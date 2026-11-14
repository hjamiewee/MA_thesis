# "made of / from / with" 전치사 분류 — DistilBERT 파인튜닝

[English](README.md) | **한국어**

영어 구문 *"X is made **of / from / with** Y"* 에서, 문장만 보고 알맞은 전치사를
예측하는 **텍스트 분류 모델**입니다. **PyTorch + HuggingFace Transformers**로
DistilBERT를 파인튜닝했습니다.

> 이 프로젝트는 저자의 석사 논문 *"Distinguishing 'Made of/from/with' Based on the
> Semantic Relations of the Arguments"* (Heejeong Wee, UNC Chapel Hill, 2024)에서
> 출발합니다. 논문에서는 **GPT-4o(무학습)** 가 이 과제에서 전체 정확도 **39%**,
> 특히 `from`은 **0%** 에 그쳤습니다. 본 프로젝트는 *"같은 과제를 데이터로 학습하면
> 얼마나 개선되는가?"* 를 직접 검증합니다.

---

## 결과 요약 (테스트셋, 141문장)

| 모델 | 정확도 | **Macro-F1** |
|------|:------:|:------------:|
| ① 다수 클래스 (항상 `of`) | 0.546 | 0.235 |
| ② TF-IDF + 로지스틱 회귀 | 0.454 | 0.379 |
| **③ DistilBERT (파인튜닝)** | **0.610** | **0.575** |

DistilBERT가 **정확도·Macro-F1 두 지표 모두**에서 베이스라인을 앞섰습니다.

**DistilBERT 클래스별 성적 (테스트셋)**

| 전치사 | precision | recall | F1 |
|--------|:---:|:---:|:---:|
| of | 0.785 | 0.662 | 0.718 |
| from | 0.444 | 0.432 | **0.438** |
| with | 0.475 | 0.704 | 0.567 |

- 논문에서 `from`이 **0%** 였던 것과 대비해, 학습 모델은 from F1 **0.438** 을 달성.
- 상세 결과: [`results/`](./results) (점수표·혼동행렬·클래스별 리포트)

> 참고: 논문의 39%는 데이터 분할·입력 방식이 달라 1:1 직접 비교가 아니라, 학습의
> 효과를 보여주는 동기 부여용 수치입니다.

---

## 과제 정의

- **입력:** 정답 전치사를 가린 문장 (`made of/from/with` → `made ___`)
- **출력:** `of` / `from` / `with` 중 하나 (3-클래스 분류)
- **정답 누출 방지:** 입력에서 전치사를 마스킹해 모델이 정답을 베끼지 못하게 함.

## 데이터

- 출처: 위 석사 논문 **Appendix 2** (원문은 COCA 2016–2017 코퍼스에서 추출).
- 규모: **939 문장** — of 511 / from 248 / with 180 (클래스 불균형 존재).
- PDF 부록을 `pdfplumber`로 좌표 기반 파싱해 생성. 상세: [`data/README.md`](./data/README.md)
- 분할: 학습 657 / 검증 141 / 테스트 141 (계층적 분할로 클래스 비율 유지).
- **데이터 파일은 저장소에 포함하지 않습니다.** COCA 이용 조건을 고려해 CSV는 로컬에만 둡니다. 만드는 방법과 형식은 [`data/README.md`](./data/README.md)에 있습니다.

## 방법 (파이프라인)

1. **데이터 분할** — train/val/test, stratified split → `notebooks/02_data_split.ipynb`
2. **베이스라인** — 다수 클래스 + TF-IDF·로지스틱 회귀 → `notebooks/03_baseline.ipynb`
3. **파인튜닝** — DistilBERT + 클래스 가중치 손실(불균형 보정), 5 epochs, MPS(Apple M4) 가속
   → `notebooks/04_finetune_distilbert.ipynb`
4. **최종 평가** — 테스트셋으로 세 모델 비교 → `notebooks/05_evaluate_test.ipynb`

평가지표는 **정확도**와 **Macro-F1**(소수 클래스를 평등하게 반영하는 불균형 핵심 지표)을 함께 사용.

---

## 재현 방법

```bash
# 1) conda 환경 생성 및 활성화
conda env create -f environment.yml
conda activate default

# 2) data/made_prepositions.csv 준비 (data/README.md 참고)

# 3) 노트북을 순서대로 실행 (Jupyter 또는 VS Code)
#    02 → 03 → 04 → 05
jupyter lab
```

> Apple Silicon(M칩)에서는 PyTorch가 자동으로 MPS 가속을 사용합니다.
> 학습된 모델은 용량 문제로 깃에 포함하지 않으며(`outputs/`는 제외), 04번 노트북을
> 실행하면 재생성됩니다.

## 저장소 구조

```
data/
  README.md                  # 데이터 카드 (출처·컬럼·분포·만드는 방법)
  made_prepositions.csv      # 전체 데이터셋 939행 (로컬 전용 · git 제외)
  splits/                    # train/val/test 분할 (02번 노트북이 생성 · git 제외)
notebooks/
  02_data_split.ipynb        # 데이터 분할
  03_baseline.ipynb          # 베이스라인
  04_finetune_distilbert.ipynb  # DistilBERT 파인튜닝 (핵심)
  05_evaluate_test.ipynb     # 테스트셋 최종 평가
results/                     # 최종 점수표·혼동행렬·리포트
environment.yml              # conda 재현 환경
```

모든 노트북은 한 줄/블록 단위 한글 주석으로 설명되어 있어, 코드를 처음부터 끝까지
따라가며 이해할 수 있습니다.

## 한계 및 향후 과제

- 데이터가 939건으로 작아 과적합 위험이 있고, 점수의 추가 향상 여지가 있음.
- 논문 결론대로 전치사 선택이 **결정론적이지 않을 수 있어**(화용·강조에 따라 복수 정답
  가능) 정확도 상한이 존재.
- 향후: 더 큰 백본(BERT-large)·LoRA 기반 LLM 파인튜닝, 입력에 product/material를 함께
  제공하는 방식 비교, 교차검증을 통한 안정성 평가.
