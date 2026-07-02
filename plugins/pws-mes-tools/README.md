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

이 플러그인의 `mssql` MCP 서버는 접속 문자열을 **환경변수**에서 읽습니다.
자격증명은 저장소에 커밋되지 않습니다.

1. `.env.example` 를 복사해 `.env` 를 만들고 실제 값을 채웁니다.

   ```
   MSSQL_CONNECTION_STRING=Data Source=192.168.x.x; Initial Catalog=SITE_PEOPLEWORKS; User ID=pws_select; Password=****; TrustServerCertificate=True;
   ```

2. claude 실행 전에 환경변수로 로드합니다.

   - PowerShell:
     ```powershell
     Get-Content .env | ForEach-Object { if ($_ -match '^\s*([^#=]+)=(.*)$') { Set-Item "env:$($matches[1].Trim())" $matches[2].Trim() } }
     ```
   - Git Bash / Linux:
     ```bash
     export $(grep -v '^#' .env | xargs)
     ```

3. `.mcp.json` 의 `${MSSQL_CONNECTION_STRING}` 이 이 값으로 치환됩니다.

> ⚠️ `.env` 는 절대 커밋하지 마세요. (루트 `.gitignore` 에 이미 제외되어 있습니다)

## 사용 후 규칙 적용

설치 후 `pws-project-init` 스킬을 실행하면 현재 프로젝트 `CLAUDE.md` 에
표준 규칙(언어·어조, Git 커밋 규칙, MSSQL 조회 규칙, 파일 인코딩)이 자동으로 심어집니다.
