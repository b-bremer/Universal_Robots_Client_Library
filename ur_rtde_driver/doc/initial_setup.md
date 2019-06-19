# Setting up a UR robot for ur_rtde_driver
## Prepare the robot
For using the *ur_rtde_driver* with a real robot you need to install the
**externalcontrol-1.0.urcap** which can be found inside the **resources** folder of this driver.

To install it you first have to copy it to the robot's **programs** folder which can be done either
via scp or using a USB stick. The installation process is similar for CB3 and eSeries robots and
will be shown side-to-side in this guide.

<tr>
<td> <img src="initial_setup_images/cb3_01_welcome.png" alt="Welcome screen of CB3" style="width: 45%;"/> </td>
<td> <img src="initial_setup_images/es_01_welcome.png" alt="Welcome screen of eSeries" style="width: 45%;"/> </td>
</tr>

On the welcome screen select *Setup Robot* and then *URCaps* to enter the URCaps installation screen
(For eSeries click on the hamburger menu in the top-right corner to get to the setup menu). There,
click the little plus sign at the bottom to open the file selector. There you should see all urcap
files stored inside the robot's programs folder or a plugged USB drive.  Select and open the
**externalcontrol-1.0.urcap** file and click *open*. Your URCaps view should now show the
**External Control** in the list of active URCaps and a notification to restart the robot. Do that
now.

<tr>
<td> <img src="initial_setup_images/cb3_05_urcaps_installed.png" alt="URCaps screen with installed
urcaps" style="width: 45%;"/> </td>
<td> <img src="initial_setup_images/es_05_urcaps_installed.png" alt="URCaps screen with installed
urcaps" style="width: 45%;"/> </td>
</tr>

After the reboot you should find the **External Control** URCaps inside the *Installation* section.
For this select *Program Robot* on the welcome screen, select the *Installation* tab and select
**External control** from the list.

<tr>
<td> <img src="initial_setup_images/cb3_07_installation_excontrol.png" alt="Installation screen of URCaps" style="width: 45%;"/> </td>
<td> <img src="initial_setup_images/es_07_installation_excontrol.png" alt="Installation screen of URCaps" style="width: 45%;"/> </td>
</tr>

Here you'll have to setup the IP address of the external PC which will be running the ROS driver.
Note that the robot and the external PC have to be in the same network, ideally in a direct
connection with each other to minimize network disturbances. The custom port should be left
untouched for now.

To use the new URCaps, create a new program and insert the **External Control** program node into
the program tree:

<tr>
<td> <img src="initial_setup_images/cb3_10_prog_structure_urcaps.png" alt="Insert the external control node" style="width: 45%;"/> </td>
<td> <img src="initial_setup_images/es_10_prog_structure_urcaps.png" alt="Insert the external control node" style="width: 45%;"/> </td>
</tr>

If you click on the *command* tab again, you'll see the settings entered inside the *Installation*.
Check that they are correct, then save the program.

<tr>
<td> <img src="initial_setup_images/cb3_11_program_view_excontrol.png" alt="Program view of external control" style="width: 45%;"/> </td>
<td> <img src="initial_setup_images/es_11_program_view_excontrol.png" alt="Program view of external control" style="width: 45%;"/> </td>
</tr>

## Prepare the ROS PC
For using the driver make sure it is installed (either by the debian package or built from source
inside a catkin workspace).

### Extract calibration information
Each UR robot is calibrated inside the factory giving exact forward and inverse kinematics. To also
make use of this in ROS, you first have to extract the calibration information from the robot.

Though this step is not necessary, to control the robot using this driver, it is highly recommended
to do so, as endeffector positions might be off in the magnitude of centimeters.

For this, there exists a helper script:

    $ rosrun ur_calibration calibration_correction \
    _robot_ip:=192.168.56.101 \
    _robot_name:=ur10_example \
    _output_package_name:=my_calibrations \
    _subfolder_name:=etc

For the parameter **robot_ip** insert the ip on which the ROS pc can reach the robot. The
**robot_name** is an arbitrary name you can give to the robot. It is recommended, to choose a unique
name that can be easily matched to the physical robot.

The script will then extract the calibration information from the robot and convert it to a yaml
syntax that can be used by the robot_description.

The resulting yaml file is stored in the package specified in the **output_package_name** parameter
inside the folder **subfolder_name** with the name **robot_name***_calibration.yaml*. The parameter
**subfolder_name** is optional and defaults to *etc* if not given.

**Note:** You'll have to provide the name of an existing package. It is recommended to have a
package storing all calibrations of all UR robots inside your organization. This way, the extraction
as described in this package has only to be performed once and the calibration can be reused by
other users.
To create a new package, go to your catkin_workspace's src folder and call

    catkin_create_pkg my_calibrations

It is recommended to adapt the new package's *package.xml* with a meaningful description.

### Start the robot driver
To actually start the robot driver use one of the existing launchfiles

    $ roslaunch ur_rtde_driver <robot_type>_bringup.launch robot_ip:=192.168.56.101 \
    kinematics_config:=$(rospack find my_calibrations)/etc/ur10_example_calibration.yaml

where **<robot_type>** is one of *ur3, ur5, ur10, ur3e, ur5e, ur10e*. Note that in this example we
load the calibration parameters for the robot "ur10_example". If the parameters in that file don't
match the ones reported from the robot, the driver will output an error during startup.

For more information on the launchfile's parameters see its own documentation.

Once the robot driver is started, load the previously generated program on the robot panel and
execute it. From that moment on the robot is fully functional. You can make use of the pause
function or even stop the program. Simply press the play button again and the ROS driver will
reconnect.