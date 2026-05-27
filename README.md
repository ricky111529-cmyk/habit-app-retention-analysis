# Habit-Forming App Retention Analysis

> 大学のマーケティング調査授業のチームプロジェクト (한양대학교 마케팅조사 수업 팀 프로젝트)
> 4人チームのリーダーとして担当 (4인 팀 리더 담당)
> 期間 (기간): 2025.03 – 2025.05

---

## 概要 / 개요

연말-연시 (2024.12.01 – 2025.01.31) 62일간의 모바일 앱 사용 로그 데이터를 활용하여 **「작심삼일」 패턴**과 사용자 이탈 요인을 정량적으로 분석한 프로젝트.

年末年始62日間のモバイルアプリ使用ログデータを用いて、ユーザーの「三日坊主」パターンと離脱要因を定量的に分析した。

---

## 仮説 / 가설

| # | 가설 / 仮説 | 분석 기법 / 手法 | 결과 / 結果 |
|---|---|---|---|
| **H1** | 12月~1月 습관형성 앱 사용시간은 U字型 곡선을 그릴 것 | 다중 선형회귀 (OLS) + 비선형 제곱항 | ✅ **支持** (p < 0.01) |
| **H2** | Education앱 이탈률이 Health·Productivity보다 높을 것 | 로지스틱 회귀 (이진 dropout) | ❌ 기각 |
| **H3** | SNS 사용 시간이 많을수록 습관형성 앱 이탈 확률이 낮을 것 | 로지스틱 회귀 + 상호작용항 (age × sns_time) | ✅ **支持** (p < 2e-16) |

---

## データ / 데이터

- **規模 / 규모**: user × app × date 단위 로그 (raw 3,436건 → 정제 후 **4,960건**)
- **期間 / 기간**: 2024-12-01 ~ 2025-01-31 (62일)
- **対象アプリ / 대상 앱**: 22개 (Productivity 1 / Health 11 / Education 5 / SNS 3 / Daily 2)
- **変数 / 변수**: `user_id`, `date`, `app_name`, `age`, `total_time`

> ⚠️ 원본 데이터(`MR_data.csv`)는 수업 자료로 본 레포에 포함되지 않음 / 原データは授業教材のため非公開

---

## 手法 / 방법

### 데이터 전처리 / 前処理
- **Missing Data**: 1월 25일 이후 미기록 + 12월 1일 미시작 유저 10명 제외
- **Outlier**: IQR 1.5배 기준 (상위 11.61% 제거)
- **0処理**: user × app × date 풀 매트릭스 생성 후 미사용일 `total_time = 0` 보강

### 파생 변수 / 派生変数
- `custom_week`: 12/1 기점 주차 (1~9주)
- `is_jaksim`: 습관형성 앱 여부 (Productivity/Health/Education = 1)
- `used_sns_apps`: SNS 1회 이상 사용 여부
- `is_weekend`: 더미 변수
- `age_x_sns`: 연령 × SNS 사용시간 **상호작용항**
- `health_app_ratio`: 유저-날짜별 Health 카테고리 비율

### 「작심삼일」 정의 함수 / 「三日坊主」定義関数

비즈니스 정의를 코드로 구현 (`check_jaksim`, `get_dropout_flag`):
1. **조건 1**: 7일 윈도우 내 3일 이상 사용한 구간 존재
2. **조건 2**: 마지막 사용일 이후 관찰 종료일까지 재사용 없음

→ `zoo::rollapply` 활용한 시계열 윈도우 분석

### 모델 / モデル

| 가설 | 모델 | 종속변수 | 주요 독립변수 | 통제변수 |
|---|---|---|---|---|
| H1 | `lm()` OLS + 제곱항 | total_time | date_num, date_num² | num_apps, health_app_ratio, age, is_weekend |
| H2 | `glm()` Logistic | dropout (0/1) | category (더미) | avg_daily_usage, age, user_count, avg_usage_days_app |
| H3 | `glm()` Logistic + 상호작용 | is_dropout (0/1) | sns_time, age_x_sns | avg_session_time, app_count, usage_duration |

---

## 結果 / 결과

### H1 — U자형 사용 패턴 (지지)
- date_num² 계수: **0.04** (p < 0.01)
- 연말로 갈수록 감소 → 연초 새해 결심으로 재증가
- `health_app_ratio` 계수 113.2 (p < 0.0001): 건강 앱 사용자가 사용 시간 압도적으로 길음

### H3 — SNS 사용과 이탈 (지지)
- `sns_time` 계수: **음수** (p < 2e-16)
- 상호작용항 `age_x_sns` 양수: 연령이 높을수록 SNS의 이탈 완화 효과 감소

---

## 마케팅적 시사점 / マーケティング示唆

1. **연말 이탈 방지 캠페인** + **연초 재유입 캠페인** = 시즌별 마케팅
2. **SNS 연동 기능** 실험적 도입 — 사회적 자극이 리텐션에 긍정 효과
3. Education 앱은 단기 목표형 → "3일 챌린지", "7일 완성" 등 짧은 목표 설계 필요

---

## 사용 도구 / Tools

- **R** (`dplyr`, `ggplot2`, `lubridate`, `tidyr`, `zoo`)
- 통계 비전공 (경영학부) — R·회귀분석 독학으로 진행

---

## 파일 구성 / ファイル構成

```
.
├── analysis.R       # 전체 분석 코드 (전처리·시각화·3개 가설 회귀)
└── README.md        # 본 문서
```

---

## 팀 / チーム

| 역할 | 이름 |
|---|---|
| **Lead (本人)** | 김민섭 / KIM MINSUP (경영학부) |
| Member | 정희연 (경영학부) |
| Member | 박소은 (관광학부) |
| Member | 김한비 (행정학과) |
