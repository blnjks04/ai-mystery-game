# 3D 현장 제작 도구

캠페인 3D 현장([설계](../../docs/roadmap/campaign-3d-design.md))의 공간, 소품, 인물을 만드는 Blender 스크립트입니다. 2026-11-19에 목원고 12시험실 하나를 이 순서로 끝까지 만들어 확인했습니다.

- 결과물(`public/assets/3d/`)과 원본(`tools/_raw/3d/`)은 **저장소에 올리지 않습니다.** 저장소만 받으면 3D 현장은 열리지 않고, 아래 순서로 다시 만들어야 합니다. Tripo에서 받은 원본 glb는 다시 만들 수 없으므로(생성 결과가 매번 다릅니다) 따로 받아야 합니다.
- 스크립트의 경로는 `D:\ai-mystery-game`으로 적혀 있습니다. 다른 자리에서 쓰려면 각 파일 위쪽의 `HERE`, `TRIPO`, `OUT`을 고칩니다.
- Blender 5.2에서 확인했습니다. Blender는 Z가 위이고, 게임(three.js)의 (x, y, z)는 Blender의 (x, z, −y)입니다.

## 파일

| 파일 | 하는 일 |
|---|---|
| `room12_layout.py` | 방의 치수와 자리. 블록아웃과 사실화가 함께 읽는다. 조사점 이름을 바꾸면 `public/js/data/sites/mokwon.js`도 같이 바꾼다 |
| `build_room12_blockout.py` | 회색 상자로 된 방, 이름 붙은 빈 오브젝트(`spot_*`, `spawn_*`, `npc_*`, `view_*`, `fx_*`), 충돌 상자(`col_*`). 컬렉션 `room12` |
| `blockout_lib.py` | 블록아웃을 짓는 도구: 상자, 벽(문과 창을 뚫는다), 방, 충돌 상자, 자리 표시. 프롤로그부터 쓴다 |
| `prologue_layout.py` | 프롤로그 네 장소(옥탑방, 아파트, 학원, 목원고 1층)의 치수와 자리. 이름을 바꾸면 `public/js/data/sites/prologue.js`도 같이 바꾼다 |
| `build_prologue_blockout.py` | 네 장소의 블록아웃을 짓고 내보낸다: `build_all()`, `show(이름)`, `export_all()`. 컬렉션 `rooftop` `apt` `academy` `gate` |
| `build_room12_real.py` | 사실화한 방. 벽·바닥·텍스 천장·창·칠판·형광등, Tripo 가구 배치, 흔적 그림(가루, 테이프, 사다리 자국, 눌린 자국), 둘째 UV `lm`. 컬렉션 `room12_real` |
| `bake_room12.py` | 조명 굽기. 형광등 / 하늘빛 / 해의 되비침 석 장 |
| `real_lib.py` | 사실화 도구(`Real`): 그림으로 만든 재질(얼룩, 나뭇결, 장판, 벽돌 따위), 상자·원통·덩어리·글자·그림판, 문과 창이 뚫린 벽, 형광등, 하늘 반구, Tripo 가구 불러와 놓기, 끝맺음(`finish`: 안 보이는 면 지우기, 조명용 UV `lm`). 프롤로그부터 쓴다 |
| `bake_site.py` | 장소 하나의 조명 굽기(석 장). `bake_room12.py`를 장소에 매이지 않게 고친 것 |
| `make_prologue_textures.py` | 프롤로그의 글자 그림(노트북 화면, 순위표, 현수막, 문패, 안내문, 일정표, 분리수거 쪽지 따위)과 열람 소품의 그림(운송장, 쪽지, 수험표, 연습지와 그 광택, 골판지, 도장, 인주, 천). Blender 밖에서 Python과 Pillow로 돌린다. `tools/_raw/3d/tex/pl_*.png` |
| `build_prologue_real.py` | 사실화한 프롤로그 네 장소. `build(이름)`, `bake_one(이름, 종류, 화소, 표본)`, `export(이름)`, `save(이름)`. 장소마다 blend 파일 하나(`tools/_raw/3d/blend/prologue_<이름>.blend`) |
| `make_prop_textures.py` | 단서 그림(펜 라벨, 점검표 스티커 두 가지, 기판 화면). Blender 밖에서 Python과 Pillow로 돌린다 |
| `build_props.py` | 열람용 소품 넷(펜, 뿔테, 천장 속, 빈 천장 속), 프롤로그의 열람 소품 셋(택배 상자, 필통, 연습지), 인물 불러오기·내보내기 |
| `export_glb.py` | 컬렉션 하나를 glb로. Draco, WebP, 동작 |

## 순서

Blender의 Python 콘솔이나 Blender MCP에서 파일을 `exec`로 불러 씁니다.

```python
run = lambda f: exec(open(rf"D:\ai-mystery-game\tools\3d\{f}", encoding="utf-8").read(), globals())
SITES = r"D:\ai-mystery-game\public\assets\3d\sites"
```

**1. 블록아웃**: 걸어 보고 동선, 시야, 조사점까지의 거리를 확인한다.

```python
run("build_room12_blockout.py")
run("export_glb.py")
export_collection("room12", SITES + r"\mokwon_room12_blockout.glb")
```

게임에서는 `/site.html?site=mokwon_room12&blockout`.

프롤로그의 네 장소는 블록아웃을 한 번에 짓습니다. 사실화는 아래 '프롤로그의 사실화'에 있습니다.

```python
run("build_prologue_blockout.py")   # blockout_lib.py와 prologue_layout.py를 스스로 읽는다
build_all()
export_all()                        # public/assets/3d/sites/prologue_{rooftop,apt,academy,gate}_blockout.glb
```

게임에서는 `/site.html?case=prologue&blockout`. `read_homefile` 같은 장면 초기화와 내보내기를 한 번의 실행에 같이 두면 내보내기가 실패합니다(문맥이 낡습니다). 따로 실행합니다.

**2. 사실화**: `tools/_raw/3d/tripo/`에 Tripo 가구(책상, 의자, 교탁, 사다리)가 있어야 한다.

```python
run("build_room12_real.py")
```

**3. 굽기**: 한 장에 45~85초(RTX 3080 Ti, 2048 화소, 표본 640). 석 장을 따로 부른다.

```python
run("bake_room12.py")
bake("lamp", 2048, 640); bake("sky", 2048, 640); bake("sun", 2048, 640)
split_tile24()   # 처음 구운 뒤에 한 번. 밀어 올릴 텍스 한 장을 떼어 낸다
```

**4. 방 내보내기**

```python
run("export_glb.py")
export_collection("room12_real", SITES + r"\mokwon_room12.glb", draco=True, apply=False, webp=82)
```

**5. 소품**: 먼저 Blender 밖에서 `python tools/3d/make_prop_textures.py`. 그리고

```python
run("build_props.py")
build_all()        # 넷을 한꺼번에. 하나만 다시 만들지 않는다 (아래 주의)
export_props()
```

**6. 인물**: Tripo에서 리깅하고 동작을 넣어 받은 glb를 `tools/_raw/3d/tripo/`에 둔다.

```python
build_char("bae.glb", "bae")
export_char("bae")   # 동작 이름은 idle, agree
```

## 게임이 읽는 이름

| 이름 | 뜻 |
|---|---|
| `spot_*` | 조사점의 자리 |
| `spawn_*`, `npc_*`, `view_*`, `fx_*` | 출발점, 인물 자리, 정해진 시점(사다리 위), 효과 자리(빛 신호가 새는 구멍) |
| `col_*` | 충돌 상자. 보이지 않고, 걷기와 길 찾기에만 쓴다 |
| `static`, `tile24` | 구운 빛을 받는 메시. 둘째 UV가 `lm` |
| `ladder_folded`, `ladder_open`, `col_ladder_open` | 사건 데이터가 이름으로 켜고 끄는 물건 |
| `detail_*` | 열람용 소품 안의 디테일 자리 |
| `view_center` | 열람할 때 돌리는 중심 |
| 재질 `*lens_film`, `*lens` | 안경의 왼쪽 알(필름)과 오른쪽 알 |

이름과 사건 데이터가 맞는지는 `tests/site.test.mjs`가 `room12_layout.py`를, `tests/prologue.test.mjs`가 `prologue_layout.py`를 읽어 확인합니다.

## 프롤로그의 사실화

장소 하나를 장면 하나로 짓고, 굽고, 내보냅니다. 블록아웃의 자리 표시와 충돌 상자는 `build()`가 블록아웃 스크립트를 불러 그대로 가져옵니다(이름과 자리가 같다).

```python
run("build_prologue_real.py")       # real_lib.py, bake_site.py, 블록아웃 스크립트를 스스로 읽는다
build("academy")                    # rooftop | apt | academy | gate
bake_one("academy", "lamp", 2048, 1024)
bake_one("academy", "sky", 2048, 1536)
bake_one("academy", "sun", 2048, 768)
export("academy")                   # public/assets/3d/sites/prologue_academy.glb + _lm_{lamp,sky,sun}.jpg
save("academy")                     # tools/_raw/3d/blend/prologue_academy.blend
```

- 먼저 `python tools/3d/make_prologue_textures.py`로 글자 그림을 만들어 둡니다.
- 게임에서는 `/site.html?site=prologue_academy`(그 장소만), `/site.html?case=prologue`(옥탑방부터). 밝기는 `public/js/data/sites/prologue.js`의 `REAL`에서 맞춥니다: 장소의 `exposure`, 빛 지도마다의 `gain`, 가구와 인물을 밝히는 환경 반사의 자리 `probes`(구역 `box`마다 하나).
- **Blender MCP로 돌릴 때는 한 번의 실행이 180초를 넘지 않게 나눕니다.** 짓기와 형광등 굽기를 한 번에, 하늘·해 굽기와 내보내기·저장을 그다음에. 실행마다 이름이 지워지므로 스크립트를 다시 읽되 `build()`는 다시 부르지 않습니다.
- `tools/_raw/3d/tripo/`에 Tripo 가구가 없으면 그 자리에 손으로 짠 대역이 놓입니다(`furn`, `seat`, `desk`, `cabinet`, `car`, `tree`). 파일이 생기면 다시 짓기만 하면 됩니다.
- Tripo 모델의 앞은 제각각입니다. 받은 그대로일 때: 침대는 길이가 Y이고 머리가 +Y, 사무 의자·책꽂이·사물함·수거함·소파·정수기·사무 책상은 앞이 −Y, 승용차는 길이가 Y이고 앞이 −Y. 새 모델은 Blender에 불러 −Y 쪽에서 한 번 보고 `rot_z`를 정합니다. 크기는 모두 가장 긴 변이 1로 맞춰져 옵니다.
- 사무 책상은 모니터·자판·화분이 한 덩어리로 붙어 옵니다. 윗판 위의 것을 걷어 내고 몸통만 쓰며(`desk_mesh`), 윗판과 책상 위의 것은 따로 짓습니다(책상마다 다르게). 나무는 밑동에 낙엽 원판이 붙어 있어 조금 묻어 심고, 잎이 높이의 0.3부터 달리므로 경비실 옆에서는 8m로 키웁니다.
- **다시 지어도 조명용 UV는 같습니다**(같은 입력이면 `static`이 같다). 그래서 가구(`furniture`)만 바뀌었으면 다시 굽지 않고 `build()` → `export()`만 해도 됩니다. 다만 `build()`가 돌려주는 `static`의 면 수가 달라졌으면 구운 그림과 어긋나므로 석 장을 다시 굽습니다. 대역이 `static` 상자로 지어지던 것(책상의 판)이 Tripo로 바뀔 때, 난수를 쓰는 차례가 바뀔 때(나무의 대역과 Tripo 나무) 그렇게 됩니다. 어긋나면 게임에서 벽이 얼룩덜룩 검게 나옵니다.
- 위층 덩어리처럼 1층 천장과 같은 높이에 밑면이 있는 상자는 밑면을 떼어 냅니다(`no_bottom`). 겹치면 게임에서 천장이 어둡게 깜빡입니다.
- **열람 소품 셋**(택배 상자, 필통, 연습지)은 12시험실 파일(`mokwon_room12.blend`)에서 짓습니다. `run("build_props.py")` → `build_all()` → `build_inspect_props()` → `export_props(INSPECT_PROPS)`. 필통이 사인펜과 안경의 메시를 빌려 쓰므로 `build_all()`이 먼저입니다. 결과는 `public/assets/3d/props/{parcel,pencilcase,practice}.glb`이고, 사건 데이터의 `INSPECTS`에 `model`로 이어져 있습니다(시점 값 `view`도 거기서 맞춥니다: glb가 있으면 사건 데이터의 `view`만 읽습니다).
  - 연습지의 광택은 거칠기 그림(`pl_practice_rough.png`)에만 있습니다. 줄의 자리는 `pl_practice.json`에 함께 적히고, Blender가 볼 곳(`detail_gloss`, `detail_margin`)의 자리를 거기서 읽습니다. 문제를 고치면 그림 둘과 json이 같이 바뀌므로 소품도 다시 지어 내보냅니다. 문제는 새로 지은 것만 싣고, 고칠 때마다 답을 직접 풀어 확인합니다.
  - 거칠기 그림은 Separate Color 노드의 초록을 거쳐 이어야 glb에 실립니다(`img_mat`).
  - `Part`는 면마다 점을 따로 만들어서 굴린 것이 각져 보입니다. 도장, 인주통, 필통은 `finish_weld()`로 점을 붙이고 꺾인 모서리만 날카롭게 남깁니다.
  - 안경 다리는 앞테 바로 뒤에서 잘라 안쪽으로 돌려 접습니다(`folded_glasses`). 돌릴 점은 **돌리기 전에** 양쪽을 다 골라 둡니다(먼저 돌린 다리가 반대쪽으로 넘어가 한 번 더 돌았다).
  - 상자 앞벽이 속을 가리므로 물건은 구긴 종이 위 7cm 높이에 놓았습니다. 물건을 옮기면 종이가 그 자리만 눌리도록 `rest`의 자리도 같이 고칩니다.
- 인물 셋(`guard`, `tutor`, `chief`)은 `build_props.py`의 `build_char()`, `export_char()`로 **한 사람씩** 내보냅니다. 여럿을 함께 불러오면 동작 이름이 `idle.001`처럼 겹칩니다.
- 충돌 상자나 자리 표시를 고쳤으면 블록아웃 glb도 다시 내보냅니다(`?blockout`이 같은 자리를 쓰도록).

## 조명을 굽는 방식

- `static`의 둘째 UV `lm`에 Cycles의 DIFFUSE를 석 장 굽습니다: 형광등(직접+간접), 하늘빛(직접+간접), 해(간접만).
- 해의 직사광과 그림자는 굽지 않습니다. 게임이 실시간으로 그립니다. 그래서 책상 그림자가 또렷하고, 사다리 같은 움직이는 물건도 그림자를 떨굽니다.
- 굽기에는 잡티 제거가 듣지 않습니다. 간접광 표본을 눌러 두고(4.0), 구운 그림을 3×3 가운뎃값과 좁은 가우스로 거릅니다.
- 8비트 sRGB JPG로 줄일 때 밝기 범위는 `RANGE`(형광등 0.8, 하늘 0.8, 해 0.45)입니다. **이 값과 해의 방향·색·세기는 `mokwon.js`의 `lightmap.maps.*.range`, `light.sun`과 같아야 합니다.**
- 게임은 석 장을 시각에 맞춰 섞습니다. 가구와 인물은 구운 빛을 받지 않으므로, 방 가운데에서 찍은 환경 반사로 밝힙니다.
- 프롤로그(`real_lib.py`, `bake_site.py`)에서 더한 것:
  - **한 화소보다 얇은 면은 구워지지 않고, 옆의 어두운 화소로 메워져 게임에서 검게 나옵니다**(칸막이의 윗면, 블라인드, 선반의 모서리). `finish()`가 UV 섬을 미터로 재어 0.045m보다 얇은 쪽을 그만큼 불리고(`_inflate_thin`), 그 뒤에 섬을 다시 채워 넣습니다(`pack_islands`, UV의 쓰임이 21%에서 55~68%로).
  - 바닥에 붙은 아랫면처럼 보이지 않는 면은 굽기 전에 지웁니다. 지우지 않으면 새까만 화소가 옆 섬으로 번집니다.
  - 구운 그림은 구워진 화소 안에서만 거르고(`smooth`), 섬의 가장자리를 바깥으로 8화소 불립니다(`dilate`). 굽기의 여백(`margin`)은 0으로 둡니다.
  - 장소마다의 해 방향과 등의 세기는 `build_prologue_real.py`의 `BAKE`에 있습니다. 밝기 범위는 12시험실과 같습니다(0.8 / 0.8 / 0.45).

## 단서는 손으로 만든다

생성 모델은 단서의 디테일을 원하는 대로 주지 않습니다. 12시험실에서는 이렇게 했습니다.

- 펜은 Tripo를 쓰지 않고 통째로 만들었습니다. 라벨의 검은 네모, 뒤 끝의 바늘구멍, 뚜껑의 창이 정확한 자리에 있어야 하기 때문입니다.
- 안경은 Tripo 모델의 알이 불투명한 면으로 나옵니다. 밝은 면에서 시작해 이어진 평평한 면을 따라가며 알을 떼어 내고, 왼쪽에만 필름 재질을 주었습니다.
- 천장 장치는 Tripo 모델에서 배관 토막을 지우고, 배관·케이블 타이·마이크·점검표·기판 화면을 손으로 더했습니다.
- 글자가 들어가는 것(라벨, 점검표, 화면)은 모두 `make_prop_textures.py`가 그립니다.

## 주의

- **`build_props.py`를 다시 부른 뒤에는 꼭 `build_all()`을 돌립니다.** 파일을 부르면 함께 쓰는 재질이 새로 만들어져, 다시 만들지 않은 소품의 재질이 빠집니다.
- 소품과 인물 컬렉션은 평소에 뷰 레이어에서 빼 둡니다(방을 구울 때 섞이지 않게). 내보낼 때는 다시 넣어야 합니다.
- Blender는 한 파일 안에서 이름이 겹치지 못합니다(`view_center.001`). `export_props()`가 내보내는 컬렉션의 이름을 되돌려 줍니다.
- 같은 인물을 다시 불러오면 동작이 `idle.001`처럼 겹칩니다. `build_char()`가 앞의 것을 지웁니다.
- Tripo 웹 화면의 순서, 설정이 되돌아가는 곳, 크레딧은 로컬의 `tools/_raw/3d/tripo/PIPELINE_NOTES.md`에 있습니다.
