# Badminton AI - UI Flow & Architecture Details

This document outlines the User Interface flow, the relationship between Java logic and XML layouts, and how the application is organized.

## 1. Application Architecture Overview
The app follows the **MVVM (Model-View-ViewModel)** architectural pattern:
*   **View**: Activity and Fragments (XML + Java logic).
*   **ViewModel**: `MainViewModel.java` (Shared across all fragments to maintain state).
*   **Navigation**: Android Navigation Component (`nav_graph.xml`) manages the screen transitions.

---

## 2. App Entry & Host
### **MainActivity**
*   **Java File**: `com.example.badminton.MainActivity.java`
*   **XML Layout**: `activity_main.xml`
*   **Role**: Serves as the primary container. It hosts the `NavHostFragment` and the `BottomNavigationView`.
*   **UI Elements**:
    *   `toolbar`: Displays fragment titles and "Up" navigation.
    *   `bottom_nav_view`: Main tab switcher.
    *   `nav_host_fragment`: The area where all other fragments are swapped in.

---

## 3. Primary Navigation Tabs (Bottom Nav)

### **A. Champion Dashboard (Home)**
*   **Java File**: `com.example.badminton.ui.SportFragment.java`
*   **XML Layout**: `fragment_sport.xml`
*   **Role**: The start destination. Handles sensor connection and high-level status.
*   **Triggers**:
    *   `btnConnect`: Launches `DeviceScanDialogFragment` (XML: `fragment_device_scan.xml`).
    *   `btnActionTrail`: Navigates to `ActionTrailFragment`.

### **B. Live Telemetry**
*   **Java File**: `com.example.badminton.ui.RealTimeFragment.java`
*   **XML Layout**: `fragment_real_time.xml`
*   **Role**: Real-time display of swing metrics.
*   **Components**: Uses multiple `GaugeView` (Custom UI) for Power, Speed, and Backswing.

### **C. AI Vision AR**
*   **Java File**: `com.example.badminton.ui.VisionFragment.java`
*   **XML Layout**: `fragment_vision.xml`
*   **Role**: Camera-based tracking (Pose + Shuttlecock).
*   **Custom View**: `VisionOverlayView.java` (Renders the skeleton/AR on top of the camera).

### **D. Performance Hub (Hub UI)**
*   **Java File**: `com.example.badminton.ui.AnalyticsHubFragment.java`
*   **XML Layout**: `fragment_explore.xml`
*   **Role**: A grid-based menu of various analysis tools.
*   **Logic**: Uses `ExploreAdapter.java` (XML: `item_explore_card.xml`) to link to detail fragments.

### **E. AI Innovation Lab (Hub UI)**
*   **Java File**: `com.example.badminton.ui.AiLabFragment.java`
*   **XML Layout**: `fragment_explore.xml`
*   **Role**: Menu for advanced experimental features (Fusion, 3DGS, Injury Risk).

---

## 4. Analysis & Detail Fragments
These fragments are typically called from the **Hubs** via the `ExploreAdapter` using Navigation IDs defined in `nav_graph.xml`.

| Analysis Feature | Java Fragment Class | Layout Used | Role |
| :--- | :--- | :--- | :--- |
| **Speed** | `SpeedAnalysisFragment.java` | `fragment_real_time.xml` | Focused tip-speed analysis. |
| **Power** | `PowerAnalysisFragment.java` | `fragment_real_time.xml` | Focused impact force analysis. |
| **Consistency** | `ConsistencyFragment.java` | `fragment_real_time.xml` | Stability of technique. |
| **History** | `HistoryFragment.java` | `fragment_sport.xml` | Summary of session totals. |
| **Calories** | `CaloriesFragment.java` | `fragment_calories.xml` | Energy expenditure tracking. |
| **Heatmap** | `HeatmapFragment.java` | `fragment_heatmap.xml` | Visual court impact map. |
| **Action Trail** | `ActionTrailFragment.java` | `fragment_action_trail.xml` | Kinematic path & MPAndroidChart. |
| **Injury Risk** | `InjuryAnalysisFragment.java` | `fragment_injury_analysis.xml` | Joint torque & safety analysis. |
| **Style Profile** | `StyleFragment.java` | `fragment_style.xml` | PieChart of shot types. |

---

## 5. UI Flow Diagram (Simplified)
1.  **Start**: `MainActivity` -> `SportFragment`.
2.  **Connection**: User clicks "Connect" -> `DeviceScanDialogFragment` -> Selects Sensor -> Connected.
3.  **Analysis**:
    *   Switch to **Live Telemetry** for real-time gauges.
    *   Switch to **AI Vision** for camera skeleton tracking.
    *   Navigate to **Performance Hub** -> Click "Speed" -> `SpeedAnalysisFragment`.
    *   Perform Swing -> `MainViewModel` updates -> All observing Fragments refresh UI.

---

## 6. Key UI-Code Links
*   **Custom Attributes**: `GaugeView.java` uses attributes defined in `res/values/attrs.xml` (e.g., `labelText`, `foregroundColor`).
*   **View Binding**: Fragments use `binding = FragmentNameBinding.inflate(...)` to interact with XML IDs directly (e.g., `binding.tvShotType`).
*   **Navigation IDs**: Transitions are called via `Navigation.findNavController(v).navigate(R.id.destination_id)`.
