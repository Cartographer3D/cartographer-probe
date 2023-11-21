# Cartographer 3D - Home of the Cartographer Probe
The Cartographer Probe, democratising Eddy Current Sensors. A huge thank you to the community, and I hope you enjoy this product. 

## Getting Started


### Mount Cartographer
Cartographer is designed to use the existing mounts available for similar products on the market. The hole spacing is 31.6mm and it takes M3 bolts. 

Ensure when you are mounting Cartographer that you recess the probe approximatly 2.5mm from the Nozzles Z height. 

### Cable Routing
Routing the USB Cable is essential, the provided cable is NOT rated for being routed through cable chains, please mount either along your Bowden or Umbilical path. 

### Install Cartographer Klipper Module 
Clone the Klipper module from GitHub using the following commands, you then need to run our installation script. 

```bash
git clone https://github.com/Cartographer3D/cartographer-klipper.git
chmod +x cartographer-klipper/install.sh
./cartographer-klipper/install.sh
```
This step will automatically create a link to the script and place it in the klipper/klipper/extra directory.

### Printer Config

Add the following configuration to the top of your Printer.cfg file. Note, you need to replace the serial path with your probes serial path, this can be found by running the following command. 
`ls /dev/serial/by-id/`

V1: For the RP2040 version of the Cartographer3D please add the following config:

```yaml
[idm]
serial:
#   Path to the serial port for the idm device. Typically has the form
#   /dev/serial/by-id/usb-idm_idm_...
speed: 40.
#   Z probing dive speed.
lift_speed: 5.
#   Z probing lift speed.
backlash_comp: 0.5
#   Backlash compensation distance for removing Z backlash before measuring
#   the sensor response.
x_offset: 0.
#   X offset of idm from the nozzle.
y_offset: 21.1
#   Y offset of idm from the nozzle.
trigger_distance: 2.
#   idm trigger distance for homing.
trigger_dive_threshold: 1.5
#   Threshold for range vs dive mode probing. Beyond `trigger_distance +
#   trigger_dive_threshold` a dive will be used.
trigger_hysteresis: 0.006
#   Hysteresis on trigger threshold for untriggering, as a percentage of the
#   trigger threshold.
cal_nozzle_z: 0.1
#   Expected nozzle offset after completing manual Z offset calibration.
cal_floor: 0.1
#   Minimum z bound on sensor response measurement.
cal_ceil:5.
#   Maximum z bound on sensor response measurement.
cal_speed: 1.0
#   Speed while measuring response curve.
cal_move_speed: 10.
#   Speed while moving to position for response curve measurement.
default_model_name: default
#   Name of default idm model to load.
mesh_main_direction: x
#   Primary travel direction during mesh measurement.
#mesh_overscan: -1
#   Distance to use for direction changes at mesh line ends. Omit this setting
#   and a default will be calculated from line spacing and available travel.
mesh_cluster_size: 1
#   Radius of mesh grid point clusters.
mesh_runs: 2
#   Number of passes to make during mesh scan.
```

V2: For the Input Shaper version of the Cartographer3D please add the following config, for the CAN version, run this command to find your boards UUID:

```bash
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```


```yaml
[cartographer]
serial:
#   Path to the serial port for the Cartographer device. Typically has the form
#   /dev/serial/by-id/usb-idm_idm_...
#   
#   If you are using the CAN Bus version, replace serial: with canbus_uuid: and add the UUID.
#   Example: canbus_uuid: 1283as878a9sd
#
speed: 40.
#   Z probing dive speed.
lift_speed: 5.
#   Z probing lift speed.
backlash_comp: 0.5
#   Backlash compensation distance for removing Z backlash before measuring
#   the sensor response.
x_offset: 0.
#   X offset of idm from the nozzle.
y_offset: 21.1
#   Y offset of idm from the nozzle.
trigger_distance: 2.
#   idm trigger distance for homing.
trigger_dive_threshold: 1.5
#   Threshold for range vs dive mode probing. Beyond `trigger_distance +
#   trigger_dive_threshold` a dive will be used.
trigger_hysteresis: 0.006
#   Hysteresis on trigger threshold for untriggering, as a percentage of the
#   trigger threshold.
cal_nozzle_z: 0.1
#   Expected nozzle offset after completing manual Z offset calibration.
cal_floor: 0.1
#   Minimum z bound on sensor response measurement.
cal_ceil:5.
#   Maximum z bound on sensor response measurement.
cal_speed: 1.0
#   Speed while measuring response curve.
cal_move_speed: 10.
#   Speed while moving to position for response curve measurement.
default_model_name: default
#   Name of default idm model to load.
mesh_main_direction: x
#   Primary travel direction during mesh measurement.
#mesh_overscan: -1
#   Distance to use for direction changes at mesh line ends. Omit this setting
#   and a default will be calculated from line spacing and available travel.
mesh_cluster_size: 1
#   Radius of mesh grid point clusters.
mesh_runs: 2
#   Number of passes to make during mesh scan.
```

You then need to remove, or comment out your `[probe]` section. 

Now add a safe_z_home section with the following inforamation. 

```yaml 
[safe_z_home]
home_xy_position: [your x-axis center coordinate], [your y-axis center coordinate]
# Example home_xy_position: 175,175 - This would be for a 350 * 350mm bed. 
z_hop: 10
```

You will also need to update your Z configuration settings. 

```yaml
[stepper_z]
endstop_pin: probe:z_virtual_endstop # use cartographer as virtual endstop
homing_retract_dist: 0 # cartographer needs this to be set to 0
```

If you purchased the model with Input Shaper, please add the following lines to your configuration
```yaml
[lis2dw]
cs_pin: cartographer:PA3
spi_bus: spi1

[resonance_tester]
accel_chip: lis2dw
probe_points:
    125, 125, 20
```

Finally, you need to ensure you have a suitable bed_mesh section. Information can be found [here](https://www.klipper3d.org/Bed_Mesh.html)

### Calibrate Cartographer

Home the machine in X and Y:
```gcode
G28 X Y
```

You will now need to position the nozzle at the center of the bed, if you have a 300 x 300 bed use the following: 
```gcode
G0 X150 Y150
```

Start the calibration process: 

V1: RP2040 Based
```
IDM_CALIBRATE
```

V2: USB or CAN w/ Input Shaper
```
CARTOGRAPHER_CALIBRATE
```
You can either use the web interface to adjust the nozzle height from the bed, or `TESTZ Z=-0.01` to lower it. Use a piece of paper to measure the offset. Once finished remove the paper and accept the position:
```
ACCEPT
```
Save the results to your config file:
```
SAVE_CONFIG
```
### Initial Tests

Home your Z (You can also just home all axis)@
```
G28 Z
```
You can test the accuracy:
```
PROBE_ACCURACY
```

You can also measure the backlash of your Z axis

V1: RP2040 Based
```
IDM_ESTIMATE_BACKLASH
```

V2: USB or CAN w/ Input Shaper
```
CARTOGRAPHER_ESTIMATE_BACKLASH
```
You can now run a Bed Mesh Calibration 
```
BED_MESH_CALIBRATE
```


