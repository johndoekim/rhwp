# PR #8296 메인테이너 검토 증거

[검토 기록](../../archives/pr_8296_review.md)의 CanvasKit font coverage 반례와 보관된 시각 근거다.
브라우저 재현은 원 head `f3e2128edc`의 TypeScript와 같은 제품 내용의 최적화 WASM을 사용했다.
원 문서의 PaintOp를 사용한 통제 face 실험이며 새로운 한컴 원본/Print fixture라고 주장하지 않는다.
원래 작업공간의 기존 검증 자료는 source와 입력 해시를 대조한 뒤 참고 자료로 재사용한다.
현재 devel의 전쪽 재출력과 최종 무회귀 검증을 완료했다고 표기하지 않는다.

진단 스크립트는 [canvaskit-probe.mjs](canvaskit-probe.mjs)다. 원 head의 checkout·검증된 web WASM·
기존 npm 의존성을 준비하고 Studio Vite를 별도 포트에서 실행한 뒤 다음과 같이 호출한다.
스크립트가 요구하는 `samples/issue8201/seal-delegation-form.hwp`는 원 head에 포함돼 있다.

```bash
RHWP_REVIEW_ROOT=/path/to/review-checkout \
RHWP_REVIEW_OUTPUT=/path/to/review-checkout/output/pr8296-canvaskit \
VITE_URL=http://127.0.0.1:7703 \
CHROME_PATH=/path/to/chrome \
node mydocs/pr/assets/pr8296-maintainer-20261101/canvaskit-probe.mjs --mode=headless
```

`RHWP_WASM_BUILD_MANIFEST`도 실제 준비한 최적화 WASM의 build/consumption manifest로 지정한다.
결과의 `innerDraws` 두 번째와 여섯 번째 호출은 glyph ID 0이고, 마지막 Public Batang 대조군은
10638이다. glyph ID의 절대 숫자는 이 공급 font bytes의 관측이며 제품 golden으로 고정하지 않는다.

## 2026-11-01 수용 보정

`acceptance-validation.json`과 `code-ci-final.json`은 보정 `3ddf77af61`의 로컬·원격 검증입니다.
base/head TSV는 최신 devel `20c9678b01`을 실제 재출력한 값으로 갱신했습니다. 수치는 이전 base와 같지만
새 Native 빌드·별도 base WASM 경로로 실행한 원본입니다. Rust 제품 소스와 Canvas2D 경로는
원 기여 head와 같아서 기존 최적화 bundle을 재사용했고 CanvasKit TypeScript 보정은 실제 브라우저에서 실행했습니다.

`canvaskit-probe-after.json`은 원래 반례와 같은 공급 조건에서 안쪽 글리프가 유효해진 결과입니다.
캡처 당시 미커밋 보정의 renderer SHA와 최종 코드의 동일성을 확인했습니다. 이후 변경은 없었습니다.
`canvaskit-full-page-after.json`과 세 전체 PNG는 실제 CanvasKit software의 portable print 재생입니다.
명시적인 공개 대체 글꼴을 공급했으며, 한컴 글꼴 자체의 피델리티 측정으로 주장하지 않습니다.
처음에는 이 공개 subset에 U+2024가 없어 명시적으로 DejaVu Sans를 공급한 뒤 재실행했습니다.
그 전의 글꼴 누락 실행은 통과로 세지 않습니다. 기존 네모 숫자는 Native 전체 PNG와
CanvasKit의 다섯 font/ratio PNG가 baseline과 바이트 단위로 같습니다.
