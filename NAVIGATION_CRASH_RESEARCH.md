# Extreme Level Research: Navigation-Induced Crashes in Badminton AI App

This research identifies the root causes of application crashes during navigation between components in the current codebase.

## 1. Asynchronous Callback Race Conditions (High Probability)
The app heavily uses asynchronous processing for AI (MediaPipe, TFLite) and CameraX.

### **Problematic Pattern in `VisionFragment.java` & `PoseFusionFragment.java`**
```java
// Inside analyzeImage (running on cameraExecutor background thread)
if (getActivity() != null) {
    getActivity().runOnUiThread(() -> {
        if (binding != null) {
            binding.visionOverlay.updateShuttlecock(overlayLocation);
        }
    });
}
```
*   **The Crash**: If the user navigates away, `getActivity()` might return a non-null value during the check, but by the time `runOnUiThread` tries to execute the lambda on the Main Thread, the Fragment may have been detached, or the Activity destroyed.
*   **Result**: `NullPointerException` when accessing `binding` (if cleared in `onDestroyView`) or `IllegalStateException` if `requireActivity()` was used.

## 2. Unsynchronized Resource Teardown (Critical Vulnerability)
The ML models are closed in `onDestroyView`, but background threads might still be accessing them.

### **Race Condition in `ShuttlecockDetector.java`**
*   **`detect()`** is `synchronized`, but **`close()`** is **NOT synchronized**.
*   **The Crash**: If `onDestroyView()` is called on the Main Thread, it invokes `shuttlecockDetector.close()`, which calls `tflite.close()`. Simultaneously, the `cameraExecutor` thread might be entering the `detect()` method or be mid-execution.
*   **Result**: `IllegalArgumentException: Internal error: The Interpreter has already been closed.`

## 3. Lifecycle-Unsafe `requireContext()` in Observers
In `PoseFusionFragment.java`:
```java
viewModel.getGhostTrajectory().observe(getViewLifecycleOwner(), landing -> {
    if (landing != null) {
        binding.tvSyncStatus.setTextColor(ContextCompat.getColor(requireContext(), android.R.color.holo_green_light));
    }
});
```
*   **The Crash**: Even when using `getViewLifecycleOwner()`, there is a micro-window during Fragment destruction where the observer might receive a final update. If `requireContext()` is called while the fragment is being detached, it throws an `IllegalStateException`.
*   **Fix**: Use `getContext()` and a null check, or use `binding.getRoot().getContext()`.

## 4. Fragment Transaction State Loss
In `DeviceScanDialogFragment.java`:
```java
DeviceScanAdapter.OnDeviceClickListener listener = device -> {
    viewModel.connectToDevice(device);
    dismiss(); // <--- Potential Crash
};
```
*   **The Crash**: If the user selects a device just as the Activity is going into the background (e.g., a phone call or rapid "Back" press), `dismiss()` will attempt a FragmentTransaction after `onSaveInstanceState`.
*   **Result**: `IllegalStateException: Can not perform this action after onSaveInstanceState`.
*   **Fix**: Use `dismissAllowingStateLoss()`.

## 5. ViewBinding Access after `onDestroyView`
All fragments clear their `binding` in `onDestroyView()`. However, several `Log.d` or background cleanup tasks in `onDestroyView` or following lifecycle methods might accidentally reference `binding` or its components.

### **Recommendations for Stability**
1.  **Synchronize Teardown**: Make `close()` methods in ML classes `synchronized` to ensure they don't execute while a `detect()` call is active.
2.  **Safe Context Access**: Avoid `requireContext()` and `requireActivity()` inside asynchronous callbacks or observers. Use `getContext()` and check for null.
3.  **Analyzer Shutdown**: In `onDestroyView`, unbind the `ImageAnalysis` use case *before* shutting down the executor and closing the ML models to ensure no more frames are sent for processing.
4.  **Halt Main Thread Posting**: Use a custom `isAdded()` or lifecycle check inside `runOnUiThread` to ensure the fragment is still valid.
