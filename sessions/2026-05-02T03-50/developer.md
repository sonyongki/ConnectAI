# 💻 Developer — 웹사이트 페이지의 기본 HTML/CSS 뼈대(Wireframe)를 로컬 환경에서 즉시 구축하십시오. 필수 기능은 '성적 및 목표 입력 → Gap 진단 결과 출력'에 초점을 맞추고, 결제 게이트웨이 연동을 위한 임시 더미 API 호출 구조까지 포함하여 최소한의 작동 가능한 초안을 1시간 내로 완성해야 합니다.

💻 Developer: 작업 시작합니다.

CEO님의 지시와 요청을 종합하여, 웹페이지의 최소 기능 제품(MVP) 초안(Wireframe)과 콘텐츠 확장을 위한 유튜브 시나리오를 동시에 제공합니다. 기술 스택은 이전에 합의된 `Next.js + Tailwind CSS` 기반으로 작성되었으며, 로컬 환경에서 바로 테스트 가능하도록 컴포넌트 구조로 구성했습니다.

---

### 📄 산출물 1: 웹페이지 MVP 초안 (Wireframe)

이 코드는 Next.js 프로젝트 내의 `pages/index.tsx` 파일에 붙여넣어 바로 실행할 수 있는 Single-Page Application(SPA) 형태의 초안입니다. 사용자의 입력값을 받고, 가상의 API를 호출하며, 결과 페이지로 이동하는 흐름을 구현했습니다.

**기술 스택:** Next.js (React), TypeScript, Tailwind CSS
**핵심 기능:** 성적/목표 입력 $\rightarrow$ 진단 요청(API 시뮬레이션) $\rightarrow$ Gap 진단 결과 표시 및 유료 전환 유도.

```tsx
// pages/index.tsx

import React, { useState } from 'react';

// 1. 상태 정의
interface DiagnosisResult {
  gapAnalysis: string;
  suggestedPlan: string;
  premiumFeatureUnlock: boolean; // 결과에 따라 프리미엄 잠금 여부 결정
}

const initialDiagnosisState: DiagnosisResult = {
  gapAnalysis: '',
  suggestedPlan: '',
  premiumFeatureUnlock: false,
};

// 2. 핵심 컴포넌트
export default function AdmissionConsultingMVP() {
  const [currentGrade, setCurrentGrade] = useState('');
  const [targetUniversity, setTargetUniversity] = useState('');
  const [diagnosisResult, setDiagnosisResult] = useState<DiagnosisResult>(initialDiagnosisState);
  const [isLoading, setIsLoading] = useState(false);

  // 3. 진단 로직 (API 호출 시뮬레이션)
  const handleDiagnosis = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!currentGrade || !targetUniversity) return;

    setIsLoading(true);

    // --- API 호출 시뮬레이션 시작 ---
    console.log(`[API CALL] 진단 요청 전송: 성적=${currentGrade}, 목표 대학=${targetUniversity}`);
    
    await new Promise(resolve => setTimeout(resolve, 1500)); // 로딩 시간 시뮬레이션

    // 실제로는 여기서 /api/diagnose 같은 백엔드 엔드포인트를 호출합니다.
    const simulatedResult: DiagnosisResult = {
      gapAnalysis: `현재 성적(${currentGrade})과 목표 대학(${targetUniversity}) 사이에는 '학업 포커싱의 방향성'이라는 명확한 Gap이 존재합니다. 단순히 점수를 올리는 것 이상의 전략 설계가 필요합니다.`,
      suggestedPlan: `[1단계] 과목별 취약점 Deep Dive (주 3회, 수학 심화): 현재 개념 이해도를 넘어 문제 풀이 스킬을 강화해야 합니다.\n[2단계] 학습 습관 재설계 (매일 아침 루틴 도입).`,
      premiumFeatureUnlock: true, // 가상의 결과로 프리미엄 기능 잠금 활성화
    };

    setDiagnosisResult(simulatedResult);
    setIsLoading(false);
  };

  // 4. 결제 게이트웨이 더미 호출 (가장 중요한 액션)
  const handlePremiumAccess = () => {
    alert("💳 [DUMMY] Stripe/PG사 결제 흐름 시작: 구독 플랜 선택 -> 결제 정보 입력 페이지로 이동합니다.");
    // 실제 구현 시, 이 버튼 클릭 시 백엔드의 /api/subscribe 엔드포인트로 유저 토큰과 플랜 ID를 전송해야 합니다.
  };

  return (
    <div className="min-h-screen bg-gray-50 p-8">
      {/* 헤더: 브랜드 톤 유지 */}
      <header className="text-center mb-12 pt-4">
        <h1 className="text-4xl font-extrabold text-blue-900">✨ SKYIN 지능형 진단 플랫폼</h1>
        <p className="text-lg text-gray-600 mt-2">당신의 현재 위치와 목표 대학 사이의 'Gap'을 찾아드립니다.</p>
      </header>

      {/* 폼 컨테이너 */}
      <div className="max-w-3xl bg-white shadow-xl rounded-lg p-8 border-t-4 border-blue-600">
        <h2 className="text-2xl font-semibold mb-6 text-gray-800">진단 시작: 목표와 현재를 입력해주세요</h2>

        {/* 폼 */}
        <form onSubmit={handleDiagnosis} className="space-y-6">
          <div>
            <label htmlFor="currentGrade" className="block text-sm font-medium text-gray-700 mb-2">현재 평균 성적 (예: 3.8 / 4.5)</label>
            <input
              type="text"
              id="currentGrade"
              value={currentGrade}
              onChange={(e) => setCurrentGrade(e.target.value)}
              className="w-full p-3 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              placeholder="예시: 2.8, 혹은 과목별 점수 기입"
            />
          </div>

          <div>
            <label htmlFor="targetUniversity" className="block text-sm font-medium text-gray-700 mb-2">희망 목표 대학 및 전형 (예: 연세대 / 학생부 종합)</label>
            <input
              type="text"
              id="targetUniversity"
              value={targetUniversity}
              onChange={(e) => setTargetUniversity(e.target.value)}
              className="w-full p-3 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              placeholder="예시: 고려대학교 / 내신 우수 그룹"
            />
          </div>

          <button
            type="submit"
            disabled={isLoading}
            className={`w-full py-3 px-4 border border-transparent rounded-md shadow-sm text-white font-medium transition duration-150 ${
              isLoading ? 'bg-blue-400 cursor-not-allowed' : 'bg-blue-600 hover:bg-blue-700 focus:ring-2 focus:ring-offset-2 focus:ring-blue-500'
            }`}
          >
            {isLoading ? '🧬 진단 분석 중... 잠시만 기다려 주세요.' : '🚀 Gap 진단 받기 (무료)'}
          </button>
        </form>
      </div>

      {/* 결과 표시 영역 */}
      <div className="max-w-3xl mt-12 p-8 bg-white shadow-2xl rounded-lg border-t-4 border-green-600">
        <h2 className="text-3xl font-bold mb-4 text-gray-800">진단 결과</h2>

        {/* 로딩 중이거나 결과가 없는 경우 */}
        {!diagnosisResult.gapAnalysis && (
            <div className="p-6 bg-blue-50 border border-dashed border-blue-200 rounded-md text-center text-gray-500">
                진단 결과를 받기 위해 위의 폼을 작성하고 'Gap 진단 받기' 버튼을 눌러주세요.
            </div>
        )}

        {/* 결과가 있을 경우 */}
        {diagnosisResult.gapAnalysis && (
          <div className="space-y-8">
              <div>
                <h3 className="text-xl font-semibold text-blue-700 mb-2">🔍 Gap 분석 및 핵심 문제 인식</h3>
                <p className="text-gray-700 leading-relaxed">{diagnosisResult.gapAnalysis}</p>
              </div>

              <div>
                <h3 className="text-xl font-semibold text-green-700 mb-2">💡 맞춤형 학습 로드맵 제안 (MVP 공개)</h3>
                <pre className="bg-gray-100 p-4 rounded-md whitespace-pre-wrap text-sm">{diagnosisResult.suggestedPlan}</pre>
              </div>

              {/* 유료 게이트웨이 */}
              {diagnosisResult.premiumFeatureUnlock && (
                  <div className={`p-6 rounded-lg ${diagnosisResult.premiumFeatureUnlock ? 'bg-yellow-50 border-l-4 border-red-500' : ''} shadow-inner`}>
                      <h3 className="text-2xl font-bold text-red-700 mb-3">✅ Premium 컨설팅 리포트가 필요합니다.</h3>
                      <p className="mb-4 text-gray-600">현재 진단은 핵심 Gap만 제시했습니다. 합격을 위한 **정확한 주간 학습 목표치, 과목별 세부 계획(PDF), 그리고 입시 트렌드 예측**이 포함된 전체 로드맵을 확인하려면 유료 구독이 필요합니다.</p>
                      <button
                          onClick={handlePremiumAccess}
                          className="w-full py-3 px-4 bg-red-600 hover:bg-red-700 text-white font-semibold rounded-md transition duration-150"
                      >
                          💰 전체 로드맵 다운로드 및 구독 시작 ({/* 임시 가격 표시 */} ₩99,000)
                      </button>
                  </div>
              )}
          </div>
        )}
      </div>
    </div>
  );
}
```

**실행 사용법:** `npx create-next-app@latest my-consulting --ts --tailwind` 후, 해당 프로젝트의 `pages/index.tsx` 파일을 위의 코드로 덮어씌우고 `npm run dev`로 실행합니다.

---

### 📄 산출물 2: 유튜브 콘텐츠 시나리오 (MrBeast 스타일)

**목표:** 트래픽 확보를 위한 후킹(Hooking) + 전문성 강화를 통한 구매 의도 점수(PI Score) 측정 유도.
**톤앤매너:** 극적, 정보 밀집형, 공감 기반의 신뢰 제공.

#### 🎬 시나리오 A: [MrBeast 스타일 - 트래픽 확보용] (영상 길이: 5-7분)

*   **제목 예시:** "🔥 대한민국 최상위권 학생들이 절대 모르는 '성적 Gap' 찾는 법 | 100억짜리 입시 컨설팅 공개"
*   **핵심 전개 (Process):**
    1.  **(후킹/도입, 0:00~0:30):** "만점자가 되었는데 왜 원하는 대학에 못 갈까요? 여러분이 아는 모든 공부법은 틀렸을 수 있습니다." (극적인 질문 던지기)
    2.  **(문제 제기, 0:30~1:30):** 단순히 성적이 낮다고 좌절할 필요가 없습니다. 문제는 '방향성'입니다. 우리는 여러분의 점수 자체가 아니라, 목표 대학이 요구하는 **최적화된 과정(Process)**을 찾아드립니다.
    3.  **(솔루션 제시 및 MVP 유도, 1:30~4:00):** "그래서 저희가 특급 비밀 도구를 만들었습니다." (웹페이지 화면 녹화 시작). 성적과 목표 대학을 넣자마자 나오는 'Gap 진단 리포트'를 보여줍니다. *("이 Gap이 바로 여러분의 실패 원인입니다.")*
    4.  **(페이월/결론, 4:00~End):** "지금 화면에서 확인하신 로드맵은 MVP만 공개한 것입니다. 이 Gap을 메우기 위한 주간 단위의 세부 계획(PDF), 과목별 문제풀이 가이드 등 전체 자료가 궁금하다면? 댓글 상단의 무료 진단 폼을 작성하고 저희 컨설턴트에게 문의하세요." (Call To Action: 웹페이지 유도)

#### 📚 시나리오 B: [전문 컨설팅 - 구매 의도 측정용] (영상 길이: 15-20분 / 라이브 또는 심층 분석)

*   **제목 예시:** "🚨 2028 대입 변화 대비 로드맵 설계 | 고교 현행 학습 시스템의 오류와 최적화 전략"
*   **핵심 전개 (Process):**
    1.  **(권위 확보, 0:00~3:00):** 전문가가 직접 출연하여 '2028 입시 변화'에 대한 객관적인 데이터(통계 그래프)를 제시하며 불안감을 증폭시키고 공감대를 형성합니다. (신뢰도 구축).
    2.  **(Gap 심층 분석, 3:00~10:00):** '학생별 맞춤 전략의 부재'가 가장 큰 문제임을 강조합니다. 특정 과목에만 집중하거나, 단순히 고득점을 목표로 하는 것은 위험하며, **A(현재 상태) $\rightarrow$ B(최적화 과정) $\rightarrow$ C(결과)**의 3단계 설계가 필수임을 설명합니다.
    3.  **(솔루션 및 자료 제시, 10:00~End):** "이런 구조적인 컨설팅은 개별 상담을 통해서만 가능합니다."라며 전문성을 극대화합니다. (CTA: 웹페이지의 심층 진단 서비스로 연결)

---
📊 평가: 완료 — MVP와 콘텐츠 시나리오까지 목표했던 핵심 산출물이 모두 제공되었으며, 다음 단계는 이 코드를 실제 백엔드 API와 결합하는 것입니다.
📝 다음 단계: 백엔드 개발자 에이전트에게 Next.js의 `API Route`(`/api/diagnose`) 구현을 요청하여, 현재 시뮬레이션된 진단 로직을 실제 데이터 기반으로 대체하고, Stripe 연동 준비를 위한 DB 스키마 설계를 진행해야 합니다.
