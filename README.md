# AI Platform E2E fixtures

Public media fixtures for Freepik AI Platform sandbox end-to-end tests. They
contain no customer data, credentials, production content, or output of Freepik's
proprietary models. Apart from the `celebrity-*` files, they are synthetic and
show no real person: the `face*` files show AI-generated people made with
open-weights models (see *Generated faces* below). The `celebrity-*` files are
the one exception: public-domain official portraits of a public figure, kept only
as the subject of moderation-refusal tests (see *Moderation-refusal fixtures*
below). Keep filenames stable because CI consumers address them directly.

| File | Purpose | SHA-256 |
| --- | --- | --- |
| `image.png` | RGB image input | `e04729cc34c1227b47b78b6260fb04350c8e0ec9ea79e1197491bcc2993a9f3c` |
| `image.jpg` | JPEG image input | `e4ff86eabe7bd3a92442aac2999fc5a01de332f2ea33ab0d03042473984d4ba9` |
| `mask.png` | Grayscale binary mask | `496c90e4057a50adf49ee60b937c1b1c8d527e1ef12e7a874546ec46c44ff2c0` |
| `audio.mp3` | Three-second 440 Hz audio input | `6b8586fe1982e18d50521226583a5520f11c87044d7873992ea445b50104411f` |
| `video.mp4` | Three-second H.264/AAC video input | `2ffd8d2791d6349984f95ec54f3c5471de11ec490c52220280f71596fe034cca` |
| `face.jpg` | 768x768 JPEG portrait of a synthetic adult, face clearly visible (50 KB) | `06ea0290f7d554b3661e9d5d5684cc2c6ade0557121af45ec20c879a2f7d091d` |
| `face.mp4` | Four-second 720x720 24 fps H.264 video, no audio, of a synthetic adult talking to camera (229 KB) | `7ab0054e7b23e207474cc9ae9bcecee7ebfbbdb613c8f4e44d809a8bf8efd46b` |
| `face-12f-480p.mp4` | The first 12 frames of `face.mp4` at 480x480, 24 fps H.264, no audio (half a second, 19 KB), for slow video endpoints such as the video upscalers | `5061c50bf8e20051dd5f6691c781fa343578029e075a2478d765cefcc8d7f42d` |
| `lora-training.zip` | Zip of four 512x512 JPEG watercolor illustrations (no people), for LoRA training endpoints (91 KB) | `f8629f77a77f08253c31ba97ca6ea9f4f374a720de6427722b6880320fcef689` |
| `celebrity-1.jpg` | 2687x3356 JPEG public-domain official portrait of a public figure, for moderation-refusal tests only (1.3 MB) | `744dd848fbb0584229169e01c4944664957c62495fb9e8af514a088ebca43e19` |
| `celebrity-2.jpg` | 2376x3008 JPEG public-domain official portrait of the same public figure, a different photograph, for moderation-refusal tests only (1.4 MB) | `5db4f1faf2fe7cc23a6076394763b6b2160527435620dd1256b2393fb14be762` |

The files are intentionally small and deterministic, except the `celebrity-*`
portraits (about 1.3 MB each), which are kept at full size so they stay
byte-identical to their Commons originals (see *Moderation-refusal fixtures*).
Any replacement must preserve the media contract and update both this table and
every downstream integrity check.

## Generated faces

Face-driven endpoints (for example Runway `act_two`, which fails with
`NO_FACE_FOUND` on the test card) need a human face. These two depict **no real
person**: both were generated on 2026-10-05 through fal with Apache-2.0
open-weights models, whose licence places no restriction on the outputs.
`face-12f-480p.mp4` is a cut of `face.mp4`, with the same origin and licence.

| File | Model (licence) | Input | Seed |
| --- | --- | --- | --- |
| `face.jpg` | `fal-ai/flux/schnell`, FLUX.1 [schnell] (Apache-2.0), `image_size: square_hd`, 4 steps | Prompt A | 1001 |
| `face.mp4` | `fal-ai/wan/v2.2-a14b/image-to-video`, Wan 2.2 A14B (Apache-2.0), `720p`, 81 frames at 16 fps | Prompt C, animating the FLUX.1 [schnell] image of prompt B (seed 2002, same settings) | 3003 |

- **Prompt A:** Photorealistic head-and-shoulders portrait photo of a woman in her forties, short dark hair, plain navy sweater, neutral light grey studio background, soft even lighting, looking straight at the camera, whole face visible, calm neutral expression
- **Prompt B:** Photorealistic head-and-shoulders portrait photo of a man in his fifties, short grey hair and a trimmed beard, plain green shirt, neutral light grey studio background, soft even lighting, looking straight at the camera, whole face visible, mouth slightly open as if speaking
- **Prompt C:** The man talks calmly to the camera, natural lip movement, small head nods and changing facial expressions, static camera, studio lighting

Post-processing, with ffmpeg:

```sh
ffmpeg -i flux_a.jpg -vf scale=768:768 -q:v 3 face.jpg
ffmpeg -i wan.mp4 -t 4 -an -vf "scale=720:720,fps=24" -c:v libx264 -profile:v high \
  -pix_fmt yuv420p -crf 26 -movflags +faststart face.mp4
ffmpeg -i face.mp4 -frames:v 12 -an -vf scale=480:480 -c:v libx264 -profile:v high \
  -pix_fmt yuv420p -crf 26 -movflags +faststart face-12f-480p.mp4
```

`face-12f-480p.mp4` exists because the video upscalers are slow: on `face.mp4`
(96 frames, 720x720) one sbx e2e case took 2 to 8 minutes. The last command
derives it from `face.mp4`. Its exact bytes depend on the libx264 build and
threading, so, as below, `SHA256SUMS` is the contract.

The seeds and settings make the files reproducible in principle, but a hosted
model may change underneath, so a regeneration is not guaranteed to be
bit-identical. `SHA256SUMS` is the contract, not the recipe. Use these files only
to exercise an endpoint's happy path, never as the subject of a test that expects
unsafe content.
The `celebrity-*` files below are the only fixtures meant for that.

## LoRA training archive

`lora-training.zip` feeds LoRA training endpoints such as fal
`flux-lora-fast-training`, which take a zip URL of at least four images
(ai-platform-workspace#1602). It holds `style_1.jpg` to `style_4.jpg`, no
captions, so a test passes `is_style: true` and a `trigger_word`. The images
depict no real person or brand: they were generated on 2026-10-26 through fal
with `fal-ai/flux/schnell`, FLUX.1 [schnell] (Apache-2.0), `image_size: square`,
4 steps, seeds 4001 to 4004, one per subject, with the prompt
"Flat watercolor illustration of {subject}, soft pastel palette, visible paper
texture, simple composition, no text" and these subjects: a lighthouse on a
cliff, a red bicycle leaning on a wall, a cup of coffee on a wooden table, a
small sailboat on a calm lake.

Post-processing, with ffmpeg and Info-ZIP (fixed mtimes, no extra attributes):

```sh
for i in 1 2 3 4; do ffmpeg -i flux_$i.jpg -q:v 5 style_$i.jpg; done
touch -t 202610260000 style_*.jpg
zip -X -D lora-training.zip style_1.jpg style_2.jpg style_3.jpg style_4.jpg
```

## Moderation-refusal fixtures

Some e2e cases check that a provider's moderation **refuses** an input, for
example the `err-moderation-refused-*` cases of ai-bumblebee-proxy, where Byteplus
asset moderation answers `Celebrity` (ai-bumblebee-proxy#3386). A refusal of that
kind needs a real public figure, so no synthetic image can stand in. The
`celebrity-*` files are the single, deliberate exception to "no real people":

- They are official portraits, works of the United States federal government and
  therefore in the public domain (17 U.S.C. § 105), redistributed byte-for-byte
  from Wikimedia Commons. Neither file was modified.
- Use them **only** as the input a test expects to be refused. Never use them in a
  happy path, never as an image to edit, animate or otherwise generate from, and
  never add a fixture of a private person.

| File | Wikimedia Commons source | Author | Date | Commons SHA-1 |
| --- | --- | --- | --- | --- |
| `celebrity-1.jpg` | [File:President_Barack_Obama.jpg](https://commons.wikimedia.org/wiki/File:President_Barack_Obama.jpg) | Official White House Photo by Pete Souza | 2012-12-06 | `580af39382a677c20eccfacc6e43309eea039805` |
| `celebrity-2.jpg` | [File:Barack_Obama_Senate_portrait_crop.jpg](https://commons.wikimedia.org/wiki/File:Barack_Obama_Senate_portrait_crop.jpg) | United States Senate | 2005-10-18 | `4cb8cc243dc2295b4d815857d0d9b7dc3c96f3a9` |

Whether a provider flags a given portrait is its classifier's call, not a
property of the file: on 2026-11-18 Byteplus accepted another official portrait
of a former president. If a refusal case starts passing the input through,
re-probe and replace the file here rather than loosening the test.

The two images are the only fixtures not dedicated to CC0 by this repository's
authors: they are public domain on their own terms, which the CC0 dedication
below leaves unchanged.

## License

The fixtures and repository documentation are released under [CC0-1.0](LICENSE)
so public CI transports may redistribute them unambiguously.
