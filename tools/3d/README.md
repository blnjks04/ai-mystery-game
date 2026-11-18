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
| `build_room12_real.py` | 사실화한 방. 벽·바닥·텍스 천장·창·칠판·형광등, Tripo 가구 배치, 흔적 그림(가루, 테이프, 사다리 자국, 눌린 자국), 둘째 UV `lm`. 컬렉션 `room12_real` |
| `bake_room12.py` | 조명 굽기. 형광등 / 하늘빛 / 해의 되비침 석 장 |
| `make_prop_textures.py` | 단서 그림(펜 라벨, 점검표 스티커 두 가지, 기판 화면). Blender 밖에서 Python과 Pillow로 돌린다 |
| `build_props.py` | 열람용 소품 넷(펜, 뿔테, 천장 속, 빈 천장 속)과 인물 불러오기·내보내기 |
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

이름과 사건 데이터가 맞는지는 `tests/site.test.mjs`가 `room12_layout.py`를 읽어 확인합니다.

## 조명을 굽는 방식

- `static`의 둘째 UV `lm`에 Cycles의 DIFFUSE를 석 장 굽습니다: 형광등(직접+간접), 하늘빛(직접+간접), 해(간접만).
- 해의 직사광과 그림자는 굽지 않습니다. 게임이 실시간으로 그립니다. 그래서 책상 그림자가 또렷하고, 사다리 같은 움직이는 물건도 그림자를 떨굽니다.
- 굽기에는 잡티 제거가 듣지 않습니다. 간접광 표본을 눌러 두고(4.0), 구운 그림을 3×3 가운뎃값과 좁은 가우스로 거릅니다.
- 8비트 sRGB JPG로 줄일 때 밝기 범위는 `RANGE`(형광등 0.8, 하늘 0.8, 해 0.45)입니다. **이 값과 해의 방향·색·세기는 `mokwon.js`의 `lightmap.maps.*.range`, `light.sun`과 같아야 합니다.**
- 게임은 석 장을 시각에 맞춰 섞습니다. 가구와 인물은 구운 빛을 받지 않으므로, 방 가운데에서 찍은 환경 반사로 밝힙니다.

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
