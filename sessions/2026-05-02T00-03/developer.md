# 💻 Developer — 1. 웹페이지 MVP 개발 로직 설계: 사용자 성적 입력(또는 목표대학/성적 입력)에 기반하여 지원 가능 대학을 제시하고, 맞춤형 주간 학습 계획을 자동 생성하는 핵심 API 및 데이터베이스 구조를 명세화해 주세요. 2. 결제 시스템 통합 준비: Stripe 또는 국내 PG사 연동이 가능한 보안 웹페이지 백엔드 프레임워크를 구축할 로직(로그인, 결제, 자료 다운로드)과 필요한 인증 플로우를 설계해주세요.

💻 Developer: 작업 시작합니다.

요청하신 웹페이지 MVP 개발 로직 설계와 결제 시스템 통합 준비는 서비스의 핵심이자 가장 복잡한 백엔드 구조를 필요로 합니다. 저는 **Python/Django 프레임워크**와 **PostgreSQL 데이터베이스**를 기반으로, 높은 보안성과 확장성을 갖춘 아키텍처를 제안합니다.

아래 산출물은 개발팀이 즉시 코딩을 시작할 수 있도록 API 명세, DB 스키마(Schema), 그리고 핵심 로직의 골격(Skeleton Code)으로 구성되어 있습니다.

---

## 🛠️ I. 웹페이지 MVP 백엔드 설계 (Django/Python 기반)

### 1. 데이터베이스 스키마 정의 (PostgreSQL / Django Models)
가장 중요한 것은 서비스 제공에 필요한 모든 데이터를 체계적으로 관리하는 것입니다.

```python
# models.py

from django.db import models
from django.contrib.auth.models import User # 기본 사용자 인증 사용 가정

class GoalProfile(models.Model):
    """사용자 성적 및 목표를 저장하는 모델 (1:1 관계)"""
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    grade_level = models.CharField(max_length=10, help_text="예: 2학년 말, N수생 등")
    # 현재 성적 입력 필드 (가장 많이 사용하는 방식)
    current_score_input = models.JSONField(null=True, blank=True, help_text="{'과목': '점수', ...}")
    # 목표 설정 입력 필드 (직무/대학 중심)
    target_major = models.CharField(max_length=100, null=True, blank=True)
    target_university = models.CharField(max_length=100, null=True, blank=True)

class UniversityInfo(models.Model):
    """대학/학과 정보 (정적 데이터베이스)"""
    uni_name = models.CharField(max_length=50)
    major_name = models.CharField(max_length=50)
    admission_criteria = models.TextField(help_text="주요 전형 방식 및 요구 역량")
    required_score_range = models.JSONField(null=True, blank=True, help_text="과목별/영역별 점수 범위 예측 데이터")

class LearningPlan(models.Model):
    """생성된 맞춤형 주간 학습 계획 (가장 핵심적인 산출물)"""
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    plan_date = models.DateField() # plan이 유효한 시작 날짜
    week_focus = models.CharField(max_length=100, help_text="이번 주 핵심 학습 목표 (예: 논리적 사고력 강화)")
    weekly_schedule = models.JSONField(help_text="주간 세부 계획 ({'요일': {'시간': '내용'}})")
    is_paid_content = models.BooleanField(default=True, help_text="유료 콘텐츠 여부")

class PaymentAccess(models.Model):
    """결제 및 접근 권한 관리 모델 (진입 장벽 설정)"""
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    access_tier = models.CharField(max_length=20, help_text="FREE | BASIC (4990원) | PREMIUM (9990원)")
    is_active = models.BooleanField(default=False)
    subscription_ends_at = models.DateTimeField(null=True, blank=True)

```

### 2. 핵심 API 엔드포인트 설계 (RESTful API 명세)

| 기능 | HTTP Method | Endpoint URL | 요청 Body (Input) | 응답 Body (Output) | 설명 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **성적/목표 등록** | `POST` | `/api/profile/goal/` | `{ "type": "score", "data": {...} }` OR `{ "type": "target", "uni": "...", "major": "..." }` | `{ "status": "success", "message": "Profile saved" }` | 사용자 성적 또는 목표 대학 정보를 저장합니다. (필수) |
| **컨설팅 로직 실행** | `POST` | `/api/consulting/generate_plan/` | `{ "user_id": 1, "access_token": "...", "input_data": {...} }` | `{ "status": "success", "recommendations": [ {..}, ... ], "weekly_plan": { .. }, "plan_details": [...] }` | **핵심 로직.** 입력 데이터 기반으로 대학 추천 및 주간 계획을 생성합니다. (결제 확인 필수) |
| **계획 다운로드** | `GET` | `/api/content/download/<plan_id>/` | (Header: Authorization Bearer Token) | `{ "status": "success", "file_url": "/downloads/..." }` | 유효한 권한을 가진 사용자에게만 컨설팅 자료(PDF/ZIP 등)를 제공합니다. |
| **결제 처리** | `POST` | `/api/payment/process/` | `{ "user_id": 1, "plan_type": "BASIC"|"PREMIUM", "stripe_checkout_session_id": "..." }` | `{ "status": "success", "message": "Payment processed. Access granted." }` | Stripe 또는 PG사 연동 후 최종 권한을 부여합니다. |

### 3. 핵심 로직 골격 (Skeleton Code)
#### A. 컨설팅 계획 생성 함수 (`generate_consulting_plan`)

이 함수는 **가장 복잡하며, 데이터 매칭과 논리적 추론**을 담당합니다.

```python
# services/consulting_service.py

def generate_consulting_plan(user_profile: GoalProfile, access_level: str) -> dict:
    """
    사용자의 프로필 데이터를 기반으로 대학 추천 및 맞춤 학습 계획을 생성합니다.
    """
    if not is_paid_access(user_profile, access_level):
        raise PermissionError("유효한 유료 액세스 권한이 필요합니다.")

    # 1. 목표대학/성적 입력 타입 판별
    is_score_based = user_profile.current_score_input is not None
    
    if is_score_based:
        # [로직 A] 성적 기반 지원 가능 대학 매칭 (과거 데이터 활용)
        recommended_unis = match_scores_to_universities(user_profile.current_score_input)
    else:
        # [로직 B] 목표대학/성적 역산 학습 계획 수립
        recommended_unis = identify_gaps_from_target(user_profile.target_university, user_profile.target_major)

    # 2. 최적화된 주간 학습 계획 생성 (논리 추론 기반)
    weekly_plan = create_adaptive_plan(recommended_unis, access_level)
    
    # 3. 최종 결과 포맷팅 및 반환
    return {
        "recommendations": recommended_unis, # 대학 리스트와 매칭 근거 포함
        "weekly_plan": weekly_plan,         # 주간 학습 스케줄 JSON
        "status": "success"
    }

def match_scores_to_universities(score_data: dict) -> list:
    """DB의 UniversityInfo를 조회하여 점수 범위와 비교 분석하는 로직 (가장 많은 DB 쿼리 발생 지점)"""
    # ... (실제 데이터베이스 쿼리와 논리가 들어갑니다.)
    pass

def create_adaptive_plan(recommendations, access):
    """추천된 목표에 맞춰 주간별 학습 초점을 설정합니다."""
    if access == "PREMIUM":
        # 프리미엄: 단순히 과목을 나열하는 것이 아니라 '역량' 단위로 재구성
        return {"week1_focus": "논리적 구조화 훈련", "schedule": {...}, "premium_content_link": "/api/advanced/"}
    else:
        # 기본: 일반적인 주차별 학습 과목 배분
        return {"week1_focus": "기초 개념 다지기", "schedule": {...}}
```

#### B. 결제 권한 확인 함수 (`is_paid_access`)

```python
def is_paid_access(user_profile: GoalProfile, requested_tier: str) -> bool:
    """사용자의 현재 결제 상태를 체크하고 접근 가능 여부를 결정합니다."""
    try:
        payment_record = PaymentAccess.objects.get(user=user_profile.user)
        
        # 1. 유효기간 확인 (구독 모델)
        if payment_record.subscription_ends_at and payment_record.subscription_ends_at < timezone.now():
            return False # 만료됨

        # 2. 요청된 등급이 현재 권한보다 낮은지 체크
        allowed_tiers = ["FREE", "BASIC", "PREMIUM"]
        current_index = allowed_tiers.index(payment_record.access_tier)
        requested_index = allowed_tiers.index(requested_tier)

        return current_index >= requested_index
    except PaymentAccess.DoesNotExist:
        # 결제 이력이 없거나, 무료 계정인 경우 처리
        if requested_tier == "FREE":
            return True
        return False
```

---

## 💰 II. 결제 시스템 통합 준비 로직 (Stripe/PG 연동)

### 1. 인증 플로우 설계 (Authentication Flow)
| 단계 | 주체 | 액션 | 목적 | 보안 고려 사항 |
| :--- | :--- | :--- | :--- | :--- |
| **Step 0** | User | 웹사이트 접속 | 로그인/회원가입을 시도. | JWT 토큰 발행 (API 통신용). |
| **Step 1** | User | 컨설팅 실행 요청 | `/api/consulting/...` 호출 시, `access_token` 포함. | 백엔드에서 `is_paid_access()`를 통해 권한 검증. |
| **Step 2 (결제)** | User & Backend | 결제 페이지 이동 | Stripe Checkout 또는 국내 PG사 Redirect URL로 리다이렉트. | 카드 정보는 절대 서버에 저장하지 않음. 세션 기반 처리. |
| **Step 3** | PG/Stripe Webhook | 백엔드 수신 | 성공적인 거래 완료 알림 (Webhook). | 이 웹훅을 통해 `PaymentAccess` 모델의 상태(Active=True, 만료일 업데이트)를 즉시 변경해야 함. (가장 중요!) |

### 2. 구현 로직: Stripe Checkout 세션 생성 예시
결제 처리는 직접 구현하기보다 게이트웨이의 공식 SDK를 사용하는 것이 필수적입니다.

```python
# payment_service/stripe_integration.py
import stripe # pip install stripe

def create_checkout_session(user_id: int, tier: str) -> dict:
    """Stripe Checkout 세션을 생성하고 클라이언트에게 URL을 반환합니다."""
    try:
        checkout_session = stripe.checkout.Session.create(
            payment_method_types=['card'],
            line_items=[{
                'price': get_stripe_price_id(tier), # 예: basic-4990, premium-9990
                'quantity': 1,
            }],
            mode='subscription', # 구독 모델이므로 subscription 모드를 사용합니다.
            success_url="YOUR_WEBHOOK_URL?session_id={CHECKOUT_SESSION_ID}",
            cancel_url="http://skyin.com/payment-failed",
        )
        return {"checkout_url": checkout_session.url}
    except Exception as e:
        print(f"Stripe Session 생성 오류: {e}")
        return {"error": str(e)}

def get_stripe_price_id(tier: str) -> str:
    """요청된 등급에 맞는 Stripe Price ID를 가져오는 헬퍼 함수 (DB에서 관리 필요)"""
    if tier == "BASIC":
        return os.environ['STRIPE_PRICE_ID_BASIC'] # 환경변수 사용
    elif tier == "PREMIUM":
        return os.environ['STRIPE_PRICE_ID_PREMIUM']
    else:
        raise ValueError("유효하지 않은 결제 등급입니다.")
```

---

📊 평가: 완료 — API 명세, 데이터 구조, 핵심 로직의 골격까지 포함하여 즉시 개발 가능한 청사진을 제시했습니다.
📝 다음 단계: 백엔드 인프라 환경 구축 및 Mock 데이터를 이용한 단위 테스트(Unit Test) 실행. (개발팀에 요청)
