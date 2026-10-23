# 03-05 — /game/br/ 를 배틀그라운드(01·02) 빌드로 되돌리고, 라스트 드롭은 따로 둔다

날짜: 2026-10-23
지시: "라스트 드롭이라는 거는 따로 하고 배틀그라운드 다시 살려"

## 무엇을 바꿨나

- `br/index.html` · `CLAUDE.md` 를 03-04 교체 직전(672af3c, 코드는 03cdee4) 상태로 되돌렸다.
- `br/game.js` 삭제 — 01·02 빌드는 한 파일(59,614 B)이다.
- 라스트 드롭은 land-grab-game 레포 `lastdrop/` 로 돌아갔다 → `/game/lastdrop/`.
  03-04 의 시작화면 버그 수정(`[hidden]{display:none!important}`, `lockMouse()`)은 그쪽에 옮겨 담았다.
- 01·02 빌드는 시작화면을 `style.display='none'` 으로 닫으므로 같은 버그가 없다.

03-04 문서는 이력으로 남겨 둔다.
