# pws-mes-tools

피플웍스 애리조나 신공장 MES 프로젝트 작업 도구 번들입니다.

## 포함 구성요소

### Skills
| 스킬 | 설명 |
|------|------|
| `mssql-query` | MES DB(SITE_PEOPLEWORKS) 안전 조회 규칙·절차 (WITH(NOLOCK) 강제, 읽기 전용) |
| `pws-user-manual` | Arizona 신공장 MES 사용자 메뉴얼(PPTX)을 표준 양식으로 작성·수정 (템플릿 포함) |
| `pws-project-init` | 프로젝트 표준 규칙(언어·커밋·MSSQL·인코딩)을 CLAUDE.md 에 설치·갱신 |

### MCP Server
| 서버 | 설명 |
|------|------|
| `mssql` | SITE_PEOPLEWORKS(MSSQL) 조회용 MCP 서버 (`mssql-mcp-server`) |

## 설치

```
/plugin marketplace add <이 저장소 git URL 또는 로컬 경로>
/plugin install pws-mes-tools@mic-pws-project
```

## MSSQL 접속 설정 (필수)

이 플러그인의 `mssql` MCP 서버는 접속 문자열을 환경변수 `MSSQL_CONNECTION_STRING` 에서 읽습니다.
자격증명은 저장소에 커밋되지 않습니다.

프로젝트 루트에 SmartUX MES 표준 파일 `Config\Datasource\mssql-datasource.json` 이 있다면, `pws-project-init` 스킬을 실행하세요. ("프로젝트 규칙 설치", "mssql 접속정보 설정" 등으로 요청)

스킬이 자동으로 처리하는 내용:

1. `Config\Datasource\mssql-datasource.json` 의 `properties.Url`(JDBC 형식), `Username`, `Password` 를 읽는다.
2. ADO.NET 형식 연결 문자열로 변환한다.

   ```
   Data Source=<host>,<port>; Initial Catalog=<db>; User ID=<user>; Password=<password>; TrustServerCertificate=True;
   ```

3. 프로젝트 `.claude/settings.local.json` 의 `env.MSSQL_CONNECTION_STRING` 에 기록한다. (다른 설정은 보존)
4. `.mcp.json` 의 `${MSSQL_CONNECTION_STRING}` 이 이 값으로 자동 치환된다. claude 재시작만 하면 적용된다.

> `mssql-datasource.json` 을 찾지 못하면 스킬이 즉시 알려주고 종료합니다.
> ⚠️ `.claude/settings.local.json` 에 비밀번호가 평문으로 기록되므로, 프로젝트 `.gitignore` 에 반드시 포함되어 있어야 합니다. (스킬이 자동으로 확인·추가합니다)

## 사용 후 규칙 적용

설치 후 `pws-project-init` 스킬을 실행하면 현재 프로젝트 `CLAUDE.md` 에
표준 규칙(언어·어조, Git 커밋 규칙, MSSQL 조회 규칙, 파일 인코딩)이 자동으로 심어지고,
위 MSSQL 접속정보도 함께 자동 설정됩니다.
