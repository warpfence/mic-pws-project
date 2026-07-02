# mssql MCP 도구 레퍼런스

데이터베이스 조회에 사용할 수 있는 `mcp__mssql__*` 도구 목록이다.
모든 도구는 `connectionName` / `connectionString` 인자를 생략하면 기본 연결(default)을 사용한다.

## 연결 / 메타

| 도구 | 용도 |
|------|------|
| `test_connection` | 연결 확인 및 서버/DB 정보 조회 |
| `list_connections` | 구성된 명명 연결 목록 |
| `list_databases` | 인스턴스의 데이터베이스 목록 |

## 객체 목록

| 도구 | 용도 |
|------|------|
| `list_tables` | 테이블 목록 |
| `list_views` | 뷰 목록 |
| `list_stored_procedures` | 스토어드 프로시저 목록 |
| `list_functions` | 함수 목록 |
| `list_triggers` | 트리거 목록 |
| `list_indexes` | 인덱스 목록 |
| `list_constraints` / `list_default_constraints` | 제약조건 목록 |
| `list_user_defined_types` | 사용자 정의 타입 목록 |

## 구조 / 정의 확인

| 도구 | 용도 |
|------|------|
| `describe_table` | 테이블 컬럼 구조 |
| `describe_view` | 뷰 정의 |
| `describe_stored_procedure` | 프로시저 시그니처 |
| `describe_trigger` | 트리거 정의 |
| `get_stored_procedure_definition` | 프로시저 본문 정의 |
| `get_multiple_stored_procedure_definitions` / `get_all_stored_procedure_definitions` | 다수/전체 프로시저 정의 |
| `search_stored_procedures_by_content` | 본문 내용으로 프로시저 검색 |
| `get_relationships` | 테이블 간 FK 관계 |

## 데이터 조회 / 분석

| 도구 | 용도 |
|------|------|
| `execute_query` | 임의 SELECT 실행 — **테이블마다 WITH(NOLOCK) 필수** |
| `sample_data` | 테이블 샘플 행 조회 |
| `analyze_table_stats` | 테이블 통계 |
| `analyze_data_distribution` | 데이터 분포 분석 |
| `analyze_null_patterns` | NULL 패턴 분석 |
| `analyze_index_usage` | 인덱스 사용 현황 |
| `analyze_database_size` | DB 용량 분석 |
| `analyze_check_constraints` | CHECK 제약조건 분석 |
| `detect_audit_columns` | 감사(audit) 컬럼 탐지 |
| `find_computed_columns` | 계산 컬럼 탐지 |
| `find_lookup_tables` | 룩업 테이블 탐지 |
| `find_missing_indexes` | 누락 인덱스 후보 |

## 사용 우선순위

- 구조만 필요하면 `list_*` / `describe_*` 전용 도구를 먼저 쓴다. (정확하고 토큰 효율적)
- 실제 데이터 조회는 `execute_query` 또는 `sample_data`를 쓴다.
- `execute_query`로 작성한 SQL은 실행 전 모든 테이블 참조에 `WITH(NOLOCK)`가 있는지 점검한다.
