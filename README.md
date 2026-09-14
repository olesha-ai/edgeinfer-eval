# EdgeInfer

**build 1.0.0** · Windows x64 · Nim · ONNX Runtime + OpenVINO · ordinary USB camera

I built this for real-time vision on a normal PC (no fancy GPU required).  
This drop is an **evaluation build**: testing, students, research. **Not** a commercial license. Source stays closed; you get the compiled package.

**Download (Windows x64 zip):** [Releases — 1.0.0](https://github.com/olesha-ai/edgeinfer-eval/releases/latest)

### What’s new in 1.0.0 (same repo — not a second project)

ImGui UI · sticky preview boxes · Out TCP + E1 + `trig.exe` in the zip · slim OpenVINO package · no white flash on start.  
Full notes: paste from `RELEASE_NOTES_1.0.0.md` into the GitHub Release body (or see the Release page).

![demo](./video.gif)

My desk setup — Core **i5-11400**, cheap rolling-shutter webcam, OpenVINO on CPU:

![probe](./proc.jpg)

![infer](./infer.jpg)

On this machine ROI sits around **~6–7 ms** (~130–170 fps class of infer). Camera is ~30 fps, so the bottleneck is the picture, not the net.

---

## Why I’m putting it up

I want numbers from **other CPUs**. I only have the 11400 here.

If you try it, open an Issue and paste:

- your CPU name  
- a few console lines like `[infer] ~6.0 ms … fps=168 (roi)`  
- camera model if you know it  

I tested with a **normal** webcam. Motion blur on fast objects is ugly. If someone has a **global shutter** USB cam and can run the same build — please tell me what you see. I think boxes will look cleaner; I’d like proof from the field.

Profile: [olesha-ai](https://github.com/olesha-ai)

---

## What it does (short)

USB camera (Media Foundation, NV12) → convert thread (`libyuv`) for preview → infer thread (letterbox / ROI → ORT + OpenVINO → boxes on UI).

Two tensor slots (example: 416 full + 256 ROI) so we don’t resize the model every frame. UI is ImGui; START / STOP are separate buttons; Space toggles. Settings lock while the stream runs.

Overnight on my box (~10 h): heap flat ~3 MiB, private working set ~182 MiB, millions of infers, no watchdog restart. Your PC may differ — that’s why I want feedback.

Honest corners I won’t hide:

- Switching models on OpenVINO MULTI: we **abandon** the old session instead of a risky Release (tiny leak on rare switch; no crash). Lines don’t hot-swap models anyway.  
- Exit uses `ExitProcess` after a clean shutdown — Windows + Nim/NiGui finalizers otherwise blow up. Fine for an exe.  
- `settings.json` is not encrypted. Advanced is for people who know what they’re doing, not DRM.

---

## Package layout

| | |
|--|--|
| `EdgeInfer.exe` | the app |
| `trig.exe` | optional LAN client (Out TCP / Find / E1) — same zip |
| `trig.json` | trig defaults (host/port/auth); edit or use UI |
| `lib/` | ORT + OpenVINO DLLs |
| `model/` | ONNX (YOLOX line) |
| `settings.json` | created/updated next to the exe |
| `licenses/` | **keep this** — OpenVINO / ORT / YOLOX / … texts |
| `LICENSE.md` | my eval license |
| `logs/` | only if you turn Memory diag on |

### settings.json

Lives next to the exe. Camera, model path, conf/iou, ORT/OV knobs, tensor sizes, etc.  
Missing file → defaults; Apply / exit can rewrite it. Better to change via UI (`S` when idle). Access Key for Advanced is **not** saved in the file.

### logs

**Off by default.** Console still prints `[infer] …` — enough for a quick speed check.  
For disk logs: Settings → Access Key → Advanced → **Memory diag** → Apply. Then `logs/run.log` and `logs/mem.log`.

---

## Run it

0. Get the zip from [Releases](https://github.com/olesha-ai/edgeinfer-eval/releases/latest).  
1. Unpack somewhere local (not a sync folder if you can help it).  
2. `lib/openvino/` and `model/*.onnx` must sit with the exe.  
3. Plug USB cam, start `EdgeInfer.exe` **from cmd** if you want to see timings.  
4. Wait for Camera / Model / Ready lamps.  
5. START or Space. STOP or Space again. Esc asks before quit.

Keys: Space start/stop · S settings (idle) · Esc exit · arrows conf/iou.

Don’t hammer Space during warmup. Give START a second before STOP.

Needs Win10/11 x64 and a UVC camera MF can see. Win N may need Media Feature Pack.

---

## License

See [LICENSE.md](LICENSE.md).

Allowed: test, teach, research, send me feedback.  
Not allowed under this paper: commercial / factory production use.

This is **not** a commercial license. If you ever need that, we talk separately.

---

## Third-party (important)

I did **not** invent OpenVINO or ONNX Runtime. EdgeInfer is my glue: capture, dual-slot pipeline, UI, memory discipline.

| | | |
|--|--|--|
| Intel OpenVINO | their runtime (Apache 2.0) | [openvino](https://github.com/openvinotoolkit/openvino) |
| ONNX Runtime | MIT | [onnxruntime](https://github.com/microsoft/onnxruntime) |
| YOLOX weights | upstream / Megvii line | [YOLOX](https://github.com/Megvii-BaseDetection/YOLOX) |
| Nim, ImGui, GLFW, libyuv | as usual | see `licenses/` |

Not affiliated with Intel, Microsoft, or Megvii. Names are trademarks of their owners — used only to say what we link against. Full texts are in **`licenses/`** (don’t strip that folder from the zip).

---

olesha-ai · eval only · tell me your CPU numbers
