# drone-death-calc
Will you die?

A tool to calculate the maximum detection distance of a drone based on camera specifications and environmental conditions.

## Features
- Calculate maximum detection distance based on camera and environmental parameters
- Analyze blur limitations from motion and sensor characteristics
- Estimate sensitivity requirements for different scenarios
- Visualize detection performance with distance charts
- Compare different detection scenarios and presets
- Sensitivity analysis with scenario modifiers

## Mathematical Model

### Core Geometry

#### Distance from Minimum Pixels
The maximum detection distance based on object size and minimum pixel requirements:

```
θ_deg = min_px / px_per_deg
θ_rad = θ_deg × π / 180
D = size_m / (2 × tan(θ_rad / 2))
```

Where:
- `size_m`: Object physical size (meters)
- `min_px`: Minimum required pixels for detection
- `px_per_deg`: Pixels per degree of field of view
- `D`: Maximum detection distance

#### Focal Length Calculation
```
f = (sensor_mm / 2) / tan(fov_deg / 2)
```

Where:
- `f`: Focal length (mm)
- `sensor_mm`: Sensor dimension (mm)
- `fov_deg`: Field of view (degrees)

### Exposure and Aperture

#### Illuminance to Exposure Value
```
EV100 = log₂(lux × K)
```

Where:
- `EV100`: Exposure value at ISO 100
- `lux`: Scene illuminance (lux)
- `K`: Calibration constant (0.30)

#### ISO-Adjusted Exposure Value
```
EV = EV100 + log₂(ISO / 100)
```

#### Required f-number
```
N = √(t_shutter × 2^EV)
```

Where:
- `N`: Required f-number
- `t_shutter`: Shutter time (seconds)
- `EV`: Exposure value

### Motion Blur Analysis

#### Object Angular Footprint
```
θ_deg = 2 × arctan(size_m / (2 × D_m))
px_footprint = θ_deg × px_per_deg
```

#### Allowed Blur Budget
```
px_allowed = blur_frac × min(px_width, px_height)
```

Where:
- `blur_frac`: Blur budget as fraction of object footprint
- `px_width`, `px_height`: Object footprint in pixels

#### Pixel Motion Rate

**Object Motion:**
```
ω = v / D_m  (rad/s)
rate_px = ω × px_per_rad
```

**Scan Motion:**
```
rate_px = scan_rate_deg/s × px_per_deg
```

#### Maximum Blur-Limited Exposure Time
```
t_max = px_allowed / (rate_obj + rate_scan)
```

### Scan Time Calculation

#### Tile Coverage
```
step_w = fov_w × (1 - overlap)
step_h = fov_h × (1 - overlap)
tiles_x = ceil(scan_az / step_w)
tiles_y = ceil(scan_el / step_h)
total_tiles = tiles_x × tiles_y
```

#### Mechanical Scan Time
```
t_pan_row = scan_az / pan_rate
t_tilt_total = (tiles_y - 1) × step_h / tilt_rate
t_mechanical = tiles_y × t_pan_row + t_tilt_total
```

#### Camera Capture Time
```
t_dwell = 1 / fps
t_camera = total_tiles × t_dwell
```

#### Total Scan Time
```
t_scan = max(t_mechanical, t_camera)
```

### Signal Processing

#### Effective Illuminance
```
lux_eff = scene_lux × transmission × efficiency
```

#### SNR Scaling
```
snr_scale = √(target_snr / 20)
f_number_adj = f_number / snr_scale
```

### Final Distance Calculation

The final maximum detection distance is limited by:
1. **Pixel resolution**: Distance where object meets minimum pixel requirement
2. **Motion blur**: Distance where blur budget allows required exposure time
3. **Travel distance**: Remaining distance after accounting for object motion during scan

```
dist_final = min(dist_pixels, dist_blur) - max_travel
```

### Algorithm Flow

1. Calculate pixels per degree from sensor resolution and FOV
2. Compute pixel-limited detection distance for width and height
3. Solve blur-limited distance using binary search (50 iterations)
4. Calculate mechanical and camera scan times
5. Determine object travel during scan
6. Compute required f-number from exposure constraints
7. Return final feasible detection distance

## Usage

1. Open `index.html` in a web browser
2. Adjust input parameters:
   - Camera specifications (FoV, sensor size, resolution)
   - Environmental conditions (illuminance, ISO)
   - Object characteristics (size, speed)
   - Scan parameters (coverage, overlap, speed)
3. Click "Recalculate" to compute maximum detection distance
4. Analyze results and sensitivity scenarios

## Assumptions and Limitations

- Assumes worst-case lateral object motion
- Uses simplified exposure model (real optics vary)
- Blur budget applied to minimum object dimension
- Scan pattern assumes row-by-row coverage
- No atmospheric effects or turbulence modeled
- Assumes object face perpendicular to camera axis
