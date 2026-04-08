---
name: Complete range drag feature
overview: "Complete the range-drag-via-long-press feature. iOS is fully implemented; Android needs two remaining changes: modify the touch listener to integrate `GestureDetector` and add the `onRangeDrag` method."
todos:
  - id: android-touch-listener
    content: Modify setProgressIndicatorTouchListener to forward events to GestureDetector and branch on isRangeDragging
    status: completed
  - id: android-range-drag
    content: Add onRangeDrag method for computing and applying range shift with clamping and haptics
    status: completed
isProject: false
---

# Complete Range Drag Feature

## Current State

The previous implementation session completed:

**iOS (DONE -- all 3 files fully changed):**
- [ios/VideoTrimmer.swift](ios/VideoTrimmer.swift) -- new control events (`didBeginDraggingRange`, `rangeDragChanged`, `didEndDraggingRange`), `rangeDragGestureRecognizer` (platform-default `UILongPressGestureRecognizer`), state variables, gesture priority wiring, and full `rangeDragPanned(_:)` handler
- [ios/VideoTrimmerViewController.swift](ios/VideoTrimmerViewController.swift) -- 3 new handler methods wired via `addTarget`

**Android (PARTIALLY DONE):**
- [android/.../VideoTrimmerView.java](android/src/main/java/com/videotrim/widgets/VideoTrimmerView.java) -- `GestureDetector` import added, state variables added (`isRangeDragging`, `rangeDragInitialRawX`, etc.), `initRangeDragDetector()` method created and called from `init()`

## Remaining Work (Android only)

### 1. Modify `setProgressIndicatorTouchListener()` (line 687)

The current method at line 687-705 needs to:
- Forward all `MotionEvent`s to `rangeDragGestureDetector` first
- Reset `isRangeDragging` on `ACTION_DOWN`
- On `ACTION_MOVE`: if `isRangeDragging`, call `onRangeDrag(event)` instead of `onTrimmerContainerPanned(event)`
- On `ACTION_UP`: if `isRangeDragging`, reset the flag and update time display

Replace the method body with:

```java
private void setProgressIndicatorTouchListener() {
    trimmerContainerBg.setOnTouchListener((view, event) -> {
      rangeDragGestureDetector.onTouchEvent(event);

      switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
          isRangeDragging = false;
          didClampWhilePanning = false;
          onMediaPause();
          onTrimmerContainerPanned(event);
          playHapticFeedback(true);
          break;
        case MotionEvent.ACTION_MOVE:
          if (isRangeDragging) {
            onRangeDrag(event);
          } else {
            onTrimmerContainerPanned(event);
          }
          break;
        case MotionEvent.ACTION_UP:
          if (isRangeDragging) {
            isRangeDragging = false;
            updateCurrentTime(true);
          }
          view.performClick();
          break;
        default:
          return false;
      }
      return true;
    });
}
```

### 2. Add `onRangeDrag()` method

Add a new method (e.g., after `onTrimmerContainerPanned` around line 753) that:
- Computes time delta from the initial touch position using x displacement
- Shifts both `startTime` and `endTime` by the same delta (keeping duration constant)
- Clamps to `[0, mDuration]`
- Plays haptic on hitting bounds
- Calls `updateHandlePositions()` and `seekTo(startTime, false)`

```java
private void onRangeDrag(MotionEvent event) {
    float deltaX = event.getRawX() - rangeDragInitialRawX;
    float containerWidth = trimmerContainerBg.getWidth();
    if (containerWidth <= 0) return;

    long rangeDuration = rangeDragInitialEndTime - rangeDragInitialStartTime;
    long deltaTime;
    if (isZoomedIn) {
      deltaTime = (long) (deltaX / containerWidth * zoomedInRangeDuration);
    } else {
      deltaTime = (long) (deltaX / containerWidth * mDuration);
    }

    long newStart = rangeDragInitialStartTime + deltaTime;
    long newEnd = newStart + rangeDuration;

    boolean didClamp = false;
    if (newStart < 0) {
      newStart = 0;
      newEnd = rangeDuration;
      didClamp = true;
    }
    if (newEnd > mDuration) {
      newEnd = mDuration;
      newStart = newEnd - rangeDuration;
      didClamp = true;
    }

    if (didClamp && !didClampWhilePanning) {
      playHapticFeedback(false);
    }
    didClampWhilePanning = didClamp;

    startTime = newStart;
    endTime = newEnd;
    updateHandlePositions();
    seekTo(startTime, false);
}
```

## Files to Change

| File | Status | Remaining |
|------|--------|-----------|
| `ios/VideoTrimmer.swift` | DONE | -- |
| `ios/VideoTrimmerViewController.swift` | DONE | -- |
| `android/.../VideoTrimmerView.java` | Partial | Modify `setProgressIndicatorTouchListener`, add `onRangeDrag` |