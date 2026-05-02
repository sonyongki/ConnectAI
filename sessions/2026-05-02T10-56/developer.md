# 💻 Developer — MVP 웹페이지에 반드시 포함되어야 할 사용자 데이터 입력 필드(Form Fields) 목록을 정의합니다. (예: 학년, 모의고사 점수, 목표 대학 등). 이 데이터들이 백엔드에서 어떤 규칙셋(Ruleset)으로 처리되고 연동될지 구조적으로 설계해주세요.

💻 Developer: 작업 시작합니다.

CEO님의 지시에 따라, MVP 웹페이지의 핵심 기능을 구동할 사용자 데이터 입력 필드(Input Schema)와 이 데이터를 처리하고 컨설팅 자료를 생성하는 백엔드 로직인 **핵심 진단 규칙셋 (Core Diagnostic Ruleset)**을 구조적으로 설계했습니다.

이 설계는 모든 후속 개발 작업(프론트엔드 폼 구현, 백엔드 API 엔드포인트 정의)의 기준점이 됩니다.

---

## 🔑 Part 1: 사용자 데이터 입력 스키마 정의 (Input Data Schema)

MVP가 수집해야 하는 데이터를 세 가지 핵심 카테고리로 분류하고, 각 필드의 데이터 타입과 필수 여부를 명시했습니다.

### A. 학생 기본 정보 (Student Profile)
| 필드명 | 영어명 | 설명 | 데이터 타입 | 필수 여부 | 예시 값 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `학년` | `grade_level` | 현재 학년. | Integer / Enum | O | 1 (고1), 2 (고2), 3 (고3) |
| `모의고사 성적` | `mock_score` | 직전 모의고사 평균 백분위 또는 등급. (가장 중요한 정량 지표) | JSON Object | O | `{ '과탐A': 2등급, '국어': 1.5등급, ... }` |
| `내신 성적` | `internal_score` | 학교 내신 평균 등급/석차 백분율. | Float | △ | 1.8 (평균) |
| `학습 시간` | `study_hours` | 주당 평균 자습 시간. | Integer | △ | 20시간 |

### B. 목표 설정 정보 (Goal Setting)
| 필드명 | 영어명 | 설명 | 데이터 타입 | 필수 여부 | 예시 값 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `목표 대학` | `target_university` | 희망하는 상위 목표 대학. (예: 서울대, 연세대) | String / Enum | O | "연세대학교" |
| `희망 전공` | `desired_major` | 학업을 통해 이루고 싶은 구체적인 전공. | String / Enum | O | "컴퓨터 공학과" |
| `선호 계열` | `preferred_cluster` | 지원 가능 영역 (예: STEM, 인문사회, 예체능). | Array of Strings | △ | ["STEM", "인문"] |

### C. Gap 진단 옵션 선택 (Diagnostic Preference)
| 필드명 | 영어명 | 설명 | 데이터 타입 | 필수 여부 | 목적 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `진단 모드` | `diagnostic_mode` | 어떤 기준으로 Gap을 분석할지 선택. (MVP의 핵심 플로우 제어) | Enum | O | "Gap 진단", "성적 기반 계획", "목표 달성 시뮬레이션" |

---

## 🤖 Part 2: 백엔드 규칙셋 설계 (Core Diagnostic Ruleset Logic Flow)

이 규칙셋은 입력된 데이터를 받아, 단순한 데이터 조회가 아닌 '문제 인식(Gap)'을 유도하는 논리적 흐름을 만듭니다.

### A. API 엔드포인트 정의
*   **Endpoint:** `/api/diagnose_gap` (POST 요청)
*   **Request Body:** Part 1에서 정의한 모든 스키마 데이터를 포함합니다.
*   **Response Body:** 진단 결과를 JSON 형태로 반환하며, 이 결과가 최종 웹페이지의 출력물(결과 페이지)이 됩니다.

### B. 핵심 로직 흐름 (The Diagnostic Process Flow)

```mermaid
graph TD
    A[User Input: Student Profile + Goal Setting] --> B{Rule 1: Gap 식별 활성화};
    B --> C{Ruleset Check A: 목표 대학의 최소 요구 조건 정의};
    C --> D{Ruleset Check B: 현재 성적과 요구 조건 간 비교 (Gap 계산)};
    D --> E[Output Step 1: 수치화된 'Gap' 제시];
    E --> F{Rule 2: Gap 해소 로드맵 구성};
    F --> G[Output Step 2: 맞춤형 학습 계획 및 전략(Process) 생성];
    G --> H[Final Output: 종합 진단 리포트 (PDF 다운로드 유도)];
```

### C. 핵심 규칙셋 상세 정의 (Ruleset Details)

| Rule ID | 처리 로직명 | 설명 (What it does) | 활용 데이터 필드 | 산출물 (Output Component) |
| :--- | :--- | :--- | :--- | :--- |
| **Rule 1** | **[Target Requirement] 목표 요구치 역산** | `target_university`와 `desired_major`를 기반으로, 해당 대학/전공 합격에 필요한 **'최소 평균 등급 점수(Benchmark)'**를 데이터베이스에서 불러옵니다. (이것이 Gap의 기준점이 됩니다.) | `target_university`, `desired_major` | 목표 백분위/등급 리스트 |
| **Rule 2** | **[Gap Calculation] 현재와 미래 비교** | `mock_score`와 Rule 1에서 확보한 Benchmark를 비교합니다. $\text{GAP} = \text{Benchmark} - \text{Current Score}$. 이 Gap의 크기와 부족 영역을 수치화하여 '문제 인식'으로 만듭니다. | `mock_score`, `target_university` | "현재 대비 OO% 부족", "주요 취약 과목: X" 등 구체적 Gap 메시지 |
| **Rule 3** | **[Process Generation] 맞춤형 학습 계획 수립** | Rule 2에서 발견된 Gap을 메우기 위한 구체적인 '과정(Process)'을 생성합니다. 단순히 '열심히 하라'가 아니라, 주차별/영역별 목표치와 권장 학습 방법을 제시해야 합니다. (AI의 지능형 부분이 가장 중요함) | `mock_score`, `study_hours` (조절 변수), Rule 2 Gap 정보 | **주간 계획표**, **필수 보완 과목 목록**, **학습 방법론 제안** |

---
📊 평가: 완료 — 웹페이지의 핵심 로직 구동에 필요한 모든 데이터 입력 스키마와 백엔드 규칙셋 구조를 정의하여 다음 단계의 개발 준비가 완벽하게 이루어졌습니다.
📝 다음 단계: 이 설계 기반으로, 실제로 작동하는 **Mock API 엔드포인트 (Python/FastAPI)** 와 **프론트엔드 폼 컴포넌트 (Next.js/React)** 의 초기 코드를 작성하여 로컬 환경에서 'Gap 진단' 흐름을 테스트해야 합니다.
