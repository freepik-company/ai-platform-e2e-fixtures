# AI Platform E2E fixtures

Public, synthetic media fixtures for Freepik AI Platform sandbox end-to-end tests.
They contain no customer data, credentials, proprietary model output, or production
content. Keep filenames stable because CI consumers address them directly.

| File | Purpose | SHA-256 |
| --- | --- | --- |
| `image.png` | RGB image input | `e04729cc34c1227b47b78b6260fb04350c8e0ec9ea79e1197491bcc2993a9f3c` |
| `image.jpg` | JPEG image input | `e4ff86eabe7bd3a92442aac2999fc5a01de332f2ea33ab0d03042473984d4ba9` |
| `mask.png` | Grayscale binary mask | `496c90e4057a50adf49ee60b937c1b1c8d527e1ef12e7a874546ec46c44ff2c0` |
| `audio.mp3` | Three-second 440 Hz audio input | `6b8586fe1982e18d50521226583a5520f11c87044d7873992ea445b50104411f` |
| `video.mp4` | Three-second H.264/AAC video input | `2ffd8d2791d6349984f95ec54f3c5471de11ec490c52220280f71596fe034cca` |

The files are intentionally small and deterministic. Any replacement must preserve
the media contract and update both this table and every downstream integrity check.
