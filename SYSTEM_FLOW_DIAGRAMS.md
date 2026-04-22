# Badminton AI System: Logic & Algorithm Flow Diagrams

This document visualizes the internal logic, algorithmic steps, and dependency chains of the Badminton AI Analysis system using Mermaid diagrams.

---

## 1. High-Level System Architecture
The system operates as a synchronized dual-stream processor: one for high-frequency IMU telemetry and one for real-time Computer Vision.

```mermaid
graph TD
    A[Sensors] --> B[CameraX 60FPS]
    A --> C[BLE IMU 100Hz]
    
    subgraph "Vision Pipeline"
        B --> B1[Bitmap Rotation 180°]
        B1 --> B2[MediaPipe Pose]
        B1 --> B3[YOLOv8n Shuttle]
        B2 --> B4[Skeleton Extraction]
        B3 --> B5[NMS + Coord Flip]
    end
    
    subgraph "Telemetry Pipeline"
        C --> C1[BLE Buffer 2560 bytes]
        C1 --> C2[Impact Peak Detection]
        C2 --> C3[TFLite Shot Classifier]
        C2 --> C4[Physics Engine: Power/Speed]
    end
    
    B4 & B5 & C3 & C4 --> D[Pose EKF Fusion]
    D --> E[AR Overlay / UI Dashboard]
```

---

## 2. Vision Pipeline Algorithm (`VisionFragment`)
This diagram detail the per-frame processing logic used for AR tracking.

```mermaid
flowchart LR
    Start(ImageProxy) --> Proxy[toBitmap]
    Proxy --> Rotate[Matrix Rotate 180°]
    
    subgraph "Parallel Inference"
        Rotate --> Pose[PoseDetector: detectLive]
        Rotate --> Shuttle[ShuttlecockDetector: detect]
    end
    
    Shuttle --> Filter{Found?}
    Filter -- Yes --> Trans[Coord Flip: 1-x, 1-y]
    Trans --> Path[Update Trajectory Path]
    
    Pose --> Callback[onPoseResult]
    
    Path & Callback --> UI[Update VisionOverlayView]
    UI --> Render[Draw Skeleton + Square + Trail]
```

---

## 3. IMU Processing & Impact Algorithm
How the system identifies a badminton shot from raw accelerometer and gyroscope data.

```mermaid
flowchart TD
    Raw[Raw IMU Packets] --> Accum[Accumulate 128 Samples]
    Accum --> Calc[Calculate Resultant G-Force]
    Calc --> Peak{G > 8.0?}
    
    Peak -- No --> Raw
    Peak -- Yes --> Extract[Extract 128x6 Window]
    
    subgraph "Feature Analysis"
        Extract --> Norm[Normalize Tensors]
        Extract --> Physics[Integrate Angular Velocity]
    end
    
    Norm --> TFLite[ShotClassifier.run]
    Physics --> Stats[Power & Speed Calc]
    
    TFLite --> Result[Shot: Smash/Clear/etc]
    Stats --> Result
```

---

## 4. Pose EKF Fusion Logic
The temporal synchronization between asynchronous video and telemetry.

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> BufferingVideo : Camera Frame Received
    BufferingVideo --> ImpactDetected : IMU Peak Found
    
    state ImpactDetected {
        [*] --> GetImpactTime
        GetImpactTime --> CalculateLatency
        CalculateLatency --> SearchFrameQueue
        SearchFrameQueue --> FoundClosestFrame
    }
    
    ImpactDetected --> Fusion : Temporal Match
    Fusion --> [*] : Update AR State
```

---

## 5. Dependency & Module Hierarchy
Mapping how different code components depend on each other.

```mermaid
graph BT
    subgraph "Hardware & Libraries"
        BLE_Lib[Nordic BLE]
        TF_Lib[TensorFlow Lite]
        MP_Lib[MediaPipe Tasks]
        CX_Lib[Jetpack CameraX]
    end

    subgraph "Core ML & Signal"
        DM[DeviceManager] --> BLE_Lib
        SC[ShotClassifier] --> TF_Lib
        PD[PoseDetector] --> MP_Lib
        SD[ShuttlecockDetector] --> TF_Lib
    end

    subgraph "State Management"
        VM[MainViewModel] --> DM
        VM --> SC
        VM --> PD
        VM --> SD
    end

    subgraph "UI Components"
        VF[VisionFragment] --> VM
        VF --> CX_Lib
        VOV[VisionOverlayView] --> VF
    end
```

---

## 6. Mathematical Algorithms

### A. Shuttlecock Speed Estimation
$$Distance = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$
$$Velocity = \frac{Distance \times ScaleFactor}{\Delta t}$$

### B. Impact Normalization (IMU)
$$InputTensor = \frac{RawValue - Mean}{StandardDeviation}$$

### C. Coordinate Mapping (180° AR Fix)
$$x_{render} = (1.0 - x_{raw}) \times ViewWidth$$
$$y_{render} = (1.0 - y_{raw}) \times ViewHeight$$
