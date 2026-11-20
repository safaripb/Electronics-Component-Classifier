# Electronic Component Classifier

[**Try the live dashboard**](https://componentbox-classifier-demo.parnia-yazdinia.chatgpt.site): Explore six sample scans, review component labels, and add parts to inventory in your browser.

**An ESP32-CAM powered electronic component recognition and inventory platform.**

Electronic Component Classifier helps electronics students, makers, and lab teams turn loose component bins into a searchable inventory. An ESP32-CAM or dashboard upload captures a component image, a FastAPI backend runs a MobileNetV2-style classifier, and the dashboard lets a human review the result, correct labels, and grow the training dataset over time.


<img width="1280" height="571" alt="image" src="https://github.com/user-attachments/assets/025ce4cb-0092-48e0-a9c0-c77c35b05310" />
<img width="1280" height="423" alt="image" src="https://github.com/user-attachments/assets/196fb201-3bd4-4766-bd9e-eb5d6666e860" />
<img width="1280" height="580" alt="image" src="https://github.com/user-attachments/assets/630e3a07-d0fd-4285-a699-8622750efafb" />


## Supported Components

The current classifier workflow supports one component per image across five classes:

- resistor
- capacitor
- jumper wire
- stepper motor
- 7-segment display

Low-confidence or unsupported images are returned as `unknown` so they can be reviewed instead of silently entering inventory with a bad label.

##WHy is this useful?

Small electronics collections become hard to manage quickly: parts get mixed, labels disappear, and project teams lose time checking whether a component is available. Electronic Component Classifier is a practical prototype for scanning parts at the bench, keeping recent captures visible, and building a better dataset from real corrections.

## Tech Stack

- **Firmware:** ESP32-CAM, Arduino C++, `WiFi`, `WebServer`, `HTTPClient`
- **Backend:** Python, FastAPI, Pydantic, OpenCV, NumPy, pytest
- **ML:** TensorFlow/Keras, MobileNetV2 transfer learning
- **Dashboard:** AngularJS 1.8, vanilla CSS, static dev server
- **Testing:** pytest, FastAPI test client

## Repository Layout

```text
componentbox/
|-- assets/                  Project images and demo media
|-- backend/                 FastAPI app, services, scripts, tests
|-- dashboard/               AngularJS scan review dashboard
|-- dataset/                 Class folders for training images
|-- docs/                    Architecture, setup, API, classifier, testing docs
|-- firmware/esp32_cam_capture/
|   |-- esp32_cam_capture.ino
|   `-- secrets.example.h
|-- mock_device/             Local image uploader that simulates the ESP32-CAM
`-- sample_data/             Example data and report placeholders
```

## Setup

### 1. Backend

```powershell
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

The API defaults to `http://localhost:8000`.

Useful environment variables:

```powershell
set RESISTOR_IMAGE_RATE_LIMIT=12
set COMPONENT_CLASSIFIER_MODEL=models/component_classifier.keras
set COMPONENT_CLASSIFIER_CONFIDENCE=0.75
set COMPONENT_SCAN_IMAGE_DIR=data/scans
set COMPONENT_SCAN_METADATA=data/scans.json
set COMPONENT_DATASET_DIR=../dataset
```

### 2. Dashboard

```powershell
cd dashboard
npm install
npm start
```

Open `http://localhost:5173`.

The dashboard uses `http://localhost:8000` by default. To point it at another backend, set `window.COMPONENTBOX_API_URL` before `src/app.js` loads, or set `COMPONENTBOX_API_URL` in browser local storage.

### 3. ESP32-CAM Firmware

Copy the example secrets file and fill in local values:

```powershell
copy firmware\esp32_cam_capture\secrets.example.h firmware\esp32_cam_capture\secrets.h
```

Edit `secrets.h`:

```cpp
const char* WIFI_SSID = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
const char* COMPONENTBOX_BACKEND_SCAN_URL = "http://YOUR_BACKEND_HOST:8000/api/component-scans";
```

Then upload `firmware/esp32_cam_capture/esp32_cam_capture.ino` from the Arduino IDE using the AI Thinker ESP32-CAM board profile.

### 4. Mock Upload Without Hardware

```powershell
python mock_device/send_fake_component_image.py assets/sample_images/esp32.jpg
```

Use `--api-url` to target a non-local backend.

## Model Training

Collect images in class folders:

```text
dataset/
|-- resistor/
|-- capacitor/
|-- wire/
|-- stepper_motor/
`-- seven_segment/
```

Install training dependencies and train:

```powershell
cd backend
pip install -r requirements-training.txt
python scripts/train_component_classifier.py --dataset ../dataset --output models/component_classifier.keras
```

The training script uses MobileNetV2 transfer learning with a validation split. The repository should not claim a production accuracy number until it is measured on a held-out dataset captured with the same ESP32-CAM setup.

## Human Correction Loop

1. Capture or upload a component image.
2. Backend returns a recommended class, confidence, or `unknown`.
3. Dashboard displays the scan in the review queue.
4. A teammate confirms or corrects the label.
5. The backend can copy the saved image into the matching dataset folder.
6. Future model training uses the corrected examples.

This keeps the project honest: uncertain predictions become useful training data instead of hidden inventory mistakes.

See [docs/api.md](docs/api.md) for request and response examples.

## Testing

```powershell
cd backend
pytest
```

The public release keeps tests focused on backend contracts and deterministic helpers. Manual hardware validation is still required for camera focus, lighting, network setup, and real-world classification quality.

