# 📋 작업 브리프

**원 명령:** # SKYIN Project: AI-Driven College Admission Consulting Platform

## 1. Project Overview
SKYIN은 MrBeast 스타일의 고자극·고효율 유튜브 콘텐츠로 트래픽을 확보하고, 이를 구독형 AI 입시 컨설팅 웹서비스로 전환하는 하이브리드 에듀테크 프로젝트이다. AI 에이전트는 본 문서를 바탕으로 유튜브 콘텐츠 기획 자동화, 웹서비스 프론트엔드/백엔드 구축, 결제 연동 및 사용자 맞춤형 학습 계획 생성 파이프라인을 구축한다.

## 2. System Architecture & Tech Stack (macOS M4 Optimized)
- **Frontend:** Next.js (React), TailwindCSS, Zustand (대시보드 상태 관리)
- **Backend/API:** Python (FastAPI), Supabase (PostgreSQL, Auth)
- **AI/LLM Engine:** 로컬 LLM(MLX Framework 활용, 프라이버시 및 비용 절감) 또는 OpenAI API (수능 계획서 생성)
- **Automation Pipeline:** n8n (유튜브 업로드 감지 -> 마케팅 이메일 발송 -> 유입 트래킹)
- **Payment Gateway:** PortOne (구 아임포트) 또는 Toss Payments (월 자동 결제 구현)

## 3. AI Agent Execution Phases

### Phase 1: MrBeast-Style YouTube Strategy & Automation (주 1회 퍼블리싱)
AI 에이전트는 유튜브 채널 운영을 위한 다음의 자동화 스크립트와 기획안을 작성한다.
1. **콘텐츠 후킹 기획안 생성기 (Python 스크립트):**
   - **MrBeast Formula 적용:** "나는 100일 만에 수능 5등급을 1등급으로 만들었습니다 (그리고 그 방법을 무료로 공개합니다)" 와 같은 극단적 성취와 시각적 증명을 강조하는 스크립트 템플릿 생성.
   - 처음 10초 이내에 가장 강력한 후킹(시각적 성적표 변화 등) 배치.
2. **Call To Action (CTA) 전략:** 
   - 영상 후반부: "이 영상에서 사용된 맞춤형 수능 100일 로드맵을 당신의 현재 성적에 맞춰 무료로 짜드립니다. 구독하고 SKYIN 웹사이트에서 확인하세요."
3. **구독자 인증 시스템:** 
   - Google YouTube Data API를 연동하여 가입 시 구독 여부를 확인하는 OAuth 플로우 구현.

### Phase 2: Web Platform Development (The Funnel)
AI 에이전트는 다음 기능이 포함된 웹 애플리케이션을 즉시 스캐폴딩한다.

#### 1. Landing Page (Conversion Focus)
- 후킹 메시지와 함께, 학생의 현재 학년/모의고사 성적을 입력받는 간단한 '진단 폼(Form)' 구현.
- 진단 완료 후 결과 보기 버튼 클릭 시 -> **회원가입 및 유튜브 구독 인증 요구 (Paywall / Auth-wall).**

#### 2. Subscription Tiers & Payment Routing (월 구독형 모델)
결제 모듈을 연동하여 다음 권한 제어를 구현한다.
- **[Free Tier] 유튜브 구독자:** 기본 진단 결과 요약본 1회 제공, 전체 계획의 10%만 블러(Blur) 해제 상태로 보여주어 호기심 유발.
- **[Basic Tier] 4,990원/월:** 
  - 웹 대시보드 전체 접근 권한 해제.
  - 수능 전까지의 주차별, 일별 상세 학습 계획 수립 및 트래킹 기능.
- **[Premium Tier] 9,990원/월:**
  - Basic Tier 모든 기능 포함.
  - 매월 말 및 평가원 모의고사 직후, 학생의 학습 데이터를 기반으로 로컬 LLM 또는 API가 분석한 **'개별 맞춤형 딥다이브 컨설팅 리포트 (PDF)'** 다운로드 제공.
  - 과목별 취약점 보완용 맞춤형 문제 추천 리스트 생성.

### Phase 3: AI Study Plan Generator (Core Logic)
AI 에이전트는 학생의 데이터를 받아 수능까지의 계획을 수립하는 Python 기반의 엔진을 개발한다.
- **Input Data:** 현재 학년, 목표 대학, 최근 모의고사 성적 (국/영/수/탐), 하루 가용 학습 시간.
- **Processing:** 
  - 수능 D-Day까지 남은 주(Weeks)를 계산.
  - 과목별 가중치를 계산하여 주차별 목표(예: "수학 기출 3회독", "국어 비문학 구조화 연습")를 할당.
- **Output:** JSON 형태로 프론트엔드 대시보드에 전달하여 시각적 타임라인(Gantt Chart 또는 Stepper)으로 렌더링.

### Phase 4: Automation & Workflow Setup (n8n 기반)
- **Trigger:** 새로운 사용자가 Basic/Premium 구독 결제 성공 시.
- **Action 1:** 데이터베이스(Supabase) 사용자 권한 업데이트.
- **Action 2:** AI 엔진을 호출하여 초기 100일 상세 계획 데이터 생성.
- **Action 3:** 가입 환영 및 계획 수립 완료 안내 이메일 자동 발송.

---

## 4. Agent Step-by-Step Task Orders
@AI-Agent: 다음 순서대로 터미널 환경(macOS)에서 작업을 실행할 것.

1. `npx create-next-app@latest skyin-web --tailwind --eslint --app` 실행하여 프론트엔드 셋업.
2. `mkdir skyin-backend && cd skyin-backend && python3 -m venv venv && source venv/bin/activate && pip install fastapi uvicorn sqlalchemy psycopg2-binary` 로 백엔드 환경 구성.
3. Supabase 프로젝트 초기화 및 Auth 플로우 (YouTube OAuth) 코드를 작성하여 프론트엔드에 적용.
4. 사용자 입력값을 바탕으로 수능 로드맵 JSON을 반환하는 프롬프트 체인(Prompt Chain) 작성.
5. PortOne(또는 Stripe) 정기결제 API 통합 로직 작성 및 `/api/subscribe` 엔드포인트 구현.
6. 유튜브 스크립트 생성을 위한 프롬프트 템플릿(MrBeast 스타일 구조 적용) 파일 작성.

## 요약
SKYIN 프로젝트의 하이브리드 에듀테크 플랫폼을 구축하는 초기 단계입니다. 핵심 목표는 MrBeast 스타일의 유튜브 트래픽을 확보하고, 이를 고도화된 구독형 AI 입시 컨설팅 웹서비스(Funnel)로 전환할 수 있는 전반적인 시스템 아키텍처를 완성하는 것입니다.

## 분배
- **💰 Business**: 제공된 전체 시스템 구조와 KPI (구매 의도 점수 측정, 구독 모델)가 일관되게 작동하는지 최종 검토하고, 각 단계(유튜브 -> 랜딩 페이지 진단 폼 -> 유료 서비스)에서 발생 가능한 사용자 이탈 지점(Friction Points)과 그에 대한 최소한의 보완 전략을 3가지 이상 제시하시오.
- **💻 Developer**: 기술 스택 기반 환경 구축 및 핵심 파이프라인 개발을 수행합니다. (1) Next.js를 활용하여 랜딩 페이지 초기 구조(TailwindCSS 적용)를 셋업하고, '진단 폼'과 '결과 확인 버튼'의 프론트엔드 컴포넌트를 구현하십시오. (2) FastAPI 백엔드를 구성하고 Supabase와의 연결을 확립한 후, 사용자 인증 및 결제 연동 엔드포인트(`/api/subscribe`)를 최우선으로 개발 완료해야 합니다.
- **✍️ Writer**: AI 핵심 로직과 마케팅 콘텐츠의 뼈대를 만듭니다. (1) 수능 대비 학습 계획을 JSON으로 출력하는 '프롬프트 체인'의 최종 버전(Input Data, Processing, Output 포맷 지정 포함)을 작성하고, 이 구조가 개발자에게 전달될 명세서를 작성하십시오. (2) 유튜브 스크립트 생성 템플릿에 MrBeast 스타일의 강력한 후킹 문구와 극적인 서사 구조를 적용하여 원본 파일을 완성하십시오.
- **🎨 Designer**: 사용자 전환(Conversion)을 최우선 목표로 하는 랜딩 페이지의 시각적 브랜딩 가이드라인을 제시합니다. 특히, '진단 폼' 입력 단계에서 사용자의 학년/성적 입력에 대한 심리적 압박감과 기대감을 동시에 주는 비주얼 요소를 설계하고, 유료 결제(Paywall)가 필요한 순간의 시각적 흐름(Visual Flow)을 명확히 정의하십시오.
- **📺 YouTube**: 개발된 'MrBeast 스타일 스크립트 템플릿'과 '랜딩 페이지 진단 폼'을 연동하는 최초의 마케팅 콘텐츠를 기획합니다. 목표는 최고 트래픽 유발입니다. 시청자의 참여(Engagement)를 극대화할 수 있는 주제 선정, 영상 콘셉트 및 첫 15초 분량의 구체적인 촬영 브리프를 작성하십시오.
