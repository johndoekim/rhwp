# PR #8296 리뷰 — 한컴 날인 기호의 1em 합성

## 최종 판정

**머지 보류 — CanvasKit이 준비된 공개 대체 글꼴을 사용하지 않아 안쪽 ‘인’을 잃는 반례가 재현됐다.**
PR의 원 문제와 핵심 수정은 유효하다. 이 한정된 글꼴 선택 경로를 보정한 뒤 수용하는 방향을 권고한다.
GitHub 승인 리뷰·댓글·push·merge는 아직 수행하지 않았다.

## 접수와 최신 devel

- PR: [#8296](https://github.com/edwardkim/rhwp/pull/8296), 작성자 `johndoekim`, 기존 merge 기여 20건.
- reviewer `edwardkim`이 이미 지정돼 있음을 API로 재확인했다. maintainer 일반 경로와 렌더링·fixture,
  실제 backend 검증·대형 PR 보조 절차를 적용한다. 첫 기여자 절차는 비해당이다.
- 검토 head: `f3e2128edca7b188080b2a94f25e0988e26c9699`, fork `johndoekim/rhwp`, base `devel`.
- 최신 devel: `20c9678b01c5a564bb510c9fccd652cceafcd212`.
  최신 코드의 F012B → `(인)` 확장은 그대로다. 동일 제품 내용의 최적화 WASM을 실제 실행한 SVG에서도
  `인감(인)또는서명`을 확인했다. [관측값](assets/pr8296-maintainer-20261101/latest-devel-result.json).
- 최신 devel과 head의 merge-tree는 `d9dc04fdf031f6c68d9c3b7aa1015097fad22ed9`이며 충돌이 없다.
- head의 CI는 35 SUCCESS·6 조건부 SKIPPED다. [API 원문](assets/pr8296-maintainer-20261101/pr-checks.json).
  녹색 CI를 실제 glyph/face 선택의 근거로 대신하지 않았다.

## P1 — 준비된 공개 안쪽 글꼴을 CanvasKit에서 소비하지 않음

[원 head 코드 L3534–3574](https://github.com/johndoekim/rhwp/blob/f3e2128edca7b188080b2a94f25e0988e26c9699/rhwp-studio/src/view/canvaskit-renderer.ts#L3534)의
seal 합성 분기는 `geometry.face`인 Haansoft Batang을 찾은 뒤, 없으면 본문 regular face를 바로 고른다.
Rust의 `enclosed_pua_geometry::inner_faces`가 지정한 `RHWP Public Batang` 후보를 소비하지 않으며,
선택한 face가 일반 ‘인’을 실제로 가지고 있는지 확인하지 않는다.

이 때문에 본문 face에 한글이 없고 Public Batang은 이미 준비된 유효한 fallback 조건에서도
`Canvas.drawText('인')`에 glyph ID `0`이 전달됐다. 실제 PNG에는 타원 안에 .notdef 네모가 보인다.
동시에 진단은 `lastRenderCompleted=true`, unsupported 없음, software surface로 보고한다.
Public Batang을 직접 지정한 정상 대조군은 같은 문자에 glyph ID `10638`을 사용하고 정상 표시한다.

- [원 문서 textRun의 글꼴 커버리지 반례](assets/pr8296-maintainer-20261101/canvaskit-original-op-missing-inner.png)
- [단일 기호 반례](assets/pr8296-maintainer-20261101/canvaskit-inner-missing.png) /
  [Public Batang 정상 대조군](assets/pr8296-maintainer-20261101/canvaskit-public-face-control.png)
- [실제 drawText glyph ID·준비한 글꼴·진단·폰트 응답 해시](assets/pr8296-maintainer-20261101/canvaskit-probe-result.json)

별도 브라우저에서 실제 제품 `CanvasKitLayerRenderer`와 공급한 font bytes를 사용했다. 원 HWP에서
생산한 textRun을 추출해 본문 face의 Hangul coverage만 바꾼 통제 반례와 단일 기호 대조군이다.
실제 렌더링을 대역으로 교체하지 않았으며 drawText 호출을 관찰했다. 정상 문서의 전체 페이지
한컴 비교나 Studio 전체 화면을 이 통제 실험으로 대체했다고 주장하지 않는다.

공개 바탕 face 요청·적재만으로 glyph가 최종 paint에서 소비되지는 않는다. CanvasKit도 공통
안쪽 face 후보 정책을 소비하고 glyph coverage를 확인한 뒤 그려야 한다. 없을 때는 누락을
성공으로 보고하지 않아야 한다. 기존 renderer-contract 검사는 ellipse와 drawText 호출은 확인하지만
실제 ‘인’ glyph가 0인지와 준비된 Public face 선택을 확인하지 않아 이 반례를 놓친다.

## 수용 가치와 보존 확인

원본은 F012B 한 문자이며 tcps 글자겹침이 아니다. `(인)` 세 문자 확장을 1em 기호로 되돌리는
방향은 입력의 편집 의도와 맞는다. #937/PR #947의 기대값, #2020/#4384의 날인선과
#6127/#7642의 네모 숫자 경로를 함께 검토했다. 측정·paint가 같은 `EnclosedPua` 분류와 em
메트릭을 쓰도록 일반화한 방향도 타당하다. 다만 위 CanvasKit의 최종 face 선택에서는 공유가 끊긴다.

입력의 PUA → display/측정 → 공통 PaintOp → SVG/Native/Canvas 소비를 추적했다.
문서 ID·페이지 수 맞춤 분기나 표 pagination 변경은 없으며 표 이어받기 컷/예약 높이 검토는 비해당이다.
렌더링 기대값은 독립 한컴 PDF를 근거로 교정했고, 새 관계 검사로 핵심 문자·seal·내용 보존을 다룬다.

다음은 기존 검증 작업공간에서 보관된 실제 Native/fresh WASM 전쪽 TSV를 source/input 해시 확인 후
재사용한 수치다. **이번에 최신 devel 전체 페이지를 다시 렌더링한 결과로 표기하지 않는다.**

| 문서 | 쪽수 | 원 head Native 최저 raw | 원 head fresh WASM 최저 raw |
| --- | ---: | ---: | ---: |
| 위임장 | 2 | 99.94% | 99.91% |
| 복학원서 | 1 | 97.23% | 97.13% |
| issue4680 장부 | 1 | 99.12% | 99.10% |

[보관된 base/head TSV와 신원](assets/pr8296-maintainer-20261101/manifest.json)의 비교 base는 `da39c53661`이다.
최신 base `20c9678b01`의 전쪽 Native/WASM 재측정과 보정 후 핵심 glyph 확대 판독은 최종 수용 전에
마쳐야 한다. 이번 P1 발견을 `90% 이상` 수치로 면제하지 않는다.
[Native 대표 review](assets/pr8296-maintainer-20261101/native-seal-form-review.png)에서 실제 서식의
내용과 셀 경계를 직접 확인했고, CanvasKit 반례 PNG도 직접 판독했다.

재현 입력과 독립 Print PDF는 원 head commit에 포함된
[위임장 자료](https://github.com/johndoekim/rhwp/tree/f3e2128edca7b188080b2a94f25e0988e26c9699/samples/issue8201),
기존 `samples/복학원서.hwp`·`pdf/복학원서-2022.pdf`, issue4680 원본·2020 PDF다.
실제 사용한 파일과 commit의 동일성은 입력 manifest에 기록했다. 통제한 Font coverage 반례는
독립 Print 측정의 추가 fixture/golden으로 등록하지 않았다.

## 완료한 검사와 다음 절차

실제 브라우저 재현과 최신 devel SVG 확인을 완료했다. 원 head의 광범위 GitHub CI는 재사용하고
로컬에서는 날인·네모 숫자·PUA 정규화·기존 저장 보존의 집중 검사 28건을 실제 재실행해
모두 통과했다. 개인·시스템 글꼴을 가린 Native Skia 경로이며 같은 suite의 비선택 1,710건은
통과 수에 포함하지 않았다. [증거 manifest](assets/pr8296-maintainer-20261101/manifest.json)에
실행 head·격리 조건·로그 해시를 기록했다. 문서 링크·스크립트 구문·공백 검사도 통과했다.

메인테이너의 제한된 보정안은 CanvasKit 안쪽 face 후보·coverage 선택, 실제 glyph 기반 회귀,
해당 backend 및 최신 base 전쪽 비교다. 다른 조판 리팩터링으로 범위를 넓힐 필요는 없다.
원격 보정 제출·댓글·승인·머지는 작업지시자의 후속 범위 결정 뒤 수행한다.
