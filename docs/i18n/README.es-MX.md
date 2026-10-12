# Motion2MixamoRig

![Motion2MixamoRig](repo_poster.png)

<p align="center">
  <a href="README.zh-CN.md">中文</a> · <a href="README.md">English</a> · <a href="README.ko.md">한국어</a> · <a href="README.ja.md">日本語</a> · <a href="README.de.md">Deutsch</a> · <a href="README.ru.md">Русский</a> · <a href="README.ar.md">العربية</a> · <a href="README.vi.md">Tiếng Việt</a> · <a href="README.fil.md">Filipino</a> · <a href="README.th.md">ไทย</a> · <a href="README.fr.md">Français</a> · Español · <a href="README.tr.md">Türkçe</a>
</p>

Transfiere el movimiento de una persona en un video — o la pose de una foto fija — a un personaje 3D riggeado con Adobe Mixamo.

Para desarrolladores de videojuegos: dale un video de acción con **una sola persona** (o una foto) y un personaje de Mixamo; genera videos de vista previa y un archivo de personaje con skinning (`.glb`) que puedes llevar directo a Blender o Unity.

> Motion2MixamoRig todavía es experimental. La versión 0.1.0 es el pipeline de línea de comandos. Este repositorio también incluye una extensión para Blender 4.2+ que ejecuta ese mismo pipeline en un entorno de Python externo. Cómo instalar y usar el complemento: [`blender_extension/README.md`](blender_extension/README.md).

Los agentes deben empezar por [`AGENTS.md`](AGENTS.md).

https://github.com/user-attachments/assets/37c67642-f36c-4815-bcf3-028ed9546a2f

<p align="center">
  <sub>Abre <a href="demo.mp4">demo.mp4</a> en este repositorio.</sub>
</p>

Otra demo:

https://github.com/user-attachments/assets/073f20c8-7283-4209-98e8-f247228bfd56

## Requisitos

Python 3.10+. Después de clonar, crea un virtualenv e instala las dependencias; con eso ya tendrás disponible el comando `m2mr`:

```bash
git clone https://github.com/MaxLiu22/Motion2MixamoRig.git
cd Motion2MixamoRig
python -m venv .venv && source .venv/bin/activate

# Instala las dependencias y este proyecto. chumpy (un paquete antiguo en la
# cadena de dependencias) importa pip en su script de setup, así que falla con
# "No module named 'pip'" en el build aislado que pip usa por defecto. Instálalo
# primero con el aislamiento desactivado y después instala este proyecto.
pip install --upgrade pip setuptools wheel \
  && pip install "numpy>=1.26" \
  && pip install --no-build-isolation "chumpy==0.70" \
  && pip install -e .
```

Los pesos de inferencia (GVHMR, etc.) no se descargan aquí. El primer `m2mr run` los descarga en `weights/` (~5 GB, solo una vez).

Se recomienda instalar ffmpeg (macOS: `brew install ffmpeg`, Ubuntu: `apt install ffmpeg`).
El pipeline funciona sin él, pero los videos de salida quedarán **sin audio** y no se podrán reproducir en el navegador.

## Antes de ejecutar: coloca estos archivos en `assets/`

Este repositorio no incluye estos archivos; tienes que conseguirlos tú. El nombre del archivo de SMPL-X debe coincidir exactamente con la tabla; los nombres del FBX, el video y la imagen pueden ser cualquiera. Cada ejecución necesita SMPL-X, un personaje de Mixamo y **o bien** un video **o bien** una foto.

| Qué | Dónde | Dónde conseguirlo |
|---|---|---|
| Modelo corporal SMPL-X | `assets/body_models/smplx/SMPLX_NEUTRAL.npz` | Regístrate en [SMPL-X](https://smpl-x.is.tue.mpg.de/) y descárgalo |
| FBX del personaje de Mixamo | `assets/mixamo/Y_Bot.fbx` | Con una cuenta de Adobe, descarga Y Bot (o cualquier otro personaje riggeado con Mixamo) desde [Mixamo](https://www.mixamo.com) |
| Video de acción | `assets/video/<your_clip>.mp4` | Un clip con **exactamente una** persona bien visible, **de pies a cabeza dentro del cuadro todo el tiempo** (puedes guardar varios archivos; cada ejecución usa el que se agregó más recientemente) |
| Foto de pose (opcional) | `assets/image/<your_photo>.jpg` | Una foto con **exactamente una** persona bien visible, **de pies a cabeza dentro del cuadro**. Úsala con `--image` |

Nota: el archivo que descargas de Mixamo se llama `Y Bot.fbx` (**con espacio**), un carácter distinto de `Y_Bot.fbx` (**con guion bajo**) en la tabla.
Cámbiale el nombre a la forma con guion bajo y se convertirá en el rig predeterminado (`m2mr run` sin `--rig`). Dejar el espacio también funciona — si no existe `Y_Bot.fbx`, se usa el primer `.fbx` de `assets/mixamo/`.

`assets/video/` puede contener varios videos. Sin `--video` ni `--image`, la ejecución usa el archivo **agregado más recientemente** a `assets/video/`; si esa carpeta está vacía, usa la foto más reciente de `assets/image/`.
Cada clip o foto debe mostrar a una sola persona, **de pies a cabeza dentro del cuadro** — nada recortado en los bordes. `m2mr doctor` / `m2mr run` se detienen antes de la extracción si hay dos personas a la vista.

## Inicio rápido

Los resultados se guardan en `outputs/<run-time>_<source>/`. Las ejecuciones con video escriben `videos/`; las ejecuciones con foto escriben `images/`.

### 1. Revisa los recursos y el entorno

Todo lo que falte se muestra junto con dónde descargarlo y dónde colocarlo.

```bash
m2mr doctor
```

### 2. Primera ejecución: ver algo en pantalla

Sin opciones, usa **el archivo más reciente de `assets/video/`** y `assets/mixamo/Y_Bot.fbx`.

```bash
m2mr run
```

Este es el paso lento. La primera ejecución descarga ~5 GB de pesos de inferencia (solo una vez) y luego extrae el movimiento humano del video.
En CPU, la extracción de un clip de ~30 segundos tarda unos 8–15 minutos; las ejecuciones posteriores de duración similar, con los pesos ya descargados, tardan unos 3–5 minutos.
El retargeting y el render suelen tardar uno o dos minutos. Reutilizar un esqueleto con `--skeleton` en la siguiente sección se salta por completo la extracción y termina en unas decenas de segundos.

### 3. El mismo movimiento, otro personaje

Descarga otro personaje de Mixamo en `assets/mixamo/`. **No vuelvas a ejecutar solo con `--rig`** — eso repite la extracción lenta.
Pasa `--skeleton` para reutilizar el esqueleto de una ejecución anterior; los minutos se vuelven decenas de segundos:

```bash
m2mr run --skeleton outputs/<previous-run-dir>/skeleton_motion.npz --rig assets/mixamo/Vampire.fbx
```

### 4. Otro video

`--video` elige el clip. Se puede combinar libremente con `--rig`.

```bash
m2mr run --video assets/video/dance.mp4 --rig assets/mixamo/Vampire.fbx
```

### 5. Una foto fija (pose estática)

`--image` reconstruye una **pose 3D estática** a partir de la foto de una persona y luego la retargetea al personaje de Mixamo igual que una ejecución con video. Las vistas previas son cuatro imágenes fijas en `images/` (las mismas vistas que en la ejecución con video), además de un clip orbital izquierda/derecha `before_after_360_compare.mp4`. No pases `--video` al mismo tiempo.

```bash
m2mr run --image assets/image/pose.jpg
m2mr run --image assets/image/pose.jpg --rig assets/mixamo/Vampire.fbx
```

`--image` sin ruta usa el archivo agregado más recientemente a `assets/image/`.

## Salidas

Cada `m2mr run` crea un directorio con el nombre de **la hora en que inició el comando**. Los cuatro archivos de la raíz son iguales para video y foto; solo cambia la carpeta de vistas previas.

```
outputs/20260829_193205_dance/          # o …_pose/ para una foto
├── run.json                            # hora de inicio, entradas, comando completo
├── skeleton_motion.npz                 # esqueleto humano 3D (reutilízalo con --skeleton)
├── mixamo_rotations.npz                # rotaciones de Mixamo por hueso
├── mixamo_character.glb                # personaje con skinning, impórtalo en Blender / Unity
└── videos/   o   images/               # vistas previas — según la entrada
```

Una **entrada de video** escribe `videos/` (con la misma relación de aspecto que el clip):

```
videos/
├── human_skeleton.mp4                  # esqueleto humano 3D reconstruido
├── mixamo_skeleton.mp4                 # esqueleto del rig de Mixamo
├── mixamo_character.mp4                # personaje de Mixamo con skinning
└── compare.mp4                         # 2×2: original (arriba izq.) / esqueleto Mixamo (arriba der.)
                                        #      esqueleto humano (abajo izq.) / personaje (abajo der.)
```

Una **entrada de foto** escribe `images/` en su lugar (las mismas cuatro vistas, más un giro completo):

```
images/
├── human_skeleton.png                  # esqueleto humano 3D reconstruido
├── mixamo_skeleton.png                 # esqueleto del rig de Mixamo
├── mixamo_character.png                # personaje de Mixamo con skinning
├── compare.png                         # 2×2: original (arriba izq.) / esqueleto Mixamo (arriba der.)
│                                       #      esqueleto humano (abajo izq.) / personaje (abajo der.)
└── before_after_360_compare.mp4        # izquierda: foto original
                                        # derecha: órbita del rig en pasos de 10° (una vuelta completa)
```

Abre `mixamo_character.glb` en Blender: File → Import → glTF 2.0 (.glb/.gltf). Si la cámara apunta a un espacio vacío, selecciona el mesh del personaje y usa View → Frame Selected. El grupo de esferas en la cabeza y las manos es la visualización de los huesos, no el mesh: en el Outliner, oculta **Icosphere** y la **armature**; deja visible el mesh con skinning. Ajusta los FPS de la escena y el último frame de la timeline según `run.json` (`fps`, `n_frames`) para que la reproducción coincida con la fuente. Compara una ejecución con video contra `videos/mixamo_character.mp4`, y una con foto contra `images/mixamo_character.png` o el clip orbital.

## Licencia

El código de este repositorio es Apache-2.0. Los recursos que descargues tienen sus propios términos y no deben redistribuirse junto con este repositorio:

- **SMPL-X**: acceso controlado por el MPI; por defecto, investigación / educación / arte sin fines comerciales; el uso comercial requiere un acuerdo aparte
- **Mixamo FBX**: se puede usar en un producto publicado; no redistribuyas el FBX original como paquete de recursos
- **GVHMR y otros pesos de inferencia**: se descargan automáticamente en `weights/` en el primer `m2mr run`, bajo la licencia de cada proyecto original

## Contacto

Si tienes algún problema, necesitas ayuda para terminar una instalación local o quieres platicar sobre una colaboración, escribe a:
[maxliu2022sz@gmail.com](mailto:maxliu2022sz@gmail.com)
