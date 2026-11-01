# PR #8296 메인테이너 검토 증거

[검토 기록](../../pr_8296_review.md)의 CanvasKit font coverage 반례와 보관된 시각 근거다.
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
