# faith-voice

[신앙일지(Faith Log)](https://csy870617.github.io/diary/)의 **자연스러운 음성 읽기**에 쓰는 음성 모델 사본입니다.
앱의 음성 설정에서 "내려받기"를 누르면 이 저장소의 파일을 한 번 받아 기기에 보관하고,
그다음부터는 인터넷 없이 기기 안에서 음성을 만듭니다.

## 들어 있는 것

| 경로 | 내용 | 출처 · 라이선스 |
|---|---|---|
| `onnx/`, `voice_styles/` | Supertonic 3 음성 모델과 목소리 10종(여성 F1–F5, 남성 M1–M5) | [supertone-oss-archive/supertonic-3](https://huggingface.co/supertone-oss-archive/supertonic-3) (revision `aafc6e32416a594460b32413efc49d7fe4ce6d46`) · **BigScience Open RAIL-M** (`LICENSE`) |
| `onnx/vector_estimator_int8.onnx` | 위 모델 중 가장 큰 `vector_estimator`를 8비트로 동적 양자화한 것 (CPU로 계산하는 기기용) | 원본에서 만든 파생물 · **BigScience Open RAIL-M** (`LICENSE`) |
| `ort/` | 브라우저 실행 엔진 onnxruntime-web 1.23.2 | Microsoft · **MIT** (`ort/LICENSE-onnxruntime-web.txt`) |
| `manifest.json` | 앱이 받을 파일 목록 · 크기 · 조각 정보 | — |

## 원본과 다른 점

원본 모델 파일의 내용은 바꾸지 않았습니다.

**추가한 파생 모델 — `vector_estimator_int8.onnx`:** GPU 없이 CPU로 계산하는 기기를 위해
`vector_estimator.onnx`를 8비트로 바꾼 것입니다(257MB → 66MB). 앱은 GPU 기기에는 원본을,
CPU 기기에는 8비트를 받습니다(`manifest.json`의 `only` 표시). 보코더는 양자화하면 음질이 무너져 원본을 씁니다.

지금 판(`int8v2`)은 onnxruntime 1.23.2의 **동적 양자화**로 만들었습니다. 계산할 때마다 값의 범위를 새로 재므로,
단계마다 값의 크기가 크게 달라지는 이 모델에서는 범위를 미리 정해 두는 정적 양자화(예전 판)보다 원본에 훨씬 가깝습니다.

```python
quantize_dynamic('vector_estimator.onnx', 'vector_estimator_int8.onnx',
                 weight_type=QuantType.QUInt8, per_channel=True,
                 op_types_to_quantize=['Conv', 'MatMul'],
                 nodes_to_exclude=[...],          # depthwise Conv 28개(group > 1)는 원본 그대로
                 extra_options={'MatMulConstBOnly': True})
```

가중치를 uint8로 둔 것은 브라우저 엔진의 `ConvInteger`가 uint8×uint8만 지원하기 때문입니다.
SHA-256 `9c6408bdf36ef1fa534a93baac7f828c0abb5b1b87b792153497603fcf0f5763`

| 같은 문장·목소리·씨앗으로 잰 값 | 원본 | 예전 판 (정적 양자화) | 지금 판 (동적 양자화) |
|---|---|---|---|
| 자연스러움 점수 UTMOS — 품질 8단계 | 3.92 | 2.90 | **3.86** |
| 〃 6 / 5 / 4단계 | 3.85 / 3.53 / 3.22 | 2.63 / 2.37 / 2.16 | **3.73 / 3.42 / 3.11** |
| 원본 8단계와의 스펙트럼 거리 — 8단계 | 0 dB | 7.4 dB | **3.9 dB** |
| 받아쓰기(Whisper) 글자 오류율 — 5단계, 문장 5개 × 목소리 4개 | – | 0.68% | **0.00%** |
| 브라우저(WASM, 스레드 1개) 단계당 시간, 약 6초 분량 | 1718 ms | 919 ms | **785 ms** |
| 〃 모델을 연 뒤 메모리 | 861 MB | 336 MB | **285 MB** |

UTMOS(UTMOS22 strong)는 영어로 학습한 점수라 절대값보다 같은 문장끼리의 비교로 봐야 합니다.

나머지는 GitHub의 파일 크기 제한(100MB) 때문에
`vector_estimator.onnx`와 `vocoder.onnx`를 40MB 조각(`*.partN`)으로 **나누기만** 했고,
앱이 받을 때 순서대로 이어 붙여 원본과 같은 파일로 되돌립니다.

## 사용 조건

음성 모델은 Open RAIL-M 라이선스를 따릅니다. 라이선스 부록 A(Attachment A)의 **사용 제한**
(불법 행위, 타인을 해치는 용도, 허위 정보·사칭, 개인정보 침해 등에 쓰지 않을 것)이 이 사본과
이 사본으로 만든 음성에도 그대로 적용됩니다. 전문은 `LICENSE`를 보세요.

Supertonic은 [Supertone Inc.](https://github.com/supertone-inc/supertonic)가 공개한 모델입니다.
