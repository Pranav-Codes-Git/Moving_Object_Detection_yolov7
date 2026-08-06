# Moving Object Detection (YOLOv7) — Setup Guide

## What this does
- Backend (`backend/app.py`): Flask server that loads pretrained YOLOv7
  (`yolov7.pt`, auto-downloaded on first run via `torch.hub`) and detects
  **person, car, motorcycle, bus, truck**. Each detection is also tagged
  **moving** (green box) or **static** (orange box) using a background-
  subtraction motion mask.
- Frontend (`frontend/`): React app with two modes — live webcam detection
  and video-file upload — both rendered as an MJPEG stream in the browser.

## 1. Backend setup
```bash
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
python app.py
```
This starts the Flask server on `http://localhost:5000`. The first run
will download `yolov7.pt` (~75MB) automatically — needs an internet
connection once.

> No GPU? It still works on CPU, just slower per frame. For a smoother demo,
> lower the webcam resolution or use a short uploaded clip.

## 2. Frontend setup
```bash
cd frontend
npm install
npm start
```
Opens `http://localhost:3000`, which talks to the Flask backend at
`http://localhost:5000` (see `BACKEND_URL` in `src/App.jsx` if you deploy
the backend elsewhere).

## 3. Using it
- **Start Webcam Detection**: opens your machine's default camera (index 0)
  and streams live annotated frames.
- **Upload Video**: pick any `.mp4`/`.avi` file; the backend saves it,
  processes it frame-by-frame, and streams the annotated result back.

## Notes for your report / viva
- Model: YOLOv7 (WongKinYiu implementation), pretrained on COCO, loaded via
  `torch.hub.load("WongKinYiu/yolov7", "custom", "yolov7.pt")`.
- Class filtering: only COCO ids `0` (person), `2` (car), `3` (motorcycle),
  `5` (bus), `7` (truck) are drawn — everything else is ignored.
- "Moving" vs "static" classification: `cv2.createBackgroundSubtractorMOG2`
  builds a foreground mask per frame; a detection is called "moving" if
  more than ~15% of pixels inside its bounding box are foreground.
- Streaming: uses HTTP **multipart/x-mixed-replace** (MJPEG) — simplest way
  to push a live annotated feed to a plain `<img>` tag without WebSockets.
