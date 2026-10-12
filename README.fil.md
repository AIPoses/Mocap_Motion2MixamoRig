# Motion2MixamoRig

![Motion2MixamoRig](repo_poster.png)

<p align="center">
  <a href="README.zh-CN.md">中文</a> · <a href="README.md">English</a> · <a href="README.ko.md">한국어</a> · <a href="README.ja.md">日本語</a> · <a href="README.de.md">Deutsch</a> · <a href="README.ru.md">Русский</a> · <a href="README.ar.md">العربية</a> · <a href="README.vi.md">Tiếng Việt</a> · Filipino · <a href="README.th.md">ไทย</a> · <a href="README.fr.md">Français</a> · <a href="README.es-MX.md">Español</a> · <a href="README.tr.md">Türkçe</a>
</p>

Ilipat ang galaw ng isang tao sa video — o ang pose sa isang still na larawan — sa isang 3D character na naka-rig gamit ang Adobe Mixamo.

Para sa mga game developer: bigyan ito ng action video na may **iisang tao** (o isang still) at isang Mixamo character; gagawa ito ng mga preview video at isang skinned na character file (`.glb`) na puwede mong ilagay agad sa Blender o Unity.

> Eksperimental pa ang Motion2MixamoRig sa ngayon. Ang bersyon 0.1.0 ay ang command-line pipeline. May kasama ring Blender 4.2+ Extension ang repository na ito na nagpapatakbo ng parehong pipeline sa isang hiwalay na Python environment. Paano i-install at gamitin ang add-on: [`blender_extension/README.md`](blender_extension/README.md).

Dapat magsimula ang mga agent sa [`AGENTS.md`](AGENTS.md).

https://github.com/user-attachments/assets/37c67642-f36c-4815-bcf3-028ed9546a2f

<p align="center">
  <sub>Buksan ang <a href="demo.mp4">demo.mp4</a> sa repository na ito.</sub>
</p>

Isa pang demo:

https://github.com/user-attachments/assets/073f20c8-7283-4209-98e8-f247228bfd56

## Mga Kailangan

Python 3.10+. Pagkatapos i-clone, gumawa ng virtualenv at i-install ang mga dependency; magagamit na pagkatapos ang command na `m2mr`:

```bash
git clone https://github.com/MaxLiu22/Motion2MixamoRig.git
cd Motion2MixamoRig
python -m venv .venv && source .venv/bin/activate

# I-install ang mga dependency at ang proyektong ito. Ang chumpy (isang lumang
# package sa dependency chain) ay nag-i-import ng pip sa setup script nito, kaya
# pumapalya ito ng "No module named 'pip'" sa default na isolated build ng pip.
# I-install muna ito nang naka-off ang isolation, saka i-install ang proyekto.
pip install --upgrade pip setuptools wheel \
  && pip install "numpy>=1.26" \
  && pip install --no-build-isolation "chumpy==0.70" \
  && pip install -e .
```

Hindi dina-download dito ang mga inference weight (GVHMR at iba pa). Ang unang `m2mr run` ang magda-download ng mga ito sa `weights/` (~5 GB, isang beses lang).

Inirerekomenda ang ffmpeg (macOS: `brew install ffmpeg`, Ubuntu: `apt install ffmpeg`).
Tatakbo ang pipeline kahit wala ito, pero **walang tunog** ang mga output video at hindi ito mapapatugtog sa browser.

## Bago tumakbo: ilagay ang mga ito sa `assets/`

Hindi kasama sa repository na ito ang mga file na ito; kailangan mo silang kunin. Dapat eksaktong tugma sa table ang filename ng SMPL-X; puwedeng kahit ano ang pangalan ng FBX, video, at larawan. Kailangan ng bawat run ang SMPL-X, isang Mixamo character, at **alinman sa** isang video **o** isang still.

| Ano | Saan ilalagay | Saan kukunin |
|---|---|---|
| SMPL-X body model | `assets/body_models/smplx/SMPLX_NEUTRAL.npz` | Mag-register sa [SMPL-X](https://smpl-x.is.tue.mpg.de/) at i-download ito |
| Mixamo character FBX | `assets/mixamo/Y_Bot.fbx` | Gamit ang Adobe account, i-download ang Y Bot (o ibang character na naka-rig sa Mixamo) mula sa [Mixamo](https://www.mixamo.com) |
| Action video | `assets/video/<your_clip>.mp4` | Isang clip na may **eksaktong isang** taong malinaw na nakikita, **mula ulo hanggang paa na nasa frame sa buong video** (puwedeng maglagay ng ilang file; ang pinakahuling idinagdag ang gagamitin sa bawat run) |
| Still ng pose (opsyonal) | `assets/image/<your_photo>.jpg` | Isang larawan na may **eksaktong isang** taong malinaw na nakikita, **mula ulo hanggang paa na nasa frame**. Gamitin kasama ng `--image` |

Paalala: `Y Bot.fbx` (**may space**) ang pangalan ng file na dina-download mula sa Mixamo, isang character lang ang pagkakaiba sa `Y_Bot.fbx` (**may underscore**) sa table.
Palitan ito ng pangalang may underscore para ito ang maging default na rig (`m2mr run` nang walang `--rig`). Ayos lang din kung iiwan ang space — kapag walang `Y_Bot.fbx`, ang unang `.fbx` sa `assets/mixamo/` ang gagamitin.

Puwedeng maglaman ng ilang video ang `assets/video/`. Kapag walang `--video` o `--image`, ang file na **pinakahuling idinagdag** sa `assets/video/` ang gagamitin; kapag walang laman ang folder na iyon, ang pinakabagong still sa `assets/image/` ang gagamitin.
Dapat iisang tao lang ang nasa bawat clip o larawan, **mula ulo hanggang paa na nasa frame** — walang naputol sa gilid. Hihinto ang `m2mr doctor` / `m2mr run` bago ang extraction kapag may dalawang tao sa frame.

## Mabilis na Pagsisimula

Mapupunta ang mga resulta sa `outputs/<run-time>_<source>/`. Sa `videos/` sumusulat ang mga video run; sa `images/` naman ang mga still run.

### 1. Suriin ang mga asset at ang environment

Ipi-print ang anumang kulang, kasama kung saan ito ida-download at kung saan ilalagay.

```bash
m2mr doctor
```

### 2. Unang run: magpakita ng resulta sa screen

Kapag walang flag, gagamitin nito ang **pinakabagong file sa `assets/video/`** at ang `assets/mixamo/Y_Bot.fbx`.

```bash
m2mr run
```

Ito ang mabagal na hakbang. Sa unang run, magda-download ito ng ~5 GB na inference weight (isang beses lang), saka kukunin ang galaw ng tao mula sa video.
Sa CPU, mga 8–15 minuto ang extraction ng ~30 segundong clip; ang mga susunod na run na kapareho ang haba, kapag nandiyan na ang mga weight, ay mga 3–5 minuto.
Karaniwang isa o dalawang minuto ang retargeting at rendering. Kapag ginamit muli ang skeleton gamit ang `--skeleton` sa susunod na seksyon, nilalaktawan nang buo ang extraction at matatapos ito sa loob ng ilang sampung segundo.

### 3. Parehong galaw, ibang character

Mag-download ng isa pang character mula sa Mixamo papunta sa `assets/mixamo/`. **Huwag patakbuhin muli gamit lang ang `--rig`** — uulitin nito ang mabagal na extraction.
Ipasa ang `--skeleton` para magamit muli ang skeleton mula sa nakaraang run; mula ilang minuto, magiging ilang sampung segundo na lang:

```bash
m2mr run --skeleton outputs/<previous-run-dir>/skeleton_motion.npz --rig assets/mixamo/Vampire.fbx
```

### 4. Ibang video

Ang `--video` ang pumipili ng clip. Malaya itong maisasama sa `--rig`.

```bash
m2mr run --video assets/video/dance.mp4 --rig assets/mixamo/Vampire.fbx
```

### 5. Isang still na larawan (static na pose)

Kinukuha ng `--image` ang isang **static na 3D pose** mula sa larawan ng isang tao, saka ito ire-retarget sa Mixamo character sa parehong paraan ng video run. Ang mga preview ay apat na still sa `images/` (parehong mga view ng video run), kasama ang isang left/right na orbit clip na `before_after_360_compare.mp4`. Huwag ipasa ang `--video` nang sabay.

```bash
m2mr run --image assets/image/pose.jpg
m2mr run --image assets/image/pose.jpg --rig assets/mixamo/Vampire.fbx
```

Kapag walang path ang `--image`, ang file na pinakahuling idinagdag sa `assets/image/` ang gagamitin.

## Mga Output

Bawat `m2mr run` ay gumagawa ng directory na pinangalanan ayon sa **oras ng pagsisimula ng command**. Pareho para sa video at still ang apat na file sa root; ang preview folder lang ang nagbabago.

```
outputs/20260829_193205_dance/          # o …_pose/ para sa still
├── run.json                            # oras ng simula, mga input, buong command
├── skeleton_motion.npz                 # 3D skeleton ng tao (gamitin muli sa --skeleton)
├── mixamo_rotations.npz                # Mixamo rotation ng bawat buto
├── mixamo_character.glb                # skinned na character, i-import sa Blender / Unity
└── videos/   o   images/               # mga preview — depende sa input
```

Ang **video input** ay sumusulat sa `videos/` (parehong aspect ratio ng clip):

```
videos/
├── human_skeleton.mp4                  # na-recover na 3D skeleton ng tao
├── mixamo_skeleton.mp4                 # skeleton ng Mixamo rig
├── mixamo_character.mp4                # skinned na Mixamo character
└── compare.mp4                         # 2×2: orihinal (kaliwa-itaas) / Mixamo skeleton (kanan-itaas)
                                        #      skeleton ng tao (kaliwa-ibaba) / character (kanan-ibaba)
```

Ang **still input** naman ay sumusulat sa `images/` (parehong apat na view, dagdag ang isang turntable):

```
images/
├── human_skeleton.png                  # na-recover na 3D skeleton ng tao
├── mixamo_skeleton.png                 # skeleton ng Mixamo rig
├── mixamo_character.png                # skinned na Mixamo character
├── compare.png                         # 2×2: orihinal (kaliwa-itaas) / Mixamo skeleton (kanan-itaas)
│                                       #      skeleton ng tao (kaliwa-ibaba) / character (kanan-ibaba)
└── before_after_360_compare.mp4        # kaliwa: orihinal na larawan
                                        # kanan: 10° na orbit ng rig (isang buong ikot)
```

Buksan ang `mixamo_character.glb` sa Blender: File → Import → glTF 2.0 (.glb/.gltf). Kung nakatutok ang camera sa bakanteng espasyo, piliin ang character mesh at gamitin ang View → Frame Selected. Ang kumpol ng mga sphere sa ulo at mga kamay ay display ng buto, hindi ang mesh: sa Outliner, itago ang **Icosphere** at ang **armature**; hayaang nakikita ang skinned na mesh. Itakda ang FPS ng scene at ang huling frame ng timeline mula sa `run.json` (`fps`, `n_frames`) para tumugma ang playback sa source. Ikumpara ang video run sa `videos/mixamo_character.mp4`, at ang still run sa `images/mixamo_character.png` o sa orbit clip.

## Lisensya

Apache-2.0 ang code ng repository na ito. May sariling mga tuntunin ang mga asset na ikaw mismo ang nagda-download, at hindi dapat ipamahagi muli kasama ng repo na ito:

- **SMPL-X**: may gate ng MPI; default ay hindi pangkomersyal na pananaliksik / edukasyon / sining; kailangan ng hiwalay na kasunduan para sa komersyal na paggamit
- **Mixamo FBX**: puwede sa isang inilabas na produkto; huwag ipamahagi muli ang raw na FBX bilang asset pack
- **GVHMR at iba pang inference weight**: awtomatikong dina-download sa `weights/` sa unang `m2mr run`, sa ilalim ng lisensya ng bawat upstream

## Contact

Kung may problema ka, kailangan mo ng tulong para matapos ang local setup, o gusto mong pag-usapan ang pakikipagtulungan, mag-email sa:
[maxliu2022sz@gmail.com](mailto:maxliu2022sz@gmail.com)
