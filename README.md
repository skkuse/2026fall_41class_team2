# SKKEY: 성균관대 스마트 컨트롤 타워
 
**소프트웨어학과 학생과 교수를 위한 대학생활 종합 지원 시스템.**
흩어진 학교 정보와 할 일을 한 화면에 모으고, 학생을 기억하는 두 AI Agent가 행사 추천과 졸업 설계를 돕는다.
 
성균관대학교 소프트웨어공학개론 (SWE3002-41) 2조
 
## 1. 제품 소개
 
학사일정, 과제, 장학, 졸업요건, 행사와 공지가 학교·학과 홈페이지, i-Campus, GLS, 게시판에 흩어져 있어 신입생은 무엇을 어디서 찾아야 하는지부터 모른다. SKKEY는 이 정보를 미리 모아 한 화면에 보여준다. 정보 수집과 정리는 수집 파이프라인(크롤러 + LLM 정리)이 맡고, AI Agent는 기억과 판단이 필요한 두 곳(행사 추천, 졸업 설계)에만 쓴다. 숫자와 순서(졸업 학점, 과목 배치, 오늘의 플랜)는 코드가 계산하고, LLM은 이해와 설명, 그리고 학생이 붙여 넣은 GLS 글 읽기만 맡는다. LLM이 읽은 숫자는 코드가 검사하고 학생이 확인한 뒤에만 쓴다.
 
### 사용자
 
웹 앱 하나에 역할에 따라 화면이 더해진다. 노트북 화면이 기준이고, 모든 화면은 로그인한 뒤에만 보인다. 가입과 로그인은 학교 이메일(`@skku.edu`, `@g.skku.edu`)로 받은 6자리 인증 코드(10분 유효)로 한다. 알림은 보내지 않고, 화면을 열면 경고 배너로 보여준다.
 
| 역할 | 주로 쓰는 기능 |
|---|---|
| 학생: 신입생 (주 사용자) | 할 일 관리, 오늘의 플랜, 꼭 알아야 할 것, 행사 추천. 첫 로그인 때 학번과 관심 분야를 답한다 |
| 학생: 재학생 | 졸업요건 Agent (진단, 로드맵, 학기별 추천 과목) |
| 교수 | 인턴십·연구실 공고 쓰기·고치기·내리기. 운영자가 계정을 승인한 뒤에 쓸 수 있다 |
| 운영자 (팀원) | 교수 승인, 오류 신고 처리, 꼭 알아야 할 것 선정, 공고·링크·졸업 규정 관리, 지표 화면. DB에서 직접 지정한다 |
 
### 기능
 
**Agent 없이 동작하는 기능**: DB에서 바로 읽거나 코드로 계산한다.
 
- **할 일 관리**: i-Campus 일정 파일(.ics)의 과제·퀴즈·강의·시험, 학사 마감, 신입생 필수 체크리스트가 자동으로 들어온다. 공지에서 찾은 기한은 후보로 보여주고 학생이 누르면 넣는다. 직접 추가와 공고 마감일 추가도 된다. 마감 24시간 안은 '임박', 지나면 '지연'으로 표시하고, 이번 주 완료율과 놓친 마감 수를 보여준다.
- **오늘의 플랜**: 코드(`plan_today`)가 마감, 예상 소요 시간, 가중치로 할 일을 정렬한다. 가중치는 학생이 ⚙ 설정에서 종류 순서(예: 강의 → 퀴즈 → 과제)를 직접 골라 정하고, 예상 소요 시간은 할 일마다 고칠 수 있다. 홈을 열 때, 할 일을 바꿀 때, '플랜 재설정'을 누를 때 다시 계산하며 AI와 LLM을 쓰지 않는다.
- **꼭 알아야 할 것**: 지금 모든 학생이 챙겨야 할 공지. 운영자가 올리고 내린다.
- **학교 소식**: 카테고리 트리(학교 전체, 학과, 장학, 동아리 모집, 신입생 필수), 글마다 한 줄 요약과 원문 링크. 모든 학생에게 같은 목록이다.
- **바로가기**: 학교 사이트 링크 모음. 매일 자동 점검한다.
- **인턴십·연구실 (교내 모집)**: 교수가 공고를 올리고, 학생은 교수에게 직접 연락한다. SKKEY 안에서 지원을 받지 않는다.
**캠퍼스 라이프 Agent**: 내 기억을 바탕으로 행사·특강·동아리를 카드로 추천한다.
 
- 카드는 매일 05:00(KST)에 최근 7일 안에 접속한 학생 것을 미리 만들고, ⚙ 설정이 바뀌면 한 번 다시 만든다. 대화창과 추천 이유 표시는 없다.
- '관심 없음'을 누른 카드는 바로 숨기고, 반응은 다음 추천에 반영한다.
- 끝난 행사와 신청 마감이 지난 행사는 뺀다. 날짜가 없는 항목은 게시 후 30일 동안 남긴다.
- 수업 시간과 겹치는 행사도 추천한다. 휴강은 수업 중에 직접 공지되는 경우가 많아 웹사이트로 알 수 없으므로, SKKEY는 휴강을 예측하지 않고 참여 여부는 학생이 판단한다.
- ⚙ 설정에 적은 글(예: "동아리는 개발 동아리만")을 추천 선호와 기억할 사실로 바꾼다. 이 글은 오늘의 플랜에는 쓰지 않는다.
- LLM을 쓸 수 없으면 관심 분야와 행사의 임베딩 유사도로 카드를 만든다.
**졸업요건 Agent** (채팅): 남은 졸업요건을 진단하고 목표 졸업 학기부터 역산한 로드맵과 추천 과목을 보여준다.
 
1. 학번, 이수 내역, 목표 졸업 학기가 없으면 먼저 묻는다. 이수 내역은 두 가지로 받는다. 간단한 방법은 GLS 졸업요건 충족현황 화면에서 복사할 수 있는 만큼 복사해 채팅에 붙여 넣는 것이다. Agent가 영역별 기준학점·취득학점을 읽어 표로 보여주고, 학생이 확인·수정한 뒤에만 쓴다. 화면의 안내가 GLS 메뉴 위치와 복사 방법을 그림으로 알려준다. 더 정확한 방법은 영역별 학점이나 들은 과목을 직접 입력하는 것이다.
2. 규칙 엔진(`diagnose`)이 남은 학점, 필수 과목, 학번별 학과 졸업 평가 항목(연구논문·작품, 인턴십, 산학협력프로젝트, 1만라인프로젝트)을 계산한다.
3. 로드맵 빌더(`build_roadmap`)가 과목을 배치하고, 검증기(`verify`)가 학점 상한·선수과목·요건을 다시 검사한다. 목표 학기가 너무 이르면 가장 빠른 졸업 학기를 보여주고, 어느 학기로도 안 되는 조건이면 그 조건을 보여주고 묻는다.
4. LLM은 질문을 이해하고 결과를 설명하며 졸업 규정 조항을 인용한다. 학생 한 명당 하루 20개까지 질문할 수 있다.
5. 학생이 진단을 바로잡으면 졸업 규정과 대조한다. 맞으면 고치고 다시 진단하며, 우리 규칙 데이터가 틀렸으면 오류 신고를 자동으로 올린다. 다르면 조항을 보여주고, 판단할 수 없으면 '확인 필요'로 표시한다.
6. 학생은 어떤 답에서든 오류 신고를 할 수 있다. 운영자가 처리하고, 규칙 데이터 오류는 회귀 시험 사례가 된다.
**GLS 붙여넣기가 실패하는 경우**
 
붙여 넣은 글은 LLM이 읽으므로 틀릴 수 있다. 읽은 숫자는 코드가 다시 검사하고, 학생이 확인하기 전에는 진단에 쓰지 않는다.
 
| 경우 | 처리 |
|---|---|
| 표의 일부만 붙여 넣어 영역이 빠짐 | 빠진 영역을 알려주고, 나머지를 붙여 넣거나 직접 입력하게 한다 |
| 칸이 밀리거나 `2(6)`, `3 / 0` 같은 표기를 잘못 읽음 | 코드가 세부 영역 합이 총학점과 맞는지, 잔여 = 기준 − 취득인지 검사한다. 어긋난 칸은 '확인 필요'로 표시하고 학생에게 묻는다 |
| 글에 없는 영역이나 숫자를 지어냄 | 붙여 넣은 글에 실제로 있는 숫자만 받는다. 없는 숫자는 버리고 그 칸을 비운다 |
| 기준학점이 저장된 졸업 규정과 다름 | 읽기 실수일 수 있으므로 학생이 확인한 뒤에만 오류 신고를 올린다 |
| GLS 화면이 아닌 글을 붙여 넣음 | 표를 만들지 않고 GLS 복사 안내를 다시 보여준다 |
| LLM을 쓸 수 없음(한도, 장애) | 직접 입력 표를 보여준다 |
 
대상은 소프트웨어학과 단일전공이며, 시간표를 짜거나 수강신청을 대신하지 않는다.
 
**내 기억**: 관심 분야, 성향·가중치, 카드 반응, 대화에서 나온 사실, 학번·이수 내역을 학생마다 저장한다. 학생은 내 기억 페이지에서 보고, 고치고, 지울 수 있다.
 
## 2. 시스템 구조
 
모듈형 모놀리스다. 백엔드 서버는 Render의 Docker 컨테이너 하나이고, 그 안의 API 모듈과 Agent 모듈은 함수 호출로 연결된다. 나머지는 설정해서 쓰는 무료 서비스다. 학교 정보는 GitHub Actions 정기 작업이 같은 백엔드 코드로 미리 모아 둔다.
 
```mermaid
flowchart TB
    WEB["웹 앱<br/>React + TypeScript (Vite)<br/>Vercel"]
 
    subgraph RENDER["Render 무료 웹 서비스 · Docker 컨테이너 1개 · 싱가포르"]
        API["API 모듈<br/>FastAPI"]
        AGENT["Agent 모듈<br/>LangGraph · 규칙 엔진"]
    end
 
    subgraph GHA["GitHub Actions 정기 작업 · 같은 백엔드 코드"]
        HOURLY["매시간: .ics 동기화<br/>3시간마다: 크롤링 · LLM 정리 · 임베딩"]
        DAILY["매일 05:00 KST: 추천 카드(Agent 코드 실행)<br/>링크 점검 · 지표 집계 · 암호화 백업 · 오래된 공지 삭제"]
    end
 
    subgraph SUPA["Supabase Free · 싱가포르"]
        AUTH["Auth<br/>이메일 인증 코드"]
        K[("knowledge<br/>학교 지식")]
        PV[("private<br/>학생·운영 데이터")]
    end
 
    LLM["LLM 클라이언트<br/>Gemini API 무료 등급"]
    SMTP["Gmail SMTP<br/>팀 계정"]
    SRC["데이터 소스<br/>학교·학과 홈페이지 · 학사일정 · 졸업 규정<br/>킹고인포 개설강좌 · i-Campus .ics"]
    UP["UptimeRobot"]
 
    WEB -->|"REST /api/v1 · SSE"| API
    WEB -->|"로그인 코드 요청·확인"| AUTH
    AUTH -->|"인증 코드 메일"| SMTP
    API -.->|"토큰 검증"| AUTH
    API -->|"함수 호출"| AGENT
    API -->|"api_rw 읽기·쓰기"| PV
    API -->|"api_rw 읽기<br/>꼭 알아야 할 것 쓰기"| K
    AGENT -.->|"agent_ro 읽기 전용 · 5개 테이블"| K
    AGENT --> LLM
    SRC -->|"수집"| GHA
    GHA -->|"pipeline_rw 쓰기"| K
    GHA -->|"pipeline_rw<br/>tasks · recommendations · metric_logs · links 쓰기<br/>users · memory · ics_links · reports 읽기"| PV
    GHA --> LLM
    UP -->|"5분마다 /health"| API
```
 
| 구성요소 | 기술 · 위치 | 하는 일 | 담당 |
|---|---|---|---|
| 웹 앱 | React + TypeScript (Vite), Vercel | 모든 화면. 백엔드 API만 부른다 | FE |
| API 모듈 | FastAPI, Render 컨테이너 | 토큰 검증과 역할 확인, 홈·할 일·소식·카드·플랜 응답, 졸업요건 채팅 스트리밍(SSE), Agent 결과와 기억 저장 | BE |
| Agent 모듈 | LangGraph, 같은 컨테이너 | 캠퍼스 라이프 Agent, 졸업요건 Agent, 기억 정리 | AI·RAG |
| 규칙 엔진 | 순수 Python (`rules/`) | `plan_today`, `diagnose`, `build_roadmap`, `verify`. 같은 입력에 같은 결과 | AI·RAG |
| 수집 | GitHub Actions | 크롤링 → `raw_documents`, .ics 동기화 → `tasks`, 링크 점검 | BE |
| 정리 | GitHub Actions | LLM 정리(분류, 한 줄 요약, 기한), 임베딩(졸업요건 자료 포함) | AI·RAG |
| LLM 클라이언트 | google-genai를 감싼 작은 모듈 → Gemini API 무료 등급 | 공지 정리, ⚙ 설정, 채팅 모두 Flash-Lite가 기본이고 임베딩은 768차원이다. 실제 하루 한도는 우리 프로젝트의 AI Studio 화면에서 확인해 하루 예산으로 정한다. 예산을 넘거나 한도에 걸리면 채팅은 LLM 설명 없이 코드가 만든 결과 글만 보낸다 | AI·RAG |
| DB와 로그인 | Supabase Free (PostgreSQL + pgvector + Auth) | 스키마 두 개, DB 로그인 네 개, 이메일 인증 코드 | BE |
| 로그인 메일, 가동 감시 | Gmail SMTP, UptimeRobot | 인증 코드 메일 발송, 5분마다 `/health` 호출 | BE |
 
**DB**: 학교 지식은 `knowledge` 스키마(`notices`, `events`, `offered_courses`, `grad_rules`, `knowledge_chunks`, `raw_documents`), 학생·운영 데이터는 `private` 스키마(`users`, `memory`, `tasks`, `ics_links`, `transcripts`, `roadmaps`, `recommendations`, `chat_messages`, `reports`, `postings`, `links`, `metric_logs`)에 둔다. 졸업 규칙의 원본은 저장소의 YAML 파일이고, 배포할 때 `grad_rules`에 싣는다. 모든 DB 연결은 Supabase 연결 풀러 주소를 쓴다.
 
| DB 로그인 | 쓰는 곳 | 권한 |
|---|---|---|
| `api_rw` | API 모듈 | `private` 읽기·쓰기, `knowledge` 읽기, 꼭 알아야 할 것 데이터 쓰기 |
| `agent_ro` | Agent 모듈 | `notices`, `events`, `offered_courses`, `grad_rules`, `knowledge_chunks` 읽기만 |
| `pipeline_rw` | 정기 작업 | `knowledge` 쓰기(배포 때 졸업 규칙 YAML 싣기 포함), `tasks`·`recommendations`·`metric_logs`·`links` 쓰기, `users`·`memory`·`ics_links`·`reports` 읽기 |
| `backup_ro` | 매일 백업 | `private` 읽기 |
 
**정기 작업**
 
| 워크플로 | 실행 시점 | 하는 일 |
|---|---|---|
| `ci.yml` | 모든 PR | 정적 검사, 단위 시험, 인터페이스 시험, 졸업요건 회귀 시험(LLM은 녹화한 응답을 재생해 실제 호출 0회), 졸업 규칙 YAML 형식 검사, Vercel CLI로 PR 미리보기 |
| `release.yml` | `main` 병합 | 마이그레이션 적용 → 졸업 규칙 YAML을 `grad_rules`에 싣기 → Render 배포 훅 → `/health` 확인 → Vercel CLI로 웹 배포. Render 자동 배포는 꺼 둔다 |
| `pipeline-hourly.yml` | 매시간 | .ics 동기화. 3시간마다 크롤링, LLM 정리, 임베딩도 함께 |
| `pipeline-daily.yml` | 매일 05:00 (KST) | 추천 카드, 링크 점검, 지표 집계, `private` 암호화 백업, 2학기 지난 공지 삭제 |
 
예약 작업이 정각에 몰리면 늦게 시작할 수 있어, cron의 분은 정각을 피한다(예: `17 * * * *`).
 
**원칙**
 
- 화면은 백엔드 API만 부른다. 로그인 코드 요청·확인만 Supabase Auth와 직접 한다.
- Agent 모듈은 DB에 쓰지 않는다. 결과와 기억은 제안만 하고, API나 정기 작업이 저장한다.
- Agent 모듈은 학교 지식만 읽는다. 학생 데이터는 API가 요청마다 한 명분만 넘기며, 이름과 이메일은 넣지 않는다.
- LLM을 실시간으로 부르는 곳은 졸업요건 채팅과 ⚙ 설정의 글 저장 두 곳뿐이다. 오늘의 플랜을 포함한 나머지는 코드로 계산하거나 미리 만든다.
- LLM에 장애가 있어도 정보 기능은 그대로 동작한다.
**보안과 수집**
 
- 학교 계정 비밀번호를 받지 않는다. 이수 내역은 학생이 입력하거나 GLS 표를 붙여 넣고, i-Campus는 .ics 주소로만 연결해 암호화 저장한다.
- LLM에는 요청한 학생 본인 데이터만 보내고, 이름과 이메일은 보내지 않는다. 붙여 넣은 GLS 글에서도 이름·학번 줄은 코드가 지운 뒤 보낸다. Gemini 무료 등급 입력이 Google 제품 개선에 쓰일 수 있음을 가입 동의 화면에서 알린다. 졸업요건 채팅 입력창 위에 "이름, 학번, 연락처 같은 개인정보는 입력하지 마세요"를 늘 보여준다..
- 로그에는 개수와 상태만 남긴다. httpx 로그는 WARNING 이상만 남기고, 이름·이메일·.ics 주소·프롬프트 본문은 어떤 로그에도 남기지 않는다. 저장소가 공개라 GitHub Actions 로그도 누구나 볼 수 있다.
- 모든 `private` 테이블에 행 수준 보안(RLS)을 켜고 브라우저 역할에는 정책을 주지 않는다.
- 비밀 값(DB 주소, Gemini 키, .ics 암호화 키, `VERCEL_TOKEN`)은 Render 환경 설정과 GitHub Secrets에만 둔다. 저장소는 공개다.
- `private` 백업은 `age`로 암호화해 7일만 보관한다.
- 공개 페이지만 수집하고, robots.txt를 지키며, 사이트마다 초당 1회 이하로 요청하고, 프로젝트를 밝히는 User-Agent를 쓴다. 같은 글은 URL과 내용 해시로 한 번만 저장하고, 공지는 2학기 동안만 보관한다.
- 학과 홈페이지의 졸업요건·교육과정·로드맵 페이지와 PDF는 매일 다시 가져오고, 바뀌면 학교 지식의 예전 조각을 바꾸거나 지운다. 사람이 하는 일은 두 가지뿐이다. 학점 기준이 바뀌면(학교 학점이수표, 학과 공지) 운영자가 졸업 규칙 YAML에 옮겨 넣어 PR로 올리고, 학생은 자기 GLS 표를 붙여 넣는다. 크롤러는 새 학점이수표나 학과 졸업요건 공지를 찾으면 운영자 확인 작업을 만들 뿐, 규칙 데이터를 직접 고치지 않는다. 크롤러는 GLS에 로그인하지 않는다.
**확장성과 한계**
 
지금은 무료 등급(Render, Supabase Free, Gemini 무료)에 맞춰 서버 하나로 시작한다. 바깥 서비스는 인터페이스 뒤에 두어, 바꿀 때 고치는 곳이 그 모듈의 설정에 머물게 한다. 다만 무료 등급에는 돈을 내야만 풀리는 한계가 있으므로, 한계마다 지금 값, 확인하는 방법, 바꾸는 곳을 적는다. 측정값은 시험하면서 채운다.
 
| 한계 | 지금 | 확인하는 방법 | 바꾸는 곳 |
|---|---|---|---|
| 백엔드 서버 | Render 무료: 메모리 512MB, 15분 동안 요청이 없으면 잠들고 깨는 데 약 1분 | Render 메모리 그래프, UptimeRobot 가동률 | 같은 Docker 이미지로 유료 Render나 Cloud Run |
| 동시 사용자 | 목표 100명 | 부하 시험으로 한계점(홈 P95가 3초를 넘는 사용자 수)을 잰다 | 서버 사양, Agent 별도 서버(`AGENT_MODE=remote`, `AGENT_URL`) |
| LLM 호출 | Gemini 무료 등급. 하루 한도는 프로젝트마다 다르다 | AI Studio의 한도 화면, `llm/`의 하루 호출 수 | `backend/skkey/llm/models.yaml`, 유료 키 |
| LLM 입력 | 무료 등급 입력은 Google 제품 개선에 쓰일 수 있다 | 프롬프트 개인정보 검사 시험 | 유료 등급 |
| DB | Supabase Free: 저장 공간 500MB | DB 크기, 공지 2학기 보관 | `DATABASE_URL_API`, `DATABASE_URL_AGENT`, `DATABASE_URL_PIPELINE`, `DATABASE_URL_BACKUP` |
| 정기 작업 | GitHub Actions 예약 작업은 늦게 시작할 수 있다 | 작업 기록의 시작 시각 | 다른 스케줄러 |
| 임베딩 | 768차원 | 상위 5개 조회 시간 | `EMBEDDING_MODEL`, `EMBEDDING_DIM` 후 재임베딩 작업 실행 |
| 수집 범위 | 소프트웨어학과 | | `backend/skkey/pipeline/sources.yaml` |
| 캐시, 로그인 메일 | 캐시 없음, Gmail SMTP | | `CACHE_BACKEND=redis`, Supabase SMTP 설정 |
 
```python
# backend/skkey/main.py (계획)
def build_agent(settings: Settings) -> AgentFacade:
    # AGENT_MODE만 바꾸면 구현체가 바뀐다. 호출하는 쪽은 고치지 않는다
    if settings.agent_mode == "remote":
        return HttpAgentClient(settings.agent_url)
    return InProcessAgent()
```
 
```yaml
# backend/skkey/llm/models.yaml (계획)
# 위에서부터 차례로 쓰고, 하루 예산이나 한도에 걸리면 다음으로 넘어간다. code_text는 LLM 없이 코드가 만든 결과 글이다.
chat:                  # 졸업요건 채팅
  - flash-lite          # 기본. 하루 예산은 AI Studio에서 확인한 한도로 정한다
  - code_text
cleanup:               # 공지 정리, ⚙ 설정
  - flash-lite
```
 
## 3. 팀 구성과 담당
 
| 팀 | 사람 | 맡는 것 |
|---|---|---|
| FE | 권유민, 김호준 | `web/`: 학생·교수·관리자·지표 화면 전부, 화면 시험 |
| BE | 강민주, 최영훈 | API(`api/`, `db/`), 수집(`pipeline/collect/`: 크롤러, .ics 동기화, 링크 점검), DB(`supabase/`), 로그인 설정, 배포와 정기 작업 |
| AI·RAG | 박연우, 성민호, 이건 | Agent(`agent/`), 규칙 엔진(`rules/`), LLM 클라이언트(`llm/`), 정리(`pipeline/process/`), 평가 데이터(`evaluation/`), 졸업 규칙 YAML(`rules/data/`) |
| 통합 담당 | [TBD: Sprint 0에서 결정, AI·RAG 1명] | 인터페이스 정의(`contracts/`), CI(`ci.yml`), 평가 데이터, 지표 집계와 주간 보고, 부하 시험 |
 
**팀 안의 역할 나누기 (제안)**
 
| 역할 | 맡는 일 | 이름 |
|---|---|---|
| FE-1 | 학생 화면: 로그인, 홈, 소식, 졸업요건 채팅과 이수 내역, 로드맵, 내 기억, 공고 목록. 컴포넌트 시험 | [TBD: 팀 합의] |
| FE-2 | 교수·관리자·지표 화면, 생성된 API 클라이언트, MSW mock, Playwright 시험 | [TBD: 팀 합의] |
| BE-1 | `api/`, `db/`, `FakeAgent`, 졸업요건 채팅 SSE, 확인한 GLS 표 저장, 권한 시험 | [TBD: 팀 합의] |
| BE-2 | 마이그레이션(`supabase/`), 로그인 설정, 크롤러, .ics 동기화, 링크 점검, 워크플로와 배포, 백업 | [TBD: 팀 합의] |
| AI·RAG-1 | 규칙 엔진(`plan_today`, `diagnose`, `build_roadmap`, `verify`), 졸업 규칙 YAML, 학생 정정 대조(`check_claim`), 회귀 사례화 | [TBD: 팀 합의] |
| AI·RAG-2 | 졸업요건 Agent, GLS 붙여넣기 읽기, 캠퍼스 라이프 Agent, ⚙ 설정 해석, 공지 정리·임베딩, LLM 클라이언트 | [TBD: 팀 합의] |
| AI·RAG-3 | 통합 담당: 인터페이스, CI, 평가 데이터, 지표 집계와 주간 보고, 부하 시험 | [TBD: 팀 합의] |
 
**여러 팀이 나눠 맡는 일**
 
| 일 | 나누는 방법 |
|---|---|
| 크롤러 → 공지 정리 | 페이지를 가져와 `raw_documents`에 저장하는 것까지 BE, 글을 이해하고 정리하는 것부터 AI·RAG |
| 졸업 규정 → `grad_rules` | 규칙 YAML과 그 형식은 AI·RAG, 테이블과 배포 때 싣는 단계는 BE, 학점 기준 입력은 운영자 |
| 오늘의 플랜 | `plan_today`는 AI·RAG, 엔드포인트는 BE, 화면은 FE |
| 매일 추천 카드 | 카드 만드는 함수는 AI·RAG, 매일 작업 실행은 BE |
| 졸업요건 채팅 | 답변 생성은 AI·RAG, SSE 전달은 BE, 화면은 FE |
| 오류 신고 | 버튼과 관리자 화면은 FE, API와 테이블은 BE, 자동 신고와 회귀 사례화는 AI·RAG |
| 로그인 | Supabase 설정, 도메인 트리거, 토큰 검증은 BE, 로그인 화면은 FE |
| 지표 | 팀마다 자기 이벤트를 기록하고, 집계·정의·주간 보고는 통합 담당, 차트는 FE |
 
## 4. 저장소 구조
 
```text
skkey/
├── web/                        # FE · React + TypeScript (Vite) → Vercel
│   └── src/api/generated/      # OpenAPI 명세로 만든 TS 클라이언트 (손으로 고치지 않음)
├── backend/                    # Docker 이미지 하나 → Render
│   ├── Dockerfile
│   ├── skkey/
│   │   ├── main.py             # 통합 담당 · 실제 구현체를 연결하는 유일한 곳
│   │   ├── contracts/          # 통합 담당 · 팀 사이 인터페이스 정의 (api.py는 BE)
│   │   ├── api/                # BE · FastAPI 라우트, 서비스
│   │   ├── db/                 # BE · DB 로그인별 연결, 리포지토리
│   │   ├── agent/              # AI·RAG · 캠퍼스 라이프, 졸업요건, 기억 정리
│   │   ├── rules/              # AI·RAG · 규칙 엔진, 검증기, 로드맵 빌더, 플랜 점수, data/ 졸업 규칙 YAML
│   │   ├── llm/                # AI·RAG · Gemini 클라이언트: 모델 순서, 하루 예산, 개인정보 검사
│   │   └── pipeline/
│   │       ├── collect/        # BE · 크롤러, .ics 동기화, 링크 점검
│   │       └── process/        # AI·RAG · LLM 정리, 임베딩
│   └── tests/                  # 각 코드의 담당 팀이 작성
├── supabase/
│   ├── migrations/             # BE · 스키마, DB 로그인, 권한 (SQL)
│   └── seed/                   # 시험 데이터
├── evaluation/                 # AI·RAG · 라벨 표본 (M-01, M-02, M-10–M-14)
└── .github/workflows/          # ci.yml은 통합 담당, 나머지는 BE
```
 
## 5. 인터페이스
 
팀이 만나는 곳은 세 군데다. 각 인터페이스는 파일 하나로 정의하고, 제공하는 팀이 Sprint 0에 먼저 만든다. 사용하는 팀은 실제 구현이 준비될 때까지 대역으로 개발하므로 서로 기다리지 않는다.
 
| 인터페이스 | 제공 → 사용 | 정의 파일 | 준비 전 대역 | 자동 확인 |
|---|---|---|---|---|
| 웹 ↔ API | BE → FE | `contracts/api.py` → `/api/v1/openapi.json` | MSW mock | Schemathesis, 생성 클라이언트 타입 검사 |
| API ↔ Agent | AI·RAG → BE | `contracts/agent.py` | `FakeAgent` | mypy, `FakeAgent`와 실제 Agent가 같은 시험 통과 |
| 수집 ↔ 정리 | BE → AI·RAG | `supabase/migrations/` (`raw_documents`) | 견본 행 | staging 마이그레이션, 견본 행으로 정리 시험 |
 
- Sprint 0이 끝나면 인터페이스를 고정한다. 이후에는 PR로만 바꾸고, 영향을 받는 두 팀이 모두 승인해야 병합한다. `/api/v1`은 하위 호환을 지키고, 호환이 깨지면 새 버전으로 낸다.
- 기능 단계가 끝날 때 양쪽이 같은 시험을 통과하면 합친 것이다.
### 웹 ↔ API (REST)
 
기준 경로는 `/api/v1`, 형식은 JSON, 요청에는 `Authorization: Bearer <token>`을 담는다. 오류 형식은 하나다: `{"error": {"code": "...", "message": "..."}}`
 
| 엔드포인트 | 호출자 | 용도 |
|---|---|---|
| `GET /me`, `POST /me/onboarding` | 모두 | 프로필, 역할, 첫 로그인 질문 |
| `GET /home` | 학생 | 홈 한 번에: 경고 배너, 할 일, 오늘의 플랜, 꼭 알아야 할 것, 카드 |
| `GET/POST /tasks`, `PATCH /tasks/{id}` | 학생 | 할 일 목록, 직접 추가, 완료 체크, 예상 소요 시간 고치기 |
| `PUT/DELETE /me/ics-link` | 학생 | i-Campus 연결, 해제 |
| `GET /tasks/candidates`, `POST /tasks/candidates/{id}/accept` | 학생 | 공지에서 찾은 기한 후보 |
| `POST /plan/reset` | 학생 | 오늘의 플랜 다시 계산 |
| `GET /notices?category=`, `GET /must-know` | 모두 | 소식, 꼭 알아야 할 것 |
| `GET /recommendations`, `POST /recommendations/{id}/reaction` | 학생 | 카드, `saved` 또는 `not_interested` |
| `PUT /me/settings` | 학생 | ⚙ 설정 저장: 플랜의 종류 순서, 추천에 쓰는 글 |
| `GET/PUT /transcript` | 학생 | 이수 내역과 영역별 학점 (직접 입력, 채팅에서 확인한 GLS 표 저장) |
| `POST /graduation/chat` (SSE) | 학생 | 졸업요건 채팅 |
| `POST /reports`, `GET /reports/mine` | 학생 | 오류 신고와 처리 결과 |
| `GET /memory`, `PATCH/DELETE /memory/{id}` | 학생 | 내 기억 |
| `GET /postings`, `POST /postings/{id}/to-tasks` | 학생 | 공고 목록, 마감일을 할 일에 넣기 |
| `POST/PATCH/DELETE /postings/{id}` | 승인된 교수 | 공고 쓰기, 고치기, 내리기 |
| `GET /links` | 모두 | 바로가기 |
| `/admin/professors`, `/admin/reports`, `/admin/must-know`, `/admin/links`, `/admin/metrics` | 운영자 | 관리자 기능 |
| `GET /health` | UptimeRobot | 백엔드와 DB 상태 |
 
채팅은 `status`(진행 상태: 이수 내역 읽는 중, 진단 중, 로드맵 만드는 중), `result`(코드가 검사한 진단, 로드맵, 과목, 붙여 넣은 GLS 글을 읽은 표), `citation`(졸업 규정 조항), `needs_check`(확인 필요), `token`(설명 글), `done` 순서로 보낸다. 설명 글은 검사한 결과를 보낸 뒤에만 보내고, `error`는 언제든 스트림을 끝낼 수 있다.
 
### API ↔ Agent
 
API는 `contracts/`에 선언된 아래 함수만 부른다. 실제 구현은 `main.py`에서 연결하고, BE는 고정된 답을 주는 `FakeAgent`로 먼저 개발한다.
 
```python
class AgentFacade(Protocol):
    async def recommend(self, ctx: StudentContext, candidates: list[EventRef]) -> list[Card]: ...
    async def interpret_settings(self, ctx: StudentContext, text: str) -> SettingsResult: ...  # 추천 선호 + 기억할 사실 (플랜에는 쓰지 않음)
    def graduation_chat(self, ctx: StudentContext, transcript: Transcript,
                        history: list[ChatTurn], message: str) -> AsyncIterator[ChatEvent]: ...
    async def check_claim(self, ctx: StudentContext, course_code: str, claimed_area: str) -> ClaimCheck: ...
 
class RuleEngine(Protocol):  # rules/의 순수 함수. 입출력과 LLM 없음
    def plan_today(self, tasks: list[Task], weights: Weights, now: datetime) -> Plan: ...
    def diagnose(self, transcript: Transcript, rules: list[GradRule]) -> Diagnosis: ...
    def build_roadmap(self, d: Diagnosis, target: Semester, credits: int, offerings: list[Offering]) -> Roadmap: ...
    def verify(self, roadmap: Roadmap, rules: list[GradRule]) -> list[Violation]: ...
```
 
- `StudentContext`: 익명 학생 id, 입학년도, 관심 분야, 가중치, 반응 요약, 저장된 사실. 이름과 이메일은 넣지 않는다.
- `ChatEvent`의 `MemoryUpdate`는 제안일 뿐이고, 저장은 API가 한다.
- `ClaimCheck`는 `match`, `mismatch`, `unverifiable`와 인용, `rule_conflict`를 돌려준다. `rule_conflict`가 참이면 API가 오류 신고를 올린다.
- `Roadmap`에는 실제로 쓴 졸업 학기가 들어간다.
### 수집 ↔ 정리
 
`knowledge.raw_documents`(`id`, `source`, `url`(유일), `title`, `posted_at`, `html`, `text`, `content_hash`, `fetched_at`, `processed_at`)가 넘겨받는 곳이다.
 
1. 수집은 `processed_at`을 비운 채 새 행을 넣는다.
2. 정리는 그 행을 읽어 `notices`, `events`, `knowledge_chunks`를 채우고 `processed_at`을 기록한다.
두 단계는 따로 실행하고, 실패하고, 재배포할 수 있다.
 
## 6. 팀별 개발 항목
 
각 항목은 완료 기준을 모두 만족해야 끝난 것이다. 괄호 안은 담당 역할(3절)과 단계다. 단계는 Sprint 0(인터페이스와 뼈대) 다음 1 로그인·할 일, 2 학교 소식·꼭 알아야 할 것, 3 추천 카드·설정, 4 졸업요건 진단, 5 로드맵·추천 과목·오류 신고, 6 인턴십·연구실·내 기억·관리자·지표 순서다. 지표는 이름 뒤에 번호(M-xx)를 붙여 적었다. 모든 항목은 담당 역할 하나와 다른 팀의 리뷰어 하나를 둔다. 규칙 엔진을 만드는 사람은 그 평가 사례를 쓰지 않는다(평가 데이터는 통합 담당).
 
### FE
 
| 개발 항목 | 만드는 것 | 완료 기준 |
|---|---|---|
| API 클라이언트와 mock<br>(FE-2 · Sprint 0) | BE의 `/api/v1/openapi.json`으로 `web/src/api/generated/`를 만든다(openapi-typescript + openapi-fetch). 손으로 고치지 않고, 명세가 바뀌면 다시 만든다. 같은 명세로 MSW mock을 만들어 엔드포인트가 준비되기 전에도 화면을 개발한다. | 다시 만든 뒤 tsc 오류 0건<br>모든 엔드포인트에 mock 응답이 있음 |
| 로그인·가입 화면<br>(FE-1 · Sprint 0–1) | 학교 이메일 입력 → 6자리 코드 입력(supabase-js로 Supabase Auth에 직접) → 첫 로그인 질문(학번, 관심 분야)을 `POST /me/onboarding`으로 보낸다. 받은 토큰을 모든 API 요청에 담는다. | 학교 이메일로 가입·로그인 후 홈으로 이동<br>gmail.com 주소와 10분 지난 코드는 오류 안내 |
| 홈<br>(FE-1 · 1, 3) | `GET /home` 한 번으로 경고 배너(임박·지연, 이번 주 완료율, 놓친 마감 수), 할 일 목록(마감순, 임박·지연 표시, 체크 `PATCH /tasks/{id}`, 직접 추가 `POST /tasks`), 기한 후보(`GET /tasks/candidates`, 수락), 오늘의 플랜(재설정 `POST /plan/reset`), 꼭 알아야 할 것, 추천 카드(저장·관심 없음 `POST /recommendations/{id}/reaction`)를 그린다. 관심 없음은 응답을 기다리지 않고 바로 숨긴다. i-Campus 연결(`PUT /me/ics-link`), 할 일마다 예상 소요 시간 고치기, ⚙ 설정(`PUT /me/settings`: 플랜의 종류 순서를 고르는 칸과 추천에 쓰는 글 칸)도 여기서 한다. | 오늘 마감인 할 일이 클릭 없이 보임<br>할 일을 체크하면 플랜이 다시 정렬됨<br>관심 없음을 누르면 카드가 바로 사라짐 |
| 학교 소식·바로가기<br>(FE-1 · 2) | 카테고리 트리(학교 전체, 학과, 장학, 동아리 모집, 신입생 필수)와 최신순 목록, 한 줄 요약, 원문 링크(`GET /notices?category=`, `GET /must-know`). 바로가기 모음(`GET /links`). | 소식을 누르면 원문 페이지로 이동<br>목록이 학생마다 같음 |
| 졸업요건 화면<br>(FE-1 · 4, 5) | 왼쪽은 이수 내역 직접 입력(영역별 학점이나 과목, `PUT /transcript`), 오른쪽은 채팅(`POST /graduation/chat`, SSE). `status`는 진행 표시로, `token`은 설명 글을 이어 붙이고, `result`는 남은 학점 표·학기별 로드맵·추천 과목, `citation`은 근거 조항 링크, `needs_check`는 '확인 필요'로 그린다. 채팅에 GLS 글을 붙여 넣으면 읽은 표를 확인·수정 카드로 보여주고, 학생이 확인하면 저장한다. GLS 메뉴 위치와 복사 방법은 그림 3단계로 안내한다(처음 한 번, '?'로 다시 열기). 입력창 위에 개인정보를 입력하지 말라는 안내를 늘 보여준다. `error`가 오면 "잠시 후 다시 시도해 주세요". 답마다 오류 신고 버튼(`POST /reports`)과 내 신고 결과(`GET /reports/mine`). | 질문 후 3초 안에 진행 상태 표시<br>이벤트 종류마다 컴포넌트 시험 통과<br>하루 20개를 넘으면 안내 표시 |
| 내 기억·공고 목록<br>(FE-1 · 6) | 내 기억 종류별 목록과 들어온 경로, 고치기·지우기(`GET /memory`, `PATCH/DELETE /memory/{id}`). 인턴십·연구실 공고 목록과 마감일을 할 일에 넣기(`POST /postings/{id}/to-tasks`). | 지운 기억이 다음 추천에 쓰이지 않음<br>공고 마감일이 할 일에 들어감 |
| 교수 화면<br>(FE-2 · 6) | 교수 계정 신청, 공고 쓰기·고치기·내리기(`POST/PATCH/DELETE /postings/{id}`). 칸: 분류(학부연구생, 연구조교, 인턴, TA, 단기 연구 프로젝트), 연구 분야, 인원, 지원 방법, 마감일(오늘 이후). | 승인 전 교수에게는 쓰기 버튼이 없음<br>등록하면 학생 목록에 바로 보임 |
| 관리자 화면<br>(FE-2 · 2, 5, 6) | `/admin/*`: 교수 승인·거절, 꼭 알아야 할 것 올리기·내리기, 바로가기 관리(끊긴 링크 표시), 오류 신고와 확인 작업(크롤러가 찾은 졸업요건 자료 변경은 원본 링크와 함께 보이고, 규칙 데이터는 YAML PR로 고친다. 원인을 학생 오해, 규칙 데이터 오류, 학교 규정 변경, 학과 개별 승인 중 하나로 골라 처리), 지표 화면(M-01–M-20 주별 추이와 목표선, 목표 미달 강조, 응답 시간 P95, Recharts). | 운영자가 아니면 들어갈 수 없음<br>지표 화면에 개인을 알아볼 수 있는 값이 없음 |
| 컴포넌트 시험<br>(FE-1, FE-2 · 모든 단계) | 화면마다 데이터 훅과 표시 컴포넌트를 나누고, Vitest + React Testing Library로 시험한다. | 모든 화면의 로딩·빈 상태·오류 상태가 그려지고 CI 통과 |
| 종단 간 시험<br>(FE-2 · 3–6) | Playwright로 유스케이스마다 1개: 가입, 로그인, 할 일, 오늘의 플랜, 학교 소식, 추천, 졸업요건 진단, 추천 과목, 공고, 내 기억, 관리. | 모든 유스케이스의 정상 흐름 통과 |
| 인수 시험<br>(FE 주관, 모든 팀 · 마지막) | 학생 10명이 1주 동안 쓰고, 정보 찾기 과제(이번 주 마감, 수강정정 기간, 장학 신청 마감)를 기존 학교 사이트·일반 LLM·SKKEY로 각각 시간 재며 수행한다. 설문을 받는다. | 만족도(M-09) 5점 중 4.0 이상<br>탐색 시간 단축률(M-15) 50% 이상<br>탐색 정답률(M-16) 95% 이상<br>주간 활성 사용자(M-17) 10명 이상<br>재방문율(M-18, 주 3일 이상 접속) 50% 이상<br>마감 놓친 비율(M-03) 10% 이하, 주간 완료율(M-04) 85% 이상, 플랜 재설정 비율(M-05) 30% 이하 |
 
### BE
 
| 개발 항목 | 만드는 것 | 완료 기준 |
|---|---|---|
| 엔드포인트 스텁<br>(BE-1 · Sprint 0) | `contracts/api.py`에 5절의 모든 엔드포인트를 정의하고, 고정된 답을 돌려주는 스텁을 먼저 연다. FE가 첫날부터 생성 클라이언트로 개발할 수 있게 한다. | Schemathesis 위반 0건<br>FE가 생성 클라이언트로 빌드 성공 |
| 실제 엔드포인트와 서비스<br>(BE-1 · 1–6) | `api/`를 라우트(HTTP만) → 서비스(규칙만, SQL 없음) → 리포지토리(SQL만) 3계층으로 만든다. 모든 요청에서 Supabase 토큰을 검증(PyJWT)하고 역할(학생, 승인된 교수, 운영자)을 확인한다. 서비스: 임박(24시간 안)·지연 상태, 완료율, 카드 반응, 공고와 교수 승인, 내 기억 저장(Agent 제안을 검사해 저장), 오류 신고와 확인 작업, 졸업요건 채팅 하루 20개 제한(`chat_messages`의 오늘 개수를 DB에서 센다. 서버가 잠들었다 깨도 유지된다). 오류 형식은 `{"error": {"code", "message"}}` 하나. | Schemathesis 위반 0건<br>토큰 없으면 401, 권한 없는 역할은 거부<br>다른 사용자의 데이터 읽기 0건<br>저장 요청 2초 안에 완료 |
| 오늘의 플랜·⚙ 설정 연결<br>(BE-1 · 1, 3) | `GET /home`과 `POST /plan/reset`에서 할 일과 학생이 고른 종류 순서로 `RuleEngine.plan_today`를 부른다(결과는 저장하지 않음). `PUT /me/settings`에서 종류 순서는 그대로 저장하고, 글이 있으면 `AgentFacade.interpret_settings`로 추천 선호와 기억할 사실을 받아 저장한 뒤 그 학생의 카드를 `recommend`로 한 번 다시 만든다. | 플랜 재설정 5초 안에 응답<br>오늘의 플랜 계산에 LLM 호출 0회<br>설정의 글을 저장하면 카드가 한 번 다시 만들어짐 |
| 졸업요건 채팅 SSE<br>(BE-1 · 4) | 학생 한 명분 `StudentContext`(익명 id, 입학년도, 관심 분야, 가중치, 반응 요약, 저장된 사실)와 이수 내역, 대화 기록을 `AgentFacade.graduation_chat`에 넘기고, 나오는 이벤트를 SSE로 그대로 전달한다. 끝나면 메시지와 기억 제안을 저장한다. `check_claim`의 `rule_conflict`가 참이면 오류 신고를 자동으로 만든다. | 진행 상태 3초, 전체 답변 20초 안<br>`StudentContext`에 이름·이메일 0건 |
| 확인한 GLS 표 저장<br>(BE-1 · 4) | 학생이 채팅에서 확인·수정한 영역별 학점표를 `PUT /transcript`로 받아 저장한다. 확인 전의 읽은 값은 진단에 쓰지 않는다. 학생이 확인한 기준학점이 그 학번의 `grad_rules`와 다르면 오류 신고를 만든다. LLM에 보내기 전에 붙여 넣은 글에서 이름·학번 줄을 지운다. 학교 계정 정보는 받지 않는다. | 확인하지 않은 표가 진단에 쓰인 경우 0건<br>LLM으로 간 글에 이름·학번 0건<br>기준학점을 일부러 바꾼 예시에서 학생 확인 후 오류 신고 생성 |
| `FakeAgent`<br>(BE-1 · Sprint 0) | `AgentFacade`를 고정된 답으로 흉내 내는 대역. AI·RAG가 준비되기 전까지 BE와 FE가 이것으로 개발한다. | 실제 Agent와 같은 인터페이스 시험 통과 |
| 마이그레이션<br>(BE-2 · Sprint 0–6) | `supabase/migrations/`: `knowledge`·`private` 스키마와 테이블, DB 로그인 4개(`api_rw`, `agent_ro`, `pipeline_rw`, `backup_ro`)와 권한, 모든 `private` 테이블의 RLS(브라우저 역할에는 정책 없음), `raw_documents`. | 빈 로컬 DB와 staging에 깨끗이 적용<br>`agent_ro`의 `private` 읽기와 브라우저 공개 키의 모든 읽기 100% 거부<br>`api_rw`, `pipeline_rw`도 정한 권한 밖은 100% 거부 |
| 로그인 설정<br>(BE-2 · Sprint 0) | Supabase Auth 이메일 코드(6자리, 10분 유효), 학교 도메인(`@skku.edu`, `@g.skku.edu`)만 받는 `auth.users` 트리거, 팀 계정 Gmail SMTP. | `@g.skku.edu`로 1분 안에 코드 도착, 스팸함 아님<br>gmail.com 주소 거부<br>10분 지난 코드 실패 |
| 크롤러<br>(BE-2 · 2, 4, 5) | `pipeline/collect/`에 출처마다 작은 크롤러 하나(공통 `Source` 인터페이스), 출처 목록은 `pipeline/sources.yaml`. 출처: 학교·학과 공지 게시판, 학사일정, 학과 홈페이지의 졸업요건·교육과정·로드맵 페이지와 PDF(매일), 킹고인포 개설강좌, 학교 교육과정 페이지(새 학점이수표가 올라왔는지만 확인). 도구: httpx + BeautifulSoup4(lxml), RSS는 feedparser, JavaScript로만 그려지는 페이지만 Playwright, PDF는 pdfplumber 또는 PyMuPDF. 공개 페이지만, robots.txt 준수, 사이트마다 초당 1회 이하, 프로젝트를 밝히는 User-Agent. `raw_documents`에 URL과 내용 해시로 저장한다. 새 학점이수표나 학과 졸업요건 공지를 찾으면 운영자 확인 작업을 만든다. | 저장한 페이지 스냅샷 100%가 기대한 제목·날짜·URL로 파싱됨<br>다시 돌려도 중복 행 0건<br>새 공지가 6시간 안에 화면에 나옴<br>robots.txt가 막은 주소 요청 0건 |
| .ics 동기화<br>(BE-2 · 1) | 학생별 암호화된 i-Campus 주소로 매시간 가져와 `tasks`(source = ics)에 넣고 바뀐 것을 고친다. 반복 일정은 icalendar + recurring-ical-events로 펼친다. | 반복 일정이 정확히 펼쳐짐<br>마감 일시 일치율(M-01) 98% 이상: 표본 100건을 원본과 대조<br>i-Campus 변경이 1시간 안에 반영 |
| 링크 점검<br>(BE-2 · 2) | 매일 바로가기 링크를 열어 보고 결과를 `links`에 기록한다. | 링크 정상률(M-19) 99% 이상<br>끊긴 링크가 관리자 화면에 보임 |
| 정기 작업과 배포<br>(BE-2 · Sprint 0–6) | `pipeline-hourly.yml`(매시간 .ics, cron은 정각을 피함, 3시간마다 크롤링과 정리), `pipeline-daily.yml`(05:00 KST 카드, 링크 점검, 지표 집계, age 암호화 백업 7일 보관, 2학기 지난 공지 삭제), `release.yml`(마이그레이션 → 졸업 규칙 YAML 싣기 → Render 배포 훅 → `/health` 확인 → Vercel CLI 웹 배포), PR마다 Vercel CLI 미리보기, UptimeRobot 5분 확인. | `ci.yml`·`release.yml` 통과, `/health` 200<br>Vercel CLI 배포와 PR 미리보기 동작<br>매일 작업이 `metric_logs`에 기록<br>Actions 로그에 이름·이메일·.ics 주소 0건 |
| 백업<br>(BE-2 · 6) | 매일 암호화 백업을 로컬 Supabase로 복원해 본다. | 2주마다 복원 성공, 행 수 일치 |
 
### AI·RAG
 
| 개발 항목 | 만드는 것 | 완료 기준 |
|---|---|---|
| `plan_today`<br>(AI·RAG-1 · 1) | `rules/`의 순수 함수. 입력은 할 일 목록, 가중치, 현재 시각이고 입출력과 LLM이 없다. 순서 규칙(제안): ① 끝낸 일은 뺀다 ② 임박(24시간 안) → 지연 → 나머지 순으로 묶는다 ③ 묶음 안에서는 마감이 이른 것 먼저 ④ 마감이 같은 날이면 학생이 ⚙ 설정에서 고른 종류 순서(기본: 강의 → 퀴즈 → 과제) ⑤ 그래도 같으면 예상 소요 시간이 짧은 것 먼저. 예상 소요 시간은 종류별 기본값(제안: 강의 60분, 퀴즈 30분, 과제 120분)이고 학생이 할 일마다 고칠 수 있다. | 규칙 ①–⑤를 하나씩 확인하는 단위 시험 통과(예: 오늘 밤 마감인 퀴즈가 모레 마감인 강의보다 먼저, 마감이 같으면 강의가 과제보다 먼저)<br>같은 할 일·가중치·시각을 넣으면 항상 같은 순서<br>할 일 500개를 0.1초 안에 정렬 |
| `diagnose`<br>(AI·RAG-1 · 4) | 이수 내역과 학번별 `grad_rules`로 영역별 남은 학점, 필수 과목, 학과 졸업 평가 항목(연구논문·작품, 인턴십, 산학협력프로젝트, 1만라인프로젝트)의 대상·충족 여부를 계산한다. 성적은 받지 않으므로, 성적 조건이 있는 항목(예: 1만라인 지정 과목 중 2과목 이상 B 이상)은 조건과 해당 과목을 보여주고 '확인 필요'로 표시한다. 인턴십처럼 학교 확인이 필요한 항목도 '확인 필요'로 표시한다. | 졸업요건 사례 20개로 진단 일치율(M-10) 95% 이상<br>같은 입력 10번에 같은 진단(M-12) 100%<br>계산 1초 안 |
| `build_roadmap`, `verify`<br>(AI·RAG-1 · 5) | 목표 졸업 학기부터 역산해 과목을 배치한다. 필수·선수과목 먼저, 선택 영역은 후보 2–3개, 개설강좌가 있으면 개설 과목 먼저, 학기당 학점 수 반영, 1만라인 지정 과목과 산학협력프로젝트 과목 배치. 목표 학기가 너무 이르면 검증을 통과하는 가장 빠른 졸업 학기를 찾는다. `verify`는 학기당 학점 상한, 선수과목, 요건을 검사하고, 맞출 수 없으면 다시 만들지 않고 그 조건을 돌려준다. | 로드맵 제약 위반(M-13) 0건<br>추천 과목 적합률(M-14, 남은 영역을 채우는 과목 ÷ 전체 추천 과목) 100%<br>속성 시험: 학점이 상한을 넘지 않고 선수과목이 항상 먼저<br>`rules/` 줄 커버리지 90% 이상, 로드맵 10초 안 |
| 졸업요건 Agent<br>(AI·RAG-2 · 4, 5) | `agent/`의 LangGraph 그래프: 빠진 정보(학번, 이수 내역, 목표 학기) 먼저 묻기 → `diagnose` → `build_roadmap` → `verify` → LLM이 결과를 설명하고 근거 조항을 인용. 숫자는 LLM이 만들지 않는다. 대화에서 나온 사실은 기억 제안(`MemoryUpdate`)으로만 낸다. | 빠진 정보를 항상 먼저 물음(대본 대화 시험)<br>근거 조항 정확도(M-11, 인용 조항이 원문에 있는 비율) 100%<br>답의 학점 숫자가 규칙 엔진 결과와 100% 일치 |
| GLS 붙여넣기 읽기<br>(AI·RAG-2 · 4) | 졸업요건 Agent의 한 단계. 채팅에 붙여 넣은 글이 GLS 졸업요건 충족현황이면 LLM 한 번(구조화 출력: 영역, 기준학점, 취득학점, 잔여학점)으로 표를 읽는다. 읽은 값은 `rules/`의 검사 함수가 다시 본다: 숫자가 붙여 넣은 글에 실제로 있는지, 세부 영역 합이 총학점과 맞는지, 잔여 = 기준 − 취득인지. 통과한 칸은 표로, 어긋나거나 빠진 칸은 '확인 필요'로 내고, 학생이 확인해야 쓴다. GLS 글이 아니거나 LLM을 쓸 수 없으면 직접 입력 표로 넘긴다. | 붙여넣기 예시 10개 이상(깨끗한 것, 일부만, 칸이 밀린 것, 다른 화면)으로 시험<br>깨끗한 예시에서 칸 95% 이상 정확<br>글에 없는 숫자를 표에 넣은 경우 0건<br>실패 예시는 모두 '확인 필요'나 직접 입력으로 넘어감 |
| `check_claim`<br>(AI·RAG-1 · 5) | 학생 정정을 졸업 규정과 대조해 `match`, `mismatch`, `unverifiable`과 인용을 돌려준다. 저장된 규칙 데이터가 졸업 규정과 다르면 `rule_conflict`. | 학생 정정 사례 20개 판정 모두 정확<br>규칙 데이터를 일부러 틀리게 한 사례에서 `rule_conflict` |
| 공지 정리·임베딩<br>(AI·RAG-2 · 2) | `pipeline/process/`: `processed_at`이 빈 `raw_documents`를 읽어 분류, 한 줄 요약, 신청·제출 기한, 행사 일시와 신청 마감을 뽑는다. 공지 10건을 LLM 호출 한 번으로 처리하고, 출력은 스키마로 검사한다(틀리면 한 번 다시, 또 틀리면 검토 대상 표시). `notices`, `events`, `knowledge_chunks`(본문 조각 + 768차원 임베딩)를 채우고 `processed_at`을 기록한다. 같은 출처의 내용이 바뀌면 예전 조각을 바꾸고, 더 이상 링크되지 않는 자료의 조각은 지운다. | 라벨 붙인 공지 50건으로 기한 추출 정확도(M-02) 90% 이상<br>모든 출력이 스키마를 통과하거나 검토 대상으로 표시됨<br>바뀐 자료의 예전 조각 0건<br>5,000개 조각 30MB 미만, 상위 5개 조회 200ms 이하 |
| 졸업 규칙 YAML<br>(AI·RAG-1 · 4, 6) | `rules/data/grad_rules/`에 학번별 YAML(영역 학점, 필수 과목, 졸업 평가 항목)을 둔다. 학점 기준은 운영자가 학교 학점이수표에서, 졸업 평가 항목과 필수 과목은 학과 졸업요건 자료에서 옮겨 넣는다. 바꿀 때는 PR로 올리고 다른 팀 리뷰어가 원본과 대조한 뒤 병합한다. 병합된 YAML만 배포 때 `grad_rules`에 실린다. 4단계에서 학번 2개, 6단계에서 나머지 학번. | 모든 값이 원본(학교 학점이수표, 학과 졸업요건 자료)과 일치(리뷰어가 대조)<br>YAML 형식 검사 CI 통과<br>병합되지 않은 변경은 규칙 엔진이 쓰지 않음 |
| 캠퍼스 라이프 Agent<br>(AI·RAG-2 · 3) | `recommend`: 내 기억과 후보 행사로 카드를 고른다. 끝난 행사와 신청 마감이 지난 행사는 빼고, 날짜가 없는 항목은 게시 후 30일까지만, 모든 카드에 출처 링크. LLM을 쓸 수 없으면 관심 분야와 행사의 임베딩 유사도로 순위를 매긴다. `interpret_settings`: ⚙ 설정 글을 추천 선호와 기억할 사실로 바꾼다. 오늘의 플랜에는 쓰지 않는다. | 지난 행사 노출(M-06) 0건<br>모든 카드에 출처 링크, LLM을 꺼도 카드가 나옴<br>운영 중 카드 저장률(M-07) 20% 이상, 관심 없음 비율(M-08) 30% 이하 |
| LLM 클라이언트<br>(AI·RAG-2 · 2) | `llm/`: google-genai를 감싼 작은 클라이언트 하나(LiteLLM은 쓰지 않는다). `models.yaml`의 모델 순서(Flash-Lite 기본), AI Studio에서 확인한 한도로 정한 하루 호출 예산(DB에서 센다), 보내기 직전 개인정보 검사(이름·이메일·학번 형태가 있으면 보내지 않음). 제공자 이름을 아는 유일한 코드이고, 프롬프트 본문은 로그에 남기지 않는다. 예산을 넘거나 한도 오류(429)가 나면 채팅은 LLM 설명 없이 코드가 만든 결과 글을 보낸다. | 429를 일부러 내도 채팅이 코드 결과 글로 끝남<br>정보 기능은 영향 없음<br>하루 생성 호출이 정한 예산 안<br>시험 프롬프트에 이름·이메일·학번 0건, 로그에 프롬프트 본문 0건 |
| 오류 신고와 회귀 사례<br>(AI·RAG-1, AI·RAG-3 · 5) | 자동 오류 신고(`rule_conflict`, 학생이 확인한 GLS 기준학점의 불일치)가 운영자에게 가고, 처리된 규칙 데이터 오류를 회귀 시험 사례로 추가한다. | 오류 신고 처리율(M-20, 7일 안에 처리한 신고 ÷ 전체) 90% 이상<br>처리된 규칙 데이터 오류는 모두 시험 사례가 됨 |
 
### 통합 담당 (AI·RAG-3)
 
| 개발 항목 | 만드는 것 | 완료 기준 |
|---|---|---|
| `contracts/agent.py`, `main.py`<br>(Sprint 0) | API ↔ Agent 인터페이스(`AgentFacade`, `RuleEngine`, 주고받는 Pydantic 모델)와 실제 구현체를 연결하는 `main.py`(`AGENT_MODE`로 같은 컨테이너 또는 별도 서버). 인터페이스는 Sprint 0이 끝나면 고정한다. | mypy 통과<br>`FakeAgent`와 실제 Agent가 같은 인터페이스 시험 통과<br>import-linter 의존 방향 위반 0건 |
| CI 게이트<br>(Sprint 0) | `ci.yml`: 정적 검사(ruff, mypy, ESLint, tsc, import-linter), 단위 시험, 인터페이스 시험, 회귀 시험. LLM은 녹화한 응답을 재생해 CI에서 실제 호출을 하지 않는다. | 정적 검사 오류 0건<br>CI의 실제 LLM 호출 0회<br>기한 추출(M-02)이나 졸업요건 지표(M-10–M-14)가 떨어지면 병합 차단 |
| 평가 데이터<br>(Sprint 0–5) | `evaluation/`: 마감 표본 100건(M-01), 라벨 공지 50건(M-02), 졸업요건 사례 20개(M-10–M-14), 학생 정정 사례 20개. 정답과 함께 저장소에 둔다. | 모든 표본에 정답과 원본 출처가 있음 |
| 추적 표시<br>(모든 단계) | 모든 시험에 다루는 기능과 지표를 적는다(예: `# 졸업요건 진단, M-10`). | 모든 기능이 시험과 연결됨 |
| 주간 지표 보고<br>(6단계부터) | 지표 화면과 `metric_logs`로 M-01–M-20을 주마다 보고하고, 목표에 못 미친 지표를 다음 주 개선 과제로 정한다. | M-01–M-20을 모두 보고 |
| 지표 집계<br>(6) | 모든 팀이 남긴 이벤트를 `metric_logs`에서 주별로 모은다. | M-01–M-20 주별 값이 빠짐없이 만들어짐 |
| 성능<br>(마지막) | Locust로 100명 동시 접속을 시험하고, 사용자를 더 늘려 홈 P95가 3초를 넘는 한계점을 잰다. Render 메모리와 UptimeRobot 가동률을 확인한다. | 100명에서 홈 P95 3초 이하, 채팅 진행 상태 3초 이하<br>메모리 400MB 미만<br>가동률 99% 이상<br>한계점을 2절 확장성과 한계 표에 기록 |