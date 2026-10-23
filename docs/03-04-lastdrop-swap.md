# 03-04 — 라스트 드롭을 /game/br/ 로 올리고, 시작화면 버그를 잡았다

날짜: 2026-10-23
지시: "https://www.valuetopic.com/game/br/ 여기에 올려야지" → "낙화 느르면 그 화면이 안 사라줘"

## 1. 무엇을 바꿨나

`/game/br/` 에 있던 01·02 단계 빌드(한 파일 59,614 B)를 **라스트 드롭 완성 빌드**로 교체했다.
라스트 드롭은 03(배틀로얄 한 판 전체 흐름) + 04(배그식 요소: 수송기 낙하·낙하산, 자세,
헬멧·방탄복, 회복 4종, 레드존, 보급 상자, 지프, 1km 맵 16명) 를 담은 빌드다.

| | 전 | 후 |
|---|---|---|
| 파일 | `br/index.html` 한 개 | `br/index.html` + `br/game.js` |
| 맵·인원 | 600 m · 12명 | 1 km · 16명 |
| 흐름 | 맵·조작·사격만 | 낙하 → 전리품 → 자기장 → 최후 1인 |

예전 01·02 코드는 커밋 `03cdee4` 에 그대로 남아 있다. 지운 게 아니라 교체다.

라스트 드롭이 `land-grab-game` 레포에도 `lastdrop/` 으로 있었고 `/game/lastdrop/` 으로도
배포돼 있었다. 같은 게임이 두 주소에 있는 건 관리가 갈라지니 **`/game/br/` 하나로 합쳤다**:
`land-grab-game/lastdrop/` 삭제, `deploy.sh` 의 lastdrop 분기 삭제, 배포본
`games/lastdrop/` 삭제. 허브 카드는 이미 br 하나만 가리키고 있었다.

## 2. 버그 — 낙하를 눌러도 시작화면이 안 사라진다

정현 보고. 재현됐고 원인도 확실하다. **내 실수였다.**

`start()` 는 `veil.hidden = true` 로 시작화면을 끈다. 그런데 내 CSS 가
`#veil{ ... display:grid ... }` 로 잡고 있다. `hidden` 속성의 `display:none` 은
**브라우저 기본 스타일(UA)** 이라서 작성자 스타일인 `display:grid` 에 그냥 밀린다.
속성은 붙지만 화면은 그대로다.

아티팩트로 올렸을 때는 멀쩡했던 이유: 아티팩트 플랫폼이 페이지를 감쌀 때
`[hidden]{display:none!important}` 를 깔아 준다. 그걸 믿고 쓴 HTML 을
**일반 웹서버인 valuetopic.com 에 그대로 올려서** 받침이 사라진 것이다.
`body{margin:0}` 도 같은 경로로 빠져 있었다.

### 고친 내용 — `br/index.html` 의 `<style>` 맨 위

```css
*{box-sizing:border-box}
body{margin:0}
img{max-width:100%}
[hidden]{display:none!important}
```

같은 원인으로 `#hud` 도 시작 전부터 보이고 있었다 (`#hud{display:grid}` + `hidden`).
한 줄 고치니 둘 다 해결됐다.

### 검증 — 헤드리스 크롬에서 실제로 버튼을 클릭해서 측정

`Input.dispatchMouseEvent` 로 `#play` 좌표를 진짜 클릭하고, 계산된 `display` 와
`getBoundingClientRect`, `elementFromPoint(화면 중앙)` 을 봤다.

| | 수정 전 | 수정 후 |
|---|---|---|
| 클릭 후 `#veil` | `grid` · 1264×657 · **보임** | `none` · 0×0 · 사라짐 |
| 클릭 후 `#hud` | `grid` (클릭 전에도 보임) | 클릭 전 `none` → 후 `grid` |
| 화면 중앙 요소 | `dd` (시작 카드 글씨) | `gl` (게임 캔버스) |
| 판정 | **버그 그대로** | **정상** |

지난번 검증이 이걸 놓친 이유를 적어 둔다: 나는 테스트에서 `veil.hidden = true` 를
직접 넣고 "속성이 붙었는지"만 봤다. 버튼을 누르지도, 눈에 보이는지(계산된 display)를
확인하지도 않았다. 앞으로 화면 전환은 **버튼을 클릭해서 계산된 스타일로** 본다.

## 3. 곁딸린 수정 — 포인터 락 거부

검증 중에 `WrongDocumentError: The root document of this element is not valid for
pointer lock.` 이 처리 안 된 프로미스 거부로 올라왔다. 헤드리스 한정 제약이고
게임은 정상 동작했지만(`playing: true`), 실제 브라우저에서도 Esc 직후 등에 거절될 수
있어서 삼키도록 했다.

```js
function lockMouse(){
  const p = canvas.requestPointerLock();
  if (p && p.catch) p.catch(() => {});
}
```
호출부 2곳(`mousedown`, `start`)을 이걸로 바꿨다.

## 4. 점검

- `node --check br/game.js` 통과
- 인라인 `<script>` 0개 / 외부 스크립트 `three.min.js`, `game.js`
- `getElementById` 대상 16개, 없는 것 0개
- `bash -n deploy.sh` 통과
- 허브 `node --check` 통과

## 5. 아직 컨펌 안 받은 것

- 방어구 등급별 피해 감소, 엎드린 표적 피격, 레드존, 보급 상자, 16인 생존 곡선은
  수치로 재 보지 않았다. 코드상으로만 들어가 있다.
- 03-R(Suroi 조사) 제안들: moveSpread, 피해량 하향, 맵·인원 밀도, 자기장 소수 단계화,
  4칸 인벤토리 + 수류탄 — 전부 컨펌 대기.
- 다음 단계는 05 온라인이다.
