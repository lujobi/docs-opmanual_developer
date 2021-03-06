# Procedure {#sec:hw_benchmark_procedure level=sec status=ready}

Author: Luzian Bieri

Maintainer: Luzian Bieri


<minitoc/>

## Environment
The environmental requirements are minimal. Use at minimum a 3x3 Loop without intersections. Ensure that the map is exclusively illuminated from the ceiling, block all light coming from the side. Use a dark, neutral surrounding for all tiles in order to no disturb the lane detection. 

## Setup
If a setup (especially the Duckiebot) is already prepared, **you might continue with the next chapter**. See <a class="only_number" href="#dw-setup"></a>. _Please mind that the hardware benchmark results aren't fully comparable, as the benchmark aims to compare different hardware setups using the same software._

### Duckietown shell
If not installed yet install the newest version of dts (duckietown shell) via the instructions provided by the [docs](https://github.com/duckietown/duckietown-shell)
Currently we are using a custom stack of the duckietown shell commands, which fix all software to fixed versions to ensure comparability among different hardware configurations. As they are not in the original repo (yet) they have to be cloned from [this fork](https://github.com/lujobi/duckietown-shell-commands-fixed-images). We are interested in the benchmarking branch. 

#### Installing
Use the following commands in a directory of your choice, recommended `/home/user/Documents/benchmarking`.

    laptop $ cd ![/path/to/commands/dir]
    laptop $ git clone git@github.com:lujobi/duckietown-shell-commands.git
    laptop $ git checkout benchmark

Export the path to the local version of duckietown shell commands

    laptop $ export DTSHELL_COMMANDS=![/path/to/commands/dir]duckietown-shell-commands/

Set dts to any version:

    laptop $ dts --set-version ![RELEASE]

In order to test whether the installation was successful enter `dts`. The output should look something like:

``` bash
INFO:dts:Commands version: daffy
INFO:dts:Using path '![/path/to/commands/dir/]duckietown-shell-commands/' as prescribed by env variable DTSHELL_COMMANDS.
INFO:dts:duckietown-shell-commands 5.0.2
INFO:duckietown-challenges:duckietown-challenges 5.1.5
INFO:zj:zuper-ipce 5.3.0
```
Enable the benchmark command via:
    
    laptop $ dts install benchmark

Add your dts token via:
    
    laptop $ dts tok set

#### Explanation benchmark commands
The branch benchmarking fixes all software versions to a specific one in order to ensure reproducibility. Additinally the command group benchmark is added. This is used to specify the version of benchmarking software.

### Duckietown world {#dw-setup}
In principal this step can be skipped as well if no localization system is around. Please continue with <a class="only_number" href="#db-setup"></a>.
#### Tiles
For the first benchmark we need the “normal” 3x3 circle circuit. Please ensure that the tiles are cleaned and assembled to [specifications](https://docs.duckietown.org/DT18/opmanual_duckietown/out/dt_ops_appearance_specifications.html). 
Especially make sure that the road has the correct width.

#### Lighting
In order to ensure reproducibility use a well illuminated room. Ensure that the light comes down from the ceiling, such that the bots are not dazzled. **Ensure that no natural light hits the bot.**

#### Watchtowers
For the 3X3 circuit use 4 watchtowers in the middle of the circuit. One per corner. Ensure a proper connection and that the watchtowers are close to the corners. 

##### Setup
As of now we use the standard procedure of setting up a loclization system and setting up the the watchtowers. **Thus make sure to use a new Terminal where the `export DTSHELL_COMMANDS` has not been executed.** Detailed instructions can be found [here](https://docs.duckietown.org/daffy/opmanual_autolab/out/watchtower_initialization.html). All in all use the following command to initialize the sd card:

    laptop $ dts init_sd_card --hostname ![WT_HOST_NAME] --linux-username mom --linux-password MomWatches --country COUNTRY --type watchtower --experimental


##### Calibration
Use the same calibration procedure as for a standard duckiebot. **But only the intrinsic part.** [Instructions](https://docs.duckietown.org/daffy/opmanual_duckiebot/out/camera_calib.html). 

Starting the camera demo:

    laptop $ dts duckiebot demo --demo_name camera --duckiebot_name ![WT_HOST_NAME] --package_name pi_camera --image 
duckietown/dt-core:daffy

Start the calibration:

    laptop $ dts duckiebot calibrate_intrinsics ![WT_HOST_NAME]

Start collecting data for the calibration. Press on the calibrate button as soon as all bars are green. Click Commit and check under `WT_HOST_NAME.local:8082/data/config/calibrations/camera_intrinsic/` that a file named `WT_HOST_NAME.yaml` exists.

#### World
Use the instructions found [here](https://docs.duckietown.org/daffy/opmanual_autolab/out/autolab_map_making.html) to set up the jupyter notebook in order to generate a new map. Note you will have to create your own fork of the duckietown-world. Make sure to leave the repo name as is!

##### Map
The yaml for a loop with floor around the map and in the center is the following: 

```yaml
tile_size: 0.585
tiles:
- - floor
  - floor
  - floor
  - floor
  - floor
- - floor
  - curve_left/W
  - straight/W
  - curve_left/N
  - floor
- - floor
  - straight/S
  - floor
  - straight/N
  - floor
- - floor
  - curve_left/S
  - straight/E
  - curve_left/E
  - floor
- - floor
  - floor
  - floor
  - floor
  - floor
```
Verify the that the map is displayed in the notebook under the name you gave to the file.

##### Apriltags
We use 4 ground Apriltags placed outside of each corner, moved  in 10 cm from both borders. Use [this link](https://docs.duckietown.org/daffy/opmanual_autolab/out/localization_apriltags_specs.html) in order to fill the map with april tags. As such use the command. 

    laptop $ python3 src/apriltag_measure/measure_ground_apriltags.py ![MAP_NAME]

Ensure that you enter your measurements **in Meters**. If you go back to the notebook you should see your map now rendered correctly with Apriltags.

#### Localization
In order to set up the basic localization use the following commands on every watchtower:

    laptop $ docker -H ![WT_HOST_NAME].local rm -f dt18_03_roscore_duckiebot-interface_1


    laptop $ docker -H ![WT_HOST_NAME].local pull duckietown/dt-duckiebot-interface:daffy-arm32v7

    laptop $ docker -H ![WT_HOST_NAME].local run --name duckiebot-interface --privileged -e ROBOT_TYPE=watchtower --restart unless-stopped -v /data:/data -dit --network=host duckietown/dt-duckiebot-interface:daffy-arm32v7

It might be that the last command fails. Use portainer to remove the `dt-duckiebot-interface`-Container.

Start the acquisition-bridge on all watchtowers:

    laptop $ docker -H ![WT_HOST_NAME].local run --name acquisition-bridge --network=host -e ROBOT_TYPE=watchtower -e LAB_ROS_MASTER_IP=![YOUR_PC_IP] -dit duckietown/acquisition-bridge:daffy-arm32v7

### Duckiebot {#db-setup}
#### Hardware Setup
Assemble the Duckiebot as prescribed in the manual of the respective version of the Kit.
Ensure that no wires are touching the wheels, or hinder the benchmark in any other way.
Clearly mark the different Duckiebots. Add an Apriltag to your Duckiebot and enter the name below. All in lowercase without whitespaces.

**Ensure that no hardware gets mixed between different configurations. Otherwise the whole benchmark will be invalidated.**

#### Init SD Card
This procedure only works if the special dt-shell-commands is installed. otherwise Proceed with the normal setup.

Decide which software version (e.g. master19, daffy, ente) you want to run. Set the benchmark version to said software version using dts:

    laptop $ dts benchmark set [!SOFTWARE_VERSION]

If the set version should be checked use the following command:

    laptop $ dts benchmark info

Use the init_sd_card command as known. (Some options which could change the software version are disabled) Use said hostname: ![HOST_NAME]

    laptop $ dts init_sd_card --hostname ![HOST_NAME] --country CH --wifi [!NETWORKS]

**If you are using a 16GB SD-Card, use the `--compress` flag.**

### Initial Setup
If you have a complete set up and calibrated bot you might want to move to the next Unit <a class="only_number" href="#sec:hw_benchmark_demo"></a>

After the `init_sd_card` procedure ist over, take any charged battery (which doesn’t belong to one of the bots to be tested) and plug the Duckiebot in. After some time the bot should be pingable, then ssh-ing into it should be possible. 
#### Portainer and compose
Open [`HOST_NAME.local:9000`](http://HOST_NAME.local:9000) in a browser. As soon as portainer is running, there should be 4 containers one of which is not running (`duckietown/rpi-duckiebot-dashboard`), start that one via portainer.
After a short time [`HOST_NAME.local`](HOST_NAME.local) should be reachable. Further progress of the installation can be see there after skipping the login. Finished setting up, enter the your duckietown-token.  

#### Verification TODO delete
Use this command to test the setup

    laptop $ dts duckiebot keyboard_control ![HOST_NAME]

and this command to test the camera:

    laptop $ dts start_gui_tools ![HOST_NAME]

then use:

    laptop $ rqt_image_view

this should display the live camera feed.


#### Calibration
To calibrate the bot we use the same command as is used in the [docs](https://docs.duckietown.org/daffy/opmanual_duckiebot/out/camera_calib.html). 
##### Camera intrinsic

As the `duckiebot-interface` should be already running use the following command: 

Daffy

    laptop $ dts duckiebot demo --demo_name camera --duckiebot_name ![HOST_NAME] --package_name pi_camera --image duckietown/dt-core:daffy@sha256:4c7633c2041f5b7846be2346e0892c9f50987d2fd98d3479ec1a4cf378f52ee6

Master19

    laptop $ dts duckiebot demo --demo_name camera --duckiebot_name ![HOST_NAME] --image duckietown/rpi-duckiebot-base:master19@sha256:80f23a1835e6b3f9d2606aae54ce824dd13085e3e7491e87c7f0216797964b5c


Then run the calibration using the command:

Daffy

    laptop $ dts duckiebot calibrate_intrinsics ![HOST_NAME] --base_image  duckietown/dt-core:daffy-amd64@sha256:d72e8a8c3191c146ecc2a812bdd036aaf15076e6c1cb9467304e0e54f9a39a10

Master19

    laptop $ dts duckiebot calibrate_intrinsics ![HOST_NAME] --base_image  duckietown/rpi-duckiebot-base:master19-no-arm@sha256:bcefefe0e249b8b1fcde21b3eaa6c7af5737fbf92003854376be9541a97257a2

*info it may take some time until the image is completely downloaded*

##### Camera extrinsic
Then run the calibration using the command:

Daffy

    laptop $ dts duckiebot calibrate_extrinsics ![HOST_NAME] --base_image  duckietown/dt-core:daffy-amd64@sha256:d72e8a8c3191c146ecc2a812bdd036aaf15076e6c1cb9467304e0e54f9a39a10

Master19

    laptop $ dts duckiebot calibrate_intrinsics ![HOST_NAME] --base_image  duckietown/rpi-duckiebot-base:master19-no-arm@sha256:bcefefe0e249b8b1fcde21b3eaa6c7af5737fbf92003854376be9541a97257a2


#### Wheels
Use thew more detailed explaination [https://docs.duckietown.org/daffy/opmanual_duckiebot/out/wheel_calibration.html](here)

Use the `gui_tools` to connect to the ROS: 

    laptop $ dts start_gui_tools ![HOST_NAME]

the use (in a different terminal) the description from the verification paragraph in order to start the keyboard control. 

Then use this command in the `gui_tools` in order to calibrate the bot. Adjust the `TRIM_VALUE` in order to do so. **Make sure the *weels can run freely* and that the bot drives straight within 10 cm on 2m.**

daffy

    container $ rosparam set /![HOST_NAME]/kinematics_node/trim TRIM_VALUE
master19

    container $ rosservice call /![HOST_NAME]/inverse_kinematics_node/set_trim -- TRIM_VALUE
