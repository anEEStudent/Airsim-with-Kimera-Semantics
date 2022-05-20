# Airsim-with-Kimera-Semantics
This documents my attempt with feeding the RGB camera and depth caemra outputs of Airsim to Kimera Semantics. [Kimera-Semantics](https://github.com/MIT-SPARK/Kimera-Semantics) is a library that is able to do semantic labels with only depth and RGB inputs. From this demonstration clip that I have recorded below, there is still some work to be done.

![Demo](/assets/images/demo.gif)

## Environment
1. Ubuntu 18.04 
2. Download ROS Melodic from the [ROS website](http://wiki.ros.org/melodic/Installation/Ubuntu)
3, Download [Kimera-Semantics](https://github.com/MIT-SPARK/Kimera-Semantics)
4. Download [Airsim](https://microsoft.github.io/AirSim/) and [UnrealEngine 4](https://www.unrealengine.com/en-US/download) (remember to check GCC version is greater than 8 when compiling Airsim) 
5. Ensure that Airsim is downloaded to the same folder as Kimera-Semantics (should be `~/catkin_ws/src`)
6. Double-check all their ROS wrappers are running by running the respective demos

## Steps to run what I have done:
1. Use my custom settings.json file for Airsim. This can be found in `~/Documents/Airsim/settings.json`. If there is nothing there manually create an Airsim folder and place my settings.json file. 
    - Brief explanation: My settings.json file launches a drone in Airsim (you can launch in car, drone, or camera mode). It also outputs the camera outputs for use in Kimera. 
2. Replace `airsim_ros_wrapper.cpp` file with my file. If you installed Airsim in `~/catkin/src`, the exact file location will be in `~/catkin_ws/src/AirSim/ros/src/airsim_ros_pkgs/src`. Otherwise, you will need to search in the location where you installed Airsim. Remember to rebuild catkin after replacing this file.
    - Brief explanation: I have changed the name of the headers of tf messages in this file to match Kimera’s naming. I have also disabled sending extra tf messages that are not required. Have added comments to things that I have changed.
3. Replace `kimera_semantics.launch` file with my file. If you installed Kimera-Semantics in `~/·catkin/src`, the exact file location will be in `~/catkin_ws/src/Kimera-Semantics/kimera_semantics_ros/launch/kimera_semantics.launch`
    - Brief explanation: Edited the launch file to be able to read topics from Airsim with some other customizations such as links to csv colour file and added tf_static transformations. 
4. Add in my csv files to `~/catkin_src/src/Kimera-Semantics/kimera_semantics_ros/cfg`
    - Brief explanation: These csv files contain the RGB values for the segmentations in the segmented image input to Kimera-Semantics. This is required for Kimera-Semantics to run. You can find the link to the csv file in the launch file under the `semantic_label_2_color_csv_filepath` argument. If you use a new environment, you will need to edit this file to match the colours of the segmentations in the semantically labelled image.
5. Replace kimera_semantics_gt.rviz launch file with my rviz file in `~/catkin_src/src/Kimera-Semantics/kimera_semantics_ros/rviz/kimera_semantics_gt.rviz`
    - Brief explanation: Changed some parameters to visualize topics from Airsim
6. Download my rosbag files. One is recorded in the Blocks environment from the UnrealEngine editor and one is recorded in the neighbourhood environment which is a precompiled environment provided by Airsim. Place them in the `~/Downloads` folder.
7. To run demo from rosbag: Run these following commands in separate terminal tabs/windows in the following order (basically Kimera needs to be launched before Airsim):
    - Launch Airsim in UnrealEngine. Ensure that the settings.json file has been placed in `~/Documents/Airsim/settings.json` before launch.
    - (In any location) `roscore`
    - (In ~/catkin/src) `roslaunch kimera roslaunch kimera_semantics_ros my_kimera_semantics.launch play_bag:=true (change path <arg name="bag_file"    default="$(find kimera_semantics_ros)/rosbags/kimera_semantics_demo.bag"/>  in kimera launch file to launch different rosbags)`
    - (In any location) `rviz -d $(rospack find kimera_semantics_ros)/rviz/my_kimera_semantics_gt.rviz`
    - Use a controller to fly the drone around!
8. To run live from simulation:
    - Launch Airsim in UnrealEngine. Ensure that the `settings.json` file has been placed in `~/Documents/Airsim/settings.json` before launch.
    - Run the following in this order in separate terminals (launch Kimera before Airsim):
    ```
    roscore
    roslaunch kimera roslaunch kimera_semantics_ros my_kimera_semantics.launch 
    roslaunch airsim_ros_pkgs airsim_node.launch 
    rviz -d $(rospack find kimera_semantics_ros)/rviz/my_kimera_semantics_gt.rviz
    ```
    - Use a controller to fly the drone around!

9. To run live from simulation in different environments (launch Kimera before Airsim):
    - Download a pre-compiled binary from here, I used the Airsim Neighbourhood environment. 
    - Run the .sh file in by running the command ./AirSimNH.sh 
    - Change the name in (b) accordingly if you downloaded another environment
    - Follow (8)d


##Parameters to take note of in the launch file for Kimera-Semantics:
1. Names of topics that Kimera reads as inputs
    - `left_cam_info_topic`
    - `right_cam_info_topic`
    - `left_cam_topic`
    - `right_cam_topic`
    - `left_cam_segmentation_topic`
    - `right_cam_segmentation_topic`
    - `left_cam_depth_topic`
2. `use_sim_time`: Setting this to true will prompt Kimera-Semantics to run only when there are messages being published in the /clock topic. I have set it to false as there seems to be some issue with clock publishing in Airsim.
3. `semantic_label_2_color_csv_filepath`: This links the csv file with RGB values for the different segmentation colours.
4. `bag_file`: This is the file path for the bag file that will be played with the `play_bag:=true` argument when launching Kimera Semantics
5. Added static tf transforms as follows to match the default Kimera behaviour:
```
<node pkg="tf2_ros" type="static_transform_publisher" name="left_cam_broadcaster" args="0.2 0 0 0 0 0 1 base_link_gt left_cam" />
<node pkg="tf2_ros" type="static_transform_publisher" name="right_cam_broadcaster" args="-0.2 0 0 0 0 0 1 base_link_gt right_cam" />
```


## Known Issues:
1. Poor mesh quality
2. Cannot install Kimera-VIO

## Suggestions for future work:
There is a video here https://www.youtube.com/watch?v=Zjevg5wQTdI that shows a demonstration by the team behind Kimera. They used Kimera-Semantics in conjunction with Kimera-VIO to create the mesh in that demonstration there. I tried to replicate what they did towards the end of my internship but I was unable to install Kimera-VIO. Perhaps this might be an area worth exploring. The professor in the video also mentioned changing the yaml settings for the camera to get better quality. Otherwise, there might be some opportunity to better the mesh quality by looking into the Voxblox library which is the backbone of Kimera-Semantics. 