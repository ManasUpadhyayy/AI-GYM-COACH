# 🏋️‍♂️ AI Real-time Gym Coach

Real-time pose detection with proactive AI voice coaching — built with Streamlit, MediaPipe, and Groq.

The app watches you through your webcam, counts your reps and sets live, checks your form (joint angles, alignment, balance), and gives you spoken feedback from an AI coach as you train.

## Features

- 📷 **Live pose tracking** via webcam using MediaPipe's Pose Landmarker
- 🔢 **Automatic rep & set counting** per exercise
- 📐 **Form analysis** — joint angles, back/torso alignment, balance and swing detection, tailored per exercise
- 🗣️ **AI voice coaching** — real-time spoken feedback powered by an LLM (Groq) + text-to-speech
- 📊 **Workout history** — past sessions logged and aggregated by exercise and date
- 🔐 **Login wall** — simple per-user session handling

### Supported exercises
- Squats
- Push-ups
- Biceps Curls (Dumbbell)
- Shoulder Press
- Lunges

## Tech stack

| Layer | Tool |
|---|---|
| UI / app framework | [Streamlit](https://streamlit.io) |
| Webcam streaming | [streamlit-webrtc](https://github.com/whitphx/streamlit-webrtc) |
| Pose detection | [MediaPipe Tasks — Pose Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker) |
| Computer vision | OpenCV (headless) |
| AI coaching / LLM | [Groq](https://groq.com) |
| Text-to-speech | gTTS |
| Data | SQLite, pandas |

## Project structure

```
gym_coach/
├── main.py                       # App entry point
├── requirements.txt              # Python dependencies
├── packages.txt                  # System-level deps (for Streamlit Cloud)
├── .env.example                  # Template for required env vars
├── static/
│   ├── style.css
│   └── AdobeClean.otf
├── ml_models/                    # NOT in git — see Setup below
│   └── pose_landmarker_full.task
└── services/
    ├── auth/
    │   └── login_wall.py
    ├── state/
    │   └── session_defaults.py
    ├── config/
    │   └── workout_config.py
    ├── ui/
    │   └── style_loader.py
    ├── persistence/
    │   └── exercise_repository.py
    ├── vision/
    │   └── exercise_video_processor.py
    ├── tracking/
    │   └── metrics.py
    └── coaching/
        ├── llm.py
        ├── tts.py
        └── voice_pipeline.py
```

## Prerequisites

- Python 3.10+
- A webcam
- A free [Groq API key](https://console.groq.com/keys)

## Setup

### 1. Clone and install dependencies

```bash
git clone <your-repo-url>
cd gym_coach
python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt
```

System-level packages listed in `packages.txt` (`libgl1`, `libglib2.0-0t64`, `libsm6`, `libxext6`) are needed by OpenCV on Linux. On Windows they're not required; on Streamlit Community Cloud they're installed automatically from this file.

### 2. Set your API key

Copy the example env file and add your real key:

```bash
copy .env.example .env        # Windows
```

Edit `.env`:
```
GROQ_API_KEY=your_actual_key_here
```

### 3. Download the pose detection model

The MediaPipe model file is not checked into git (it's a large binary). Download it into `ml_models/`:

```powershell
mkdir ml_models -Force
curl.exe -L -o ml_models\pose_landmarker_full.task https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_full/float16/1/pose_landmarker_full.task
```

Confirm it downloaded correctly — the file should be several MB, not a few KB.

### 4. Run locally

```bash
streamlit run main.py
```

Open the sidebar, set your exercise/sets/reps, and click **Start Workout**.

## Deploying to Streamlit Community Cloud

1. Push this repo to GitHub (`.env`, `data.db`, and `ml_models/` will be skipped automatically thanks to `.gitignore`).
2. Go to [share.streamlit.io](https://share.streamlit.io) and connect your repo, with `main.py` as the entry point.
3. In your app's **Settings → Secrets**, add:
   ```toml
   GROQ_API_KEY = "your_actual_key_here"
   ```
4. Since `ml_models/` isn't in the repo, add a startup step (e.g. in `main.py` or a setup script) that downloads `pose_landmarker_full.task` from the URL above if it isn't already present on disk.

## Troubleshooting

- **`RuntimeError: Unable to open file at ...pose_landmarker_full.task`** — the model file is missing; see Setup step 3.
- **"Voice pipeline failed to init" / no spoken feedback** — check that `GROQ_API_KEY` is set correctly in `.env` (and that `load_dotenv()` is being called) or in Streamlit Secrets if deployed.
- **Camera doesn't start** — make sure your browser has webcam permission for the app's URL.

