# Dead-Reckoning

Hybrid probabilistic + ML dead-reckoning stack focused on robust GNSS-denied navigation:

- Deterministic physics for safety-critical state estimation
- ML modules for denoising, speed observability, and adaptive bias correction

## Recommended Tech Stack

### 1) Mobile app (Android-first)
- Kotlin
- Jetpack Compose
- Foreground Service
- NDK (C++ core)
- ONNX Runtime Mobile / TFLite
- Room DB
- MapLibre GL + offline OSM tiles (MBTiles)

### 2) Edge engine (portable)
- C++17 core library (shared with mobile core)
- Eigen + Ceres
- protobuf/gRPC interface
- Optional Rust wrapper for embedded deployment

### 3) Fusion/mapping core
- Error-state INS + UKF (or invariant EKF)
- Chi-square innovation gating
- GNSS outage detector
- HMM map-matching (OSM graph + turn restrictions)
- NHC constraints in vehicle frame

### 4) ML training stack (cloud/desktop)
- Python
- PyTorch
- Hydra
- MLflow / Weights & Biases
- DVC
- Jupyter for analysis
- Export quantized ONNX/TFLite artifacts

### 5) Data/ETL
- ROS bag/CSV parsers
- Polars/Pandas
- Sequence-level split tooling for IO-VNBD
- Deterministic feature pipeline

### 6) Validation/CI
- GitHub Actions
- Unit + replay tests
- Scenario benchmarks (tunnel/urban canyon/parking)
- Reproducible plot generator scripts

## Design Plan

1. **State definition**  
   Position, velocity, attitude, gyro/accel bias, scale factors; strict body→nav transforms; gravity compensation before integration.
2. **Calibration/alignment module**  
   Automatic phone-to-vehicle yaw/pitch/roll estimation with continuous misalignment monitoring.
3. **Outage handler**  
   Explicit GNSS quality triggers (C/N0, HDOP, innovation-gate failure), mode-switch FSM, measured switch-latency KPI.
4. **ML modules**  
   - IMU denoise/vibration suppression (1D temporal model)  
   - Speed estimator from IMU windows (self-supervised against GNSS/odometry where available)  
   - Optional bias drift predictor as pseudo-measurement into filter
5. **Map matching**  
   HMM candidate generation + topology-aware transitions; confidence score fed back into fusion as a soft constraint.
6. **Dual-rate architecture**  
   100–200 Hz inertial propagation; 10 Hz fused navigation output for phone UI; independent high-rate edge output path.
7. **Evaluation protocol**  
   Drive-level train/val/test split (no leakage); report DR drift (% distance), absolute trajectory error, speed MAE/RMSE, outage recovery error, and 10 Hz latency profile.
8. **Deliverables**  
   Mobile APK, edge SDK, pretrained model package, IO-VNBD reproducible benchmark report, and trajectory plots.
