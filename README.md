# OptiTrack Motion Capture Setup Guide with VRPN on ROS Melodic and WSL (Windows Subsystem for Linux)

## Prerequisites

* **ROS Melodic** is already installed on your WSL machine.
* **OptiTrack Motive** is installed and running on your Windows machine.

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

   * Open OptiTrack Motive on your Windows machine.
   * Go to **Data Streaming Settings** and enable **VRPN**.
   * Copy the **port number** (default is `3883`) from Motive for later use in the ROS launch file.

2. **Check Firewall Settings**:

   * Allow traffic on the VRPN port in your Windows firewall.
   * Go to **Windows Firewall with Advanced Security** > **Inbound Rules**, and create a new rule allowing UDP traffic on the VRPN port.

### 3. Configure ROS Launch File

1. **Copy the Sample Launch File**:
   Instead of creating a launch file from scratch, copy the sample launch file provided by the `vrpn_client_ros` package:

   ```
   mkdir -p ~/catkin_ws/src/mocap_vrpn/launch
   cp /opt/ros/melodic/share/vrpn_client_ros/launch/sample.launch ~/catkin_ws/src/mocap_vrpn/launch/mocap_vrpn.launch
   ```

2. **Check the Windows IP Address**:

   Before editing the launch file, open PowerShell or Command Prompt on Windows and run:

   ```
   ipconfig
   ```

   Find the **IPv4 Address** of the active network adapter used to connect to the network. For example:

   ```
   IPv4 Address. . . . . . . . . . . : 10.8.2.67
   ```

3. **Edit the Launch File**:
   Open the copied launch file and adjust the `server` address to point to the Windows IP:

   ```
   nano ~/catkin_ws/src/mocap_vrpn/launch/mocap_vrpn.launch
   ```

   For example:

   ```yaml
   server: "10.8.2.67"
   port: 3883
   ```

   **Important:** The Windows IP may change. If the connection stops working, run `ipconfig` again and verify that the IP in the launch file is still correct.

4. **Edit Trackers**:
   Ensure the `trackers` section matches your rigid body name(s) in Motive:

   ```yaml
   trackers:
     - mobico1
   ```

5. **Build and Source the Workspace**:

   ```
   cd ~/catkin_ws
   catkin_make
   source devel/setup.bash
   ```

### 4. Important Note for WSL Users

When using WSL, do **not** use `127.0.0.1` for the VRPN server address. The IP `127.0.0.1` refers to the WSL environment and not your Windows machine. Instead, use the IPv4 address of the active Windows network adapter, which you can find by running `ipconfig` in Windows.

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

   You should see a topic like:

   ```
   /vrpn_client_node/mobico1/pose
   ```

4. **Inspect the Pose Data**:
   You can inspect the pose data of your tracked body in ROS:

   ```
   rostopic echo /vrpn_client_node/mobico1/pose
   ```

5. **Visualize the Pose Data in RViz**:

   Open RViz by running:

   ```
   rviz
   ```

   In RViz:

   1. Set **Fixed Frame** to `world`.
   2. Click **Add**.
   3. Add a **Pose** display.
   4. Set the topic to:

      ```
      /vrpn_client_node/mobico1/pose
      ```

   RViz will now display the current tracked pose of the object in real time.

6. **Visualize the Motion Path in RViz**:

   To display the complete OptiTrack trajectory rather than only the current pose, open another terminal and run:

   ```
   python - <<'PY'
   import rospy
   from geometry_msgs.msg import PoseStamped
   from nav_msgs.msg import Path

   rospy.init_node('pose_to_path')
   pub = rospy.Publisher('/mobico1/path', Path, queue_size=1)
   path = Path()

   def cb(msg):
       path.header = msg.header
       path.poses.append(msg)
       pub.publish(path)

   rospy.Subscriber('/vrpn_client_node/mobico1/pose', PoseStamped, cb)
   rospy.spin()
   PY
   ```

   In RViz:

   1. Click **Add** > **Path**.
   2. Set the topic to:

      ```
      /mobico1/path
      ```

   RViz will now display the accumulated OptiTrack trajectory. To clear the path, stop the Python command with `Ctrl+C` and run it again.

### 6. Optional: Install VRPN for Testing

If you want to test VRPN connections outside of ROS, you can install VRPN from source to use tools like `vrpn_print_devices`. This step is **optional** and not required for normal ROS operations.

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
   Use the rigid body name, Windows IP, and VRPN port:

   ```
   vrpn_print_devices mobico1@<Your_Windows_IP>:3883
   ```

   For example:

   ```
   vrpn_print_devices mobico1@10.8.2.67:3883
   ```

   If the connection is working, the tracker data should continuously update.

## Troubleshooting

* **Connection Issues**: Ensure that Motive and ROS can communicate over the same network. Run `ipconfig` on Windows and verify that the current IPv4 address matches the `server` address in the ROS launch file.
* **Only `/rosout` and `/rosout_agg` appear**: The VRPN node may be running but not receiving tracker data. Check the Windows IP and test the connection with `vrpn_print_devices`.
* **Firewall**: Double-check firewall rules to ensure no traffic is blocked between WSL and Windows.
* **Tracker Name**: Make sure the tracker name in the launch file exactly matches the rigid body name in Motive.
* **Network Configuration**: When using WSL, use the Windows network IP rather than `127.0.0.1` or the WSL virtual adapter IP.

With these steps, your setup should successfully stream OptiTrack motion capture data into ROS and allow both live pose and trajectory visualization in RViz.
