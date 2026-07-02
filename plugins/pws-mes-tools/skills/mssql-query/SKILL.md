---
name: mssql-query
description: SmartUX Infac MES 데이터베이스(MSSQL)를 조회할 때 사용한다. "DB 조회", "테이블 조회", "데이터 확인", "쿼리 실행", "스토어드 프로시저 확인", "스키마 확인" 등 SITE_PEOPLEWORKS 데이터베이스의 데이터·구조를 들여다봐야 할 때 호출한다. 조회는 반드시 mssql MCP 도구로 수행하며, 테이블 조회 시 항상 WITH(NOLOCK)를 강제한다.
---

# MSSQL 데이터베이스 조회

SmartUX Infac MES 플랫폼의 MSSQL 데이터베이스(`SITE_PEOPLEWORKS`)를 안전하게 조회하기 위한 규칙과 절차를 제공한다.

## 핵심 규칙 (반드시 준수)

1. **MCP 도구만 사용한다.** 데이터베이스 조회는 반드시 `mcp__mssql__*` 도구로 수행한다. 다른 방식(직접 sqlcmd 실행, 임의 클라이언트 등)을 쓰지 않는다.
2. **테이블 조회 시 항상 `WITH(NOLOCK)`를 붙인다.** `FROM` 또는 `JOIN`으로 참조하는 **모든** 테이블 별칭 뒤에 `WITH(NOLOCK)`를 명시한다. 운영 DB의 락 경합을 피하기 위한 필수 규칙이다.
3. **읽기 전용 조회만 한다.** `SELECT`/구조 조회 외의 `INSERT`/`UPDATE`/`DELETE`/`MERGE`/DDL은 실행하지 않는다. 변경이 필요하면 먼저 사용자에게 확인한다.

## 조회 절차

1. **연결 확인** — 첫 조회 전 또는 연결이 의심될 때 `mcp__mssql__test_connection`으로 확인한다. 기본 연결(default)을 사용하며 별도 명명 연결은 없다.
   - 서버: `SQLDB` / 기본 DB: `SITE_PEOPLEWORKS`
2. **구조 파악** — 테이블/컬럼을 모를 때 임의 쿼리 대신 전용 도구를 먼저 쓴다.
   - 테이블 목록: `mcp__mssql__list_tables`
   - 컬럼 구조: `mcp__mssql__describe_table`
   - 샘플 데이터: `mcp__mssql__sample_data`
3. **쿼리 실행** — 데이터 조회는 `mcp__mssql__execute_query`로 실행한다. 작성한 SQL에 `WITH(NOLOCK)`가 빠지지 않았는지 실행 전 반드시 점검한다.

전용 도구 전체 목록과 용도는 `references/mcp-tools.md` 참고.

## WITH(NOLOCK) 작성 예시

올바른 예 (모든 테이블에 NOLOCK):

```sql
SELECT a.WORK_ORDER_ID, a.PROD_QTY, b.EQP_NAME
FROM TB_WORK_ORDER a WITH(NOLOCK)
INNER JOIN TB_EQUIPMENT b WITH(NOLOCK)
  ON a.EQP_ID = b.EQP_ID
WHERE a.WORK_DATE = '2026-06-30'
```

잘못된 예 (NOLOCK 누락 — 사용 금지):

```sql
SELECT * FROM TB_WORK_ORDER WHERE WORK_DATE = '2026-06-30'   -- ✗ NOLOCK 없음
```

서브쿼리, 파생 테이블, CTE 내부의 테이블 참조에도 동일하게 `WITH(NOLOCK)`를 적용한다.

## 차단 키워드 우회 (오탐지 대응)

`mcp__mssql__execute_query`는 DDL/변경 방지를 위해 특정 키워드(`create`, `update`, `delete`, `insert`, `drop`, `alter` 등)를 **단순 문자열 포함(substring) 방식**으로 차단한다. 이 때문에 **컬럼명에 해당 문자열이 들어가면** SELECT 조회인데도 차단된다.

대표 오탐지 사례:
- `CREATED_TIME`, `CREATED_QTY`, `CREATOR` → `create` 로 차단
- `UPDATED_TIME`, `UPDATER` → `update` 로 차단
- `DELETED_FLAG`, `IS_DELETED` → `delete` 로 차단

> 예: `... ORDER BY CREATED_TIME DESC` 실행 시
> `Error executing query: Query contains blocked keyword: create`

### 우회 방법

1. **`ORDER BY` 절** — 컬럼명 대신 **SELECT 결과의 컬럼 순번(ordinal)** 으로 정렬한다.
   - 먼저 `describe_table`로 대상 컬럼의 `ORDINAL_POSITION`을 확인한다.
   - `SELECT *`로 조회하면 컬럼이 ordinal 순서로 나오므로 그 순번을 그대로 쓴다.

   ```sql
   -- CREATED_TIME(50번째 컬럼) 기준 최신 1건
   SELECT TOP 1 * FROM WPM_TB_LOT WITH(NOLOCK) ORDER BY 50 DESC
   ```

2. **`SELECT` 대상 컬럼** — 차단어가 든 컬럼명을 직접 쓰지 않고 `SELECT *`로 전체를 가져온 뒤 결과에서 해당 컬럼을 읽는다.

3. **`WHERE` 절에서 불가피할 때** — 조건에 차단어 컬럼이 꼭 필요하면, 가능하면 차단어가 없는 다른 컬럼(예: `MODIFIED_TIME`, `LAST_TXN_TIME`)으로 대체 가능한지 먼저 검토한다. 대체 불가하면 사용자에게 상황을 알리고 방법을 협의한다.

> 우회하더라도 **모든 테이블에 `WITH(NOLOCK)`** 규칙은 그대로 지킨다.

## 참고

- 가능하면 읽기 전용 계정(`pws_select`)으로 동작하는 연결을 사용한다.
- 큰 결과가 예상되면 `TOP N` 또는 페이징으로 행 수를 제한한다.
