# 평가 틀 정의 형식 (v0.1)

평가 대상이 바뀌면 이 형식에 맞춘 평가 틀 파일만 새로 작성한다. 엔진은 그대로 쓴다.

## 1. 최상위 구조

| 키 | 내용 |
|---|---|
| `meta` | 평가 틀 이름, 버전, 평가 목적, 평가 대상 유형, 기준 시점, 작성자 |
| `scale` | 점수 범위(기본 1~5)와 점수 의미 |
| `axes` | 평가 범주 목록 (id, 이름, 설명) |
| `inputs` | 수집할 자료 항목 목록 (지표들이 공유) |
| `derived` | 입력값으로 계산하는 파생값 목록 |
| `indicators` | 지표 목록 |
| `external_factors` | 점수 대상에서 제외하고 서술로만 다루는 외부 여건 |
| `references` | 단계 기준의 근거 문서 목록 |

## 2. 자료 항목 (`inputs`)

| 키 | 내용 |
|---|---|
| `id` | 영문 식별자 |
| `label` | 화면에 표시할 이름 |
| `type` | `number`, `integer`, `choice`, `text` |
| `unit` | 단위 (%, 억 원, 건 등) |
| `source_hint` | 어디서 찾는지 (사람과 AI 추출 모두 참고) |
| `extraction_prompt` | AI 추출 단계에서 쓸 지침 |
| `options` | `choice`일 때 선택지 |

모든 입력값은 값과 함께 `evidence`(근거 문장, 출처, 기준일)를 기록한다. 값이 없으면 `null`로 두고 해당 지표는 '확인 불가'로 처리한다.

## 3. 파생값 (`derived`)

| 연산 | 의미 |
|---|---|
| `percent` | args[0] / args[1] × 100 |
| `divide` | args[0] / args[1] |
| `subtract` | args[0] − args[1] |

## 4. 채점 규칙 (`indicators[].rule`)

채점은 언어모델의 판단 없이 아래 규칙으로만 한다. 같은 입력이면 항상 같은 점수가 나온다.

### `range` : 수치를 구간에 대입
```json
{ "type": "range", "field": "max_segment_share",
  "bands": [ {"min": 70, "score": 1}, {"min": 55, "max": 70, "score": 2}, ... ] }
```
`min`은 이상, `max`는 미만. 위에서부터 처음 맞는 구간의 점수를 쓴다.

### `ordinal` : 정의된 단계 중 사실에 해당하는 단계 선택
```json
{ "type": "ordinal", "field": "judgment_disclosure_level" }
```
선택지는 `inputs`의 `options`에 점수와 함께 정의한다. 선택지 문구는 해석 여지가 없도록 확인 가능한 사실로 쓴다.

### `conditions` : 여러 값을 함께 따지는 경우
```json
{ "type": "conditions",
  "cases": [ {"when": {"all": [{"field": "loss_years", "op": ">=", "value": 2}]}, "score": 1}, ... ],
  "default": 3 }
```
위에서부터 처음 맞는 조건의 점수를 쓴다. `op`는 `>=`, `>`, `<=`, `<`, `==`. `all`(모두)·`any`(하나 이상) 사용.

## 5. 지표 (`indicators`)

| 키 | 내용 |
|---|---|
| `id`, `name`, `axis` | 식별자, 이름, 소속 범주 |
| `question` | 이 지표가 답하려는 질문 |
| `anchor_type` | `external`(외부 기준) 또는 `factual`(사실 기준, 구간은 분석자가 설정) |
| `anchor_ref` | 근거 문서 id (`references`) |
| `levels` | 1~5점 각 단계의 설명 (보고서·화면 표시용) |
| `rule` | 채점 규칙 |
| `controllable` | `yes`, `partial`, `no` — 방안으로 점수를 올릴 수 있는지 |
| `importance` | `high`, `medium`, `low` — 다음 성장 단계에 걸리는 정도. 과제 도출에 사용 |
| `target` | 목표 점수와 목표 시점 (방안 설계 단계에서 입력) |

## 6. 엔진 출력

- 지표별: 점수, 적용된 규칙 구간, 입력값과 근거, 확인 불가 여부
- 범주별: 점수 분포 그림 (총점은 내지 않는다)
- 과제 후보: `importance`가 high이고 점수가 2 이하인 지표, 그중 `controllable`이 yes·partial인 것
- 현재 점수와 목표 점수 비교 그림

## 7. 사람의 확정

엔진이 낸 점수는 초안이다. 평가자가 근거를 확인하고 확정하며, 확정 시 수정 사유를 기록한다.
