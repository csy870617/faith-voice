# faith-voice

[신앙일지(Faith Log)](https://csy870617.github.io/diary/)의 **자연스러운 음성 읽기**에 쓰는 음성 모델 사본입니다.
앱의 음성 설정에서 "내려받기"를 누르면 이 저장소의 파일을 한 번 받아 기기에 보관하고,
그다음부터는 인터넷 없이 기기 안에서 음성을 만듭니다.

## 들어 있는 것

| 경로 | 내용 | 출처 · 라이선스 |
|---|---|---|
| `onnx/`, `voice_styles/` | Supertonic 3 음성 모델과 목소리 10종(여성 F1–F5, 남성 M1–M5) | [supertone-oss-archive/supertonic-3](https://huggingface.co/supertone-oss-archive/supertonic-3) (revision `aafc6e32416a594460b32413efc49d7fe4ce6d46`) · **BigScience Open RAIL-M** (`LICENSE`) |
| `ort/` | 브라우저 실행 엔진 onnxruntime-web 1.23.2 | Microsoft · **MIT** (`ort/LICENSE-onnxruntime-web.txt`) |
| `manifest.json` | 앱이 받을 파일 목록 · 크기 · 조각 정보 | — |

## 원본과 다른 점

모델 내용은 바꾸지 않았습니다. GitHub의 파일 크기 제한(100MB) 때문에
`vector_estimator.onnx`와 `vocoder.onnx`를 40MB 조각(`*.partN`)으로 **나누기만** 했고,
앱이 받을 때 순서대로 이어 붙여 원본과 같은 파일로 되돌립니다.

## 사용 조건

음성 모델은 Open RAIL-M 라이선스를 따릅니다. 라이선스 부록 A(Attachment A)의 **사용 제한**
(불법 행위, 타인을 해치는 용도, 허위 정보·사칭, 개인정보 침해 등에 쓰지 않을 것)이 이 사본과
이 사본으로 만든 음성에도 그대로 적용됩니다. 전문은 `LICENSE`를 보세요.

Supertonic은 [Supertone Inc.](https://github.com/supertone-inc/supertonic)가 공개한 모델입니다.
