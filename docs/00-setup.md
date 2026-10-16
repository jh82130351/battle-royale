# 00 - 배틀로얄 레포 셋업

## 무엇을 했나
세 번째 게임 **배틀로얄(총 게임)** 의 레포를 세우고, 앞선 두 게임과 같은 절차를
담은 `CLAUDE.md` 와 본 보고서를 넣었다. 기본안·로드맵은 오퍼스 확정본을 그대로 적었다.
**다만 GitHub 레포 생성은 토큰 권한에 막혔다 — 아래 1번.** 로컬 커밋까지는 끝났다.

---

## 1. 레포 정보

| 항목 | 값 |
| --- | --- |
| 이름 | `jh82130351/battle-royale` — **가칭, 정현 확정 대기** |
| 브랜치 | `main` |
| 작업 위치 | `~/battle-royale` |
| 게임 본체 | `br/index.html` (아직 없음 — 01 작업) |
| 배포 경로 | `https://valuetopic.com/game/br/` (계획) |
| 공개 범위 | public (앞선 두 레포와 동일) |

### ⚠ 레포를 아직 못 만들었다
```
$ gh repo create battle-royale --public
GraphQL: Resource not accessible by personal access token (createRepository)
```
서버 PAT에 **레포 생성(Administration) 권한이 없다.**

헷갈리기 쉬운 점을 적어 둔다. `gh api` 로 물어보면 이렇게 답한다:
```
land-grab-game     push=true admin=true
fps-prototype      push=true admin=true
```
그런데 **fps-prototype 은 실제로 푸시가 안 된다**(403, 커밋 6개가 로컬에 쌓여 있다).
`gh api` 가 말하는 건 **계정 권한**이고, 막는 건 **토큰에 선택된 레포 목록**이다.
그래서 권한 확인은 `git push` 로 직접 해야 한다.

**정현님께 부탁드릴 것 — 둘 중 하나:**
1. **웹에서 직접 레포를 만들어 주신다** (github.com → New repository → `battle-royale`, Public,
   README·.gitignore·license **전부 체크 해제**). 그러면 제가 바로 푸시한다.
   로컬 커밋과 원격 주소는 이미 맞춰 뒀다.
2. **토큰을 고쳐 주신다** — fps-prototype 도 같이 풀린다.
   Settings → Developer settings → Fine-grained tokens → 해당 토큰
   - Repository access: **All repositories** (또는 battle-royale·fps-prototype 추가)
   - Permissions → Repository → **Administration: Read and write** (레포 생성용)
   - Permissions → Repository → **Contents: Read and write** (푸시용)

**2번을 권한다.** fps-prototype 커밋 6개가 같은 이유로 2주째 묶여 있다.

---

## 2. 기본안 표 (오퍼스 확정)

| 항목 | 확정값 |
| --- | --- |
| 인원 | **온라인 최대 8인** (방 설정 2~8). 빈자리는 봇으로 채운다 |
| 맵 | **1000×1000** 평지+언덕·건물 몇 채. 땅따먹기/FPS 지형 재활용 |
| 시작 | 공중/외곽 랜덤 낙하 → **맨손** |
| 파밍 | 맵에 총·탄약·방어구·회복 랜덤 스폰. **E로 줍기** |
| 총 | **5종** — 권총·SMG·소총·샷건·저격. 데미지·연사·탄창·반동 차등 |
| 자기장 | 시간 따라 안전구역 축소, 밖은 지속 데미지 |
| 승리 | 최후 1인(또는 1팀). 종료 시 순위표 |
| 시점 | **1인칭** (FPS 컨트롤 재활용) |
| 컨트롤 | WASD / Shift / Space / 마우스 · 좌클릭 발사 · R 재장전 · 1~5 무기 · E 줍기 |
| 네트워크 | **자체 WS 서버** (land-grab 방식, host-authoritative) — **단 05단계부터.** 01~04는 봇전 오프라인 |

### 표와 다르게 적은 칸이 하나 있다
표 본문에는 맵이 `400×400` 으로 적혀 있었으나,
**같은 지시에서 "8인 배틀로얄에 400은 좁다 → 1000×1000" 으로 정정**해 주셨다.
**확정값은 1000×1000** 이다. 위 표는 그 정정을 반영한 것이다.

16인은 **05 온라인 단계에서 성능을 보고 결정**한다. 일단 8인으로 간다.

---

## 3. 네트워크 — 자체 WS 서버 (확정), 단 05단계부터

배틀로얄은 온라인이 곧 게임이지만, **01~04는 봇전 오프라인으로 만든다.**
이 순서가 맞다고 본다. 쏘는 맛과 자기장이 봇 상대로 먼저 돌아가야
온라인을 붙일 때 "네트워크가 문제인지 게임이 문제인지"를 가릴 수 있다.

앞선 두 게임의 상태는 이렇다. 확정안이 자체 WS 쪽인 이유이기도 하다.

| 게임 | 방식 | 지금 상태 |
| --- | --- | --- |
| 땅따먹기 | 자체 WS 서버 (`/game/ws`, 포트 5002) | **잘 돈다** |
| FPS Builder | Firebase Realtime DB | **죽어 있다** |

FPS 쪽이 죽은 이유는 24 작업 때부터 보고드린 그대로다. 공개 쓰기를 막아 주신 건 잘 된 일인데,
**이 게임은 로그인을 안 하고 Firebase Auth 자체가 설정돼 있지 않아서** 접속 버튼이 아무 일도 안 한다.

총 게임은 **판정이 서버에 있어야 한다.** 클라이언트가 "맞췄다"고 말하게 두면 그 순간 치트가 된다.
host-authoritative 는 땅따먹기에서 이미 검증됐다.

**05 단계에서 미리 정해야 할 것:** `ws_server.py` 는 땅따먹기 격자 전용이라 그대로 못 쓴다.
포트를 하나 더 열거나(예: 5003) 한 서버에서 게임별 경로를 나누게 된다.
**프로덕션을 건드리는 일이라 그때 별도 지시를 받는다.**

---

## 4. 서빙 계획 (지금은 계획만, 코드 안 건드렸다)

```
valuetopic.com/game/        ← 허브        (land-grab-game 레포 hub/index.html)
valuetopic.com/game/land-grab/
valuetopic.com/game/fps/
valuetopic.com/game/br/     ← 여기 추가   (이 레포 br/index.html)
```

Flask 는 `games/` 아래 폴더를 그대로 서빙한다. **`app.py` 는 안 고쳐도 되고 재기동도 필요 없다.**
24 작업 때 이미 그렇게 만들어 뒀다.

필요한 건 둘뿐이다.

**(1) `land-grab-game/deploy.sh` 에 br 분기 추가** — fps 와 같은 꼴
```bash
BR_SRC=${BR_SRC:-/tmp/battle-royale-deploy}
mkdir -p "$GAMES/br"
cp "$BR_SRC/br/index.html" "$GAMES/br/index.html"
```
fps 분기에 있는 **미푸시 커밋 경고**도 같이 넣는다. 그게 없으면
푸시 못 한 작업이 조용히 사라진 채 옛 버전이 배포된다 (실제로 겪었다).

**(2) 허브 카드 한 줄** — `land-grab-game/hub/index.html` 의 `GAMES` 배열
```js
{ id: 'br', name: '배틀로얄', desc: '...', icon: '🪂',
  tags: ['3D', '멀티', '총 · 서바이벌'], ready: false },
```
**본체가 생기기 전까지는 `ready: false`** 로 둔다. 눌러도 안 열리는 카드가 뜨면 안 된다.
`SOON` 숫자도 1 줄인다.

> 둘 다 **land-grab-game 레포를 고치는 일**이라 이 레포 작업이 아니다.
> 01 작업에서 본체 뼈대와 함께 넣는다.

---

## 5. 로드맵 (오퍼스 확정)

| 단계 | 내용 |
| --- | --- |
| **00** | 레포 셋업 · 본 문서 — **지금** |
| 01 | **3D 맵 + 컨트롤러** |
| 02 | **총기·발사·히트 판정** (봇 표적) |
| 03 | **파밍·인벤·무기 교체** |
| 04 | **자기장·낙하·승패** → 봇전 완성 |
| 05 | **온라인 8인** |
| 06 | **밸런스·연출·소리** |

01 에는 `deploy.sh` br 분기와 허브 카드도 같이 넣는다 (4번 참조).
본체가 걸어다닐 수 있게 되기 전까지 허브 카드는 `ready: false` 다.

> 제가 00 초안에서 "사격 손맛을 앞으로 빼자"고 제안드렸는데,
> 확정 로드맵이 02에 총기를 두고 있어 그 취지가 이미 반영돼 있다. 그대로 간다.

## 6. 변경 파일 목록

| 파일 | 내용 |
| --- | --- |
| `CLAUDE.md` | 신규. land-grab-game 것을 가져와 프로젝트명·경로를 바꾸고, 앞선 두 게임에서 데인 것 6가지와 토큰 주의를 더했다 |
| `docs/00-setup.md` | 신규. 본 문서 |
| `.gitignore` | 신규 |

## 7. 실행 명령
```bash
gh repo create battle-royale --public      # ← 막혔다 (1번)
mkdir -p ~/battle-royale/docs
cd ~/battle-royale && git init -b main
git remote add origin https://github.com/jh82130351/battle-royale.git
git add . && git commit
git push -u origin main                    # ← 레포가 생겨야 된다
```

## 8. 문법 검사 결과
**해당 없음.** 이번 작업에 코드가 없다 (`.md` 와 `.gitignore` 뿐).
`br/index.html` 이 생기는 01 작업부터 `node --check` 를 돌린다.

## 9. 새 커밋 해시
`(아래 커밋 참조 — 로컬에만 있다. 원격 레포가 생기면 그대로 올라간다.)`

---

## 10. 감독 확인 요청 사항

**1번 하나만 막혀 있다. 나머지는 확정안으로 다 채웠다.**

1. **레포를 만들어 주셔야 푸시가 된다.** 둘 중 하나다.
   - **(권장) 토큰 수정** — fps-prototype 커밋 6개도 같이 풀린다.
     Fine-grained token → Repository access **All repositories**,
     Permissions → **Administration: Read and write** + **Contents: Read and write**
   - **웹에서 직접 생성** — `battle-royale`, Public, README·.gitignore·license **전부 체크 해제**.
     바로 푸시한다. 다만 fps-prototype 은 계속 막힌 채다.
2. **레포 이름** `battle-royale` 로 굳혀도 되는지 (가칭이라 하셨다). 지금 바꾸는 게 제일 싸다.
3. **01 을 바로 시작해도 되는지.** 기본안이 확정됐으니 막는 건 없다.
   레포 생성과 무관하게 `br/index.html` 작업은 로컬에서 진행 가능하다.

## 11. 남은 이슈
- **레포 생성 막힘** (10번 1).
- fps-prototype 커밋 6개가 여전히 푸시 안 됨 (같은 토큰 문제).
- FPS Builder 멀티가 Firebase Auth 미설정으로 동작하지 않음 (별도 지시 대기 중).
- **05 온라인 단계에서 WS 서버 포트·경로 결정 필요** (3번). 프로덕션 작업이라 그때 지시를 받는다.
- **16인 여부는 05 에서 성능 보고 결정.** 01~04 는 8인 기준으로 짠다.
