# The Ultimate "Donkey-Level" Guide to the Badminton AI App

Hello! If you are a donkey (or just a human who likes very simple explanations), this guide is for you. We are going to walk through every single part of this app's "body" and "brain."

---

## 1. THE BIG BOSS (Navigation & Control)

### `MainActivity.java`
- **What it is:** The front door of the house.
- **What it does:** When you open the app, this file sets up the side menu (the "Drawer") and makes sure you can switch between different rooms (Fragments) without getting lost. It also asks you for permission to use the camera.

### `MainViewModel.java`
- **What it is:** The app's Short-Term Memory.
- **What it does:** It holds data while you move from screen to screen. If the Bluetooth hears a swing, the ViewModel remembers it so that the History screen can show it later. It is the "glue" that holds everything together.

---

## 2. THE EARS (Bluetooth & Communication)
This package is called `com.example.badminton.ble`. It’s how the phone talks to the racket.

### `DeviceManager.java`
- **What it is:** The Bluetooth Radio Operator.
- **What it does:** It scans for the racket, connects to it, and waits for the racket to "shout" data. When data arrives, it logs it so we can see the raw bytes.

### `BadmintonBLEManager.java` & `NordicBleManager.java`
- **What they are:** Assistants to the Radio Operator.
- **What they do:** They handle the boring technical stuff of Bluetooth (like handshakes and error checking) so the `DeviceManager` can focus on the data.

### `BLEForegroundService.java`
- **What it is:** The "Stay Awake" pill.
- **What it does:** It makes sure the Bluetooth stays connected even if you put the phone in your pocket or switch to another app. It shows that little icon in your notification bar.

---

## 3. THE EYES (Computer Vision & AI)
This package is called `com.example.badminton.ml`. It uses Artificial Intelligence to "see."

### `PoseDetector.java`
- **What it is:** The Skeleton Mapper.
- **What it does:** It looks at the video and finds 33 points on your body (like your nose, elbows, and knees). It turns your body into a digital stick figure.

### `ShuttlecockDetector.java`
- **What it is:** The Birdie Hunter.
- **What it does:** It is trained to look for one thing only: the shuttlecock. It ignores the trees, the net, and your face, and only draws a box around the white birdie.

### `ShotClassifier.java`
- **What it is:** The Professional Judge.
- **What it does:** It looks at the wiggles in the movement data from the racket. It knows that a "Smash" has a very sharp wiggle and a "Drop" has a soft wiggle.

---

## 4. THE MATH TEACHER (Calculations)
This package is called `com.example.badminton.processing`.

### `SignalProcessor.java`
- **What it is:** The Calculator.
- **What it does:** It takes raw sensor numbers and turns them into "Real World" numbers. 
    - **Force:** $F = m \times a$ (Mass of racket times acceleration).
    - **Speed:** It calculates how fast the birdie flew based on your swing rotation.

---

## 5. THE ARTIST (The Screen & AR)
This package is called `com.example.badminton.ui`. This is everything you actually touch and see.

### `VisionFragment.java`
- **What it is:** The Camera View.
- **What it does:** It shows the live video and manages the AI "Eyes" in real-time.

### `VisionOverlayView.java`
- **What it is:** The AR Glass.
- **What it does:** This is a transparent layer on top of the video. It draws the Cyan/Orange skeleton and the Yellow box for the shuttlecock. It also draws the "breadcrumb trail" (the path) of the birdie.

### `AnalyticsHubFragment.java`
- **What it is:** The Dashboard.
- **What it does:** The main menu for all the cool AI tools like Power, Speed, and Heatmaps.

### `PowerAnalysisFragment.java` & `GaugeView.java`
- **What they are:** The Strength Meter.
- **What they do:** They show a cool speedometer-style gauge that tells you how hard you hit the birdie.

### `InjuryAnalysisFragment.java`
- **What it is:** The Body Guard.
- **What it does:** It checks if your arm moved too violently. If you swing in a way that might hurt your elbow, it gives you a warning score.

### `HeatmapFragment.java`
- **What it is:** The Paint Splatter Map.
- **What it does:** Every time you hit the birdie, it puts a dot on a picture of a court. After an hour, you can see if you play mostly on the left or right side.

### `NarrativeCoachFragment.java`
- **What it is:** The Talking Coach.
- **What it does:** It looks at all the data and writes a human sentence, like: "Your smash was powerful, but try to snap your wrist faster!"

### `HistoryFragment.java` & `HistoryAdapter.java`
- **What they are:** The Scrapbook.
- **What they do:** They show a list of every swing you’ve ever done, saved in the database.

---

## 6. THE LIBRARY (Storing Data)
This package is called `com.example.badminton.db`.

### `AppDatabase.java`
- **What it is:** The File Cabinet.
- **What it does:** The actual storage house where all your swing data lives permanently on the phone's hard drive.

### `SwingDao.java`
- **What it is:** The Librarian.
- **What it does:** This is the code that actually writes or reads data from the File Cabinet. If you want to "Save a Swing," you ask the Librarian.

### `SwingSession.java`
- **What it is:** The Log Entry.
- **What it does:** This defines what information we save for each swing (Time, Power, Speed, Type).

---

## 7. THE BOXES (Data Containers)
This package is called `com.example.badminton.model`.

### `SwingData.java`
- **What it is:** A Simple Box.
- **What it does:** It’s a container used to move swing info from the Bluetooth to the UI.

### `ExploreItem.java`
- **What it is:** A Menu Box.
- **What it does:** It holds the Icon and Title for the buttons in the Analytics Hub.

---

## 8. SPECIAL TOOLS

### `PoseFusionFragment.java`
- **What it is:** The Master Merger.
- **What it does:** It combines the Body Skeleton (Vision) and the Racket Data (Bluetooth) into one single view.

### `ActionTrailView.java`
- **What it is:** The Ghost Path.
- **What it does:** It draws that cool 3D-looking line showing exactly how your racket moved through the air.

---

## THE FULL CIRCLE (How it works in 10 seconds)
1. **You** swing the racket.
2. **ESP32** sends movement data via **Bluetooth** (`DeviceManager`).
3. **Phone Camera** sees your body (`PoseDetector`) and the birdie (`ShuttlecockDetector`).
4. **The Brain** (`MainViewModel`) matches the "Hit" to the "Video frame."
5. **The Math Teacher** (`SignalProcessor`) calculates your speed.
6. **The Artist** (`VisionOverlayView`) draws a yellow trail on the screen.
7. **The Librarian** (`SwingDao`) saves it to the **File Cabinet** (`AppDatabase`).
8. **The Coach** (`NarrativeCoach`) tells you "Great Smash!" 

🐴 **Now even a donkey understands!**
