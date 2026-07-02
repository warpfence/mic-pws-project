# mic-pws-project

피플웍스(Peopleworks) 애리조나 신공장 MES 구축 프로젝트용 **Claude Code 플러그인 마켓플레이스**입니다.

## 수록 플러그인

| 플러그인 | 설명 |
|----------|------|
| [`pws-mes-tools`](./plugins/pws-mes-tools) | MES 작업용 스킬(MSSQL 조회, 사용자 메뉴얼 작성, 프로젝트 규칙 설치) + `mssql` MCP 서버 |

## 설치 방법

### 1) 마켓플레이스 등록

git 저장소 또는 로컬 경로 둘 다 가능합니다.

```bash
# 로컬 경로로 (push 없이 바로 테스트)
/plugin marketplace add "D:\SynologyDrive\...\00.sourcecode\mic-pws-project"

# git 저장소로 (팀 배포)
/plugin marketplace add https://github.com/warpfence/mic-pws-project.git
/plugin marketplace add warpfence/mic-pws-project   # GitHub 단축표기
```

### 2) 플러그인 설치

```bash
/plugin install pws-mes-tools@mic-pws-project
```
형식은 `플러그인명@마켓플레이스명` 입니다.

### 3) MSSQL 접속 설정 (설치 후 1회)

`mssql` MCP 는 접속문자열을 환경변수에서 읽습니다. claude 실행 **전에** 설정하세요.

```powershell
# .env.example 복사 → .env 에 실제 값 기입 후 로드
Get-Content .env | ForEach-Object { if ($_ -match '^\s*([^#=]+)=(.*)$') { Set-Item "env:$($matches[1].Trim())" $matches[2].Trim() } }
```
> 미설정 시 `mssql` MCP 만 연결 실패하고, 나머지 스킬은 정상 동작합니다.
> 자세한 접속 설정은 [`plugins/pws-mes-tools/README.md`](./plugins/pws-mes-tools/README.md) 참고.

### 4) 프로젝트 규칙 심기 (선택)

설치 후 `pws-project-init` 스킬을 실행하면 현재 프로젝트 `CLAUDE.md` 에
표준 규칙(언어·어조, Git 커밋, MSSQL 조회, 파일 인코딩)이 자동으로 설치됩니다.

## 관리 명령

| 명령 | 설명 |
|------|------|
| `/plugin` | 플러그인 관리 UI (설치/활성화/비활성화) |
| `/plugin marketplace list` | 등록된 마켓플레이스 목록 |
| `/plugin marketplace update mic-pws-project` | 마켓플레이스 최신화 (git pull) |
| `/plugin marketplace remove mic-pws-project` | 마켓플레이스 등록 해제 |
| `/plugin uninstall pws-mes-tools@mic-pws-project` | 플러그인 제거 |

> 💡 업데이트: repo 에 변경을 push → 사용자는 `/plugin marketplace update` 후 재설치하면 최신 스킬·MCP 가 반영됩니다.

## 구조

```
mic-pws-project/
├── .claude-plugin/
│   └── marketplace.json          # 마켓플레이스 정의
├── .gitignore                    # .env 등 자격증명 제외
└── plugins/
    └── pws-mes-tools/
        ├── .claude-plugin/plugin.json
        ├── .mcp.json             # mssql MCP (접속문자열은 env 참조)
        ├── .env.example          # MSSQL_CONNECTION_STRING 예시
        ├── README.md
        └── skills/
            ├── mssql-query/
            ├── pws-user-manual/  (assets/ 에 PPTX 템플릿 포함)
            └── pws-project-init/
```
