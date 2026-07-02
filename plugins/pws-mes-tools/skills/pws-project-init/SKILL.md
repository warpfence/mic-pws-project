---
name: pws-project-init
description: 피플웍스 MES 프로젝트의 표준 작업 규칙(언어·어조, Git 커밋 규칙, MSSQL 조회 규칙, 파일 인코딩)을 현재 프로젝트의 CLAUDE.md 에 설치·갱신하고, 프로젝트의 `Config\Datasource\mssql-datasource.json` 에서 MSSQL 접속정보를 읽어 `.claude/settings.local.json` 의 env 에 `MSSQL_CONNECTION_STRING` 을 자동 설정한다. "프로젝트 규칙 설치", "MES 규칙 세팅", "pws init", "표준 규칙 적용", "mssql 접속정보 설정" 등을 요청할 때 사용한다.
---

# PWS MES 프로젝트 규칙 설치

피플웍스(Peopleworks) 애리조나 신공장 MES 프로젝트의 표준 작업 규칙을 현재 작업 중인 프로젝트의 `CLAUDE.md` 에 설치하거나 갱신하고, MSSQL 접속정보를 자동으로 설정하는 스킬이다. 마켓플레이스 플러그인으로 배포된 규칙을 각 사용자/프로젝트에 일관되게 심기 위한 부트스트랩 용도로 쓴다.

## 동작 절차

### 1) CLAUDE.md 표준 규칙 설치

1. 현재 프로젝트 루트의 `CLAUDE.md` 를 읽는다. 없으면 새로 만든다.
2. 아래 **표준 규칙 블록** 을 찾는다.
   - `<!-- PWS-STANDARD:START -->` ~ `<!-- PWS-STANDARD:END -->` 마커가 이미 있으면 그 사이 내용을 최신 규칙으로 **교체**한다.
   - 마커가 없으면 파일 **맨 앞**(기존 제목 다음)에 규칙 블록을 **추가**한다.
3. 기존 CLAUDE.md의 다른 내용(프로젝트 아키텍처 설명 등)은 절대 지우거나 변형하지 않는다.
4. 파일은 UTF-8 로 저장한다.

### 2) MSSQL 접속정보 자동 설정 (.claude/settings.local.json)

1. 현재 프로젝트 루트에서 `Config\Datasource\mssql-datasource.json` 파일을 찾는다.
   - **파일이 없으면** 이 단계를 진행하지 않고, `Config\Datasource\mssql-datasource.json` 을 찾지 못했다는 사실을 사용자에게 즉시 알린 뒤 종료한다. (임의로 다른 값을 넣거나 추측하지 않는다)
2. 파일이 있으면 JSON을 읽어 `properties` 아래 값을 추출한다.
   - `Url` — JDBC 형식: `jdbc:sqlserver://<host>:<port>;databasename=<db>` → `<host>`, `<port>`, `<db>` 분리
   - `Username`
   - `Password`
3. 추출한 값을 `mssql` MCP 가 사용하는 ADO.NET 형식 연결 문자열로 변환한다.

   ```
   Data Source=<host>,<port>; Initial Catalog=<db>; User ID=<Username>; Password=<Password>; TrustServerCertificate=True;
   ```

   예시 — `Url: jdbc:sqlserver://211.106.97.178:17118;databasename=SITE_PEOPLEWORKS_REAL`, `Username: pws_dev`, `Password: pwsuserp@ss` 인 경우:

   ```
   Data Source=211.106.97.178,17118; Initial Catalog=SITE_PEOPLEWORKS_REAL; User ID=pws_dev; Password=pwsuserp@ss; TrustServerCertificate=True;
   ```

4. 프로젝트 루트의 `.claude/settings.local.json` 을 읽는다. 없으면 `{}` 로 새로 만든다.
5. 최상위 `env` 객체(없으면 새로 생성) 아래 `MSSQL_CONNECTION_STRING` 키만 위 값으로 갱신한다. 파일의 다른 설정(`permissions` 등)과 `env` 의 다른 키는 그대로 보존한다.
6. 저장 후 `.claude/settings.local.json` 이 프로젝트 `.gitignore` 에 제외 대상으로 등록되어 있는지 확인한다. 없으면 추가할지 사용자에게 확인 후 반영한다. (자격증명 평문 노출 방지)
7. `mssql-datasource.json` 자체에 비밀번호가 평문으로 들어 있으므로, `settings.local.json` 으로 복사하는 것 이상의 보안 위험은 없다. 단, 연결 문자열(특히 비밀번호)을 사용자에게 그대로 출력하거나 로그에 남기지 않는다.

### 3) 완료 보고

저장 후, CLAUDE.md 에 어떤 규칙이 추가/갱신되었는지, 그리고 MSSQL 접속정보가 `.claude/settings.local.json` 에 자동 설정되었는지(또는 `mssql-datasource.json` 을 찾지 못해 설정하지 못했는지) 사용자에게 한국어로 요약 보고한다. `env` 값은 claude 재시작 시 자동 적용되므로 별도의 수동 로드 절차가 필요 없다는 점도 안내한다.

## 설치할 표준 규칙 블록

아래 내용을 그대로 CLAUDE.md 에 심는다 (마커 포함):

```markdown
<!-- PWS-STANDARD:START (pws-mes-tools 로 관리됨 — 수동 편집 대신 pws-project-init 스킬로 갱신) -->
## 언어·어조
- 한국어로 응답한다.
- 상냥하고 부드러운 존댓말을 사용한다.

## Git Commit Message 규칙
- 제목 머리말에 `[feat]` / `[fix]` / `[docs]` 등 속성 태그를 붙인다.
- 본문은 대쉬(-) 글머리 기호로 작성한다.
- footer 에 `Co-Authored-By: Claude` 관련 메시지는 넣지 않는다.

## MSSQL 데이터베이스 조회 규칙
- DB 조회·데이터 확인·스키마 확인은 `mssql-query` 스킬을 우선 사용한다.
- 데이터베이스 접근은 반드시 `mcp__mssql__*` MCP 도구로만 한다. (직접 sqlcmd 등 금지)
- 테이블/뷰 SELECT 시 항상 `WITH(NOLOCK)` 를 붙인다. (JOIN·서브쿼리·CTE 내부 포함)
- 읽기 전용 조회만 한다. INSERT/UPDATE/DELETE/MERGE/DDL 은 사용자 확인 후에만 진행한다.

## 파일 인코딩
- 모든 파일은 UTF-8 인코딩을 사용한다.
<!-- PWS-STANDARD:END -->
```

## 주의사항

- 이 스킬은 규칙을 **프로젝트 CLAUDE.md 에 영구히 심어** 이후 세션에서 자동 적용되게 한다. 일회성 지식 제공이 아니라 설정 설치가 목적이다.
- 전역(`~/.claude/CLAUDE.md`)이 아니라 **현재 프로젝트의 CLAUDE.md** 를 대상으로 한다. 전역에 적용하려면 사용자에게 명시적으로 확인받는다.
- `mssql-query` 스킬과 `mssql` MCP 서버가 같은 플러그인에 포함되어 있으므로, 규칙 설치 후 해당 도구들이 사용 가능한지 함께 안내한다.
