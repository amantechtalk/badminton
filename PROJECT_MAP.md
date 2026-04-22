# Project Map: Badminton AI Analysis System

This file provides a structured mapping of the entire codebase, detailing the purpose of each package and file, along with their primary dependencies.

---

## 1. Core Logic & Orchestration (`com.example.badminton`)

| File | Function | Dependencies |
| :--- | :--- | :--- |
| **`MainActivity.java`** | App entry point, handles navigation drawer, permissions, and fragment switching. | `androidx.navigation`, `MainViewModel` |
| **`MainViewModel.java`** | Central state manager. Fuses IMU and Video data, manages BLE connection state, and AI results. | `DeviceManager`, `ShotClassifier`, `ShuttlecockDetector` |
| **`PoseTestActivity.java`** | Diagnostic tool for testing Pose and Shuttlecock detection on static high-res images. | `PoseDetector`, `ShuttlecockDetector` |
| **`ActionTrailView.java`** | Custom view for rendering 3D-projected racket movement paths. | `android.graphics.Path` |

---

## 2. BLE Telemetry (`com.example.badminton.ble`)

| File | Function | Dependencies |
| :--- | :--- | :--- |
| **`DeviceManager.java`** | Primary BLE handler. Manages connection, MTU requests, and data notification callbacks. | `no.nordicsemi.android:ble` |
| **`NordicBleManager.java`** | Wrapper for Nordic's BLE library to simplify service discovery. | `no.nordicsemi.android:ble` |
| **`BadmintonBLEManager.java`** | Specific implementation of BLE logic for the Badminton sensor hardware. | `DeviceManager` |
| **`BLEForegroundService.java`** | Keeps the BLE connection alive while the app is in the background or recording. | `android.app.Service` |

---

## 3. Machine Learning & AI (`com.example.badminton.ml`)

| File | Function | Dependencies |
| :--- | :--- | :--- |
| **`PoseDetector.java`** | Body tracking using MediaPipe. Extracts 33 skeletal landmarks. | `com.google.mediapipe:tasks-vision` |
| **`ShuttlecockDetector.java`** | Object detection using YOLOv8 to find and track the shuttlecock. | `org.tensorflow:tensorflow-lite` |
| **`ShotClassifier.java`** | Classifies IMU bursts into shot types (Smash, Clear, etc.) using CNN. | `org.tensorflow:tensorflow-lite` |

---

## 4. Signal Processing (`com.example.badminton.processing`)

| File | Function | Dependencies |
| :--- | :--- | :--- |
| **`SignalProcessor.java`** | Physics engine for calculating Power (N), Speed (km/h), and Torque. | `org.apache.commons:math3` |

---

## 5. User Interface & AR (`com.example.badminton.ui`)

### Core Vision & AR
- **`VisionFragment.java`**: Orchestrates CameraX feed and real-time AI inference.
- **`VisionOverlayView.java`**: Renders the AR skeleton, shuttlecock box, and trajectory trail.
- **`PoseFusionFragment.java`**: Unified view for Body Pose and Racket Sensor fusion.

### Analysis & Dashboard
- **`AnalyticsHubFragment.java`**: Main dashboard for all AI-driven insights.
- **`ExploreAdapter.java`**: Recycler adapter for the Analytics Hub menu.
- **`PowerAnalysisFragment.java`**: Displays impact force metrics using `GaugeView`.
- **`SpeedAnalysisFragment.java`**: Visualizes racket tip and shuttlecock speeds.
- **`InjuryAnalysisFragment.java`**: Analyzes movement "jerk" to provide safety scores.
- **`HeatmapFragment.java`**: Maps strike locations onto a court diagram.
- **`NarrativeCoachFragment.java`**: Generates natural language feedback based on AI data.

### Tracking & Performance
- **`StaminaFragment.java`**: Monitors intensity and calculates energy expenditure.
- **`AccuracyFragment.java`**: Tracks how well the athlete hits the "sweet spot."
- **`ConsistencyFragment.java`**: Measures the repeatability of different shot types.
- **`ActionTrailFragment.java`**: 3D visualization of the racket swing arc.

### Management & History
- **`HistoryFragment.java`**: Lists past sessions using `SwingDao`.
- **`DeviceScanDialogFragment.java`**: Popup for scanning and connecting to ESP32 sensors.
- **`DeviceScanAdapter.java`**: Lists available Bluetooth devices.

---

## 6. Data Persistence (`com.example.badminton.db`)

| File | Function | Dependencies |
| :--- | :--- | :--- |
| **`AppDatabase.java`** | Room database definition. | `androidx.room` |
| **`SwingDao.java`** | Interface for SQL operations (Insert, Query, Delete). | `androidx.room` |
| **`SwingSession.java`** | Entity class representing a single stored swing or session. | `androidx.room` |

---

## 7. Data Models (`com.example.badminton.model`)

| File | Function | Dependencies |
| :--- | :--- | :--- |
| **`SwingData.java`** | Data class for an active swing result (Power, Speed, Type). | `java.io.Serializable` |
| **`ExploreItem.java`** | Simple model for dashboard menu items. | `None` |
