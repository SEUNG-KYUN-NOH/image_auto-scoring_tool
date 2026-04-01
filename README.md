# image_auto-scoring_tool
4인 평가 루브릭을 바탕으로 인지성·이미지품질·클릭유도력을 구조화하고, 평균뿐 아니라 합의도·불일치·경계사례까지 반영해 이미지 레이블링을 자동화하는 데이터셋 및 평가 파이프라인 프로젝트.
A rubric-based image labeling automation project that structures recognizability, image quality, and click appeal while capturing averages, agreement, disagreement, and edge cases in a practical evaluation pipeline.

* test_page_url: https://seung-kyun-noh.github.io/image_auto-scoring_tool/


# 🤖 image_auto-scoring_tool

> 저화질 이미지 레이블링 평가자동화 도구  
> Rubric-based image quality auto-scoring pipeline using Gemini API

[![Version](https://img.shields.io/badge/version-v1.0.0-blue)](#) [![Python](https://img.shields.io/badge/python-3.10+-blue)](#) [![Colab](https://img.shields.io/badge/platform-Google%20Colab-orange)](#) [![License](https://img.shields.io/badge/license-MIT-green)](#)

---

## 📌 한 줄 요약

쇼핑몰 상품 이미지를 AI(Gemini)에게 보여주면, **인지성 · 이미지품질 · 클릭유도성** 3항목을 자동으로 채점(0~2점)해 CSV로 저장하는 파이프라인입니다.

---

## 🗂 프로젝트 구조

```
image_auto-scoring_tool/
├── index.html                          # 웹 프로토타입 (시연용)
├── _ver1_0_0__Auto_scoring_for_gemini_api.ipynb   # Colab 실행 파일 (v1.0.0)
└── README.md
```

---

## ⚙️ 파이프라인 구조

```
CSV 입력 (product_id, product_name, image_url)
    │
    ▼
[이미지 다운로드] ── 비동기(aiohttp) 병렬 처리
    │
    ▼
[STEP 1] Gatekeeper (Gemini API 1차 호출)
    ├─ is_fatal = true  → 전 항목 0점, STEP 2 스킵 (API 비용 절약)
    └─ is_fatal = false → STEP 2 진행
    │
    ▼
[STEP 2] Scorer (Gemini API 2차 호출)
    └─ 인지성 / 이미지품질 / 클릭유도성  각 0~2점 채점
    │
    ▼
결과 DataFrame → CSV 저장
```

---

## 📊 채점 기준 (Rubric)

| 항목 | 2점 (Good) | 1점 (Fair) | 0점 (Bad) |
|------|-----------|-----------|----------|
| **인지성** | 한눈에 인식, 수량 명확 | 배경 복잡, 인식 지연 | 상품 불일치, 외국어만 |
| **이미지품질** | 스튜디오급, 고해상도 | 살짝 흐림, 어두운 배경 | 스마트폰 날사진, 극심한 노이즈 |
| **클릭유도성** | 전문 레이아웃, 시선 집중 | 평범한 구성, 기본 정보 | 촌스러운 색상, 일러스트만 |

### Gatekeeper 즉시 거부 조건

1. 이미지 비율 왜곡 (찌그러짐/늘어짐)
2. 과도하게 흐리거나 인식 불가
3. 극심한 노이즈 / 저해상도
4. 외국어 텍스트 과다
5. 도매용 포장 이미지 (20kg 포대, 박스째 촬영 등)

---

## 🚀 실행 방법 (Colab)

```python
# 1. 라이브러리 설치
!pip install google-genai

# 2. API 키 설정
import os
os.environ["GEMINI_API_KEY"] = "YOUR_API_KEY"

# 3. 평가 데이터 입력
data = {
    'product_id': ['688464776'],
    'product_name': ['이츠웰 쇠고기다시 1kg'],
    'image_url': ['https://...jpg']
}

# 4. 평가 실행
results_dict = await process_batch(df)
```

결과 파일: `톡딜_이미지평가_최종결과.csv`

---

## 🗺 버전 로드맵

| 버전 | 상태 | 핵심 목표 |
|------|------|---------|
| **v1.0.0** | ✅ 완료 | Colab 단건 채점 검증, 프롬프트 체이닝 구조 확립 |
| **v1.1** | 🔄 개발 중 | CSV 일괄 처리, TrainSet 캐시 도입 |
| **v1.2** | 📋 계획 | 이미지 센서(로컬 사전 필터링) 추가 |
| **v2.0** | 📋 계획 | API 다중화, 결과 대시보드 |

---

## 💡 설계 포인트

**프롬프트 체이닝 (Prompt Chaining)**
- 하나의 복잡한 프롬프트 대신 Gatekeeper → Scorer 두 단계로 분리
- 결함 이미지는 1차에서 차단 → API 호출 횟수 최소화

**비동기 병렬 처리 (asyncio + aiohttp)**
- 이미지 다운로드와 API 호출을 동시에 처리
- 순차 처리 대비 처리 속도 대폭 향상

**API 비용 최적화 전략**
```
이미지 센서 (로컬) → TrainSet 캐시 → Gatekeeper → Scorer
     API 호출 없음      API 호출 없음    저비용 판단   정밀 채점
```

---

## 🛠 기술 스택

| 기술 | 역할 |
|------|------|
| Google Gemini 2.5 Flash | AI 이미지 분석 엔진 |
| Python asyncio | 비동기 병렬 처리 |
| aiohttp | 이미지 URL 비동기 다운로드 |
| pandas DataFrame | 데이터 관리 및 CSV 저장 |
| Google Colab | 실행 환경 |

---

## 🌐 웹 프로토타입

`index.html` 파일을 브라우저에서 열면 시연용 UI를 확인할 수 있습니다.

- CSV 업로드 / 직접 입력
- 평가 진행 실시간 로그
- 결과 필터링 및 검색
- 패턴 분석 대시보드
- 설정 (모델 / 센서 / 캐시)

---

## 👤 Author

**SEUNG-KYUN-NOH** · adrian.101  
작성일: 2026.03.31
