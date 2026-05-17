# 🏋️‍♂️ AI Real-time GYM Coach

> A real-time AI-powered fitness coaching application that uses computer vision to analyze your exercise form and delivers live voice feedback — right in your browser.

---

## ✨ Features

- **🎥 Real-time Pose Detection** — Streams your webcam feed and detects body landmarks using Google MediaPipe's `pose_landmarker_full` model.
- **🤸 5 Exercise Detectors** — Supports Squats, Push-ups, Biceps Curls (Dumbbell), Shoulder Press, and Lunges with exercise-specific form analysis.
- **🤖 AI Voice Coach** — Powered by Groq's `llama-3.3-70b-versatile` LLM, the coach proactively gives spoken feedback on form issues, set completions, and workout milestones.
- **🗣️ Text-to-Speech** — Coach responses are converted to audio via Google TTS (`gTTS`) and autoplayed in the browser.
- **📊 Live Metrics Sidebar** — Displays real-time angle measurements and form status (e.g. knee angle, back angle, depth status) per exercise.
- **📅 Workout History** — Persists every session to a local SQLite database and shows an aggregated history table per user.
- **🔐 Username-based Auth** — Simple login wall that creates or retrieves a user record, scoping all history to that user.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| UI Framework | [Streamlit](https://streamlit.io/) `1.54.0` |
| Webcam Streaming | [streamlit-webrtc](https://github.com/whitphx/streamlit-webrtc) `0.64.5` |
| Pose Estimation | [MediaPipe](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker) `0.10.14` |
| Computer Vision | [OpenCV](https://opencv.org/) (headless) `4.10.0.84` |
| LLM Inference | [Groq API](https://console.groq.com/) (`llama-3.3-70b-versatile`) |
| Text-to-Speech | [gTTS](https://gtts.readthedocs.io/) `2.5.3` |
| Data Manipulation | [Pandas](https://pandas.pydata.org/) `2.2.3` |
| Database | SQLite (via `sqlite3` stdlib) |
| Env Management | [python-dotenv](https://github.com/theskumar/python-dotenv) |

---

## 📂 Project Structure

```
AI Gym Coach/
├── main.py                          # Streamlit app entry point
├── requirements.txt
├── data.db                          # SQLite database (auto-created)
├── .env                             # API keys (not committed)
│
├── core/
│   └── base_exercise.py             # Abstract base class for all exercise detectors
│
├── detectors/                       # Exercise-specific form detectors
│   ├── squat.py
│   ├── pushup.py
│   ├── biceps_curl.py
│   ├── shoulder_press.py
│   └── lunges.py
│
├── ml_models/
│   └── pose_landmarker_full.task    # MediaPipe pose landmarker model
│
├── services/
│   ├── auth/
│   │   └── login_wall.py            # Username-based session auth
│   ├── coaching/
│   │   ├── llm.py                   # Groq LLM coach wrapper
│   │   ├── tts.py                   # gTTS text-to-speech wrapper
│   │   └── voice_pipeline.py        # Event-driven coaching orchestrator
│   ├── config/
│   │   └── workout_config.py        # Exercise options, metrics fields, prompt
│   ├── persistence/
│   │   └── exercise_repository.py   # SQLite CRUD for users & exercises
│   ├── state/
│   │   └── session_defaults.py      # Streamlit session state initialization
│   ├── tracking/
│   │   └── metrics.py               # Syncs video processor metrics to session state
│   ├── ui/
│   │   └── style_loader.py          # CSS & font injection helpers
│   └── vision/
│       └── exercise_video_processor.py  # WebRTC video processor (pose + overlay)
│
└── static/
    ├── style.css                    # Custom app styles
    └── AdobeClean.otf               # Custom font
```

---

## 🚀 Getting Started

### Prerequisites

- Python `3.10+`
- A [Groq API key](https://console.groq.com/) (free tier available)
- A webcam

### 1. Clone the Repository

```bash
git clone https://github.com/ShubhamZoro/AI-GYM-COACH.git
cd AI-GYM-COACH
```

### 2. Install Dependencies

Using `uv` (recommended):

```bash
uv venv
uv pip install -r requirements.txt
```

Or using plain `pip`:

```bash
python -m venv .venv
.venv\Scripts\activate      # Windows
source .venv/bin/activate   # macOS/Linux
pip install -r requirements.txt
```

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
```

> Alternatively, set `GROQ_API_KEY` in `.streamlit/secrets.toml` for Streamlit Cloud deployments.

### 4. Run the App

```bash
uv run streamlit run main.py
# or
streamlit run main.py
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

---

## 🎮 How to Use

1. **Login** — Enter a unique username to start your session.
2. **Plan your workout** — In the sidebar, select an exercise, number of sets, and reps per set.
3. **Start Workout** — Click **Start Workout** to activate the webcam feed.
4. **Exercise** — Perform your chosen exercise in front of the camera. The skeleton overlay will appear and metrics will update live.
5. **Listen to your coach** — The AI coach will speak proactive feedback on your form, congratulate you on completed sets, and cheer you at the end.
6. **End Workout** — Click **End Workout** when done. Your session is saved to history automatically.
7. **Review history** — Scroll down on the main page to see an aggregated workout history table.

---

## 🤸 Supported Exercises & Tracked Metrics

| Exercise | Tracked Metrics |
|---|---|
| **Squats** | Knee Angle, Back Angle, Depth Status |
| **Push-ups** | Elbow Angle, Body Alignment, Hip Position |
| **Biceps Curls (Dumbbell)** | Elbow Angle, Shoulder Stability, Swing Detection |
| **Shoulder Press** | Elbow Angle, Arm Extension, Back Arch |
| **Lunges** | Front Knee Angle, Torso Angle, Balance Status |

---

## 🧠 AI Coaching Architecture

The coaching system is event-driven and rate-limited to avoid spamming:

```
WebRTC Frame → MediaPipe Pose Landmarker → Exercise Detector → Metrics
                                                                  ↓
                                                         VoicePipeline.process_event()
                                                                  ↓
                                                    Form Issue Detection (rule-based)
                                                                  ↓
                                                    LLMCoach (Groq llama-3.3-70b)
                                                                  ↓
                                                    TextToSpeech (gTTS) → Autoplay Audio
```

**Events that trigger coaching:**
- `workout_started` — Always fires on session start
- `set_completed` — Fires every time a set is completed
- `workout_completed` — Fires once when all sets are done
- `ongoing_form_check` — Fires at most every **5 seconds** when a form issue is detected
- `no_pose_detected` — Fires when the user is out of frame

---

## ☁️ Deploying to Streamlit Cloud

1. Push your code to GitHub (ensure `.env` is in `.gitignore`).
2. Go to [share.streamlit.io](https://share.streamlit.io) and connect your repo.
3. Add `GROQ_API_KEY` under **App Settings → Secrets**:
   ```toml
   GROQ_API_KEY = "your_key_here"
   ```
4. Deploy. WebRTC requires HTTPS — Streamlit Cloud provides this automatically.

> **Note:** `data.db` is ephemeral on Streamlit Cloud. For persistent storage, swap the SQLite layer with a hosted database (e.g. Supabase, PlanetScale).

---

## 📄 License

This project is open source. Feel free to fork, modify, and build upon it.

---

## 🙌 Acknowledgements

- [Google MediaPipe](https://ai.google.dev/edge/mediapipe) for the pose landmarker model
- [Groq](https://groq.com/) for blazing-fast LLM inference
- [streamlit-webrtc](https://github.com/whitphx/streamlit-webrtc) for making real-time webcam streaming in Streamlit possible
