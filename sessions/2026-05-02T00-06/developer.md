# 💻 Developer — 기술 스택 기반 환경 구축 및 핵심 파이프라인 개발을 수행합니다. (1) Next.js를 활용하여 랜딩 페이지 초기 구조(TailwindCSS 적용)를 셋업하고, '진단 폼'과 '결과 확인 버튼'의 프론트엔드 컴포넌트를 구현하십시오. (2) FastAPI 백엔드를 구성하고 Supabase와의 연결을 확립한 후, 사용자 인증 및 결제 연동 엔드포인트(`/api/subscribe`)를 최우선으로 개발 완료해야 합니다.

💻 Developer: 작업 시작합니다.

CEO님의 지시에 따라 웹 서비스의 핵심 기능을 구축하는 것이 최우선입니다. 가장 시급한 **프론트엔드의 진단 폼 구조**와 이 데이터를 처리할 수 있는 **백엔드 API 엔드포인트 골격**을 먼저 구성하겠습니다.

전체 시스템 아키텍처를 고려하여, 다음 세 가지 핵심 요소를 분리된 코드로 제공합니다.

1.  **Frontend (Next.js):** 진단 폼 컴포넌트 구현 및 상태 관리 설정.
2.  **Backend (FastAPI):** 사용자 데이터 수신 및 가짜 AI 로직을 거치는 `/api/diagnose` 엔드포인트 구축.
3.  **Payment Endpoint (FastAPI):** 구독 인증용 `/api/subscribe` 스켈레톤 구현.

---

### 💾 1. Frontend Setup: Diagnosis Form Component (Next.js)

이 코드는 `app/components/DiagnosisForm.tsx`에 위치합니다. Next.js의 클라이언트 컴포넌트로 작성하며, 상태 관리는 React Hooks를 사용합니다.

**설치 및 실행 명령어:**
```bash
# 프로젝트 루트에서 다음 설치
npm install tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

**`app/components/DiagnosisForm.tsx`**
```tsx
'use client'; // Next.js 클라이언트 컴포넌트임을 선언

import React, { useState } from 'react';

// 진단 폼 상태 정의
interface FormData {
  grade: string; // 학년 (예: 2학년)
  mockMath: number | '';
  mockKorean: number | '';
  mockEnglish: number | '';
  mockScience: number | '';
  mockSocial: number | '';
}

export default function DiagnosisForm() {
  const [formData, setFormData] = useState<FormData>({
    grade: '',
    mockMath: '',
    mockKorean: '',
    mockEnglish: '',
    mockScience: '',
    mockSocial: '',
  });
  const [isLoading, setIsLoading] = useState(false);
  const [diagnosisResult, setDiagnosisResult] = useState<string | null>(null);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: name === 'grade' ? value : parseFloat(value) || '' }));
  };

  // 폼 제출 핸들러 (API 호출 시뮬레이션)
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!formData.grade && !formData.mockMath) {
      alert("진단 정보를 최소한 하나 입력해주세요.");
      return;
    }

    setIsLoading(true);
    setDiagnosisResult(null);

    try {
      // ⚠️ 실제 API 호출 부분 (FastAPI 백엔드와 연결)
      const response = await fetch('/api/diagnose', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(formData),
      });

      if (!response.ok) throw new Error('진단 실패');
      
      const data = await response.json();
      setDiagnosisResult(data.message); // 백엔드에서 받은 임시 결과 메시지 표시

    } catch (error) {
      console.error("API 호출 오류:", error);
      setDiagnosisResult("데이터 처리 중 오류가 발생했습니다. 잠시 후 다시 시도해주세요.");
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="max-w-3xl mx-auto p-8 bg-white shadow-2xl rounded-xl">
      <h2 className="text-4xl font-extrabold text-indigo-800 mb-6 text-center">
        🎯 SKYIN AI 진단: 당신의 합격 로드맵을 찾아드립니다.
      </h2>
      <p className="text-gray-600 mb-10 text-center">
        현재 성적과 목표를 입력하고, 맞춤형 학습 컨설팅을 받아보세요. (진단 결과는 회원가입/결제 필요)
      </p>

      <form onSubmit={handleSubmit} className="space-y-6 bg-gray-50 p-8 rounded-lg border">
        {/* 학년 입력 */}
        <div>
          <label htmlFor="grade" className="block text-sm font-medium text-gray-700 mb-1">학년 (필수)</label>
          <select
            id="grade"
            name="grade"
            value={formData.grade}
            onChange={handleChange}
            required
            className="w-full p-3 border border-gray-300 rounded-md focus:ring-indigo-500 focus:border-indigo-500 transition duration-150"
          >
            <option value="" disabled>학년을 선택해주세요</option>
            <option value="1">고등학교 1학년</option>
            <option value="2">고등학교 2학년</option>
            <option value="3">고등학교 3학년</option>
          </select>
        </div>

        {/* 모의고사 성적 그리드 (국/영/수) */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            {[
                { name: 'mockKorean', label: '국어 (모의)', value: formData.mockKorean },
                { name: 'mockEnglish', label: '영어 (모의)', value: formData.mockEnglish },
                { name: 'mockMath', label: '수학 (모의)', value: formData.mockMath },
            ].map((field) => (
                <div key={field.name}>
                    <label htmlFor={field.name} className="block text-sm font-medium text-gray-700 mb-1">{field.label}</label>
                    <input
                        type="number"
                        id={field.name}
                        name={field.name}
                        value={field.value || ''}
                        onChange={handleChange}
                        placeholder="점수 입력 (예: 85)"
                        min="0"
                        max="150"
                        className="w-full p-3 border border-gray-300 rounded-md focus:ring-indigo-500 focus:border-indigo-500 transition duration-150"
                    />
                </div>
            ))}
        </div>

         {/* 모의고사 성적 그리드 (탐구) */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
             {[
                { name: 'mockScience', label: '과학 (모의)', value: formData.mockScience },
                { name: 'mockSocial', label: '사탐/과탐 (모의)', value: formData.mockSocial },
            ].map((field) => (
                <div key={field.name}>
                    <label htmlFor={field.name} className="block text-sm font-medium text-gray-700 mb-1">{field.label}</label>
                    <input
                        type="number"
                        id={field.name}
                        name={field.name}
                        value={field.value || ''}
                        onChange={handleChange}
                        placeholder="점수 입력 (예: 90)"
                        min="0"
                        max="100"
                        className="w-full p-3 border border-gray-300 rounded-md focus:ring-indigo-500 focus:border-indigo-500 transition duration-150"
                    />
                </div>
            ))}
        </div>


        {/* 제출 버튼 */}
        <button
          type="submit"
          disabled={isLoading}
          className={`w-full flex justify-center items-center py-3 px-4 border border-transparent rounded-md shadow-sm text-lg font-medium ${
            isLoading ? 'bg-indigo-300 cursor-not-allowed' : 'bg-indigo-600 hover:bg-indigo-700 focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500'
          } text-white transition duration-150`}
        >
          {isLoading ? (
            <div className="flex items-center">
                <svg className="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle><path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12h12a7.962 7.962 0 01-12 0z"></path></svg>
                진단 중... 로드마드를 분석합니다.
            </div>
          ) : (
            '✅ 진단 결과 확인 및 로드맵 받기 (무료 체험)'
          )}
        </button>

      </form>

       {/* 임시 결과 메시지 영역 */}
      {diagnosisResult && !isLoading && (
        <div className="mt-8 p-6 bg-yellow-50 border-l-4 border-yellow-500 text-yellow-800 rounded">
            <p className="font-bold mb-2">✨ 진단 결과 요약:</p>
            <p>{diagnosisResult}</p>
             <button className="mt-3 text-sm font-semibold underline hover:text-indigo-600 cursor-pointer">
                ➡️ 상세 리포트 보기 (Premium)
            </button>
        </div>
      )}

    </div>
  );
}
```

---

### 🐍 2. Backend Setup: FastAPI Core API (`fastapi_app/main.py`)

이 코드는 백엔드 폴더 `fastapi_app` 내의 `main.py`에 위치합니다. Supabase와 포트원 연동을 위한 구조를 잡습니다.

**설치 및 실행 명령어:**
```bash
# 1. 가상 환경 활성화 (아직 안 했다면)
python3 -m venv venv
source venv/bin/activate

# 2. 필요한 라이브러리 설치
pip install fastapi uvicorn pydantic python-dotenv psycopg2-binary supabase requests
```

**`.env` 파일 설정 (필수)**
프로젝트 루트에 `.env` 파일을 만들고 다음 정보를 입력합니다. 이 정보는 환경변수로만 사용됩니다.
```ini
# Supabase Credentials
SUPABASE_URL=YOUR_SUPABASE_URL
SUPABASE_KEY=YOUR_SUPABASE_ANON_KEY

# Payment Gateway (PortOne 또는 Toss Payments) Secret Key
PG_SECRET_KEY=YOUR_PORTONE_SECRET_KEY 
```

**`fastapi_app/main.py`**
```python
from fastapi import FastAPI, HTTPException, Body
from pydantic import BaseModel
import os
from dotenv import load_dotenv
from supabase import create_client, Client
# 가짜 라이브러리 임포트 (실제 PG 연동 시 사용)
# from portone import PortOneClient 

# .env 파일 로드
load_dotenv()

app = FastAPI(title="SKYIN Consulting API")

# --- 초기화 및 DB 연결 설정 ---
try:
    SUPABASE_URL = os.environ.get("SUPABASE_URL")
    SUPABASE_KEY = os.environ.get("SUPABASE_KEY")

    if SUPABASE_URL and SUPABASE_KEY:
        supabase: Client = create_client(SUPABASE_URL, SUPABASE_KEY)
        print("✅ Supabase 연결 성공.")
    else:
        # DB 연결 실패 시 에러 처리 로직 추가 필요
        supabase = None
except Exception as e:
    print(f"❌ Supabase 연결 초기화 오류 발생: {e}")


# --- 데이터 모델 정의 (Pydantic) ---

class DiagnosisInput(BaseModel):
    """프론트엔드에서 전송되는 진단 폼 데이터 구조"""
    grade: str | None = None
    mockKorean: float | None = None
    mockEnglish: float | None = None
    mockMath: float | None = None
    mockScience: float | None = None
    mockSocial: float | None = None

class SubscriptionRequest(BaseModel):
    """구독 인증 요청 바디 구조"""
    user_id: str
    payment_token: str # PG사에서 발급받은 결제 토큰 (강력한 보안 처리 필요)
    tier: str # Basic 또는 Premium


# --- 🚀 API 엔드포인트 구현 ---

@app.get("/")
def read_root():
    return {"status": "success", "message": "SKYIN AI Consulting API is running."}


@app.post("/api/diagnose")
async def diagnose_student(data: DiagnosisInput = Body(...)):
    """
    [진단 폼 처리]: 사용자 성적 데이터를 받아 학습 로드맵을 생성하는 핵심 엔드포인트.
    실제로는 이 함수 내에서 AI 엔진 (LLM Call)이 호출되어야 합니다.
    """
    if not supabase:
        raise HTTPException(status_code=503, detail="데이터베이스 서비스가 초기화되지 않았습니다.")

    print(f"--- 진단 데이터 수신: {data.model_dump()} ---")
    
    # 1. Supabase에 사용자 데이터를 임시 저장 (트래킹 목적)
    try:
        supabase.table('diagnoses').insert({
            'user_id': 'temp_user', # 실제 로그인된 user_id 사용해야 함
            'data': data.model_dump(),
            'created_at': 'now()'
        }).execute()
    except Exception as e:
         print(f"
