# mic-pws-project

피플웍스(Peopleworks) 애리조나 신공장 MES 구축 프로젝트용 **Claude Code 플러그인 마켓플레이스**입니다.

## 수록 플러그인

| 플러그인 | 설명 |
|----------|------|
| [`pws-mes-tools`](./plugins/pws-mes-tools) | MES 작업용 스킬(MSSQL 조회, 사용자 메뉴얼 작성, 프로젝트 규칙 설치) + `mssql` MCP 서버 |

## 사용 방법

```
# 마켓플레이스 등록 (git 저장소 또는 로컬 경로)
/plugin marketplace add <git URL 또는 로컬 경로>

# 플러그인 설치
/plugin install pws-mes-tools@mic-pws-project
```

설치 후 MSSQL 접속 설정은 [`plugins/pws-mes-tools/README.md`](./plugins/pws-mes-tools/README.md) 를 참고하세요.

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
