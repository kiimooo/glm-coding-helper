# Backend Configuration

The backend can choose the fastest available runtime automatically and can be
overridden from the command line or environment variables.

## Recommended Startup

First-time setup on Windows when Python may not be installed:

```powershell
powershell -ExecutionPolicy Bypass -File scripts\bootstrap_windows.ps1 -Target auto
```

`bootstrap_windows.ps1` will:

- find an existing Python 3.12 installation if available
- install Python 3.12 with `winget` when possible
- fall back to downloading the official Python installer
- create the CPU/GPU backend virtual environment
- install backend dependencies
- check the detector weight path

First-time setup when Python is already installed:

```powershell
powershell -ExecutionPolicy Bypass -File scripts\setup_backend.ps1 -Target auto
```

CPU-only setup:

```powershell
powershell -ExecutionPolicy Bypass -File scripts\setup_backend.ps1 -Target cpu
```

GPU setup:

```powershell
powershell -ExecutionPolicy Bypass -File scripts\setup_backend.ps1 -Target gpu
```

The setup script creates local virtual environments:

- `.venv_paddle` for CPU inference
- `.venv_paddle_gpu` for GPU inference

The GPU requirements include CUDA runtime wheels used by Paddle, so users do
not need to manually install the CUDA toolkit for the default Windows setup.
They still need an NVIDIA driver that supports the GPU runtime.

GUI mode:

```powershell
python scripts\tools\start_backend.py --mode auto
```

Windows wrapper:

```powershell
powershell -ExecutionPolicy Bypass -File scripts\start_backend.ps1 -Mode auto
```

Headless mode:

```powershell
python scripts\tools\start_backend.py --headless --mode auto
```

Force GPU:

```powershell
python scripts\tools\start_backend.py --mode gpu
```

Force CPU with explicit worker count:

```powershell
python scripts\tools\start_backend.py --mode cpu --cpu-workers 3
```

## Automatic Defaults

- `--mode auto` probes `.venv_paddle_gpu` and selects GPU when Paddle can see a
  CUDA device.
- If GPU is unavailable, the backend falls back to `cpu_parallel`.
- CPU workers default to:
  - `1` worker on 1-2 CPU cores
  - `2` workers on 3-6 CPU cores
  - `3` workers on 7+ CPU cores
- Prompt-constrained OCR decoding is enabled by default on both CPU and GPU.

## Setup Script

Python entry:

```powershell
python scripts\setup_backend.py --target auto
```

Targets:

| Target | Behavior |
| --- | --- |
| `auto` | Uses `nvidia-smi` to choose GPU when an NVIDIA GPU is visible, otherwise CPU |
| `cpu` | Creates `.venv_paddle` and installs CPU dependencies |
| `gpu` | Creates `.venv_paddle_gpu` and installs GPU dependencies |
| `both` | Creates both environments |

Useful options:

```powershell
python scripts\setup_backend.py --target cpu --recreate
python scripts\setup_backend.py --target gpu --no-smoke-test
python scripts\setup_backend.py --target cpu --pip-arg -i --pip-arg https://pypi.tuna.tsinghua.edu.cn/simple
```

The setup script also checks for the YOLO detector weight:

```text
models/weights/yolo-captcha-detector.pt
```

## Command-Line Options

| Option | Environment variable | Default |
| --- | --- | --- |
| `--host` | `CNCAPTCHA_HOST` | `0.0.0.0` |
| `--port` | `CNCAPTCHA_PORT` | `8888` |
| `--mode auto/gpu/cpu` | `CNCAPTCHA_OCR_MODE` | `auto` |
| `--cpu-workers N` | `CNCAPTCHA_CPU_OCR_WORKERS` | auto |
| `--yolo-device DEVICE` | `CNCAPTCHA_YOLO_DEVICE` | `0` on GPU, `cpu` on CPU |
| `--yolo-imgsz N` | `CNCAPTCHA_YOLO_IMGSZ` | `448` |
| `--cpu-model MODEL` | `CNCAPTCHA_CPU_OCR_MODEL` | `hybrid` |
| `--gpu-model MODEL` | `CNCAPTCHA_GPU_OCR_MODEL` | `PP-OCRv5_server_rec` |
| `--no-constrained` | `CNCAPTCHA_OCR_CONSTRAINED=0` | constrained enabled |

Additional advanced variables:

```powershell
$env:CNCAPTCHA_CPU_OCR_FAST_MODEL='PP-OCRv5_mobile_rec'
$env:CNCAPTCHA_CPU_OCR_FALLBACK_MODEL='PP-OCRv5_server_rec'
$env:CNCAPTCHA_GPU_OCR_DEVICE='gpu:0'
$env:CNCAPTCHA_SKIP_GPU_DETECT='1'
```

## Health Check

The server exposes the resolved configuration:

```powershell
Invoke-RestMethod http://127.0.0.1:8888/health
```

The response includes `backend.ocr_mode`, `backend.cpu_workers`,
`backend.gpu_available`, and the selected YOLO/OCR settings.

## Browser Capture Tips

The backend captures the visible browser window when the userscript sends a
captcha request. Chrome is not required. Current browser matching covers
Chrome, Edge, Firefox, Brave, Opera, `bigmodel`, `Z.ai`, `GLM`, and Chinese
Zhipu page titles.

The released userscript does not need a browser-specific change for this
backend update. Keep the backend GUI dropdown on `Auto` for normal use. If more
than one browser window is open and the backend captures the wrong one, choose
the exact window from the `Browser window` dropdown and click `Refresh` after
opening or switching browser windows.

If the backend says it cannot find the browser window:

- Keep the backend GUI dropdown on `Auto` unless there are multiple similar
  browser windows.
- If needed, use the backend GUI `Browser window` dropdown to select the exact
  browser window. Click `Refresh` after opening or switching browser windows.
- Keep the GLM Coding page visible and not minimized.
- Use a normal browser window instead of a PWA/app window.
- Make sure the title contains GLM, Z.ai, bigmodel, or Zhipu. Opening the
  normal `bigmodel.cn/glm-coding` page is preferred.
- Run this diagnostic command:

```powershell
python scripts\monitor\window_helper.py
```

If `window_helper.py` can capture the browser but the backend cannot, update to
the latest script/backend files and restart the backend.

If the backend says it did not get a qualified captcha image:

- Put the browser and captcha dialog on the same monitor as the backend session.
- Avoid hiding the captcha dialog behind the backend GUI or another window.
- Try a medium browser size first, for example 1280x800 to 1920x1080.
- If Windows display scaling is very high, try 100%, 125%, or 150%.
- Keep browser zoom at 50% or above. 67%, 75%, 90%, and 100% are safer choices.
- If the monitor is very large or high-DPI, avoid a tiny browser window; make
  the captcha image at least about 180 px wide and 150 px tall on screen.
- Keep the whole captcha dialog visible, including the prompt line, image, and
  confirm button.

The GUI checkboxes that previously appeared in the backend window were removed
because they did not control the released userscript flow.
