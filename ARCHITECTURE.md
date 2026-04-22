# Badminton AI Analysis System - Technical Architecture

This document provides a comprehensive technical breakdown of the Badminton AI Analysis application, reflecting the latest synchronized EKF, YOLOv8n, and MediaPipe Pose implementations.

---

## 1. System Design Philosophy

The application is built on a **Modular MVVM (Model-View-ViewModel)** architecture. This design ensures that high-frequency telemetry processing (100Hz IMU) and heavy computer vision tasks (60FPS CameraX) do not block the UI thread. Extensive debug logging is integrated using `_DEBUG` TAGs for real-time observability.

### Core Architecture Layers:
1.  **UI Layer (Fragments/Activities):** Purely responsible for displaying data and capturing user input. It observes state changes via `LiveData`.
2.  **Domain/ViewModel Layer:** The `MainViewModel` acts as the orchestrator. It manages session state, telemetry buffering, and temporal synchronization.
3.  **Service/Data Layer:**
    *   **BLE Engine (`DeviceManager.java`):** Encapsulates hardware communication using Nordic BLE library.
    *   **ML Engine (`ShotClassifier.java`):** Handles TFLite inference for stroke classification.
    *   **Vision Engine (`ShuttlecockDetector.java`):** Handles YOLOv8n object detection for shuttlecocks.
    *   **Pose Engine (`PoseDetector.java`):** Handles MediaPipe Pose Landmarker for skeleton extraction.
    *   **Persistence:** Room database for swing session history.

---

## 2. Debugging & Observability

The system implements a standardized logging strategy across all core modules. Use Logcat with the `_DEBUG` filter to monitor the system state.

| Tag | Purpose | Example Message |
| :--- | :--- | :--- |
| `MainViewModel_DEBUG` | Sync, Buffering, Result Ingress | "Pose EKF Sync Success! Jitter: 42ms" |
| `DeviceManager_DEBUG` | BLE Lifecycle, Scanning, GATT | "Scan Discovered Unnamed [AA:BB:CC] RSSI: -65" |
| `POSE_FLOW` | MediaPipe Pipeline, Landmarks | "MATCH! AI detected athlete skeleton. Landmarks size: 33" |
| `POSE_RENDER` | Overlay Rendering, Redraws | "VISION ENGINE ACTIVE" |
| `ShotClassifier_DEBUG` | IMU Inference, ML Fallbacks | "Classification Result: Smash (Confidence: 0.88)" |
| `ShuttlecockDetector_DEBUG` | YOLOv8n Inference Logs | "parseYOLOv8Output: 5 candidates found." |

---

## 3. Detailed Data Flows

### A. Telemetry Pipeline (IMU)
1.  **Ingress:** Notifications arrive -> `bleBuffer`. (Logged in `DeviceManager_DEBUG`)
2.  **Windowing:** Once 2560 bytes (128 samples) are accumulated, `processSwingData()` triggers. (Logged in `MainViewModel_DEBUG`)
3.  **Physics:** Calculates peak G-force (Power) and angular velocity magnitude (Speed).
4.  **Classification:** 128x6 tensor is passed to TFLite. (Logged in `ShotClassifier_DEBUG`)

### B. Vision Pipeline (Shuttle & Pose Tracking)
1.  **Capture:** CameraX `ImageAnalysis` -> `toBitmap()` with a **180-degree rotation** fix.
2.  **Pose Inference:** MediaPipe `PoseLandmarker` identifies 33 landmarks. Optimized with a `0.3f` confidence threshold for distant subjects.
3.  **Shuttle Inference:** YOLOv8n identifies bounding box. Results are transformed using $(1-x, 1-y)$ to align with the rotated frame.
4.  **AR Overlay**: `VisionOverlayView` renders the skeleton, a yellow square (Shuttle), and a 30-frame path trail (Trajectory).

---

## 4. Pose EKF & Temporal Sync
*   **The Problem:** IMU data arrives at 100Hz in bursts, while Video arrives at 60Hz continuously.
*   **The Solution:** `MainViewModel` maintains a `videoFrames` queue. When an IMU impact is detected, the system searches the queue for the video frame closest to the impact time.
*   **Extended Kalman Filter:** Fuses "Local Orientation" (IMU) with "Global Position" (Vision). Success/failure is tracked via `MainViewModel_DEBUG`.

---

## 5. Distant Subject Tracking (Far-Court AI)
*   **Threshold Sensitivity**: Detection confidence lowered to `0.3f`.
*   **Static Analysis**: `PoseTestActivity` allows developers to verify AI performance on high-resolution static images to ensure distant players are correctly mapped.
*   **Adaptive Rendering**: Overlay stroke widths and font sizes scale relative to the image resolution to maintain legibility.

---

## 6. Technology Stack

| Component | Technology |
| :--- | :--- |
| **BLE** | Nordic BLE Android Library (v2.7.1) |
| **Vision** | Jetpack CameraX + YOLOv8n (TFLite) |
| **Pose** | MediaPipe Tasks Vision (Pose Landmarker) |
| **ML** | TensorFlow Lite Support (v0.4.4) |
| **Charts** | MPAndroidChart (v3.1.0) |
| **Architecture** | MVVM with LiveData & ViewBinding |
