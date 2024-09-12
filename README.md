# OptiTrack Motion Capture Setup Guide with VRPN on ROS Melodic and WSL (Windows Subsystem for Linux)

## Prerequisites
- **ROS Melodic** is already installed on your WSL machine.
- **OptiTrack Motive** is installed and running on your Windows machine.

## Steps to Configure and Connect VRPN to ROS

### 1. Install the `vrpn_client_ros` Package

1. **Install the VRPN Client ROS Package**:
   Open a terminal in WSL and run:
   ```
   sudo apt-get update
   sudo apt-get install ros-melodic-vrpn-client-ros
   ```

### 2. Configure Motive (on Windows)

1. **Enable VRPN Streaming**:
   - Open OptiTrack Motive on your Windows machine.
   - Go to **Data Streaming Settings** and enable **VRPN**.
   - Copy the **port number** (default is `3883`) from Motive for later use in the ROS launch file.

2. **Check Firewall Settings**:
   - Allow traffic on the VRPN port in your Windows firewall.
   - Go to **Windows Firewall with Advanced Security** > **Inbound Rules**, and create a new rule allowing UDP traffic on the VRPN port.

### 3. Configure ROS Launch File

1. **Copy the Sample Launch File**:
   Instead of creating a launch file from scratch, copy the sample launch file provided by the `vrpn_client_ros` package:
   ```
   mkdir -p ~/catkin_ws/src/mocap_vrpn/launch
   cp /opt/ros/melodic/share/vrpn_client_ros/launch/sample.launch ~/catkin_ws/src/mocap_vrpn/launch/mocap_vrpn.launch
   ```

2. **Edit the Launch File**:
   Open the copied launch file and adjust the `server` argument to point to your Windows IP address (instead of `127.0.0.1`):
   ```
   nano ~/catkin_ws/src/mocap_vrpn/launch/mocap_vrpn.launch
   ```

   Modify the following line:
   ```xml
   <arg name="server" default="127.0.0.1"/>
   ```

   Change it to your Windows IP address:
   ```xml
   <arg name="server" default="Your_Windows_IP"/>
   ```

   For example, if your Windows IP is `10.8.2.136`:
   ```xml
   <arg name="server" default="10.8.2.136"/>
   ```

3. **Edit Trackers**:
   Ensure the `trackers` section matches your rigid body name(s) in Motive:
   ```xml
   trackers:
     - TestBody  # Name of the rigid body from Motive
   ```

4. **Build and Source the Workspace**:
   ```
   cd ~/catkin_ws
   catkin_make
   source devel/setup.bash
   ```

### 4. Important Note for WSL Users

When using WSL, do **not** use `127.0.0.1` for the VRPN server address. The IP `127.0.0.1` refers to the WSL environment and not your Windows machine. Instead, use the IP address of your Windows machine (e.g., `10.8.2.136`), which you can find by running `ipconfig` in a Windows command prompt.

### 5. Test the Setup

1. **Start Motive** on your Windows machine and ensure the VRPN server is running.

2. **Launch the VRPN client node in ROS**:
   ```
   roslaunch mocap_vrpn mocap_vrpn.launch
   ```

3. **Verify ROS Topics**:
   Check for the available topics using:
   ```
   rostopic list
   ```

   You should see a topic like `/vrpn/TestBody/pose`.

4. **Inspect the Pose Data**:
   You can inspect the pose data of your tracked body in ROS:
   ```
   rostopic echo /vrpn/TestBody/pose
   ```


5. **Visualize the Pose Data in RViz**:

   Open RViz by running:
   ```
    rviz
   ```
   In RViz, follow these steps:

   1. Click the **"Add"** button in the lower-left corner.
   2. Select **"By Topic"** from the list.
   3. Expand the `/vrpn` topic and select the **"Pose"** option under your tracked object (e.g., `/vrpn/TestBody/pose`).
   4. RViz will now display the tracked pose of the object in real-time.

   Adjust Visualization (Optional):

   - You can customize the display by adjusting the pose markers (e.g., setting up arrows, axes, or shapes).
   - Ensure that you have the **TF** (transform frames) visualization active if needed, to help understand the frame transformations.

### 6. Optional: Install VRPN for Testing (Optional)

If you want to test VRPN connections outside of ROS, you can install VRPN from source to use tools like `vrpn_print_devices` to check if the connection to the VRPN server is working. This step is **optional** and not required for normal ROS operations.

#### Steps for Installing VRPN for Testing:
1. **Install Dependencies**:
   ```
   sudo apt-get install cmake g++ libudev-dev libusb-1.0-0-dev
   ```

2. **Clone and Build VRPN**:
   ```
   git clone https://github.com/vrpn/vrpn.git
   cd vrpn
   mkdir build
   cd build
   cmake ..
   make
   ```

3. **Test with VRPN Tools**:
   After building, you can use the following tools to test the connection:
   - `vrpn_print_devices <Your_Windows_IP> 3883`
   - `vrpn_print_messages <Your_Windows_IP> 3883`

   These tools allow you to verify if the VRPN server is correctly streaming data.

## Troubleshooting

- **Connection Issues**: Ensure that Motive and ROS can communicate over the same network. Use the correct IP address of the Windows machine in the ROS configuration.
- **Firewall**: Double-check firewall rules to ensure no traffic is blocked between WSL and Windows.
- **Network Configuration**: If using WSL, ensure you're using the Windows IP (`10.8.x.x`) and not `127.0.0.1` for the VRPN server.

With these steps, your setup should be successfully streaming OptiTrack motion capture data into ROS.
