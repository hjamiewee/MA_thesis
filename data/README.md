# made_prepositions.csv

> **이 파일은 저장소에 포함하지 않습니다.** 원문 문장이 COCA에서 나온 것이라, 이용 조건을 고려해 CSV는 로컬에만 둡니다.
> 아래 "재생성" 방법으로 만든 뒤 `data/made_prepositions.csv`에 두면 노트북을 그대로 실행할 수 있습니다.
>
> *Not included in this repository.* The sentences come from COCA, so the CSV is kept local. Rebuild it as described under "재생성" (rebuild) and place it at `data/made_prepositions.csv`.

석사 논문 *"Distinguishing 'Made of/from/with' Based on the Semantic Relations of the
Arguments"* (Heejeong Wee, UNC Chapel Hill, 2024)의 **Appendix 2**를 파싱해 만든
분류용 데이터셋입니다. 원본 문장은 COCA (2016–2017) 코퍼스에서 추출되었습니다.

## 태스크
문장(또는 product·material 쌍)을 보고 알맞은 전치사 **of / from / with** 를 분류.

## 컬럼
| 컬럼 | 설명 |
|------|------|
| `id` | 행 번호 (1부터) |
| `label` | 정답 전치사 — `of` / `from` / `with` |
| `product` | 만들어지는 대상 (make의 표면 주어) |
| `material` | 재료 (전치사의 목적어) |
| `sentence` | COCA 원문 문장 |
| `sentence_masked` | `made of/from/with` → `made ___` 로 마스킹한 문장 (정답 누출 방지, 학습 입력용) |

## 분포
| label | rows |
|-------|------|
| of    | 511  |
| from  | 248  |
| with  | 180  |
| **합계** | **939** |

> 논문 본문 기준 수치는 of 511 / from 249 / with 180. `from`이 1행 적은 이유는
> 페이지 경계에서 분할된 한 행을 하나의 레코드로 올바르게 병합했기 때문이며,
> 빈 필드·중복·오병합은 없습니다.

## 주의
- 원문은 코퍼스 토큰화 형태라 구두점 앞에 공백이 있습니다(`wood ,`).
  `-LRB-`/`-RRB-` 등은 괄호로 복원했고, `@ @ @`는 원문에서 마스킹된 비식별 토큰입니다.
- 클래스 불균형(of 54% / from 26% / with 19%)이 있으니 학습 시 stratified split 또는
  class weight 적용을 권장합니다.

## 재생성
`Wee_unc_0153M_22785.pdf` 의 PDF p50–147 (논문 p42–139)을 `pdfplumber` 로 좌표 기반
파싱하여 생성. 세로 경계선 x=[72,125,210,321,540] 를 명시해 4개 열(category/product/
material/sentence)로 분리.
