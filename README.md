# Lane-Detection-with-OpenCV
A classical computer-vision pipeline (no deep learning) that detects lane boundaries in road images and dashcam videos, and overlays them in real time. 
Features

- Works on both single images and full videos
- Detects left and right lane lines independently using Hough Line Transform
- Robust classification of lines by extrapolated position (not just raw slope), so it holds up near the vanishing point
- Weighted line fitting — longer, more confident segments (e.g. a solid line) count more than short/noisy fragments (e.g. a dashed line)
- Solid vs. dashed line detection based on edge coverage ratio
- Visual "lane width" indicator (arrow + label) between the two detected boundaries
- A debug mode that visualizes every stage of the pipeline (edges → ROI mask → raw Hough segments) for quick tuning on new footage

## How it works

1. **Edge detection** — grayscale + Gaussian blur + Canny edge detector
2. **Region of interest** — masks out the sky/horizon, keeping the road area
3. **Hough Transform** — `cv2.HoughLinesP` finds straight line segments in the masked edge map
4. **Classification** — each segment is assigned to the left or right lane boundary by its slope sign *and* its extrapolated x-position at the bottom of the frame
5. **Line fitting** — `np.polyfit`, weighted by segment length, produces one clean line per side
6. **Overlay** — the two lines (and, optionally, a lane-width arrow/label) are drawn back onto the original frame

## Installation

```bash
pip install opencv-python-headless numpy matplotlib
```

## Usage

Open `lane_detection.ipynb` in Jupyter or Google Colab.

**Single image:**
```python
result = lane_detection(image)
```

**Video (frame-by-frame):**
```python
cap = cv2.VideoCapture("input.mp4")
out = cv2.VideoWriter("output.mp4", cv2.VideoWriter_fourcc(*"mp4v"), fps, (width, height))

while True:
    ret, frame = cap.read()
    if not ret:
        break
    result_frame = lane_detection(frame)
    out.write(result_frame)

cap.release()
out.release()
```

**Debugging on new footage:**
```python
debug_pipeline(image)  # shows edges, ROI mask, and raw Hough segments side by side
```

## Notes / limitations

- The region-of-interest and Hough parameters are tuned for a fairly standard front-facing dashcam angle; very different camera mounts or fields of view may need re-tuning — use `debug_pipeline()` to check quickly.
- This is a classical (non-ML) approach, so it works best on clearly painted lane markings in reasonable lighting/weather.

## License

MIT — see [LICENSE](LICENSE)
