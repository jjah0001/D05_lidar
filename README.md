# m1ct_d2 — ROS2 LiDAR Driver

ROS2 driver for the M1CT Coin D2 LiDAR sensor. Publishes laser scan data on the `/scan` topic and broadcasts a static TF transform from `base_link` to `laser_link`.

## Requirements

- ROS2 Jazzy
- A Linux machine with the M1CT D2 LiDAR connected via USB (`/dev/ttyUSB0`)

## Setup

### 1. Grant USB port access

```bash
sudo chmod 777 /dev/ttyUSB0
```

> **Note:** This resets on reboot. For a permanent fix, copy the provided udev rules file:
> ```bash
> sudo cp m1ct_d2/sc_mini.rules /etc/udev/rules.d/
> sudo udevadm control --reload-rules && sudo udevadm trigger
> ```

### 2. Build the package

From the workspace root (`m1ct_d4_ros2_v16/`):

```bash
colcon build
```

### 3. Source the environment

```bash
source /opt/ros/jazzy/setup.bash
source install/setup.bash
```

## Running the Node

```bash
ros2 run m1ct_d2 m1ctd2
```

The node will:
- Connect to the LiDAR on `/dev/ttyUSB0` at 230400 baud
- Publish laser scans on the `/scan` topic at ~10 Hz
- Broadcast a static TF from `base_link` → `laser_link`

### Parameters

Default parameters are in `m1ct_d2/params/m1ct_d2.yaml`:

| Parameter         | Default         | Description                        |
|-------------------|-----------------|------------------------------------|
| `port`            | `/dev/ttyUSB0`  | Serial port                        |
| `baudrate`        | `230400`        | Serial baud rate                   |
| `frame_id`        | `laser_link`    | TF frame for scan messages         |
| `angle_min`       | `-180.0`        | Minimum scan angle (degrees)       |
| `angle_max`       | `180.0`         | Maximum scan angle (degrees)       |
| `min_range`       | `0.01`          | Minimum range (meters)             |
| `max_range`       | `16.0`          | Maximum range (meters)             |
| `frequency`       | `10.0`          | Scan frequency (Hz)                |
| `version`         | `4`             | Sensor version (4 = M1CT Coin D2)  |

## Visualizing in RViz

### 1. Launch RViz

In a new terminal (after sourcing the environment):

```bash
source /opt/ros/jazzy/setup.bash
source install/setup.bash
rviz2
```

### 2. Configure RViz

1. In the **Displays** panel, click **Add**.
2. Select **By topic** → `/scan` → **LaserScan**, then click **OK**.
3. In the **Fixed Frame** field (top of the Displays panel), set it to `laser_link`.
4. The laser scan should now appear as a ring of colored dots in the 3D view.

### Optional: adjust display settings

In the **LaserScan** display entry you can tune:
- **Size (m)**: point size in the visualization (e.g. `0.05`)
- **Color Transformer**: `Intensity` to colour points by reflectivity, or `FlatColor` for a solid colour
- **Min/Max Color**: colour range when using the `Intensity` transformer

## Topics

| Topic          | Type                          | Description              |
|----------------|-------------------------------|--------------------------|
| `/scan`        | `sensor_msgs/msg/LaserScan`   | 2D laser scan output     |
| `/lsd_error`   | `std_msgs/msg/String`         | Error/status messages    |
| `/lidar_status`| `std_msgs/msg/UInt16` (sub)   | Control commands (in)    |

### Lidar status control commands

Publish to `/lidar_status` to control the sensor at runtime:

| Value | Action               |
|-------|----------------------|
| `1`   | Start LiDAR          |
| `2`   | Stop LiDAR           |
| `3`   | High exposure mode   |
| `4`   | Low exposure mode    |
| `5`   | Clear error state    |
| `6`   | High rotation speed  |
| `7`   | Low rotation speed   |

Example:

```bash
ros2 topic pub /lidar_status std_msgs/msg/UInt16 "{data: 1}"
```

## Troubleshooting

| Symptom                          | Fix                                                              |
|----------------------------------|------------------------------------------------------------------|
| `Permission denied` on serial    | Run `sudo chmod 777 /dev/ttyUSB0`                                |
| No scan data in RViz             | Check **Fixed Frame** is set to `laser_link`                     |
| `node_lidar init error`          | Verify the LiDAR is plugged in and the port is correct           |
| Node exits after ~3 s            | LiDAR may be unresponsive — unplug, replug, and restart the node |
