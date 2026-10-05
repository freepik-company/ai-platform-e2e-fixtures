# AI Platform E2E fixtures

Public, synthetic media fixtures for Freepik AI Platform sandbox end-to-end tests.
They contain no customer data, credentials, production content, real people, or
output of Freepik's proprietary models. The two `face.*` files are AI-generated
people made with open-weights models (see *Generated faces* below). Keep filenames
stable because CI consumers address them directly.

| File | Purpose | SHA-256 |
| --- | --- | --- |
| `image.png` | RGB image input | `e04729cc34c1227b47b78b6260fb04350c8e0ec9ea79e1197491bcc2993a9f3c` |
| `image.jpg` | JPEG image input | `e4ff86eabe7bd3a92442aac2999fc5a01de332f2ea33ab0d03042473984d4ba9` |
| `mask.png` | Grayscale binary mask | `496c90e4057a50adf49ee60b937c1b1c8d527e1ef12e7a874546ec46c44ff2c0` |
| `audio.mp3` | Three-second 440 Hz audio input | `6b8586fe1982e18d50521226583a5520f11c87044d7873992ea445b50104411f` |
| `video.mp4` | Three-second H.264/AAC video input | `2ffd8d2791d6349984f95ec54f3c5471de11ec490c52220280f71596fe034cca` |
| `face.jpg` | 768x768 JPEG portrait of a synthetic adult, face clearly visible (50 KB) | `06ea0290f7d554b3661e9d5d5684cc2c6ade0557121af45ec20c879a2f7d091d` |
| `face.mp4` | Four-second 720x720 24 fps H.264 video, no audio, of a synthetic adult talking to camera (229 KB) | `7ab0054e7b23e207474cc9ae9bcecee7ebfbbdb613c8f4e44d809a8bf8efd46b` |

The files are intentionally small and deterministic. Any replacement must preserve
the media contract and update both this table and every downstream integrity check.

## Generated faces

Face-driven endpoints (for example Runway `act_two`, which fails with
`NO_FACE_FOUND` on the test card) need a human face. These two depict **no real
person**: both were generated on 2026-10-05 through fal with Apache-2.0
open-weights models, whose licence places no restriction on the outputs.

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
```

The seeds and settings make the files reproducible in principle, but a hosted
model may change underneath, so a regeneration is not guaranteed to be
bit-identical. `SHA256SUMS` is the contract, not the recipe. Use these files only
to exercise an endpoint's happy path, never as the subject of a test that expects
unsafe content.

## License

The fixtures and repository documentation are released under [CC0-1.0](LICENSE)
so public CI transports may redistribute them unambiguously.
