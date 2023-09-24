# sovol-sv04-notes

This is Tim's commentary on Sovol's customizations of the Marlin firmware for their Sovol SV04 IDEX printer.

They are based on version 2.0.9.2 of Marlin: https://github.com/MarlinFirmware/Marlin/tree/2.0.9.2

Sovol's github is: https://github.com/Sovol3d/SV04-IDEX-3D-Printer-Mainboard-Source-code

They haven't made any changes since April 2023. This is all based on `1867fcb` as Sovol's latest commit.

It's confusing because they changed the version the source to say it's version 1.1.0.

They posted this on Facebook, on January 9th, 2022 when they released it:

> Dears, the SV04 IDEX firmware is updated. Please download it here.
> https://sovol3d.com/pages/download
> There are files about introductions of the new firmware and how to flash the two firmware. The latest firmware is based on Marlin 2.0.9.2& 2.0.9.3 and it's named V1.0.9, and the corresponding touch screen firmware version is V2.0. 
> It fixes the possible layer shift problem caused by the Marlin2.0.7 stepper DIR signal bug on IDEX machines, improves the G34 auto Z align function, and adds a save button for Z offset, X/Y offset, z offset during printing; With it, you need to click the save button to save the adjusted values. 
> Comment below if you have any questions.

I generated a diff with this command `git diff --stat -p -w -M -C -D 3b1b64aac71d159~1...HEAD`
after first grafting Sovol's git repo onto Marlin's official git repo with `git replace --graft`.

```sh
git replace -l --format=long
3b1b64aac71d159dbedec0af0b4fe1d18a896723 (commit) -> 33544ceeb4b1b543c39729a4c99fe28d1c1be293 (commit)
```

## diffstat
First we'll go over the diffstat.

Some files had no lines change, just different permissions and I omitted those.

### Removing github files
It makes sense they would remove all the `.github` files. Well, it would
make more sense to replace them with their own. But I guess they aren't
interested in accepting PR's, contributations, etc.

```diff
 .github/FUNDING.yml                                |    3 -
 .github/ISSUE_TEMPLATE/bug_report.yml              |  169 --
 .github/ISSUE_TEMPLATE/config.yml                  |   20 -
 .github/ISSUE_TEMPLATE/feature_request.yml         |   44 -
 .github/code_of_conduct.md                         |   46 -
 .github/contributing.md                            |  143 -
 .github/issue_template.md                          |   35 -
 .github/lock.yml                                   |   40 -
 .github/pull_request_template.md                   |   33 -
 .github/workflows/bump-date.yml                    |   36 -
 .github/workflows/check-pr.yml                     |   32 -
 .github/workflows/clean-closed.yml                 |   39 -
 .github/workflows/close-stale.yml                  |   28 -
 .github/workflows/lock-closed.yml                  |   32 -
 .github/workflows/test-builds.yml                  |  145 -
 .github/workflows/unlock-reopened.yml              |   22 -
 ```

### Marlin configuration
 Of course they customized these files which are used to configure Marlin.
 Ideally this would be the only customization. But of course, there's
 a lot more going on.
 ```
 Marlin/Configuration.h                             |  163 +-
 Marlin/Configuration_adv.h                         |   52 +-
 ````
 
 And the rest!

 For a lot of these with just a few lines changed, they added code
 to talk to the touch screen, or to customize something with IDEX.

 `Version.h` is weird, where they changed the version to an older one.

`LCD_RTS.cpp` is the biggest additional, which controls the touch screen.

After scanning through that file, I'm starting to understand why every firmware update needs an separate update to the touchware firmware. Rather than sending the touch screen information, or the touch screen sending G-Code commands, it seems like it tells the touch screen which screen to display, and the touch screen meanwhile tells it which square was touched. Which makes them highly coupled.


 ```diff
 Marlin/src/HAL/STM32/Sd2Card_sdio_stm32duino.cpp   |    3 +-
 Marlin/src/MarlinCore.cpp                          |   38 +-
 Marlin/src/MarlinCore.h                            |    2 +
 Marlin/src/feature/bedlevel/abl/abl.h              |    1 +
 Marlin/src/feature/pause.cpp                       |   38 +-
 Marlin/src/feature/powerloss.cpp                   |   51 +-
 Marlin/src/feature/powerloss.h                     |    3 +
 Marlin/src/gcode/bedlevel/abl/G29.cpp              |   21 +-
 Marlin/src/gcode/calibrate/G28.cpp                 |   20 +-
 Marlin/src/gcode/calibrate/G34_M422.cpp            |   13 +
 Marlin/src/gcode/control/M605.cpp                  |    2 +-
 Marlin/src/gcode/control/T.cpp                     |    3 +-
 Marlin/src/gcode/feature/pause/M600.cpp            |   10 +-
 Marlin/src/gcode/motion/G0_G1.cpp                  |    7 +-
 Marlin/src/gcode/queue.cpp                         |   22 +-
 Marlin/src/gcode/sd/M24_M25.cpp                    |    2 +-
 Marlin/src/gcode/temp/M106_M107.cpp                |   16 +-
 Marlin/src/inc/Conditionals_LCD.h                  |   16 +-
 Marlin/src/inc/Conditionals_post.h                 |    6 +-
 Marlin/src/inc/SanityCheck.h                       |    2 +-
 Marlin/src/inc/Version.h                           |   10 +-
 Marlin/src/lcd/e3v2/common/encoder.cpp             |    2 +-
 Marlin/src/lcd/e3v2/creality/LCD_RTS.cpp           | 2917 ++++++++++++++++++++
 Marlin/src/lcd/e3v2/creality/LCD_RTS.h             |  307 ++
 Marlin/src/lcd/marlinui.cpp                        |    8 +-
 Marlin/src/module/endstops.cpp                     |   10 +-
 Marlin/src/module/motion.cpp                       |   16 +-
 Marlin/src/module/motion.h                         |    2 +-
 Marlin/src/module/planner.cpp                      |   16 +-
 Marlin/src/module/settings.cpp                     |   41 +-
 Marlin/src/module/temperature.cpp                  |   78 +-
 Marlin/src/pins/stm32f1/pins_CREALITY_V4.h         |  149 +-
 Marlin/src/sd/cardreader.cpp                       |   16 +-
 README.md                                          |  114 +-
 buildroot/share/PlatformIO/scripts/exc.S           |  104 -
 buildroot/share/PlatformIO/scripts/random-bin.py   |    2 +-
 .../variants/marlin_CHITU_F103/wirish/start.S      |   57 -
 .../variants/marlin_MEEB_3DP/wirish/start.S        |   57 -
 buildroot/share/cmake/CMakeLists.txt               |  123 -
 ini/features.ini                                   |    1 +
 ini/stm32f1-maple.ini                              |    2 +-
 ini/stm32f1.ini                                    |    7 +-
 platformio.ini                                     |    2 +-
 185 files changed, 3863 insertions(+), 1536 deletions(-)
```

## Full diff

Here's the details. I'll omit files that were deleted or only had their permissions changed.

### Configuration

- Set some version strings
- turn off SHOW_BOOTSCREEN
- motherboard: BOARD_CREALITY_V4 (but is it really?)
- extruders: 2
- `#define HOTEND_OFFSET_X { 0.0, 188.00 } // (mm) relative X-offset for each nozzle` why 188mm? When parked, T0 is at -62 and T1 at 362. So this makes no sense to me, either it's nonsense or I don't understand what it's actually used for.
- they comment out the _DRIVER_TYPE lines while also changing them to TMC2208
- they really lower the default acceleration (from 3000 to 500)
- they turn on classic jerk
- define the X and Y bed size to 302 by 302
- define X_MIN_POS to -62
- max Z is 400
- bed leveling is bilinear
- restore level after G28
- fade height is 1mm (default is 10mm)
- grid points are 4
- extrapolate beyond grid is on
- NOZZLE_PARK_FEATURE is on, max rate 50


```diff
diff --git a/Marlin/Configuration.h b/Marlin/Configuration.h
index c50af25e63..b226c55234 100644
--- a/Marlin/Configuration.h
+++ b/Marlin/Configuration.h
@@ -69,8 +69,12 @@
 // @section info
 
 // Author info of this build printed to the host during boot and M115
-#define STRING_CONFIG_H_AUTHOR "(none, default config)" // Who made the changes.
-//#define CUSTOM_VERSION_FILE Version.h // Path from the root directory (no quotes)
+#define STRING_CONFIG_H_AUTHOR "SV04" // Who made the changes.
+#define CUSTOM_VERSION_FILE Version.h // Path from the root directory (no quotes)
+#define MACVERSION      STRING_CONFIG_H_AUTHOR
+#define SOFTVERSION     SHORT_BUILD_VERSION
+#define CORP_WEBSITE    "www.sovol3d.com"
+#define Screen_version  "Screen: V2.0"
 
 /**
  * *** VENDORS PLEASE READ ***
@@ -84,7 +88,7 @@
  */
 
 // Show the Marlin bootscreen on startup. ** ENABLE FOR PRODUCTION **
-#define SHOW_BOOTSCREEN
+//#define SHOW_BOOTSCREEN
 
 // Show the bitmap in Marlin/_Bootscreen.h on startup.
 //#define SHOW_CUSTOM_BOOTSCREEN
@@ -102,7 +106,7 @@
  *
  * :[-1, 0, 1, 2, 3, 4, 5, 6, 7]
  */
-#define SERIAL_PORT 0
+#define SERIAL_PORT 1
 
 /**
  * Serial Port Baud Rate
@@ -115,7 +119,7 @@
  *
  * :[2400, 9600, 19200, 38400, 57600, 115200, 250000, 500000, 1000000]
  */
-#define BAUDRATE 250000
+#define BAUDRATE 115200
 //#define BAUD_RATE_GCODE     // Enable G-code M575 to set the baud rate
 
 /**
@@ -123,7 +127,7 @@
  * Currently Ethernet (-2) is only supported on Teensy 4.1 boards.
  * :[-2, -1, 0, 1, 2, 3, 4, 5, 6, 7]
  */
-//#define SERIAL_PORT_2 -1
+//#define SERIAL_PORT_2 3
 //#define BAUDRATE_2 250000   // Enable to override BAUDRATE
 
 /**
@@ -139,7 +143,7 @@
 
 // Choose the name from boards.h that matches your setup
 #ifndef MOTHERBOARD
-  #define MOTHERBOARD BOARD_RAMPS_14_EFB
+  #define MOTHERBOARD BOARD_CREALITY_V4
 #endif
 
 // Name displayed in the LCD "Ready" message and Info menu
@@ -192,7 +196,7 @@
 
 // This defines the number of extruders
 // :[0, 1, 2, 3, 4, 5, 6, 7, 8]
-#define EXTRUDERS 1
+#define EXTRUDERS 2
 
 // Generally expected filament diameter (1.75, 2.85, 3.0, ...). Used for Volumetric, Filament Width Sensor, etc.
 #define DEFAULT_NOMINAL_FILAMENT_DIA 1.75
@@ -351,9 +355,9 @@
 // Offset of the extruders (uncomment if using more than one and relying on firmware to position when changing).
 // The offset has to be X=0, Y=0 for the extruder 0 hotend (default extruder).
 // For the other hotends it is their distance from the extruder 0 hotend.
-//#define HOTEND_OFFSET_X { 0.0, 20.00 } // (mm) relative X-offset for each nozzle
-//#define HOTEND_OFFSET_Y { 0.0, 5.00 }  // (mm) relative Y-offset for each nozzle
-//#define HOTEND_OFFSET_Z { 0.0, 0.00 }  // (mm) relative Z-offset for each nozzle
+#define HOTEND_OFFSET_X { 0.0, 188.00 } // (mm) relative X-offset for each nozzle
+#define HOTEND_OFFSET_Y { 0.0, 0.00 }  // (mm) relative Y-offset for each nozzle
+#define HOTEND_OFFSET_Z { 0.0, 0.00 }  // (mm) relative Z-offset for each nozzle
 
 // @section machine
 
@@ -488,14 +492,14 @@
  *
  */
 #define TEMP_SENSOR_0 1
-#define TEMP_SENSOR_1 0
+#define TEMP_SENSOR_1 1
 #define TEMP_SENSOR_2 0
 #define TEMP_SENSOR_3 0
 #define TEMP_SENSOR_4 0
 #define TEMP_SENSOR_5 0
 #define TEMP_SENSOR_6 0
 #define TEMP_SENSOR_7 0
-#define TEMP_SENSOR_BED 0
+#define TEMP_SENSOR_BED 1
 #define TEMP_SENSOR_PROBE 0
 #define TEMP_SENSOR_CHAMBER 0
 #define TEMP_SENSOR_COOLER 0
@@ -513,15 +517,15 @@
 //#define MAX31865_CALIBRATION_OHMS_1 430
 
 #define TEMP_RESIDENCY_TIME         10  // (seconds) Time to wait for hotend to "settle" in M109
-#define TEMP_WINDOW                  1  // (°C) Temperature proximity for the "temperature reached" timer
+#define TEMP_WINDOW                  2  // (°C) Temperature proximity for the "temperature reached" timer
 #define TEMP_HYSTERESIS              3  // (°C) Temperature proximity considered "close enough" to the target
 
 #define TEMP_BED_RESIDENCY_TIME     10  // (seconds) Time to wait for bed to "settle" in M190
-#define TEMP_BED_WINDOW              1  // (°C) Temperature proximity for the "temperature reached" timer
+#define TEMP_BED_WINDOW              2  // (°C) Temperature proximity for the "temperature reached" timer
 #define TEMP_BED_HYSTERESIS          3  // (°C) Temperature proximity considered "close enough" to the target
 
 #define TEMP_CHAMBER_RESIDENCY_TIME 10  // (seconds) Time to wait for chamber to "settle" in M191
-#define TEMP_CHAMBER_WINDOW          1  // (°C) Temperature proximity for the "temperature reached" timer
+#define TEMP_CHAMBER_WINDOW          2  // (°C) Temperature proximity for the "temperature reached" timer
 #define TEMP_CHAMBER_HYSTERESIS      3  // (°C) Temperature proximity considered "close enough" to the target
 
 /**
@@ -564,7 +568,7 @@
 #define HEATER_5_MAXTEMP 275
 #define HEATER_6_MAXTEMP 275
 #define HEATER_7_MAXTEMP 275
-#define BED_MAXTEMP      150
+#define BED_MAXTEMP      125
 #define CHAMBER_MAXTEMP  60
 
 /**
@@ -589,8 +593,8 @@
 #define PID_K1 0.95      // Smoothing factor within any PID loop
 
 #if ENABLED(PIDTEMP)
-  //#define PID_EDIT_MENU         // Add PID editing to the "Advanced Settings" menu. (~700 bytes of PROGMEM)
-  //#define PID_AUTOTUNE_MENU     // Add PID auto-tuning to the "Advanced Settings" menu. (~250 bytes of PROGMEM)
+  #define PID_EDIT_MENU         // Add PID editing to the "Advanced Settings" menu. (~700 bytes of PROGMEM)
+  #define PID_AUTOTUNE_MENU     // Add PID auto-tuning to the "Advanced Settings" menu. (~250 bytes of PROGMEM)
   //#define PID_PARAMS_PER_HOTEND // Uses separate PID parameters for each extruder (useful for mismatched extruders)
                                   // Set/get with gcode: M301 E[extruder number, 0-2]
 
@@ -601,9 +605,9 @@
     #define DEFAULT_Ki_LIST {   1.08,   1.08 }
     #define DEFAULT_Kd_LIST { 114.00, 114.00 }
   #else
-    #define DEFAULT_Kp  22.20
-    #define DEFAULT_Ki   1.08
-    #define DEFAULT_Kd 114.00
+    #define DEFAULT_Kp  23.81
+    #define DEFAULT_Ki   1.93
+    #define DEFAULT_Kd  73.64
   #endif
 #endif // PIDTEMP
 
@@ -624,7 +628,7 @@
  * heater. If your configuration is significantly different than this and you don't understand
  * the issues involved, don't use bed PID until someone else verifies that your hardware works.
  */
-//#define PIDTEMPBED
+#define PIDTEMPBED
 
 //#define BED_LIMIT_SWITCHING
 
@@ -638,13 +642,13 @@
 
 #if ENABLED(PIDTEMPBED)
   //#define MIN_BED_POWER 0
-  //#define PID_BED_DEBUG // Sends debug data to the serial port.
+  #define PID_BED_DEBUG // Sends debug data to the serial port.
 
   // 120V 250W silicone heater into 4mm borosilicate (MendelMax 1.5+)
   // from FOPDT model - kp=.39 Tp=405 Tdead=66, Tc set to 79.2, aggressive factor of .15 (vs .1, 1, 10)
-  #define DEFAULT_bedKp 10.00
-  #define DEFAULT_bedKi .023
-  #define DEFAULT_bedKd 305.4
+  #define DEFAULT_bedKp 312.84
+  #define DEFAULT_bedKi 52.04
+  #define DEFAULT_bedKd 1253.64
 
   // FIND YOUR OWN: "M303 E-1 C8 S90" to run autotune on the bed at 90 degreesC for 8 cycles.
 #endif // PIDTEMPBED
@@ -718,7 +722,7 @@
  * Note: For Bowden Extruders make this large enough to allow load/unload.
  */
 #define PREVENT_LENGTHY_EXTRUDE
-#define EXTRUDE_MAXLENGTH 200
+#define EXTRUDE_MAXLENGTH 1000
 
 //===========================================================================
 //======================== Thermal Runaway Protection =======================
@@ -739,8 +743,8 @@
 
 #define THERMAL_PROTECTION_HOTENDS // Enable thermal protection for all extruders
 #define THERMAL_PROTECTION_BED     // Enable thermal protection for the heated bed
-#define THERMAL_PROTECTION_CHAMBER // Enable thermal protection for the heated chamber
-#define THERMAL_PROTECTION_COOLER  // Enable thermal protection for the laser cooling
+//#define THERMAL_PROTECTION_CHAMBER // Enable thermal protection for the heated chamber
+//#define THERMAL_PROTECTION_COOLER  // Enable thermal protection for the laser cooling
 
 //===========================================================================
 //============================= Mechanical Settings =========================
@@ -783,7 +787,7 @@
 //#define USE_IMIN_PLUG
 //#define USE_JMIN_PLUG
 //#define USE_KMIN_PLUG
-//#define USE_XMAX_PLUG
+#define USE_XMAX_PLUG
 //#define USE_YMAX_PLUG
 //#define USE_ZMAX_PLUG
 //#define USE_IMAX_PLUG
@@ -829,13 +833,14 @@
 #endif
 
 // Mechanical endstop with COM to ground and NC to Signal uses "false" here (most common setup).
-#define X_MIN_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
-#define Y_MIN_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
+#define X_MIN_ENDSTOP_INVERTING true // Set to true to invert the logic of the endstop.
+#define Y_MIN_ENDSTOP_INVERTING true // Set to true to invert the logic of the endstop.
 #define Z_MIN_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
+#define Z2_MIN_ENDSTOP_INVERTING true // Set to true to invert the logic of the Z2 endstop.
 #define I_MIN_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
 #define J_MIN_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
 #define K_MIN_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
-#define X_MAX_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
+#define X_MAX_ENDSTOP_INVERTING true // Set to true to invert the logic of the endstop.
 #define Y_MAX_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
 #define Z_MAX_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
 #define I_MAX_ENDSTOP_INVERTING false // Set to true to invert the logic of the endstop.
@@ -861,9 +866,9 @@
  *          TMC5130, TMC5130_STANDALONE, TMC5160, TMC5160_STANDALONE
  * :['A4988', 'A5984', 'DRV8825', 'LV8729', 'L6470', 'L6474', 'POWERSTEP01', 'TB6560', 'TB6600', 'TMC2100', 'TMC2130', 'TMC2130_STANDALONE', 'TMC2160', 'TMC2160_STANDALONE', 'TMC2208', 'TMC2208_STANDALONE', 'TMC2209', 'TMC2209_STANDALONE', 'TMC26X', 'TMC26X_STANDALONE', 'TMC2660', 'TMC2660_STANDALONE', 'TMC5130', 'TMC5130_STANDALONE', 'TMC5160', 'TMC5160_STANDALONE']
  */
-#define X_DRIVER_TYPE  A4988
-#define Y_DRIVER_TYPE  A4988
-#define Z_DRIVER_TYPE  A4988
+//#define X_DRIVER_TYPE  TMC2208
+//#define Y_DRIVER_TYPE  TMC2208
+//#define Z_DRIVER_TYPE  TMC2208
 //#define X2_DRIVER_TYPE A4988
 //#define Y2_DRIVER_TYPE A4988
 //#define Z2_DRIVER_TYPE A4988
@@ -872,7 +877,7 @@
 //#define I_DRIVER_TYPE  A4988
 //#define J_DRIVER_TYPE  A4988
 //#define K_DRIVER_TYPE  A4988
-#define E0_DRIVER_TYPE A4988
+//#define E0_DRIVER_TYPE TMC2208
 //#define E1_DRIVER_TYPE A4988
 //#define E2_DRIVER_TYPE A4988
 //#define E3_DRIVER_TYPE A4988
@@ -883,7 +888,7 @@
 
 // Enable this feature if all enabled endstop pins are interrupt-capable.
 // This will remove the need to poll the interrupt pins, saving many CPU cycles.
-//#define ENDSTOP_INTERRUPTS_FEATURE
+#define ENDSTOP_INTERRUPTS_FEATURE
 
 /**
  * Endstop Noise Threshold
@@ -920,21 +925,21 @@
  * following movement settings. If fewer factors are given than the
  * total number of extruders, the last value applies to the rest.
  */
-//#define DISTINCT_E_FACTORS
+#define DISTINCT_E_FACTORS
 
 /**
  * Default Axis Steps Per Unit (steps/mm)
  * Override with M92
  *                                      X, Y, Z [, I [, J [, K]]], E0 [, E1[, E2...]]
  */
-#define DEFAULT_AXIS_STEPS_PER_UNIT   { 80, 80, 400, 500 }
+#define DEFAULT_AXIS_STEPS_PER_UNIT   { 64, 80, 400, 415, 415 }
 
 /**
  * Default Max Feed Rate (mm/s)
  * Override with M203
  *                                      X, Y, Z [, I [, J [, K]]], E0 [, E1[, E2...]]
  */
-#define DEFAULT_MAX_FEEDRATE          { 300, 300, 5, 25 }
+#define DEFAULT_MAX_FEEDRATE          { 500, 500, 5, 25, 25 }
 
 //#define LIMITED_MAX_FR_EDITING        // Limit edit via M203 or LCD to DEFAULT_MAX_FEEDRATE * 2
 #if ENABLED(LIMITED_MAX_FR_EDITING)
@@ -947,7 +952,7 @@
  * Override with M201
  *                                      X, Y, Z [, I [, J [, K]]], E0 [, E1[, E2...]]
  */
-#define DEFAULT_MAX_ACCELERATION      { 3000, 3000, 100, 10000 }
+#define DEFAULT_MAX_ACCELERATION      { 500, 500, 100, 5000,5000 }
 
 //#define LIMITED_MAX_ACCEL_EDITING     // Limit edit via M201 or LCD to DEFAULT_MAX_ACCELERATION * 2
 #if ENABLED(LIMITED_MAX_ACCEL_EDITING)
@@ -962,9 +967,9 @@
  *   M204 R    Retract Acceleration
  *   M204 T    Travel Acceleration
  */
-#define DEFAULT_ACCELERATION          3000    // X, Y, Z and E acceleration for printing moves
-#define DEFAULT_RETRACT_ACCELERATION  3000    // E acceleration for retracts
-#define DEFAULT_TRAVEL_ACCELERATION   3000    // X, Y, Z acceleration for travel (non printing) moves
+#define DEFAULT_ACCELERATION          500    // X, Y, Z and E acceleration for printing moves
+#define DEFAULT_RETRACT_ACCELERATION  500    // E acceleration for retracts
+#define DEFAULT_TRAVEL_ACCELERATION   500    // X, Y, Z acceleration for travel (non printing) moves
 
 /**
  * Default Jerk limits (mm/s)
@@ -974,7 +979,7 @@
  * When changing speed and direction, if the difference is less than the
  * value set here, it may happen instantaneously.
  */
-//#define CLASSIC_JERK
+#define CLASSIC_JERK
 #if ENABLED(CLASSIC_JERK)
   #define DEFAULT_XJERK 10.0
   #define DEFAULT_YJERK 10.0
@@ -1033,7 +1038,7 @@
 #define Z_MIN_PROBE_USES_Z_MIN_ENDSTOP_PIN
 
 // Force the use of the probe for Z-axis homing
-//#define USE_PROBE_FOR_Z_HOMING
+#define USE_PROBE_FOR_Z_HOMING
 
 /**
  * Z_MIN_PROBE_PIN
@@ -1087,7 +1092,7 @@
 /**
  * The BLTouch probe uses a Hall effect sensor and emulates a servo.
  */
-//#define BLTOUCH
+#define BLTOUCH
 
 /**
  * Touch-MI Probe by hotends.fr
@@ -1179,11 +1184,11 @@
  *     |    [-]    |
  *     O-- FRONT --+
  */
-#define NOZZLE_TO_PROBE_OFFSET { 10, 10, 0 }
+#define NOZZLE_TO_PROBE_OFFSET { 0, 25, 0 }
 
 // Most probes should stay away from the edges of the bed, but
 // with NOZZLE_AS_PROBE this can be negative for a wider probing area.
-#define PROBING_MARGIN 10
+#define PROBING_MARGIN 25
 
 // X and Y axis travel speed (mm/min) between probes
 #define XY_PROBE_FEEDRATE (133*60)
@@ -1321,9 +1326,9 @@
 // @section machine
 
 // Invert the stepper direction. Change (or reverse the motor connector) if an axis goes the wrong way.
-#define INVERT_X_DIR false
-#define INVERT_Y_DIR true
-#define INVERT_Z_DIR false
+#define INVERT_X_DIR true
+#define INVERT_Y_DIR false
+#define INVERT_Z_DIR true
 //#define INVERT_I_DIR false
 //#define INVERT_J_DIR false
 //#define INVERT_K_DIR false
@@ -1331,8 +1336,8 @@
 // @section extruder
 
 // For direct drive extruder v9 set to true, for geared extruder set to false.
-#define INVERT_E0_DIR false
-#define INVERT_E1_DIR false
+#define INVERT_E0_DIR true
+#define INVERT_E1_DIR true
 #define INVERT_E2_DIR false
 #define INVERT_E3_DIR false
 #define INVERT_E4_DIR false
@@ -1352,7 +1357,7 @@
  */
 //#define Z_IDLE_HEIGHT Z_HOME_POS
 
-//#define Z_HOMING_HEIGHT  4      // (mm) Minimal Z height before homing (G28) for Z clearance above the bed, clamps, ...
+#define Z_HOMING_HEIGHT  10      // (mm) Minimal Z height before homing (G28) for Z clearance above the bed, clamps, ...
                                   // Be sure to have this much clearance over your Z_MAX_POS to prevent grinding.
 
 //#define Z_AFTER_HOMING  10      // (mm) Height to move to after homing Z
@@ -1369,16 +1374,16 @@
 // @section machine
 
 // The size of the printable area
-#define X_BED_SIZE 200
-#define Y_BED_SIZE 200
+#define X_BED_SIZE 302
+#define Y_BED_SIZE 302
 
 // Travel limits (mm) after homing, corresponding to endstop positions.
-#define X_MIN_POS 0
+#define X_MIN_POS -62
 #define Y_MIN_POS 0
 #define Z_MIN_POS 0
 #define X_MAX_POS X_BED_SIZE
 #define Y_MAX_POS Y_BED_SIZE
-#define Z_MAX_POS 200
+#define Z_MAX_POS 400
 //#define I_MIN_POS 0
 //#define I_MAX_POS 50
 //#define J_MIN_POS 0
@@ -1536,7 +1541,7 @@
  */
 //#define AUTO_BED_LEVELING_3POINT
 //#define AUTO_BED_LEVELING_LINEAR
-//#define AUTO_BED_LEVELING_BILINEAR
+#define AUTO_BED_LEVELING_BILINEAR
 //#define AUTO_BED_LEVELING_UBL
 //#define MESH_BED_LEVELING
 
@@ -1545,7 +1550,7 @@
  * these options to restore the prior leveling state or to always enable
  * leveling immediately after G28.
  */
-//#define RESTORE_LEVELING_AFTER_G28
+#define RESTORE_LEVELING_AFTER_G28
 //#define ENABLE_LEVELING_AFTER_G28
 
 /**
@@ -1575,7 +1580,7 @@
   // The height can be set with M420 Z<height>
   #define ENABLE_LEVELING_FADE_HEIGHT
   #if ENABLED(ENABLE_LEVELING_FADE_HEIGHT)
-    #define DEFAULT_LEVELING_FADE_HEIGHT 10.0 // (mm) Default fade height.
+    #define DEFAULT_LEVELING_FADE_HEIGHT 1.0 // (mm) Default fade height.
   #endif
 
   // For Cartesian machines, instead of dividing moves on mesh boundaries,
@@ -1587,7 +1592,7 @@
   /**
    * Enable the G26 Mesh Validation Pattern tool.
    */
-  //#define G26_MESH_VALIDATION
+  #define G26_MESH_VALIDATION
   #if ENABLED(G26_MESH_VALIDATION)
     #define MESH_TEST_NOZZLE_SIZE    0.4  // (mm) Diameter of primary nozzle.
     #define MESH_TEST_LAYER_HEIGHT   0.2  // (mm) Default layer height for G26.
@@ -1603,7 +1608,7 @@
 #if EITHER(AUTO_BED_LEVELING_LINEAR, AUTO_BED_LEVELING_BILINEAR)
 
   // Set the number of grid points per dimension.
-  #define GRID_MAX_POINTS_X 3
+  #define GRID_MAX_POINTS_X 4
   #define GRID_MAX_POINTS_Y GRID_MAX_POINTS_X
 
   // Probe along the Y axis, advancing X after each column
@@ -1613,7 +1618,7 @@
 
     // Beyond the probed grid, continue the implied tilt?
     // Default is to maintain the height of the nearest edge.
-    //#define EXTRAPOLATE_BEYOND_GRID
+    #define EXTRAPOLATE_BEYOND_GRID
 
     //
     // Experimental Subdivision of the grid by Catmull-Rom method.
@@ -1714,7 +1719,7 @@
  * Commands to execute at the end of G29 probing.
  * Useful to retract or move the Z probe out of the way.
  */
-//#define Z_PROBE_END_SCRIPT "G1 Z10 F12000\nG1 X15 Y330\nG1 Z0.5\nG1 Z10"
+#define Z_PROBE_END_SCRIPT "G28 XY"
 
 // @section homing
 
@@ -1737,7 +1742,7 @@
  * - Allows Z homing only when XY positions are known and trusted.
  * - If stepper drivers sleep, XY homing may be required again before Z homing.
  */
-//#define Z_SAFE_HOMING
+#define Z_SAFE_HOMING
 
 #if ENABLED(Z_SAFE_HOMING)
   #define Z_SAFE_HOMING_X_POINT X_CENTER  // X point for Z homing
@@ -1822,7 +1827,7 @@
  *   M501 - Read settings from EEPROM. (i.e., Throw away unsaved changes)
  *   M502 - Revert settings to "factory" defaults. (Follow with M500 to init the EEPROM.)
  */
-//#define EEPROM_SETTINGS     // Persistent storage with M500 and M501
+#define EEPROM_SETTINGS     // Persistent storage with M500 and M501
 //#define DISABLE_M503        // Saves ~2700 bytes of PROGMEM. Disable for release!
 #define EEPROM_CHITCHAT       // Give feedback on EEPROM commands. Disable to save PROGMEM.
 #define EEPROM_BOOT_SILENT    // Keep M503 quiet and only give errors during first load
@@ -1856,14 +1861,14 @@
 // Preheat Constants - Up to 5 are supported without changes
 //
 #define PREHEAT_1_LABEL       "PLA"
-#define PREHEAT_1_TEMP_HOTEND 180
-#define PREHEAT_1_TEMP_BED     70
+#define PREHEAT_1_TEMP_HOTEND 200
+#define PREHEAT_1_TEMP_BED     60
 #define PREHEAT_1_TEMP_CHAMBER 35
 #define PREHEAT_1_FAN_SPEED     0 // Value from 0 to 255
 
 #define PREHEAT_2_LABEL       "ABS"
 #define PREHEAT_2_TEMP_HOTEND 240
-#define PREHEAT_2_TEMP_BED    110
+#define PREHEAT_2_TEMP_BED     80
 #define PREHEAT_2_TEMP_CHAMBER 35
 #define PREHEAT_2_FAN_SPEED     0 // Value from 0 to 255
 
@@ -1878,7 +1883,7 @@
  *    P1  Raise the nozzle always to Z-park height.
  *    P2  Raise the nozzle by Z-park amount, limited to Z_MAX_POS.
  */
-//#define NOZZLE_PARK_FEATURE
+#define NOZZLE_PARK_FEATURE
 
 #if ENABLED(NOZZLE_PARK_FEATURE)
   // Specify a park position as { X, Y, Z_raise }
@@ -1886,7 +1891,7 @@
   //#define NOZZLE_PARK_X_ONLY          // X move only is required to park
   //#define NOZZLE_PARK_Y_ONLY          // Y move only is required to park
   #define NOZZLE_PARK_Z_RAISE_MIN   2   // (mm) Always raise Z by at least this distance
-  #define NOZZLE_PARK_XY_FEEDRATE 100   // (mm/s) X and Y axes feedrate (also used for delta Z axis)
+  #define NOZZLE_PARK_XY_FEEDRATE  50   // (mm/s) X and Y axes feedrate (also used for delta Z axis)
   #define NOZZLE_PARK_Z_FEEDRATE    5   // (mm/s) Z axis feedrate (not used for delta printers)
 #endif
 
@@ -2095,7 +2100,8 @@
  * SD Card support is disabled by default. If your controller has an SD slot,
  * you must uncomment the following option or it won't work.
  */
-//#define SDSUPPORT
+#define SDSUPPORT
+#define SDIO_SUPPORT
 
 /**
  * SD CARD: ENABLE CRC
@@ -2778,6 +2784,7 @@
 //
 //#define DWIN_MARLINUI_PORTRAIT
 //#define DWIN_MARLINUI_LANDSCAPE
+#define RTS_AVAILABLE
 
 //
 // Touch Screen Settings
@@ -2821,7 +2828,7 @@
 
 // Set number of user-controlled fans. Disable to use all board-defined fans.
 // :[1,2,3,4,5,6,7,8]
-//#define NUM_M106_FANS 1
+#define NUM_M106_FANS 2
 
 // Increase the FAN PWM frequency. Removes the PWM noise but increases heating in the FET/Arduino
 //#define FAST_PWM_FAN
@@ -2836,7 +2843,7 @@
 // However, control resolution will be halved for each increment;
 // at zero value, there are 128 effective control positions.
 // :[0,1,2,3,4,5,6,7]
-#define SOFT_PWM_SCALE 0
+#define SOFT_PWM_SCALE 5
 
 // If SOFT_PWM_SCALE is set to a value higher than 0, dithering can
 // be used to mitigate the associated resolution loss. If enabled,
 ```

 Now the advanced configuration.

- X2_MAX_POS 362
- they change LIN_ADVANCE_K to 0.09, which is odd since the feature isn't on
- 

```diff
diff --git a/Marlin/Configuration_adv.h b/Marlin/Configuration_adv.h
index 6e44f009ed..8b876d830b 100644
--- a/Marlin/Configuration_adv.h
+++ b/Marlin/Configuration_adv.h
@@ -692,7 +692,7 @@
 //
 // For Z set the number of stepper drivers
 //
-#define NUM_Z_STEPPER_DRIVERS 1   // (1-4) Z options change based on how many
+#define NUM_Z_STEPPER_DRIVERS 2   // (1-4) Z options change based on how many
 
 #if NUM_Z_STEPPER_DRIVERS > 1
   // Enable if Z motor direction signals are the opposite of Z1
@@ -749,12 +749,12 @@
  *       Set the initial X offset and temperature differential with M605 S2 X[offs] R[deg] and
  *       follow with M605 S3 to initiate mirrored movement.
  */
-//#define DUAL_X_CARRIAGE
+#define DUAL_X_CARRIAGE
 #if ENABLED(DUAL_X_CARRIAGE)
   #define X1_MIN_POS X_MIN_POS   // Set to X_MIN_POS
   #define X1_MAX_POS X_BED_SIZE  // Set a maximum so the first X-carriage can't hit the parked second X-carriage
-  #define X2_MIN_POS    80       // Set a minimum to ensure the  second X-carriage can't hit the parked first X-carriage
-  #define X2_MAX_POS   353       // Set this to the distance between toolheads when both heads are homed
+  #define X2_MIN_POS   0         // Set a minimum to ensure the  second X-carriage can't hit the parked first X-carriage
+  #define X2_MAX_POS   362       // Set this to the distance between toolheads when both heads are homed
   #define X2_HOME_DIR    1       // Set to 1. The second X-carriage always homes to the maximum endstop position
   #define X2_HOME_POS X2_MAX_POS // Default X2 home position. Set to X2_MAX_POS.
                       // However: In this mode the HOTEND_OFFSET_X value for the second extruder provides a software
@@ -766,7 +766,7 @@
   #define DEFAULT_DUAL_X_CARRIAGE_MODE DXC_AUTO_PARK_MODE
 
   // Default x offset in duplication mode (typically set to half print bed width)
-  #define DEFAULT_DUPLICATION_X_OFFSET 100
+  #define DEFAULT_DUPLICATION_X_OFFSET 150
 
   // Default action to execute following M605 mode change commands. Typically G28X to apply new mode.
   //#define EVENT_GCODE_IDEX_AFTER_MODECHANGE "G28X"
@@ -871,7 +871,7 @@
  * Z Steppers Auto-Alignment
  * Add the G34 command to align multiple Z steppers using a bed probe.
  */
-//#define Z_STEPPER_AUTO_ALIGN
+#define Z_STEPPER_AUTO_ALIGN
 #if ENABLED(Z_STEPPER_AUTO_ALIGN)
   // Define probe X and Y positions for Z1, Z2 [, Z3 [, Z4]]
   // If not defined, probe limits will be used.
@@ -1307,10 +1307,10 @@
 // LCD Print Progress options
 #if EITHER(SDSUPPORT, LCD_SET_PROGRESS_MANUALLY)
   #if ANY(HAS_MARLINUI_U8GLIB, EXTENSIBLE_UI, HAS_MARLINUI_HD44780, IS_TFTGLCD_PANEL, IS_DWIN_MARLINUI)
-    //#define SHOW_REMAINING_TIME         // Display estimated time to completion
+    #define SHOW_REMAINING_TIME         // Display estimated time to completion
     #if ENABLED(SHOW_REMAINING_TIME)
-      //#define USE_M73_REMAINING_TIME    // Use remaining time from M73 command instead of estimation
-      //#define ROTATE_PROGRESS_DISPLAY   // Display (P)rogress, (E)lapsed, and (R)emaining time
+      #define USE_M73_REMAINING_TIME    // Use remaining time from M73 command instead of estimation
+      #define ROTATE_PROGRESS_DISPLAY   // Display (P)rogress, (E)lapsed, and (R)emaining time
     #endif
   #endif
 
@@ -1383,9 +1383,9 @@
    * an option on the LCD screen to continue the print from the last-known
    * point in the file.
    */
-  //#define POWER_LOSS_RECOVERY
+  #define POWER_LOSS_RECOVERY
   #if ENABLED(POWER_LOSS_RECOVERY)
-    #define PLR_ENABLED_DEFAULT   false // Power Loss Recovery enabled by default. (Set with 'M413 Sn' & M500)
+    #define PLR_ENABLED_DEFAULT   true // Power Loss Recovery enabled by default. (Set with 'M413 Sn' & M500)
     //#define BACKUP_POWER_SUPPLY       // Backup power / UPS to move the steppers on power loss
     //#define POWER_LOSS_ZRAISE       2 // (mm) Z axis raise on resume (on power loss with UPS)
     //#define POWER_LOSS_PIN         44 // Pin to detect power loss. Set to -1 to disable default pin on boards without module.
@@ -1860,7 +1860,7 @@
  *
  * Warning: Does not respect endstops!
  */
-//#define BABYSTEPPING
+#define BABYSTEPPING
 #if ENABLED(BABYSTEPPING)
   //#define INTEGRATED_BABYSTEPPING         // EXPERIMENTAL integration of babystepping into the Stepper ISR
   //#define BABYSTEP_WITHOUT_HOMING
@@ -1868,7 +1868,7 @@
   //#define BABYSTEP_XY                     // Also enable X/Y Babystepping. Not supported on DELTA!
   #define BABYSTEP_INVERT_Z false           // Change if Z babysteps should go the other way
   //#define BABYSTEP_MILLIMETER_UNITS       // Specify BABYSTEP_MULTIPLICATOR_(XY|Z) in mm instead of micro-steps
-  #define BABYSTEP_MULTIPLICATOR_Z  1       // (steps or mm) Steps or millimeter distance for each Z babystep
+  #define BABYSTEP_MULTIPLICATOR_Z  40       // (steps or mm) Steps or millimeter distance for each Z babystep
   #define BABYSTEP_MULTIPLICATOR_XY 1       // (steps or mm) Steps or millimeter distance for each XY babystep
 
   //#define DOUBLECLICK_FOR_Z_BABYSTEPPING  // Double-click on the Status Screen for Z Babystepping.
@@ -1910,7 +1910,7 @@
 //#define LIN_ADVANCE
 #if ENABLED(LIN_ADVANCE)
   //#define EXTRA_LIN_ADVANCE_K // Enable for second linear advance constants
-  #define LIN_ADVANCE_K 0.22    // Unit: mm compression per 1mm/s extruder speed
+  #define LIN_ADVANCE_K 0.09    // Unit: mm compression per 1mm/s extruder speed
   //#define LA_DEBUG            // If enabled, this will generate debug information output over USB.
   //#define EXPERIMENTAL_SCURVE // Enable this option to permit S-Curve Acceleration
 #endif
@@ -1952,8 +1952,8 @@
 #if PROBE_SELECTED && !IS_KINEMATIC
   //#define PROBING_MARGIN_LEFT PROBING_MARGIN
   //#define PROBING_MARGIN_RIGHT PROBING_MARGIN
-  //#define PROBING_MARGIN_FRONT PROBING_MARGIN
-  //#define PROBING_MARGIN_BACK PROBING_MARGIN
+  //#define PROBING_MARGIN_FRONT 10
+  //#define PROBING_MARGIN_BACK 10
 #endif
 
 #if EITHER(MESH_BED_LEVELING, AUTO_BED_LEVELING_UBL)
@@ -1974,7 +1974,7 @@
  */
 //#define G29_RETRY_AND_RECOVER
 #if ENABLED(G29_RETRY_AND_RECOVER)
-  #define G29_MAX_RETRIES 3
+  #define G29_MAX_RETRIES 1
   #define G29_HALT_ON_FAILURE
   /**
    * Specify the GCODE commands that will be executed when leveling succeeds,
@@ -2300,9 +2300,9 @@
  */
 #if HAS_MULTI_EXTRUDER
   // Z raise distance for tool-change, as needed for some extruders
-  #define TOOLCHANGE_ZRAISE                 2 // (mm)
+  #define TOOLCHANGE_ZRAISE                 0 // (mm)
   //#define TOOLCHANGE_ZRAISE_BEFORE_RETRACT  // Apply raise before swap retraction (if enabled)
-  //#define TOOLCHANGE_NO_RETURN              // Never return to previous position on tool-change
+  #define TOOLCHANGE_NO_RETURN              // Never return to previous position on tool-change
   #if ENABLED(TOOLCHANGE_NO_RETURN)
     //#define EVENT_GCODE_AFTER_TOOLCHANGE "G12X"   // Extra G-code to run after tool-change
   #endif
@@ -2390,12 +2390,12 @@
  */
 //#define ADVANCED_PAUSE_FEATURE
 #if ENABLED(ADVANCED_PAUSE_FEATURE)
-  #define PAUSE_PARK_RETRACT_FEEDRATE         60  // (mm/s) Initial retract feedrate.
+  #define PAUSE_PARK_RETRACT_FEEDRATE         10  // (mm/s) Initial retract feedrate.
   #define PAUSE_PARK_RETRACT_LENGTH            2  // (mm) Initial retract.
                                                   // This short retract is done immediately, before parking the nozzle.
-  #define FILAMENT_CHANGE_UNLOAD_FEEDRATE     10  // (mm/s) Unload filament feedrate. This can be pretty fast.
-  #define FILAMENT_CHANGE_UNLOAD_ACCEL        25  // (mm/s^2) Lower acceleration may allow a faster feedrate.
-  #define FILAMENT_CHANGE_UNLOAD_LENGTH      100  // (mm) The length of filament for a complete unload.
+  #define FILAMENT_CHANGE_UNLOAD_FEEDRATE      6  // (mm/s) Unload filament feedrate. This can be pretty fast.
+  #define FILAMENT_CHANGE_UNLOAD_ACCEL        20  // (mm/s^2) Lower acceleration may allow a faster feedrate.
+  #define FILAMENT_CHANGE_UNLOAD_LENGTH       60  // (mm) The length of filament for a complete unload.
                                                   //   For Bowden, the full length of the tube and nozzle.
                                                   //   For direct drive, the full length of the nozzle.
                                                   //   Set to 0 for manual unloading.
@@ -2419,11 +2419,11 @@
                                                   // Filament Unload does a Retract, Delay, and Purge first:
   #define FILAMENT_UNLOAD_PURGE_RETRACT       13  // (mm) Unload initial retract length.
   #define FILAMENT_UNLOAD_PURGE_DELAY       5000  // (ms) Delay for the filament to cool after retract.
-  #define FILAMENT_UNLOAD_PURGE_LENGTH         8  // (mm) An unretract is done, then this length is purged.
-  #define FILAMENT_UNLOAD_PURGE_FEEDRATE      25  // (mm/s) feedrate to purge before unload
+  #define FILAMENT_UNLOAD_PURGE_LENGTH         6  // (mm) An unretract is done, then this length is purged.
+  #define FILAMENT_UNLOAD_PURGE_FEEDRATE      20  // (mm/s) feedrate to purge before unload
 
   #define PAUSE_PARK_NOZZLE_TIMEOUT           45  // (seconds) Time limit before the nozzle is turned off for safety.
-  #define FILAMENT_CHANGE_ALERT_BEEPS         10  // Number of alert beeps to play when a response is needed.
+  //#define FILAMENT_CHANGE_ALERT_BEEPS         10  // Number of alert beeps to play when a response is needed.
   #define PAUSE_PARK_NO_STEPPER_TIMEOUT           // Enable for XYZ steppers to stay powered on during filament change.
   //#define FILAMENT_CHANGE_RESUME_ON_INSERT      // Automatically continue / load filament when runout sensor is triggered again.
   //#define PAUSE_REHEAT_FAST_RESUME              // Reduce number of waits by not prompting again post-timeout before continuing.
```

- Some SDIO_Dx_PIN's are commented out.
  - not sure why

```diff
diff --git a/Marlin/src/HAL/STM32/Sd2Card_sdio_stm32duino.cpp b/Marlin/src/HAL/STM32/Sd2Card_sdio_stm32duino.cpp
index 54e1820c78..7c1f04f777 100644
--- a/Marlin/src/HAL/STM32/Sd2Card_sdio_stm32duino.cpp
+++ b/Marlin/src/HAL/STM32/Sd2Card_sdio_stm32duino.cpp
@@ -50,13 +50,14 @@
 #endif
 
 // Fixed
+/*
 #define SDIO_D0_PIN   PC8
 #define SDIO_D1_PIN   PC9
 #define SDIO_D2_PIN   PC10
 #define SDIO_D3_PIN   PC11
 #define SDIO_CK_PIN   PC12
 #define SDIO_CMD_PIN  PD2
-
+*/
 SD_HandleTypeDef hsd;  // create SDIO structure
 // F4 supports one DMA for RX and another for TX, but Marlin will never
 // do read and write at same time, so we use the same DMA for both.
 ```

- check if `RTS_AVAILABLE` (which is defined earlier in a config file, so `true`)
  - include the custom LCD_RTS header
  - don't include encoder.h
- define some global vars: `active_extruder_font`, `dualXPrintingModeStatus`
  - `TERN_(HAS_T_COMMAND, active_extruder = active_extruder_font);`
- some code related to `next_check_axes_ms` is removed/commented
  - `planner.check_axes_activity();` now runs every time instead of having the: `// Limit check_axes_activity frequency to 10Hz` limit
- code related to wiring up the touch screen
  - it's taking the form of replacing DWIN_xxx with RTSyyy, often inside `#if HAS_DWIN_E3V2_BASIC` but sometime s not
- as part of `setup()` it runs `queue.enqueue_now_P(PSTR("M280 P0 S160"));`: set servo 0 position to 160

 ```diff
diff --git a/Marlin/src/MarlinCore.cpp b/Marlin/src/MarlinCore.cpp
index 1b9c8885b1..bc6c069b09 100644
--- a/Marlin/src/MarlinCore.cpp
+++ b/Marlin/src/MarlinCore.cpp
@@ -71,8 +71,10 @@
 #endif
 
 #if HAS_DWIN_E3V2
-  #include "lcd/e3v2/common/encoder.h"
-  #if ENABLED(DWIN_CREALITY_LCD)
+  //#include "lcd/e3v2/common/encoder.h"
+  #if ENABLED(RTS_AVAILABLE)
+    #include "lcd/e3v2/creality/LCD_RTS.h"
+  #elif ENABLED(DWIN_CREALITY_LCD)
     #include "lcd/e3v2/creality/dwin.h"
   #elif ENABLED(DWIN_CREALITY_LCD_ENHANCED)
     #include "lcd/e3v2/enhanced/dwin.h"
@@ -247,6 +249,8 @@ MarlinState marlin_state = MF_INITIALIZING;
 // For M109 and M190, this flag may be cleared (by M108) to exit the wait loop
 bool wait_for_heatup = true;
 
+uint8_t active_extruder_font;
+uint8_t dualXPrintingModeStatus;
 // For M0/M1, this flag may be cleared (by M108) to exit the wait-for-user loop
 #if HAS_RESUME_CONTINUE
   bool wait_for_user; // = false;
@@ -709,11 +713,12 @@ inline void manage_inactivity(const bool no_stepper_sleep=false) {
   TERN_(MONITOR_L6470_DRIVER_STATUS, L64xxManager.monitor_driver());
 
   // Limit check_axes_activity frequency to 10Hz
-  static millis_t next_check_axes_ms = 0;
-  if (ELAPSED(ms, next_check_axes_ms)) {
+  // static millis_t next_check_axes_ms = 0;
+  // if (ELAPSED(ms, next_check_axes_ms)) {
+  //   planner.check_axes_activity();
+  //   next_check_axes_ms = ms + 100UL;
+  // }
   planner.check_axes_activity();
-    next_check_axes_ms = ms + 100UL;
-  }
   
   #if PIN_EXISTS(FET_SAFETY)
     static millis_t FET_next;
@@ -805,7 +810,7 @@ void idle(bool no_stepper_sleep/*=false*/) {
   TERN_(USE_BEEPER, buzzer.tick());
 
   // Handle UI input / draw events
-  TERN(HAS_DWIN_E3V2_BASIC, DWIN_Update(), ui.update());
+  TERN_(HAS_DWIN_E3V2_BASIC, RTSUpdate());
 
   // Run i2c Position Encoders
   #if ENABLED(I2C_POSITION_ENCODERS)
@@ -1113,9 +1118,13 @@ void setup() {
   #define SETUP_RUN(C) do{ SETUP_LOG(STRINGIFY(C)); C; }while(0)
 
   MYSERIAL1.begin(BAUDRATE);
-  millis_t serial_connect_timeout = millis() + 1000UL;
+  uint32_t serial_connect_timeout = millis() + 1000UL;
   while (!MYSERIAL1.connected() && PENDING(millis(), serial_connect_timeout)) { /*nada*/ }
 
+  LCD_SERIAL.begin(BAUDRATE);
+  serial_connect_timeout = millis() + 1000UL;
+  while (!LCD_SERIAL.connected() && PENDING(millis(), serial_connect_timeout)) { /*nada*/ }
+
   #if HAS_MULTI_SERIAL && !HAS_ETHERNET
     #ifndef BAUDRATE_2
       #define BAUDRATE_2 BAUDRATE
@@ -1272,7 +1281,7 @@ void setup() {
   // (because EEPROM code calls the UI).
 
   #if HAS_DWIN_E3V2_BASIC
-    SETUP_RUN(DWIN_Startup());
+    SETUP_RUN(RTSUpdate());
   #else
     SETUP_RUN(ui.init());
     #if BOTH(HAS_WIRED_LCD, SHOW_BOOTSCREEN)
@@ -1301,6 +1310,9 @@ void setup() {
 
   SETUP_RUN(settings.first_load());   // Load data from EEPROM if available (or use defaults)
                                       // This also updates variables in the planner, elsewhere
+  #if HAS_MULTI_EXTRUDER
+    TERN_(HAS_T_COMMAND, active_extruder = active_extruder_font);
+  #endif
 
   #if HAS_ETHERNET
     SETUP_RUN(ethernet.init());
@@ -1547,11 +1559,7 @@ void setup() {
   #endif
 
   #if HAS_DWIN_E3V2_BASIC
-    Encoder_Configuration();
-    HMI_Init();
-    HMI_SetLanguageCache();
-    HMI_StartFrame(true);
-    DWIN_StatusChanged_P(GET_TEXT(WELCOME_MSG));
+    SETUP_RUN(rtscheck.RTS_Init());
   #endif
 
   #if HAS_SERVICE_INTERVALS && !HAS_DWIN_E3V2_BASIC
@@ -1590,8 +1598,8 @@ void setup() {
   #endif
 
   marlin_state = MF_RUNNING;
-
   SETUP_LOG("setup() completed.");
+  queue.enqueue_now_P(PSTR("M280 P0 S160"));
 }
 
 /**
 ```

- the two new global variables, defined in the cpp file, are added to the header, making them mutable and readable from anywhere


 ```diff
diff --git a/Marlin/src/MarlinCore.h b/Marlin/src/MarlinCore.h
index c3698d616d..f3d8f19aa9 100644
--- a/Marlin/src/MarlinCore.h
+++ b/Marlin/src/MarlinCore.h
@@ -62,6 +62,8 @@ bool printingIsPaused();
 void startOrResumeJob();
 
 extern bool wait_for_heatup;
+extern uint8_t active_extruder_font;
+extern uint8_t dualXPrintingModeStatus;
 
 #if HAS_RESUME_CONTINUE
   extern bool wait_for_user;
```

- abl.h now includes bedlevel.h for some reason

```diff
diff --git a/Marlin/src/feature/bedlevel/abl/abl.h b/Marlin/src/feature/bedlevel/abl/abl.h
index 3d54c55695..2fb632cc06 100644
--- a/Marlin/src/feature/bedlevel/abl/abl.h
+++ b/Marlin/src/feature/bedlevel/abl/abl.h
@@ -22,6 +22,7 @@
 #pragma once
 
 #include "../../../inc/MarlinConfigPre.h"
+#include "../../../feature/bedlevel/bedlevel.h"
 
 extern xy_pos_t bilinear_grid_spacing, bilinear_start;
 extern xy_float_t bilinear_grid_factor;
```

- touch screen changes
- some comments in Chinese, translated by google (these might actually have been chinese translated of english strings to start with, I didn't check the constants like PAUSE_MESSAGE_UNLOAD were defiend to)
  - "加热喷嘴等待界面" => "Heating nozzle waiting interface"
  - "插入丝料并继续" => "Insert filament and continue"
  - "等待清理喷嘴" => "Waiting for nozzle cleaning"
  - "等待卸下丝料" => "Waiting to unload the filament" 
  - "按下确认，加热喷嘴" => "Press to confirm and heat the nozzle"


```diff
diff --git a/Marlin/src/feature/pause.cpp b/Marlin/src/feature/pause.cpp
index d54326116e..ee461cd5c7 100644
--- a/Marlin/src/feature/pause.cpp
+++ b/Marlin/src/feature/pause.cpp
@@ -55,6 +55,8 @@
   #include "../lcd/extui/ui_api.h"
 #elif ENABLED(DWIN_CREALITY_LCD_ENHANCED)
   #include "../lcd/e3v2/enhanced/dwin.h"
+#elif ENABLED(RTS_AVAILABLE)
+  #include "../lcd/e3v2/creality/LCD_RTS.h"
 #endif
 
 #include "../lcd/marlinui.h"
@@ -139,7 +141,9 @@ static bool ensure_safe_temperature(const bool wait=true, const PauseMode mode=P
       thermalManager.setTargetHotend(thermalManager.extrude_min_temp, active_extruder);
   #endif
 
-  ui.pause_show_message(PAUSE_MESSAGE_HEATING, mode); UNUSED(mode);
+  ui.pause_show_message(PAUSE_MESSAGE_HEATING, mode); UNUSED(mode);//加热喷嘴等待界面
+  rtscheck.RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+  rtscheck.RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
 
   if (wait) return thermalManager.wait_for_hotend(active_extruder);
 
@@ -186,11 +190,18 @@ bool load_filament(const_float_t slow_load_length/*=0*/, const_float_t fast_load
   }
 
   if (pause_for_user) {
-    if (show_lcd) ui.pause_show_message(PAUSE_MESSAGE_INSERT, mode);
+    if (show_lcd)
+    {
+      ui.pause_show_message(PAUSE_MESSAGE_INSERT, mode);//插入丝料并继续
+      // rtscheck.RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+      // rtscheck.RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+      // rtscheck.RTS_SndData(ExchangePageBase + 17, ExchangepageAddr);
+    } 
     SERIAL_ECHO_MSG(_PMSG(STR_FILAMENT_CHANGE_INSERT));
 
     first_impatient_beep(max_beep_count);
     
+
     KEEPALIVE_STATE(PAUSED_FOR_USER);
     wait_for_user = true;    // LCD click or M108 will clear this
 
@@ -265,8 +276,12 @@ bool load_filament(const_float_t slow_load_length/*=0*/, const_float_t fast_load
     do {
       if (purge_length > 0) {
         // "Wait for filament purge"
-        if (show_lcd) ui.pause_show_message(PAUSE_MESSAGE_PURGE);
-
+        if (show_lcd)
+        {
+          ui.pause_show_message(PAUSE_MESSAGE_PURGE);//等待清理喷嘴
+          rtscheck.RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+          rtscheck.RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+        }
         // Extrude filament to get into hotend
         unscaled_e_move(purge_length, ADVANCED_PAUSE_PURGE_FEEDRATE);
       }
@@ -340,8 +355,12 @@ bool unload_filament(const_float_t unload_length, const bool show_lcd/*=false*/,
     return false;
   }
 
-  if (show_lcd) ui.pause_show_message(PAUSE_MESSAGE_UNLOAD, mode);
-
+  if (show_lcd) 
+  {
+    ui.pause_show_message(PAUSE_MESSAGE_UNLOAD, mode);//等待卸下丝料
+    rtscheck.RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+    rtscheck.RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+  }
   // Retract filament
   unscaled_e_move(-(FILAMENT_UNLOAD_PURGE_RETRACT) * mix_multiplier, (PAUSE_PARK_RETRACT_FEEDRATE) * mix_multiplier);
 
@@ -485,6 +504,9 @@ void show_continue_prompt(const bool is_reload) {
   DEBUG_ECHOLNPGM("... is_reload:", is_reload);
 
   ui.pause_show_message(is_reload ? PAUSE_MESSAGE_INSERT : PAUSE_MESSAGE_WAITING);
+  rtscheck.RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+  rtscheck.RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+  rtscheck.RTS_SndData(Beep, SoundAddr);
   SERIAL_ECHO_START();
   SERIAL_ECHOPGM_P(is_reload ? PSTR(_PMSG(STR_FILAMENT_CHANGE_INSERT) "\n") : PSTR(_PMSG(STR_FILAMENT_CHANGE_WAIT) "\n"));
 }
@@ -525,7 +547,9 @@ void wait_for_confirmation(const bool is_reload/*=false*/, const int8_t max_beep
     // Wait for the user to press the button to re-heat the nozzle, then
     // re-heat the nozzle, re-show the continue prompt, restart idle timers, start over
     if (nozzle_timed_out) {
-      ui.pause_show_message(PAUSE_MESSAGE_HEAT);
+      ui.pause_show_message(PAUSE_MESSAGE_HEAT);//按下确认，加热喷嘴
+      rtscheck.RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+      rtscheck.RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
       SERIAL_ECHO_MSG(_PMSG(STR_FILAMENT_CHANGE_HEAT));
 
       TERN_(HOST_PROMPT_SUPPORT, host_prompt_do(PROMPT_USER_CONTINUE, GET_TEXT(MSG_HEATER_TIMEOUT), GET_TEXT(MSG_REHEAT)));
```

- power loss recovery stuff
  - write some status to new variable `info.recovery_flag`
  - add a `T0` or `T1` gcode if we're in single mode
  - a silly `if/else if/else` chain just to set `dualXPrintingModeStatus = save_dual_x_carriage_mode;`, or to `0` if it's not one of the valid modes (normally 0-3, but 0-4 because they made 4 single mode, right extruder)
  - and other stuff


```diff
diff --git a/Marlin/src/feature/powerloss.cpp b/Marlin/src/feature/powerloss.cpp
index 8db31daa40..2306ce398e 100644
--- a/Marlin/src/feature/powerloss.cpp
+++ b/Marlin/src/feature/powerloss.cpp
@@ -53,7 +53,9 @@ uint32_t PrintJobRecovery::cmd_sdpos, // = 0
 #include "../module/printcounter.h"
 #include "../module/temperature.h"
 #include "../core/serial.h"
-
+#if ENABLED(RTS_AVAILABLE)
+  #include "../lcd/e3v2/creality/LCD_RTS.h"
+#endif
 #if ENABLED(FWRETRACT)
   #include "fwretract.h"
 #endif
@@ -228,7 +230,7 @@ void PrintJobRecovery::save(const bool force/*=false*/, const float zraise/*=POW
     // Misc. Marlin flags
     info.flag.dryrun = !!(marlin_debug_flags & MARLIN_DEBUG_DRYRUN);
     info.flag.allow_cold_extrusion = TERN0(PREVENT_COLD_EXTRUSION, thermalManager.allow_cold_extrude);
-
+    info.recovery_flag = PoweroffContinue;
     write();
   }
 }
@@ -478,7 +480,11 @@ void PrintJobRecovery::resume() {
       }
     }
   #endif
-
+  if((dualXPrintingModeStatus == 0) || (dualXPrintingModeStatus == 4)) 
+  {
+      sprintf_P(cmd, PSTR("T%i"), info.active_extruder);
+      gcode.process_subcommands_now(cmd);
+  }
   // Restore the previously active tool (with no_move)
   #if HAS_MULTI_EXTRUDER || HAS_MULTI_HOTEND
     sprintf_P(cmd, PSTR("T%i S"), info.active_extruder);
@@ -526,6 +532,45 @@ void PrintJobRecovery::resume() {
     gcode.process_subcommands_now_P(PSTR("G12"));
   #endif
 
+  #if ENABLED(DUAL_X_CARRIAGE)
+    if(save_dual_x_carriage_mode == 2)
+    {
+      dualXPrintingModeStatus = save_dual_x_carriage_mode;
+    }
+    else if(save_dual_x_carriage_mode == 3)
+    {
+      dualXPrintingModeStatus = save_dual_x_carriage_mode;
+    }
+    else if(save_dual_x_carriage_mode == 1)
+    {
+      dualXPrintingModeStatus = save_dual_x_carriage_mode;
+    }
+    else if(save_dual_x_carriage_mode == 0)
+    {
+      dualXPrintingModeStatus = save_dual_x_carriage_mode;
+    }
+    else if(save_dual_x_carriage_mode == 4)
+    {
+      dualXPrintingModeStatus = save_dual_x_carriage_mode;
+    }
+    else
+    {
+      dualXPrintingModeStatus = 0;
+    }
+
+    gcode.process_subcommands_now_P(PSTR("G1 E3 F3000"));
+
+    save_dual_x_carriage_mode = dualXPrintingModeStatus;
+    if(save_dual_x_carriage_mode == 1)
+    {
+      gcode.process_subcommands_now_P(PSTR("M605 S1"));
+      if(info.active_extruder == 0)
+      {
+        sprintf_P(cmd, PSTR("T%i"), info.active_extruder);
+        gcode.process_subcommands_now(cmd);
+      }
+    }
+  #endif
   // Move back over to the saved XY
   sprintf_P(cmd, PSTR("G1X%sY%sF3000"),
     dtostrf(info.current_position.x, 1, 3, str_1),
```

add `recovery_flag` to struct

```diff
diff --git a/Marlin/src/feature/powerloss.h b/Marlin/src/feature/powerloss.h
index 6a13c92df7..9b9609f322 100644
--- a/Marlin/src/feature/powerloss.h
+++ b/Marlin/src/feature/powerloss.h
@@ -115,6 +115,9 @@ typedef struct {
   // Relative axis modes
   uint8_t axis_relative;
 
+    // recovery flag
+  uint8_t recovery_flag;
+
   // Misc. Marlin flags
   struct {
     bool raised:1;                // Raised before saved
```

Changes related to G29 bilinear bed leveling.

- comment out `z_values[i][j] = rz;`
  - what's that about?
- code to send the touch screen the bed leveling info
- other touch screen code
- if I wanted to change the number of points (like to John's 7x7) but
  not update the touch screen firmware, I could probably fake it in 

```diff
diff --git a/Marlin/src/gcode/bedlevel/abl/G29.cpp b/Marlin/src/gcode/bedlevel/abl/G29.cpp
index 0eb13dba96..02df6e4c8d 100644
--- a/Marlin/src/gcode/bedlevel/abl/G29.cpp
+++ b/Marlin/src/gcode/bedlevel/abl/G29.cpp
@@ -34,6 +34,7 @@
 #include "../../../module/planner.h"
 #include "../../../module/stepper.h"
 #include "../../../module/probe.h"
+#include "../../../module/settings.h"
 #include "../../queue.h"
 
 #if ENABLED(PROBE_TEMP_COMPENSATION)
@@ -62,6 +63,8 @@
   #include "../../../lcd/e3v2/creality/dwin.h"
 #elif ENABLED(DWIN_CREALITY_LCD_ENHANCED)
   #include "../../../lcd/e3v2/enhanced/dwin.h"
+#elif ENABLED(RTS_AVAILABLE)
+  #include "../../../lcd/e3v2/creality/LCD_RTS.h"
 #endif
 
 #if HAS_MULTI_HOTEND
@@ -299,7 +302,7 @@ G29_TYPE GcodeSuite::G29() {
         }
         if (WITHIN(i, 0, (GRID_MAX_POINTS_X) - 1) && WITHIN(j, 0, (GRID_MAX_POINTS_Y) - 1)) {
           set_bed_leveling_enabled(false);
-          z_values[i][j] = rz;
+          //z_values[i][j] = rz;
           TERN_(ABL_BILINEAR_SUBDIVISION, bed_level_virt_interpolate());
           TERN_(EXTENSIBLE_UI, ExtUI::onMeshUpdate(i, j, rz));
           set_bed_leveling_enabled(abl.reenable);
@@ -602,9 +605,11 @@ G29_TYPE GcodeSuite::G29() {
 
       abl.measured_z = 0;
 
+      uint8_t showcount = 0;
+
       // Outer loop is X with PROBE_Y_FIRST enabled
       // Outer loop is Y with PROBE_Y_FIRST disabled
-      for (PR_OUTER_VAR = 0; PR_OUTER_VAR < PR_OUTER_SIZE && !isnan(abl.measured_z); PR_OUTER_VAR++) {
+      for (PR_OUTER_VAR = 0, showcount = 0; PR_OUTER_VAR < PR_OUTER_SIZE && !isnan(abl.measured_z); PR_OUTER_VAR++) {
 
         int8_t inStart, inStop, inInc;
 
@@ -666,6 +671,13 @@ G29_TYPE GcodeSuite::G29() {
             const float z = abl.measured_z + abl.Z_offset;
             z_values[abl.meshCount.x][abl.meshCount.y] = z;
             TERN_(EXTENSIBLE_UI, ExtUI::onMeshUpdate(abl.meshCount, z));
+            #if ENABLED(RTS_AVAILABLE)
+              if((showcount ++) < GRID_MAX_POINTS_X * GRID_MAX_POINTS_Y)
+              {
+                rtscheck.RTS_SndData(showcount, AUTO_BED_LEVEL_ICON_VP);
+              }
+              rtscheck.RTS_SndData(z*1000, AUTO_BED_LEVEL_1POINT_VP + (showcount - 1) * 2);
+            #endif
 
           #endif
 
@@ -885,8 +897,11 @@ G29_TYPE GcodeSuite::G29() {
     process_subcommands_now_P(PSTR(Z_PROBE_END_SCRIPT));
   #endif
 
-  TERN_(HAS_DWIN_E3V2_BASIC, DWIN_CompletedLeveling());
+  //TERN_(HAS_DWIN_E3V2_BASIC, DWIN_CompletedLeveling());
 
+  #if ENABLED(RTS_AVAILABLE)
+    RTS_AutoBedLevelPage();
+  #endif
   report_current_position();
 
   TERN_(FULL_REPORT_TO_HOST_FEATURE, set_and_report_grblstate(M_IDLE));
```

G28 is the homing command.

- some code to set the current x0 and x1 to X_MIN_POS and X2_MAX_POS
  - I wonder why that is needed, i thought marlin supported dual x carriage out of the box?
- some TERM (LCD) code commented out, and some added or modified for RTS
- mode 4 (single right extruder) code, to home the left axis when printing from card

```diff
diff --git a/Marlin/src/gcode/calibrate/G28.cpp b/Marlin/src/gcode/calibrate/G28.cpp
index dc93ba3d2f..5700ac2522 100644
--- a/Marlin/src/gcode/calibrate/G28.cpp
+++ b/Marlin/src/gcode/calibrate/G28.cpp
@@ -53,6 +53,8 @@
   #include "../../lcd/e3v2/creality/dwin.h"
 #elif ENABLED(DWIN_CREALITY_LCD_ENHANCED)
   #include "../../lcd/e3v2/enhanced/dwin.h"
+#elif ENABLED(RTS_AVAILABLE)
+  #include "../../lcd/e3v2/creality/LCD_RTS.h"
 #endif
 
 #if HAS_L64XX                         // set L6470 absolute position registers to counts
@@ -141,6 +143,11 @@
 
     destination.set(okay_homing_xy, current_position.z);
 
+    #if ENABLED(DUAL_X_CARRIAGE)
+      current_position_x1_axis = X2_MAX_POS;
+      current_position_x0_axis = X_MIN_POS;
+    #endif
+
     TERN_(HOMING_Z_WITH_PROBE, destination -= probe.offset_xy);
 
     if (position_is_reachable(destination)) {
@@ -239,7 +246,7 @@ void GcodeSuite::G28() {
     return;
   }
 
-  TERN_(HAS_DWIN_E3V2_BASIC, DWIN_StartHoming());
+  //TERN_(HAS_DWIN_E3V2_BASIC, DWIN_StartHoming());
   TERN_(EXTENSIBLE_UI, ExtUI::onHomingStart());
 
   planner.synchronize();          // Wait for planner moves to finish!
@@ -473,6 +480,12 @@ void GcodeSuite::G28() {
 
       TERN_(IMPROVE_HOMING_RELIABILITY, end_slow_homing(saved_motion_state));
     }
+    else if((save_dual_x_carriage_mode == 4) && card.isPrinting()){
+      // Home the 1st (left) extruder
+      active_extruder = 0;
+      homeaxis(X_AXIS);
+
+    }
 
   #endif // DUAL_X_CARRIAGE
 
@@ -521,9 +534,10 @@ void GcodeSuite::G28() {
     #endif
   #endif // HAS_HOMING_CURRENT
 
-  ui.refresh();
+  //ui.refresh();
+  //RTSUpdate();
 
-  TERN_(HAS_DWIN_E3V2_BASIC, DWIN_CompletedHoming());
+  TERN_(RTS_AVAILABLE, RTS_MoveAxisHoming());
   TERN_(EXTENSIBLE_UI, ExtUI::onHomingComplete());
 
   report_current_position();
```

G34 / M422
Z-Stepped alignment /  Set Z Motor XY

- add a G28 X after the G28 Z after z alignment if not card printing
- add regular G28 otherwise

```diff
diff --git a/Marlin/src/gcode/calibrate/G34_M422.cpp b/Marlin/src/gcode/calibrate/G34_M422.cpp
index dd1dd5622a..a53b0c0dc7 100644
--- a/Marlin/src/gcode/calibrate/G34_M422.cpp
+++ b/Marlin/src/gcode/calibrate/G34_M422.cpp
@@ -55,6 +55,10 @@
   #endif
 #endif
 
+#if ENABLED(RTS_AVAILABLE)
+  #include "../../lcd/e3v2/creality/LCD_RTS.h"
+#endif
+
 /**
  * G34: Z-Stepper automatic alignment
  *
@@ -433,7 +437,16 @@ void GcodeSuite::G34() {
         // After this operation the z position needs correction
         set_axis_never_homed(Z_AXIS);
         // Home Z after the alignment procedure
+        if(!card.isPrinting())
+        {
           process_subcommands_now_P(PSTR("G28 Z"));
+          process_subcommands_now_P(PSTR("G28 X"));
+          rtscheck.RTS_SndData(ExchangePageBase + 22, ExchangepageAddr);
+        }
+        else
+        {
+          process_subcommands_now_P(PSTR("G28"));
+        }
       #else
         // Use the probed height from the last iteration to determine the Z height.
         // z_measured_min is used, because all steppers are aligned to z_measured_min.
```

M605 - multi nozzle mode

They change:
- `-          if (parser.seen('X')) duplicate_extruder_x_offset = _MAX(parser.value_linear_units(), (X2_MIN_POS) - (X1_MIN_POS));`
- `+          if (parser.seen('X')) duplicate_extruder_x_offset = _MAX(parser.value_linear_units(), (82) - (X1_MIN_POS));`

So that last part, instead of being "62" becomes "144".
This, I think, is supposed to be a safety check, to keep the extruders far enough apart.

This is one of those changes that I'm trying to understand why it's needed. (If it's a bugfix, it should be improved and submitted upstream, otherwise, the less customizaton, the easier to upgrade Marlin)

```diff
diff --git a/Marlin/src/gcode/control/M605.cpp b/Marlin/src/gcode/control/M605.cpp
index 08efaab59d..73a8d35a94 100644
--- a/Marlin/src/gcode/control/M605.cpp
+++ b/Marlin/src/gcode/control/M605.cpp
@@ -78,7 +78,7 @@
 
         case DXC_DUPLICATION_MODE:
           // Set the X offset, but no less than the safety gap
-          if (parser.seen('X')) duplicate_extruder_x_offset = _MAX(parser.value_linear_units(), (X2_MIN_POS) - (X1_MIN_POS));
+          if (parser.seen('X')) duplicate_extruder_x_offset = _MAX(parser.value_linear_units(), (82) - (X1_MIN_POS));
           if (parser.seen('R')) duplicate_extruder_temp_offset = parser.value_celsius_diff();
           // Always switch back to tool 0
           if (active_extruder != 0) tool_change(0);
```

T command: tool change (change extruder)
T0 changes to the left extruder (sometimes called extrduer 1)
T1 changes to right extruder (extruder 2)

- the sovol added global variable `active_extruder_font` is set to `parser.boolval('S');`
  - The docs say "S1           Don't move the tool in XY after change"
  - is this something special the screen sends? or just nonsense? 


```diff
diff --git a/Marlin/src/gcode/control/T.cpp b/Marlin/src/gcode/control/T.cpp
index 5e8f6b5436..5bb139f951 100644
--- a/Marlin/src/gcode/control/T.cpp
+++ b/Marlin/src/gcode/control/T.cpp
@@ -22,7 +22,7 @@
 
 #include "../gcode.h"
 #include "../../module/tool_change.h"
-
+#include "../../MarlinCore.h"
 #if EITHER(HAS_MULTI_EXTRUDER, DEBUG_LEVELING_FEATURE)
   #include "../../module/motion.h"
 #endif
@@ -67,4 +67,5 @@ void GcodeSuite::T(const int8_t tool_index) {
       || parser.boolval('S')
     #endif
   );
+  active_extruder_font = parser.boolval('S');
 }
```

M660 filament change

They added:
- `SERIAL_ECHOLNPGM("helwr");`
- `SERIAL_ECHOLNPGM("hello wr");`
- `rtscheck.RTS_SndData(ExchangePageBase + 6, ExchangepageAddr);`

Not sure what any of that is about. The last one is letting the touch screen know. The first two, no idea. Off the cuff, looks like "hello world" test code, but I haven't looked at what that macro does, so I could be off base.

```diff
diff --git a/Marlin/src/gcode/feature/pause/M600.cpp b/Marlin/src/gcode/feature/pause/M600.cpp
index 541fa08350..0491dbc813 100644
--- a/Marlin/src/gcode/feature/pause/M600.cpp
+++ b/Marlin/src/gcode/feature/pause/M600.cpp
@@ -45,7 +45,9 @@
 #if HAS_FILAMENT_SENSOR
   #include "../../../feature/runout.h"
 #endif
-
+#if ENABLED(RTS_AVAILABLE)
+  #include "../../../lcd/e3v2/creality/LCD_RTS.h"
+#endif
 /**
  * M600: Pause for filament change
  *
@@ -62,7 +64,7 @@
  *  Default values are used for omitted arguments.
  */
 void GcodeSuite::M600() {
-
+SERIAL_ECHOLNPGM("helwr");
   #if ENABLED(MIXING_EXTRUDER)
     const int8_t target_e_stepper = get_target_e_stepper_from_command();
     if (target_e_stepper < 0) return;
@@ -75,7 +77,9 @@ void GcodeSuite::M600() {
 
     const int8_t target_extruder = active_extruder;
   #else
+  SERIAL_ECHOLNPGM("hello wr");
     const int8_t target_extruder = get_target_extruder_from_command();
+    
     if (target_extruder < 0) return;
   #endif
 
@@ -97,6 +101,8 @@ void GcodeSuite::M600() {
     ui.pause_show_message(PAUSE_MESSAGE_CHANGING, PAUSE_MODE_PAUSE_PRINT, target_extruder);
   #endif
  
+  rtscheck.RTS_SndData(ExchangePageBase + 6, ExchangepageAddr);
+ 
   #if ENABLED(HOME_BEFORE_FILAMENT_CHANGE)
     // If needed, home before parking for filament change
     home_if_needed(true);
```

G0 / G1 are the move and move and extrude commands.

- looks like we just add code to let the touch screen know


```diff
diff --git a/Marlin/src/gcode/motion/G0_G1.cpp b/Marlin/src/gcode/motion/G0_G1.cpp
index cc6979b74c..1042919687 100644
--- a/Marlin/src/gcode/motion/G0_G1.cpp
+++ b/Marlin/src/gcode/motion/G0_G1.cpp
@@ -24,7 +24,9 @@
 #include "../../module/motion.h"
 
 #include "../../MarlinCore.h"
-
+#if ENABLED(RTS_AVAILABLE)
+  #include "../../lcd/e3v2/creality/LCD_RTS.h"
+#endif
 #if BOTH(FWRETRACT, FWRETRACT_AUTORETRACT)
   #include "../../feature/fwretract.h"
 #endif
@@ -128,5 +130,8 @@ void GcodeSuite::G0_G1(TERN_(HAS_FAST_MOVES, const bool fast_move/*=false*/)) {
     #else
       TERN_(FULL_REPORT_TO_HOST_FEATURE, report_current_grblstate_moving());
     #endif
+    #if ENABLED(RTS_AVAILABLE)
+      RTS_PauseMoveAxisPage();
+    #endif
   }
 }
```

Added code to report to touch screen the print finishing from an sd card.

```diff
diff --git a/Marlin/src/gcode/queue.cpp b/Marlin/src/gcode/queue.cpp
index d11b2823f2..7d6279fe98 100644
--- a/Marlin/src/gcode/queue.cpp
+++ b/Marlin/src/gcode/queue.cpp
@@ -56,7 +56,9 @@ GCodeQueue queue;
 #if ENABLED(GCODE_REPEAT_MARKERS)
   #include "../feature/repeat.h"
 #endif
-
+#if ENABLED(RTS_AVAILABLE)
+  #include "../lcd/e3v2/creality/LCD_RTS.h"
+#endif
 // Frequently used G-code strings
 PGMSTR(G28_STR, "G28");
 
@@ -592,6 +594,24 @@ void GCodeQueue::get_serial_commands() {
       }
       else
         process_stream_char(sd_char, sd_input_state, command.buffer, sd_count);
+
+      #if ENABLED(RTS_AVAILABLE)
+        // the printing results
+        if (card_eof)
+        {
+          rtscheck.RTS_SndData(100, PRINT_PROCESS_VP);
+          delay(1);
+          rtscheck.RTS_SndData(100, PRINT_PROCESS_ICON_VP);
+          delay(1);
+          rtscheck.RTS_SndData(0, PRINT_SURPLUS_TIME_HOUR_VP);
+          delay(1);
+          rtscheck.RTS_SndData(0, PRINT_SURPLUS_TIME_MIN_VP);
+          delay(1);  
+          rtscheck.RTS_SndData(ExchangePageBase + 9, ExchangepageAddr);
+          
+          card.fileHasFinished();         // Handle end of file reached
+        }
+      #endif
     }
   }
 
```

M24 / M25: start and pause sd card print

- let the touch screen know?

```diff
diff --git a/Marlin/src/gcode/sd/M24_M25.cpp b/Marlin/src/gcode/sd/M24_M25.cpp
index 4cb040feb3..9b9ac9afdd 100644
--- a/Marlin/src/gcode/sd/M24_M25.cpp
+++ b/Marlin/src/gcode/sd/M24_M25.cpp
@@ -113,7 +113,7 @@ void GcodeSuite::M25() {
 
     TERN_(DGUS_LCD_UI_MKS, MKS_pause_print_move());
 
-    IF_DISABLED(DWIN_CREALITY_LCD, ui.reset_status());
+    IF_DISABLED(RTS_AVAILABLE, ui.reset_status());
 
     #if ENABLED(HOST_ACTION_COMMANDS)
       TERN_(HOST_PROMPT_SUPPORT, host_prompt_open(PROMPT_PAUSE_RESUME, PSTR("Pause SD"), PSTR("Resume")));
```

M106 / M107: fan speed / fan off

- set both fans when `dxc_is_parked` when setting speed or turning off

```diff
diff --git a/Marlin/src/gcode/temp/M106_M107.cpp b/Marlin/src/gcode/temp/M106_M107.cpp
index 3f85c53d78..7fb3f79be9 100644
--- a/Marlin/src/gcode/temp/M106_M107.cpp
+++ b/Marlin/src/gcode/temp/M106_M107.cpp
@@ -89,7 +89,13 @@ void GcodeSuite::M106() {
 
   // Set speed, with constraint
   thermalManager.set_fan_speed(pfan, speed);
-
+  #if ENABLED(DUAL_X_CARRIAGE)
+    if (dxc_is_parked())
+    {
+      thermalManager.set_fan_speed(0, speed);
+      thermalManager.set_fan_speed(1, speed);
+    }
+  #endif
   TERN_(LASER_SYNCHRONOUS_M106_M107, planner.buffer_sync_block(BLOCK_FLAG_SYNC_FANS));
 
   if (TERN0(DUAL_X_CARRIAGE, idex_is_duplicating()))  // pfan == 0 when duplicating
@@ -107,7 +113,13 @@ void GcodeSuite::M107() {
   #endif
 
   thermalManager.set_fan_speed(pfan, 0);
-
+  #if ENABLED(DUAL_X_CARRIAGE)
+   if (dxc_is_parked())
+    {
+      thermalManager.set_fan_speed(0, 0);
+      thermalManager.set_fan_speed(1, 0);
+    }
+  #endif
   if (TERN0(DUAL_X_CARRIAGE, idex_is_duplicating()))  // pfan == 0 when duplicating
     thermalManager.set_fan_speed(1 - pfan, 0);
 
```

They add touch screen stuff in here. 
Use serial port 1 for four particular motherboards, otherwise use part 3 (for creality 4.x board it says).
The ones that use port 1 are: `BTT_SKR_MINI_E3_V1_0`, `BTT_SKR_MINI_E3_V1_2`, `BTT_SKR_MINI_E3_V2_0`, `BTT_SKR_E3_TURBO`

It also enables some features based on having the touch screen.

```diff
diff --git a/Marlin/src/inc/Conditionals_LCD.h b/Marlin/src/inc/Conditionals_LCD.h
index 99b360f9f0..a46cc80bbf 100644
--- a/Marlin/src/inc/Conditionals_LCD.h
+++ b/Marlin/src/inc/Conditionals_LCD.h
@@ -87,6 +87,18 @@
   #define IS_RRD_SC 1
   #define U8GLIB_ST7920
 
+#elif ENABLED(RTS_AVAILABLE)
+  #define SERIAL_CATCHALL 0
+  #ifndef LCD_SERIAL_PORT
+    #if MB(BTT_SKR_MINI_E3_V1_0, BTT_SKR_MINI_E3_V1_2, BTT_SKR_MINI_E3_V2_0, BTT_SKR_E3_TURBO)
+      #define LCD_SERIAL_PORT 1
+    #else
+      #define LCD_SERIAL_PORT 3 // Creality 4.x board
+    #endif
+  #endif
+  #define HAS_LCD_BRIGHTNESS 1
+  #define LCD_BRIGHTNESS_MAX 250
+
 #elif ENABLED(ZONESTAR_12864OLED)
   #define IS_RRD_SC 1
   #define U8GLIB_SH1106
@@ -493,10 +505,10 @@
 #endif
 
 // Aliases for LCD features
-#if EITHER(DWIN_CREALITY_LCD, DWIN_CREALITY_LCD_ENHANCED)
+#if EITHER(RTS_AVAILABLE, DWIN_CREALITY_LCD_ENHANCED)
   #define HAS_DWIN_E3V2_BASIC 1
 #endif
-#if EITHER(HAS_DWIN_E3V2_BASIC, DWIN_CREALITY_LCD_JYERSUI)
+#if EITHER(HAS_DWIN_E3V2_BASIC, RTS_AVAILABLE)
   #define HAS_DWIN_E3V2 1
 #endif
 
```

- Adds check for touch screen to turn on `#define M600_PURGE_MORE_RESUMABLE 1`
- Defines `HAS_T_COMMAND 1` if `HAS_MULTI_EXTRUDER`
  - usually the code is checking for dual x axis, so that's slightly interesting
  - This is only used once place, also added by sovol:
    - `Marlin/src/MarlinCore.cpp:1314  TERN_(HAS_T_COMMAND, active_extruder = active_extruder_font);`


```diff
diff --git a/Marlin/src/inc/Conditionals_post.h b/Marlin/src/inc/Conditionals_post.h
index 1db4208a1f..bebf74b226 100644
--- a/Marlin/src/inc/Conditionals_post.h
+++ b/Marlin/src/inc/Conditionals_post.h
@@ -2970,7 +2970,7 @@
  * Advanced Pause - Filament Change
  */
 #if ENABLED(ADVANCED_PAUSE_FEATURE)
-  #if ANY(HAS_LCD_MENU, EXTENSIBLE_UI, DWIN_CREALITY_LCD_ENHANCED, DWIN_CREALITY_LCD_JYERSUI) || BOTH(EMERGENCY_PARSER, HOST_PROMPT_SUPPORT)
+  #if ANY(HAS_LCD_MENU, EXTENSIBLE_UI, DWIN_CREALITY_LCD_ENHANCED, DWIN_CREALITY_LCD_JYERSUI, RTS_AVAILABLE) || BOTH(EMERGENCY_PARSER, HOST_PROMPT_SUPPORT)
     #define M600_PURGE_MORE_RESUMABLE 1
   #endif
   #ifndef FILAMENT_CHANGE_SLOW_LOAD_LENGTH
@@ -3200,6 +3200,10 @@
   #endif
 #endif
 
+#if HAS_MULTI_EXTRUDER
+  #define HAS_T_COMMAND 1
+#endif
+
 #if HAS_LCD_MENU
   // LCD timeout to status screen default is 15s
   #ifndef LCD_TIMEOUT_TO_STATUS
```

SanityCheck.h

- Addded the touch screen to a condtion that makes sure only 1 touch screen option is enabled


```diff
diff --git a/Marlin/src/inc/SanityCheck.h b/Marlin/src/inc/SanityCheck.h
index 58c02e7ebd..78e9b701ab 100644
--- a/Marlin/src/inc/SanityCheck.h
+++ b/Marlin/src/inc/SanityCheck.h
@@ -2665,7 +2665,7 @@ static_assert(Y_MAX_LENGTH >= Y_BED_SIZE, "Movement bounds (Y_MIN_POS, Y_MAX_POS
   + COUNT_ENABLED(ANYCUBIC_LCD_I3MEGA, ANYCUBIC_LCD_CHIRON, ANYCUBIC_TFT35) \
   + COUNT_ENABLED(DGUS_LCD_UI_ORIGIN, DGUS_LCD_UI_FYSETC, DGUS_LCD_UI_HIPRECY, DGUS_LCD_UI_MKS, DGUS_LCD_UI_RELOADED) \
   + COUNT_ENABLED(ENDER2_STOCKDISPLAY, CR10_STOCKDISPLAY) \
-  + COUNT_ENABLED(DWIN_CREALITY_LCD, DWIN_CREALITY_LCD_ENHANCED, DWIN_CREALITY_LCD_JYERSUI, DWIN_MARLINUI_PORTRAIT, DWIN_MARLINUI_LANDSCAPE) \
+  + COUNT_ENABLED(DWIN_CREALITY_LCD, DWIN_CREALITY_LCD_ENHANCED, DWIN_CREALITY_LCD_JYERSUI, DWIN_MARLINUI_PORTRAIT, DWIN_MARLINUI_LANDSCAPE,RTS_AVAILABLE) \
   + COUNT_ENABLED(FYSETC_MINI_12864_X_X, FYSETC_MINI_12864_1_2, FYSETC_MINI_12864_2_0, FYSETC_MINI_12864_2_1, FYSETC_GENERIC_12864_1_1) \
   + COUNT_ENABLED(LCD_SAINSMART_I2C_1602, LCD_SAINSMART_I2C_2004) \
   + COUNT_ENABLED(MKS_12864OLED, MKS_12864OLED_SSD1306) \
```

They changed the version number from `2.0.9.2` to `V1.1.0`. WTF, why?

I wonder if the touch screen is expecting that or something? (Does it even send that to the touch screen?)

Otherwise, maybe just an error while applying their 1.1.0 patches to 2.x?

Other changes are less exciting.

- update distro date
- bumped protocol version from 1.0 to 1.1.0
  - I wonder why
- machine name to SV04
- set the corp website to svold3d.com


```diff
diff --git a/Marlin/src/inc/Version.h b/Marlin/src/inc/Version.h
index 12baa64fe7..6aa679f23b 100644
--- a/Marlin/src/inc/Version.h
+++ b/Marlin/src/inc/Version.h
@@ -25,12 +25,13 @@
  * Release version. Leave the Marlin version or apply a custom scheme.
  */
 #ifndef SHORT_BUILD_VERSION
-  #define SHORT_BUILD_VERSION "2.0.9.2"
+  #define SHORT_BUILD_VERSION "V1.1.0"
 #endif
 
 /**
  * Verbose version identifier containing a unique identifier, such as the
  * vendor name, download location, GitHub account, etc.
+ * 
  */
 #ifndef DETAILED_BUILD_VERSION
   #define DETAILED_BUILD_VERSION SHORT_BUILD_VERSION
@@ -42,7 +43,7 @@
  * version was tagged.
  */
 #ifndef STRING_DISTRIBUTION_DATE
-  #define STRING_DISTRIBUTION_DATE "2021-09-03"
+  #define STRING_DISTRIBUTION_DATE "2021-12-22"
 #endif
 
 /**
@@ -66,14 +67,14 @@
  * (Other behaviors are given by the firmware version and capabilities report.)
  */
 #ifndef PROTOCOL_VERSION
-  #define PROTOCOL_VERSION "1.0"
+  #define PROTOCOL_VERSION "1.1.0"
 #endif
 
 /**
  * Define a generic printer name to be output to the LCD after booting Marlin.
  */
 #ifndef MACHINE_NAME
-  #define MACHINE_NAME "3D Printer"
+  #define MACHINE_NAME "SV04"
 #endif
 
 /**
@@ -120,3 +121,4 @@
   #define USB_DEVICE_PRODUCT_NAME         MACHINE_NAME
 #endif
 #define USB_DEVICE_SERIAL_NAME            "123985739853"
+#define	CORP_WEBSITE_E	"sovol3d.com"
\ No newline at end of file
```

An "encoder" is a knob that you can turn, like the ones on many 3d printer lcds.

The SV04 touch screen doesn't have one, so this really doesn't matter to use.

Change the conditions from `HAS_DWIN_E3V2` to `DWIN_CREALITY_LCD_ENHANCED`
  - Is that just a different lcd they support?


Encoders are kind of neat. You can turn them forever, and they send little clicks to the microcontroller, so it knows how many notches you turned it and in what direction. But it doesn't have a "state". Compare to pots, which change resistance / create voltage dividers, which you can only turn for a fixed range (typically less than 360°), but doesn't have discrete clicks and is stateful.

```diff
diff --git a/Marlin/src/lcd/e3v2/common/encoder.cpp b/Marlin/src/lcd/e3v2/common/encoder.cpp
index edfaf60aae..589301af45 100644
--- a/Marlin/src/lcd/e3v2/common/encoder.cpp
+++ b/Marlin/src/lcd/e3v2/common/encoder.cpp
@@ -27,7 +27,7 @@
 
 #include "../../../inc/MarlinConfigPre.h"
 
-#if HAS_DWIN_E3V2
+#if DWIN_CREALITY_LCD_ENHANCED
 
 #include "encoder.h"
 #include "../../buttons.h"
```
LCD_RTS.cpp: new file added for touch screen

I'm not going to cover this in too much detail. It's hopefully less exciting from an upgrading perspective since this is the isolated part that isn't intermingled with the rest of the code.

It exports a bunch of methods that the rest of the code uses to report things to it. And it sends commands to the rest of the firmware.

It is a lot more coupled to the touch screen's firmware than I expected. I guess that explains why everyone's custom firmware also needs custom touch screen firmware.

For a lot of things, it calls functions.
But also for a lot of things, it queues G Code. Here's a list of some of the gcode it sends:

- M24: start/resume sd print
- G28: homeing
- G92.9: 
 - With POWER_LOSS_RECOVERY:
   -   G92.9 : Set NATIVE Current Position to the given X Y Z E.
- M84: disable steppers
- M420 S1 / S0: get or set bed leveling state. (So there's some condition where it turns it off)
- M218 T1 X/Y/Z: set hotend offset (from each other)
- G28 Z0: homing
- G29: bilinear leveling
- G34: z axix alignment
- T0 / T1: tool change / switch extruder
- M605 S1 / S2 / S2 X68 R0 / S3 / S0: multi nozzle mode (0 = full control, 1 = autopark, 2 = copy mode, 3 = mirror mode, R = temp difference, X= difference between two extruders)
- M1000: 
  - /**
  -  * M1000: Resume from power-loss (undocumented)
  -  *   - With 'S' go to the Resume/Cancel menu
  -  *   - With no parameters, run recovery commands
  -  */

```
    queue.enqueue_now_P(PSTR("M420 S1"));
          queue.enqueue_now_P(PSTR("M24"));
        queue.enqueue_now_P(PSTR("G28"));
        queue.enqueue_now_P(PSTR("G1 F200 Z0.0"));
          sprintf_P(commandbuf, PSTR("G92.9 X%6.3f"), 
        queue.enqueue_now_P(PSTR("M84"));
          queue.enqueue_now_P(PSTR("M420 S1"));
        sprintf_P(commandbuf, PSTR("M218 T1 X%4.1f"), hotend_offset[1].x);
        sprintf_P(commandbuf, PSTR("M218 T1 Y%4.1f"), hotend_offset[1].y);
        sprintf_P(commandbuf, PSTR("M218 T1 Z%4.1f"), hotend_offset[1].z);
          queue.enqueue_now_P(PSTR("G28 Z0"));
        queue.enqueue_now_P(PSTR("G1 F200 Z0.0"));
        queue.enqueue_now_P(PSTR("M420 S0"));
          queue.enqueue_now_P(PSTR("G29"));
          queue.enqueue_now_P(PSTR("G1 F600 Z3"));
          queue.enqueue_now_P(PSTR("G1 X150 Y150 F3000"));
        queue.enqueue_now_P(PSTR("G28 X"));
        queue.enqueue_now_P(PSTR("G34"));
            queue.enqueue_now_P(PSTR("G28 X"));
            queue.enqueue_now_P(PSTR("T1"));
            queue.enqueue_now_P(PSTR("T0"));
              queue.enqueue_now_P(PSTR("M605 S1"));
              queue.enqueue_now_P(PSTR("M605 S2"));
              queue.enqueue_now_P(PSTR("M605 S2 X68 R0"));
              queue.enqueue_now_P(PSTR("M605 S3"));
              queue.enqueue_now_P(PSTR("M605 S0"));
          queue.enqueue_now_P(PSTR("M1000"));
        queue.enqueue_now_P(PSTR("M24"));
          queue.enqueue_now_P(PSTR("M420 S1"));
        sprintf_P(commandbuf, PSTR("M218 T1 X%4.1f"), hotend_offset[1].x);
        sprintf_P(commandbuf, PSTR("M218 T1 Y%4.1f"), hotend_offset[1].y);
        sprintf_P(commandbuf, PSTR("M218 T1 Z%4.1f"), hotend_offset[1].z);
```

```diff
diff --git a/Marlin/src/lcd/e3v2/creality/LCD_RTS.cpp b/Marlin/src/lcd/e3v2/creality/LCD_RTS.cpp
new file mode 100644
index 0000000000..2a63c281c8
--- /dev/null
+++ b/Marlin/src/lcd/e3v2/creality/LCD_RTS.cpp
@@ -0,0 +1,2917 @@
+#include "LCD_RTS.h"
+#include <WString.h>
+#include <stdio.h>
+#include <string.h>
+#include <Arduino.h>
+#include "../../../MarlinCore.h"
+#include "../../../inc/MarlinConfig.h"
+#include "../../../module/settings.h"
+#include "../../../core/serial.h"
+#include "../../../core/macros.h"
+
+#include "../../fontutils.h"
+#include "../../../sd/cardreader.h"
+#include "../../../feature/powerloss.h"
+#include "../../../feature/babystep.h"
+#include "../../../module/temperature.h"
+#include "../../../module/printcounter.h"
+#include "../../../module/motion.h"
+#include "../../../module/planner.h"
+#include "../../../gcode/queue.h"
+#include "../../../gcode/gcode.h"
+#include "../../../module/probe.h"
+
+#include "../../../feature/bedlevel/abl/abl.h"
+
+#if ENABLED(RTS_AVAILABLE)
+
+#define CHECKFILEMENT true
+
+float zprobe_zoffset;
+float last_zoffset = 0.0;
+
+const float manual_feedrate_mm_m[] = {50 * 60, 50 * 60, 4 * 60, 60};
+
+int startprogress = 0;
+CRec CardRecbuf;
+float pause_z = 0;
+float pause_e = 0;
+bool sdcard_pause_check = true;
+bool print_preheat_check = false;
+
+float ChangeFilament0Temp = 200;
+float ChangeFilament1Temp = 200;
+
+float current_position_x0_axis = X_MIN_POS;
+float current_position_x1_axis = X2_MAX_POS;
+int StartFlag = 0;   
+int PrintFlag = 0;
+
+int heatway = 0;
+millis_t next_rts_update_ms = 0;
+int last_target_temperature[4] = {0};
+int last_target_temperature_bed;
+char waitway = 0;
+int recnum = 0;
+unsigned char Percentrecord = 0;
+
+bool pause_action_flag = false;
+int power_off_type_yes = 0;
+// represents to update file list
+bool CardUpdate = false;
+
+extern CardReader card;
+// represents SD-card status, true means SD is available, false means opposite.
+bool lcd_sd_status;
+
+char Checkfilenum = 0;
+int FilenamesCount = 0;
+char cmdbuf[20] = {0};
+float Filament0LOAD = 10;
+float Filament1LOAD = 10;
+float XoffsetValue ;
+
+// 0 for 10mm, 1 for 1mm, 2 for 0.1mm
+unsigned char AxisUnitMode;
+float axis_unit = 10;
+unsigned char AutoHomeIconNum;
+RTSSHOW rtscheck;
+int Update_Time_Value = 0;
+
+bool PoweroffContinue = false;
+char commandbuf[30];
+bool active_extruder_flag = false;
+
+static int change_page_number = 0; 
+
+char save_dual_x_carriage_mode = 0;
+
+uint16_t remain_time = 0;
+
+static bool last_card_insert_st;
+bool card_insert_st;
+bool sd_printing;
+bool sd_printing_autopause;
+inline void RTS_line_to_current(AxisEnum axis)
+{
+  if (!planner.is_full())
+  {
+    planner.buffer_line(current_position, MMM_TO_MMS(manual_feedrate_mm_m[(int8_t)axis]), active_extruder);
+  }
+}
+
+RTSSHOW::RTSSHOW()
+{
+  recdat.head[0] = snddat.head[0] = FHONE;
+  recdat.head[1] = snddat.head[1] = FHTWO;
+  memset(databuf, 0, sizeof(databuf));
+}
+
+void RTSSHOW::RTS_SDCardInit(void)
+{
+  if(RTS_SD_Detected())
+  {
+    card.mount();
+  }
+  if(CardReader::flag.mounted)
+  {
+    uint16_t fileCnt = card.get_num_Files();
+    card.getWorkDirName();
+    if(card.filename[0] != '/')
+    {
+      card.cdup();
+    }
+
+    int addrnum = 0;
+    int num = 0;
+    for(uint16_t i = 0;(i < fileCnt) && (i < (MaxFileNumber + addrnum));i ++)
+    {
+      card.selectFileByIndex(fileCnt - 1 - i);
+      char *pointFilename = card.longFilename;
+      int filenamelen = strlen(card.longFilename);
+      int j = 1;
+      while((strncmp(&pointFilename[j], ".gcode", 6) && strncmp(&pointFilename[j], ".GCODE", 6)) && ((j ++) < filenamelen));
+      if(j >= filenamelen)
+      {
+        addrnum++;
+        continue;
+      }
+
+      if (j >= TEXTBYTELEN)
+      {
+        strncpy(&card.longFilename[TEXTBYTELEN - 3], "..", 2);
+        card.longFilename[TEXTBYTELEN - 1] = '\0';
+        j = TEXTBYTELEN - 1;
+      }
+
+      strncpy(CardRecbuf.Cardshowfilename[num], card.longFilename, j);
+
+      strcpy(CardRecbuf.Cardfilename[num], card.filename);
+      CardRecbuf.addr[num] = FILE1_TEXT_VP + (num * 20);
+      RTS_SndData(CardRecbuf.Cardshowfilename[num], CardRecbuf.addr[num]);
+      CardRecbuf.Filesum = (++num);
+    }
+    for(int j = CardRecbuf.Filesum;j < MaxFileNumber;j ++)
+    {
+      CardRecbuf.addr[j] = FILE1_TEXT_VP + (j * 20);
+      RTS_SndData(0, CardRecbuf.addr[j]);
+    }
+    for(int j = 0;j < 20;j ++)
+    {
+      // clean print file
+      RTS_SndData(0, PRINT_FILE_TEXT_VP + j);
+    }
+    lcd_sd_status = IS_SD_INSERTED();
+  }
+  else
+  {
+    if(sd_printing_autopause == true)
+    {
+      RTS_SndData(CardRecbuf.Cardshowfilename[CardRecbuf.recordcount], PRINT_FILE_TEXT_VP);
+      card.mount();
+    }
+    else
+    {
+      // clean filename Icon
+      for(int j = 0;j < MaxFileNumber;j ++)
+      {
+        // clean filename Icon
+        for(int i = 0;i < TEXTBYTELEN;i ++)
+        {
+          RTS_SndData(0, CardRecbuf.addr[j] + i);
+        }
+      }
+      memset(&CardRecbuf, 0, sizeof(CardRecbuf));
+    }
+  }
+}
+
+bool RTSSHOW::RTS_SD_Detected(void)
+{
+  static bool last;
+  static bool state;
+  static bool flag_stable;
+  static uint32_t stable_point_time;
+
+  bool tmp = IS_SD_INSERTED();
+
+  if(tmp != last)
+  {
+    flag_stable = false;
+  }
+  else
+  {
+    if(!flag_stable)
+    {
+      flag_stable = true;
+      stable_point_time = millis();
+    }
+  }
+
+  if(flag_stable)
+  {
+    if(millis() - stable_point_time > 30)
+    {
+      state = tmp;
+    }
+  }
+
+  last = tmp;
+
+  return state;
+}
+
+void RTSSHOW::RTS_SDCardUpate(void)
+{
+  const bool sd_status = RTS_SD_Detected();
+  if (sd_status != lcd_sd_status)
+  {
+    if (sd_status)
+    {
+      // SD card power on
+      card.mount();
+      RTS_SDCardInit();
+    }
+    else
+    {
+      card.release();
+      if(sd_printing_autopause == true)
+      {
+        RTS_SndData(CardRecbuf.Cardshowfilename[CardRecbuf.recordcount], PRINT_FILE_TEXT_VP);
+      }
+      else
+      {
+        for(int i = 0;i < CardRecbuf.Filesum;i ++)
+        {
+          for(int j = 0;j < 20;j ++)
+          {
+            RTS_SndData(0, CardRecbuf.addr[i] + j);
+          }
+          RTS_SndData((unsigned long)0xA514, FilenameNature + (i + 1) * 16);
+        }
+
+        for(int j = 0;j < 20;j ++)
+        {
+          // clean screen.
+          RTS_SndData(0, PRINT_FILE_TEXT_VP + j);
+          RTS_SndData(0, SELECT_FILE_TEXT_VP + j);
+        }
+        memset(&CardRecbuf, 0, sizeof(CardRecbuf));
+      }
+    }
+    lcd_sd_status = sd_status;
+  }
+
+  // represents to update file list
+  if(CardUpdate && lcd_sd_status && RTS_SD_Detected())
+  {
+    for(uint16_t i = 0;i < CardRecbuf.Filesum;i ++)
+    {
+      delay(1);
+      RTS_SndData(CardRecbuf.Cardshowfilename[i], CardRecbuf.addr[i]);
+      RTS_SndData((unsigned long)0xA514, FilenameNature + (i + 1) * 16);
+    }
+    CardUpdate = false;
+  }
+}
+
+void RTSSHOW::RTS_Init()
+{
+  AxisUnitMode = 1;
+  active_extruder = active_extruder_font;
+  #if ENABLED(DUAL_X_CARRIAGE)
+    save_dual_x_carriage_mode = dualXPrintingModeStatus;
+    if(save_dual_x_carriage_mode == 1)
+    {
+      RTS_SndData(1, PRINT_MODE_ICON_VP);
+      RTS_SndData(1, SELECT_MODE_ICON_VP);
+    }
+    else if(save_dual_x_carriage_mode == 2)
+    {
+      RTS_SndData(2, PRINT_MODE_ICON_VP);
+      RTS_SndData(2, SELECT_MODE_ICON_VP);
+    }
+    else if(save_dual_x_carriage_mode == 3)
+    {
+      RTS_SndData(3, PRINT_MODE_ICON_VP);
+      RTS_SndData(3, SELECT_MODE_ICON_VP);
+    }
+    else if(save_dual_x_carriage_mode == 4)
+    {
+      RTS_SndData(5, PRINT_MODE_ICON_VP);
+      RTS_SndData(5, SELECT_MODE_ICON_VP);
+    }
+    else 
+    {
+      RTS_SndData(4, PRINT_MODE_ICON_VP);
+      RTS_SndData(4, SELECT_MODE_ICON_VP);
+    }
+  #endif
+  #if ENABLED(AUTO_BED_LEVELING_BILINEAR)
+    bool zig = false;
+    int8_t inStart, inStop, inInc, showcount;
+    showcount = 0;
+    //settings.load();
+    for (int y = 0; y < GRID_MAX_POINTS_Y; y++)
+    {
+      // away from origin
+      if (zig)
+      {
+        inStart = 0;
+        inStop = GRID_MAX_POINTS_X;
+        inInc = 1;
+      }
+      else
+      {
+        // towards origin
+        inStart = GRID_MAX_POINTS_X - 1;
+        inStop = -1;
+        inInc = -1;
+      }
+      zig ^= true;
+      for (int x = inStart; x != inStop; x += inInc)
+      {
+        RTS_SndData(z_values[x][y] * 1000, AUTO_BED_LEVEL_1POINT_VP + showcount * 2);
+        showcount++;
+      }
+    }
+    queue.enqueue_now_P(PSTR("M420 S1"));
+  #endif
+  last_zoffset = zprobe_zoffset = probe.offset.z;
+  RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+  RTS_SndData((hotend_offset[1].x - X2_MAX_POS) * 100, TWO_EXTRUDER_HOTEND_XOFFSET_VP);
+  RTS_SndData(hotend_offset[1].y * 100, TWO_EXTRUDER_HOTEND_YOFFSET_VP);
+  RTS_SndData(hotend_offset[1].z * 100, TWO_EXTRUDER_HOTEND_ZOFFSET_VP);
+
+  last_target_temperature_bed = thermalManager.temp_bed.target;
+  last_target_temperature[0] = thermalManager.temp_hotend[0].target;
+  last_target_temperature[1] = thermalManager.temp_hotend[1].target;
+  feedrate_percentage = 100;
+  RTS_SndData(feedrate_percentage, PRINT_SPEED_RATE_VP);
+
+  /***************turn off motor*****************/
+  RTS_SndData(1, MOTOR_FREE_ICON_VP);
+
+  /***************transmit temperature to screen*****************/
+  RTS_SndData(0, HEAD0_SET_TEMP_VP);
+  RTS_SndData(0, HEAD1_SET_TEMP_VP);
+  RTS_SndData(0, BED_SET_TEMP_VP);
+  RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+  RTS_SndData(thermalManager.temp_hotend[1].celsius, HEAD1_CURRENT_TEMP_VP);
+  RTS_SndData(thermalManager.temp_bed.celsius, BED_CURRENT_TEMP_VP);
+
+  /***************transmit Fan speed to screen*****************/
+  // turn off fans
+  thermalManager.set_fan_speed(0, 0);
+  thermalManager.set_fan_speed(1, 0);
+  RTS_SndData(1, HEAD0_FAN_ICON_VP);
+  RTS_SndData(1, HEAD1_FAN_ICON_VP);
+  delay(5);
+
+  /*********transmit SD card filename to screen***************/
+  RTS_SDCardInit();
+
+  /***************transmit Printer information to screen*****************/
+  char sizebuf[20] = {0};
+  sprintf(sizebuf, "%d X %d X %d", X_MAX_POS - 2, Y_MAX_POS - 2, Z_MAX_POS);
+  RTS_SndData(MACVERSION, PRINTER_MACHINE_TEXT_VP);
+  RTS_SndData(SOFTVERSION, PRINTER_VERSION_TEXT_VP);
+  RTS_SndData(sizebuf, PRINTER_PRINTSIZE_TEXT_VP);
+  RTS_SndData(CORP_WEBSITE, PRINTER_WEBSITE_TEXT_VP);
+  RTS_SndData(Screen_version, Screen_Version_VP);
+  /**************************some info init*******************************/
+  RTS_SndData(0, PRINT_PROCESS_ICON_VP);
+  if(CardReader::flag.mounted)
+  {
+    change_page_number = 1;
+  }
+  else
+  {
+    change_page_number = 0;
+  }
+}
+
+int RTSSHOW::RTS_RecData()
+{
+	int frame_index = 0;
+	int timeout = 0;
+	int framelen = 0;
+	bool frame_flag = false;
+	if(LCD_SERIAL.available() <= 0){
+		return -1;
+	}
+	do{
+		if(LCD_SERIAL.available() > 0){
+			databuf[frame_index] = LCD_SERIAL.read();
+			timeout = 0;
+			/* 0x5A */
+			if((frame_index == 0) && (databuf[frame_index] == FHONE)){
+				frame_index++;
+				continue;
+			}
+			/* 0xA5 */
+			else if(frame_index == 1){
+				if(databuf[frame_index] == FHTWO){
+					frame_index++;
+				}
+				else{
+					frame_index = 0;
+				}
+				continue;
+			}
+			/* 长度 */
+			else if(frame_index == 2){
+				framelen = databuf[frame_index];
+				frame_index++;
+				continue;
+			}
+			else if(frame_index != 0){
+				frame_index++;
+				/* 一帧数据提取完毕，剩余的串口数据下次进入这个函数会在处理 */
+				if(frame_index == (framelen + 3)){
+					frame_flag = true;
+					break;
+				}
+			}
+		}
+		else{
+			timeout++;
+			delay(1);
+		}
+	}while(timeout < 50); /* 超时函数 */
+//	MYSERIAL0.write(0xBB);
+	
+	if(frame_flag == true){
+		recdat.head[0] = databuf[0];
+		recdat.head[1] = databuf[1];
+		recdat.len = databuf[2];
+		recdat.command = databuf[3];
+		for(int idx = 0; idx < frame_index; idx++){
+		}
+
+	}
+	else{
+		return -1;
+	}
+    // response for writing byte
+    if ((recdat.len == 0x03) && 
+		((recdat.command == 0x82) || (recdat.command == 0x80)) && 
+		(databuf[4] == 0x4F) && 
+		(databuf[5] == 0x4B)){
+		memset(databuf, 0, sizeof(databuf));
+		recnum = 0;
+		return -1;
+    }
+    else if (recdat.command == 0x83){
+		// response for reading the data from the variate
+		recdat.addr = databuf[4];
+		recdat.addr = (recdat.addr << 8) | databuf[5];
+		recdat.bytelen = databuf[6];
+		for (unsigned int i = 0; i < recdat.bytelen; i += 2){
+			recdat.data[i / 2] = databuf[7 + i];
+			recdat.data[i / 2] = (recdat.data[i / 2] << 8) | databuf[8 + i];
+		}
+    }
+    else if (recdat.command == 0x81){
+		// response for reading the page from the register
+		recdat.addr = databuf[4];
+		recdat.bytelen = databuf[5];
+		for (unsigned int i = 0; i < recdat.bytelen; i ++){
+			recdat.data[i] = databuf[6 + i];
+			// recdat.data[i] = (recdat.data[i] << 8 )| databuf[7 + i];
+		}
+    }
+	else{
+		memset(databuf, 0, sizeof(databuf));
+		recnum = 0;
+		// receive the wrong data
+		return -1;
+	}
+	memset(databuf, 0, sizeof(databuf));
+	recnum = 0;
+
+	return 2;
+}
+
+void RTSSHOW::RTS_SndData(void)
+{
+  if((snddat.head[0] == FHONE) && (snddat.head[1] == FHTWO) && (snddat.len >= 3))
+  {
+    databuf[0] = snddat.head[0];
+    databuf[1] = snddat.head[1];
+    databuf[2] = snddat.len;
+    databuf[3] = snddat.command;
+    // to write data to the register
+    if(snddat.command == 0x80)
+    {
+      databuf[4] = snddat.addr;
+      for(int i = 0;i <(snddat.len - 2);i ++)
+      {
+        databuf[5 + i] = snddat.data[i];
+      }
+    }
+    else if((snddat.len == 3) && (snddat.command == 0x81))
+    {
+      // to read data from the register
+      databuf[4] = snddat.addr;
+      databuf[5] = snddat.bytelen;
+    }
+    else if(snddat.command == 0x82)
+    {
+      // to write data to the variate
+      databuf[4] = snddat.addr >> 8;
+      databuf[5] = snddat.addr & 0xFF;
+      for(int i =0;i <(snddat.len - 3);i += 2)
+      {
+        databuf[6 + i] = snddat.data[i/2] >> 8;
+        databuf[7 + i] = snddat.data[i/2] & 0xFF;
+      }
+    }
+    else if((snddat.len == 4) && (snddat.command == 0x83))
+    {
+      // to read data from the variate
+      databuf[4] = snddat.addr >> 8;
+      databuf[5] = snddat.addr & 0xFF;
+      databuf[6] = snddat.bytelen;
+    }
+     for(int i = 0;i < (snddat.len + 3); i ++)
+     {
+       LCD_SERIAL.write(databuf[i]);
+     }
+
+    memset(&snddat, 0, sizeof(snddat));
+    memset(databuf, 0, sizeof(databuf));
+    snddat.head[0] = FHONE;
+    snddat.head[1] = FHTWO;
+  }
+}
+
+void RTSSHOW::RTS_SndData(const String &s, unsigned long addr, unsigned char cmd /*= VarAddr_W*/)
+{
+  if(s.length() < 1)
+  {
+    return;
+  }
+  RTS_SndData(s.c_str(), addr, cmd);
+}
+
+void RTSSHOW::RTS_SndData(const char *str, unsigned long addr, unsigned char cmd/*= VarAddr_W*/)
+{
+  int len = strlen(str);
+  if(len > 0)
+  {
+    databuf[0] = FHONE;
+    databuf[1] = FHTWO;
+    databuf[2] = 3 + len;
+    databuf[3] = cmd;
+    databuf[4] = addr >> 8;
+    databuf[5] = addr & 0x00FF;
+    for(int i = 0;i < len;i ++)
+    {
+      databuf[6 + i] = str[i];
+    }
+
+    for(int i = 0;i < (len + 6);i ++)
+    {
+      LCD_SERIAL.write(databuf[i]);
+      //delayMicroseconds(1);
+    }
+    memset(databuf, 0, sizeof(databuf));
+  }
+}
+
+void RTSSHOW::RTS_SndData(char c, unsigned long addr, unsigned char cmd /*= VarAddr_W*/)
+{
+  snddat.command = cmd;
+  snddat.addr = addr;
+  snddat.data[0] = (unsigned long)c;
+  snddat.data[0] = snddat.data[0] << 8;
+  snddat.len = 5;
+  RTS_SndData();
+}
+
+void RTSSHOW::RTS_SndData(unsigned char *str, unsigned long addr, unsigned char cmd) { RTS_SndData((char *)str, addr, cmd); }
+
+void RTSSHOW::RTS_SndData(int n, unsigned long addr, unsigned char cmd /*= VarAddr_W*/)
+{
+  if (cmd == VarAddr_W)
+  {
+    if (n > 0xFFFF)
+    {
+      snddat.data[0] = n >> 16;
+      snddat.data[1] = n & 0xFFFF;
+      snddat.len = 7;
+    }
+    else
+    {
+      snddat.data[0] = n;
+      snddat.len = 5;
+    }
+  }
+  else if (cmd == RegAddr_W)
+  {
+    snddat.data[0] = n;
+    snddat.len = 3;
+  }
+  else if (cmd == VarAddr_R)
+  {
+    snddat.bytelen = n;
+    snddat.len = 4;
+  }
+  snddat.command = cmd;
+  snddat.addr = addr;
+  RTS_SndData();
+}
+
+void RTSSHOW::RTS_SndData(unsigned int n, unsigned long addr, unsigned char cmd) { RTS_SndData((int)n, addr, cmd); }
+
+void RTSSHOW::RTS_SndData(float n, unsigned long addr, unsigned char cmd) { RTS_SndData((int)n, addr, cmd); }
+
+void RTSSHOW::RTS_SndData(long n, unsigned long addr, unsigned char cmd) { RTS_SndData((unsigned long)n, addr, cmd); }
+
+void RTSSHOW::RTS_SndData(unsigned long n, unsigned long addr, unsigned char cmd /*= VarAddr_W*/)
+{
+  if (cmd == VarAddr_W)
+  {
+    if (n > 0xFFFF)
+    {
+      snddat.data[0] = n >> 16;
+      snddat.data[1] = n & 0xFFFF;
+      snddat.len = 7;
+    }
+    else
+    {
+      snddat.data[0] = n;
+      snddat.len = 5;
+    }
+  }
+  else if (cmd == VarAddr_R)
+  {
+    snddat.bytelen = n;
+    snddat.len = 4;
+  }
+  snddat.command = cmd;
+  snddat.addr = addr;
+  RTS_SndData();
+}
+
+void RTSSHOW::RTS_SDcard_Stop()
+{
+  waitway = 7;
+  change_page_number = 1;
+  #if ENABLED(DUAL_X_CARRIAGE)
+    extruder_duplication_enabled = false;
+    dual_x_carriage_mode = DEFAULT_DUAL_X_CARRIAGE_MODE;
+    active_extruder = 0;
+  #endif
+  card.flag.abort_sd_printing = true;
+
+  #if DISABLED(SD_ABORT_NO_COOLDOWN)
+    thermalManager.disable_all_heaters();
+  #endif
+  print_job_timer.reset();
+  
+  thermalManager.setTargetHotend(0, 0);
+  RTS_SndData(0, HEAD0_SET_TEMP_VP);
+  thermalManager.setTargetHotend(0, 1);
+  RTS_SndData(0, HEAD1_SET_TEMP_VP);
+  thermalManager.setTargetBed(0);
+  RTS_SndData(0, BED_SET_TEMP_VP);
+  thermalManager.zero_fan_speeds();
+  wait_for_heatup = wait_for_user = false;
+  PoweroffContinue = false;
+  sd_printing_autopause = false;
+  if(CardReader::flag.mounted)
+  {
+    #if ENABLED(SDSUPPORT) && ENABLED(POWER_LOSS_RECOVERY)
+      card.removeJobRecoveryFile();
+    #endif
+  }
+  #ifdef EVENT_GCODE_SD_STOP
+    queue.inject_P(PSTR(EVENT_GCODE_SD_STOP));
+  #endif
+
+  // shut down the stepper motor.
+  // queue.enqueue_now_P(PSTR("M84"));
+  RTS_SndData(0, MOTOR_FREE_ICON_VP);
+
+  RTS_SndData(0, PRINT_PROCESS_ICON_VP);
+  RTS_SndData(0, PRINT_PROCESS_VP);
+  delay(2);
+  for(int j = 0;j < 20;j ++)
+  {
+    // clean screen.
+    RTS_SndData(0, PRINT_FILE_TEXT_VP + j);
+    // clean filename
+    RTS_SndData(0, SELECT_FILE_TEXT_VP + j);
+  }
+}
+
+void RTSSHOW::RTS_HandleData()
+{
+  int Checkkey = -1;
+  // for waiting
+  if(waitway > 0)
+  {
+    memset(&recdat, 0, sizeof(recdat));
+    recdat.head[0] = FHONE;
+    recdat.head[1] = FHTWO;
+    return;
+  }
+  for(int i = 0;Addrbuf[i] != 0;i ++)
+  {
+    if(recdat.addr == Addrbuf[i])
+    {
+      if(Addrbuf[i] >= ChangePageKey)
+      {
+        Checkkey = i;
+      }
+      break;
+    }
+  }
+
+  if(Checkkey < 0)
+  {
+    memset(&recdat, 0, sizeof(recdat));
+    recdat.head[0] = FHONE;
+    recdat.head[1] = FHTWO;
+    return;
+  }
+
+  switch(Checkkey)
+  {
+    case MainPageKey:
+      if(recdat.data[0] == 1)
+      {
+        CardUpdate = true;
+        CardRecbuf.recordcount = -1;
+        if(CardReader::flag.mounted)
+        {
+          for (int j = 0; j < 20; j ++)
+            {
+              RTS_SndData(0, SELECT_FILE_TEXT_VP + j);
+            }
+          RTS_SndData(ExchangePageBase + 2, ExchangepageAddr);
+        }
+        else
+        {
+          RTS_SndData(ExchangePageBase + 47, ExchangepageAddr);
+        }
+      }
+      else if(recdat.data[0] == 2)
+      {
+        card.flag.abort_sd_printing = true;
+        print_job_timer.reset();
+        queue.clear();
+        quickstop_stepper();
+        RTS_SndData(0, MOTOR_FREE_ICON_VP);
+        RTS_SndData(0, PRINT_PROCESS_ICON_VP);
+        RTS_SndData(0, PRINT_PROCESS_VP);
+        delay(2);
+        RTS_SndData(0, PRINT_TIME_HOUR_VP);
+        RTS_SndData(0, PRINT_TIME_MIN_VP);
+        RTS_SndData(0, PRINT_SURPLUS_TIME_HOUR_VP);
+        RTS_SndData(0, PRINT_SURPLUS_TIME_MIN_VP);
+
+        sd_printing_autopause = false;
+        change_page_number = 1;
+        RTS_SndData(ExchangePageBase + 1, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 3)
+      {
+        thermalManager.fan_speed[0] ? RTS_SndData(0, HEAD0_FAN_ICON_VP) : RTS_SndData(1, HEAD0_FAN_ICON_VP);
+        thermalManager.fan_speed[1] ? RTS_SndData(0, HEAD1_FAN_ICON_VP) : RTS_SndData(1, HEAD1_FAN_ICON_VP);
+        RTS_SndData(ExchangePageBase + 15, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 4)
+      {
+        RTS_SndData(ExchangePageBase + 21, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 5)
+      {
+        #if ENABLED(DUAL_X_CARRIAGE)
+          save_dual_x_carriage_mode = dualXPrintingModeStatus;
+          if(save_dual_x_carriage_mode == 1)
+          {
+            RTS_SndData(1, TWO_COLOR_MODE_ICON_VP);
+            RTS_SndData(0, COPY_MODE_ICON_VP);
+            RTS_SndData(0, MIRROR_MODE_ICON_VP);
+            RTS_SndData(0, SINGLE_MODE_ICON_VP);
+
+            RTS_SndData(1, PRINT_MODE_ICON_VP);
+            RTS_SndData(1, SELECT_MODE_ICON_VP);
+          }
+          else if(save_dual_x_carriage_mode == 2)
+          {
+            RTS_SndData(0, TWO_COLOR_MODE_ICON_VP);
+            RTS_SndData(1, COPY_MODE_ICON_VP);
+            RTS_SndData(0, MIRROR_MODE_ICON_VP);
+            RTS_SndData(0, SINGLE_MODE_ICON_VP);
+
+            RTS_SndData(2, PRINT_MODE_ICON_VP);
+            RTS_SndData(2, SELECT_MODE_ICON_VP);
+          }
+          else if(save_dual_x_carriage_mode == 3)
+          {
+            RTS_SndData(0, TWO_COLOR_MODE_ICON_VP);
+            RTS_SndData(0, COPY_MODE_ICON_VP);
+            RTS_SndData(1, MIRROR_MODE_ICON_VP);
+            RTS_SndData(0, SINGLE_MODE_ICON_VP);
+
+            RTS_SndData(3, PRINT_MODE_ICON_VP);
+            RTS_SndData(3, SELECT_MODE_ICON_VP);
+          }
+          else if(save_dual_x_carriage_mode == 4)
+          {
+            RTS_SndData(0, TWO_COLOR_MODE_ICON_VP);
+            RTS_SndData(0, COPY_MODE_ICON_VP);
+            RTS_SndData(0, MIRROR_MODE_ICON_VP);
+            RTS_SndData(2, SINGLE_MODE_ICON_VP);
+
+            RTS_SndData(5, PRINT_MODE_ICON_VP);
+            RTS_SndData(5, SELECT_MODE_ICON_VP);
+          }
+          else
+          {
+            RTS_SndData(0, TWO_COLOR_MODE_ICON_VP);
+            RTS_SndData(0, COPY_MODE_ICON_VP);
+            RTS_SndData(0, MIRROR_MODE_ICON_VP);
+            RTS_SndData(1, SINGLE_MODE_ICON_VP);
+
+            RTS_SndData(4, PRINT_MODE_ICON_VP);
+            RTS_SndData(4, SELECT_MODE_ICON_VP);
+          }
+
+          RTS_SndData(ExchangePageBase + 34, ExchangepageAddr);
+        #endif
+      }
+      break;
+
+    case AdjustmentKey:
+      if(recdat.data[0] == 1)
+      {
+        thermalManager.fan_speed[0] ? RTS_SndData(0, HEAD0_FAN_ICON_VP) : RTS_SndData(1, HEAD0_FAN_ICON_VP);
+        thermalManager.fan_speed[1] ? RTS_SndData(0, HEAD1_FAN_ICON_VP) : RTS_SndData(1, HEAD1_FAN_ICON_VP);
+        RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+        RTS_SndData(ExchangePageBase + 14, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 2)
+      {
+        if(PrintFlag == 1)
+          RTS_SndData(ExchangePageBase + 12, ExchangepageAddr);
+        else
+          RTS_SndData(ExchangePageBase + 11, ExchangepageAddr);
+        //settings.save();
+      }
+      else if(recdat.data[0] == 3)
+      {
+        if (thermalManager.fan_speed[0])
+        {
+          RTS_SndData(1, HEAD0_FAN_ICON_VP);
+          thermalManager.set_fan_speed(0, 0);
+        }
+        else
+        {
+          RTS_SndData(0, HEAD0_FAN_ICON_VP);
+          thermalManager.set_fan_speed(0, 255);
+        }
+      }
+      else if(recdat.data[0] == 4)
+      {
+        if (thermalManager.fan_speed[1])
+        {
+          RTS_SndData(1, HEAD1_FAN_ICON_VP);
+          thermalManager.set_fan_speed(1, 0);
+        }
+        else
+        {
+          RTS_SndData(0, HEAD1_FAN_ICON_VP);
+          thermalManager.set_fan_speed(1, 255);
+        }
+      }
+      break;
+
+    case PrintSpeedKey:
+      feedrate_percentage = recdat.data[0];
+      RTS_SndData(feedrate_percentage, PRINT_SPEED_RATE_VP);
+      break;
+
+    case StopPrintKey:
+      if((recdat.data[0] == 1) || (recdat.data[0] == 0xF1))
+      {
+        RTS_SndData(ExchangePageBase + 40, ExchangepageAddr);
+        RTS_SndData(0, PRINT_TIME_HOUR_VP);
+        RTS_SndData(0, PRINT_TIME_MIN_VP);
+        RTS_SndData(0, PRINT_SURPLUS_TIME_HOUR_VP);
+        RTS_SndData(0, PRINT_SURPLUS_TIME_MIN_VP);
+        Update_Time_Value = 0;
+        RTS_SDcard_Stop();
+        PrintFlag = 0;
+      }
+      else if(recdat.data[0] == 0xF0)
+      {
+        if(card.isPrinting)
+        {
+          RTS_SndData(ExchangePageBase + 11, ExchangepageAddr);
+        }
+        else if(sdcard_pause_check == false)
+        {
+          RTS_SndData(ExchangePageBase + 12, ExchangepageAddr);
+        }
+        else
+        {
+          RTS_SndData(ExchangePageBase + 10, ExchangepageAddr);
+        }
+      }
+      break;
+
+    case PausePrintKey:
+      if(recdat.data[0] == 0xF0)
+      {
+        break;
+      }
+      else if(recdat.data[0] == 0xF1)
+      {
+        RTS_SndData(ExchangePageBase + 40, ExchangepageAddr);
+        // reject to receive cmd
+        waitway = 1;
+        pause_z = current_position[Z_AXIS];
+        card.pauseSDPrint();
+        pause_action_flag = true;
+        Update_Time_Value = 0;
+        sdcard_pause_check = false;
+        PrintFlag = 1;
+        change_page_number = 12;
+      }
+      break;
+
+    case ResumePrintKey:
+      if(recdat.data[0] == 1)
+      {
+        #if ENABLED(CHECKFILEMENT)
+          if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+          else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+          else if((4 != save_dual_x_carriage_mode) && (0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+          {
+            rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+        #endif
+        RTS_SndData(ExchangePageBase + 40, ExchangepageAddr);
+
+        card.startOrResumeFilePrinting();
+        print_job_timer.start();
+
+        Update_Time_Value = 0;
+        sdcard_pause_check = true;
+        pause_action_flag = false;
+        RTS_SndData(ExchangePageBase + 11, ExchangepageAddr);
+        PrintFlag = 2;
+      }
+      else if(recdat.data[0] == 2)
+      {
+        #if ENABLED(CHECKFILEMENT)
+          if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+          else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+          else if((4 != save_dual_x_carriage_mode) && (0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+          {
+            rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+          else
+          {
+            RTS_SndData(ExchangePageBase + 40, ExchangepageAddr);
+
+            card.startOrResumeFilePrinting();
+            print_job_timer.start();
+
+            Update_Time_Value = 0;
+            pause_action_flag = false;
+            sdcard_pause_check = true;
+            RTS_SndData(ExchangePageBase + 11, ExchangepageAddr);
+            PrintFlag = 2;
+          }
+        #endif
+      }
+      else if(recdat.data[0] == 3)
+      {
+        if(PoweroffContinue == true)
+        {
+          #if ENABLED(CHECKFILEMENT)
+            if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+            {
+              RTS_SndData(0, CHANGE_FILAMENT_ICON_VP);
+            }
+            else if((0 == save_dual_x_carriage_mode) && (1 == READ(CHECKFILEMENT0_PIN)))
+            {
+              RTS_SndData(1, CHANGE_FILAMENT_ICON_VP);
+            }
+            else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+            {
+              RTS_SndData(0, CHANGE_FILAMENT_ICON_VP);
+            }
+            else if((4 == save_dual_x_carriage_mode) && (1 == READ(CHECKFILEMENT1_PIN)))
+            {
+              RTS_SndData(1, CHANGE_FILAMENT_ICON_VP);
+            }
+            else if((4 != save_dual_x_carriage_mode) &&(0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+            {
+              RTS_SndData(0, CHANGE_FILAMENT_ICON_VP);
+            }
+            else if((4 != save_dual_x_carriage_mode) &&(0 != save_dual_x_carriage_mode) && (1 == READ(CHECKFILEMENT0_PIN)) && (1 == READ(CHECKFILEMENT1_PIN)))
+            {
+              RTS_SndData(1, CHANGE_FILAMENT_ICON_VP);
+            }
+          #endif
+          RTS_SndData(ExchangePageBase + 8, ExchangepageAddr);
+        }
+        else if(PoweroffContinue == false)
+        {
+          #if ENABLED(CHECKFILEMENT)
+            if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+            {
+              RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+              break;
+            }
+            else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+            {
+              RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+              break;
+            }
+            else if((4 != save_dual_x_carriage_mode) && (0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+            {
+              RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+              break;
+            }
+          #endif
+
+          char cmd[30];
+          char *c;
+          sprintf_P(cmd, PSTR("M23 %s"), CardRecbuf.Cardfilename[FilenamesCount]);
+          for (c = &cmd[4]; *c; c++)
+            *c = tolower(*c);
+
+          queue.enqueue_one_now(cmd);
+          delay(20);
+          queue.enqueue_now_P(PSTR("M24"));
+          // clean screen.
+          for (int j = 0; j < 20; j ++)
+          {
+            RTS_SndData(0, PRINT_FILE_TEXT_VP + j);
+          }
+          RTS_SndData(CardRecbuf.Cardshowfilename[CardRecbuf.recordcount], PRINT_FILE_TEXT_VP);
+          delay(2);
+          #if ENABLED(BABYSTEPPING)
+            RTS_SndData(0, AUTO_BED_LEVEL_ZOFFSET_VP);
+          #endif
+          feedrate_percentage = 100;
+          RTS_SndData(feedrate_percentage, PRINT_SPEED_RATE_VP);
+          zprobe_zoffset = last_zoffset;
+          RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+          PoweroffContinue = true;
+          RTS_SndData(ExchangePageBase + 10, ExchangepageAddr);
+          sdcard_pause_check = true;
+        }
+      }
+      else if(recdat.data[0] == 4)
+      {
+        if(!CardReader::flag.mounted)
+        {
+          CardUpdate = true;
+          RTS_SDCardUpate();
+          RTS_SndData(ExchangePageBase + 46, ExchangepageAddr);
+        }
+        else
+        {
+          #if ENABLED(CHECKFILEMENT)
+          if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+          else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+          else if((4 != save_dual_x_carriage_mode) && (0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+          {
+            rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          }
+          #endif
+          RTS_SndData(ExchangePageBase + 40, ExchangepageAddr);
+
+          card.startOrResumeFilePrinting();
+          print_job_timer.start();
+
+          Update_Time_Value = 0;
+          sdcard_pause_check = true;
+          pause_action_flag = false;
+          PrintFlag = 2;
+
+          for(uint16_t i = 0;i < CardRecbuf.Filesum;i ++) 
+          {
+            if(!strcmp(CardRecbuf.Cardfilename[i], &recovery.info.sd_filename[1]))
+            {
+              rtscheck.RTS_SndData(CardRecbuf.Cardshowfilename[i], PRINT_FILE_TEXT_VP);
+            }
+          }
+          sd_printing_autopause = true;
+          RTS_SndData(ExchangePageBase + 11, ExchangepageAddr);
+        }
+      }
+      break;
+
+    case TempScreenKey:
+      if (recdat.data[0] == 1)
+      {
+        if (thermalManager.fan_speed[0] == 0)
+        {
+          RTS_SndData(1, HEAD0_FAN_ICON_VP);
+        }
+        else
+        {
+          RTS_SndData(0, HEAD0_FAN_ICON_VP);
+        }
+        RTS_SndData(ExchangePageBase + 16, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 2)
+      {
+        if (thermalManager.fan_speed[1] == 0)
+        {
+          RTS_SndData(1, HEAD1_FAN_ICON_VP);
+        }
+        else
+        {
+          RTS_SndData(0, HEAD1_FAN_ICON_VP);
+        }
+        RTS_SndData(ExchangePageBase + 17, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 3)
+      {
+        RTS_SndData(ExchangePageBase + 18, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 4)
+      {
+        RTS_SndData(ExchangePageBase + 15, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 0xF1)
+      {
+        #if FAN_COUNT > 0
+          for (uint8_t i = 0; i < FAN_COUNT; i++)
+          {
+            thermalManager.fan_speed[i] = 255;
+          }
+        #endif
+
+        thermalManager.setTargetHotend(0, 0);
+        RTS_SndData(0, HEAD0_SET_TEMP_VP);
+        delay(1);
+        thermalManager.setTargetHotend(0, 1);
+        RTS_SndData(0, HEAD1_SET_TEMP_VP);
+        delay(1);
+        thermalManager.setTargetBed(0);
+        RTS_SndData(0, BED_SET_TEMP_VP);
+        delay(1);
+
+        RTS_SndData(ExchangePageBase + 15, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 0xF0)
+      {
+        RTS_SndData(ExchangePageBase + 15, ExchangepageAddr);
+      }
+      break;
+
+    case CoolScreenKey:
+      if (recdat.data[0] == 1)
+      {
+        thermalManager.setTargetHotend(0, 0);
+        RTS_SndData(0, HEAD0_SET_TEMP_VP);
+        thermalManager.fan_speed[0] = 255;
+        RTS_SndData(0, HEAD0_FAN_ICON_VP);
+      }
+      else if (recdat.data[0] == 2)
+      {
+        thermalManager.setTargetBed(0);
+        RTS_SndData(0, BED_SET_TEMP_VP);
+      }
+      else if (recdat.data[0] == 3)
+      {
+        thermalManager.setTargetHotend(0, 1);
+        RTS_SndData(0, HEAD1_SET_TEMP_VP);
+        thermalManager.fan_speed[1] = 255;
+        RTS_SndData(0, HEAD1_FAN_ICON_VP);
+      }
+      else if (recdat.data[0] == 4)
+      {
+        RTS_SndData(ExchangePageBase + 15, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 5)
+      {
+        thermalManager.temp_hotend[0].target = PREHEAT_1_TEMP_HOTEND;
+        thermalManager.setTargetHotend(thermalManager.temp_hotend[0].target, 0);
+        RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+        thermalManager.temp_bed.target = PREHEAT_1_TEMP_BED;
+        thermalManager.setTargetBed(thermalManager.temp_bed.target);
+        RTS_SndData(thermalManager.temp_bed.target, BED_SET_TEMP_VP);
+      }
+      else if (recdat.data[0] == 6)
+      {
+        thermalManager.temp_hotend[0].target = PREHEAT_2_TEMP_HOTEND;
+        thermalManager.setTargetHotend(thermalManager.temp_hotend[0].target, 0);
+        RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+        thermalManager.temp_bed.target = PREHEAT_2_TEMP_BED;
+        thermalManager.setTargetBed(thermalManager.temp_bed.target);
+        RTS_SndData(thermalManager.temp_bed.target, BED_SET_TEMP_VP);
+      }
+      else if (recdat.data[0] == 7)
+      {
+        thermalManager.temp_hotend[1].target = PREHEAT_1_TEMP_HOTEND;
+        thermalManager.setTargetHotend(thermalManager.temp_hotend[1].target, 1);
+        RTS_SndData(thermalManager.temp_hotend[1].target, HEAD1_SET_TEMP_VP);
+        thermalManager.temp_bed.target = PREHEAT_1_TEMP_BED;
+        thermalManager.setTargetBed(thermalManager.temp_bed.target);
+        RTS_SndData(thermalManager.temp_bed.target, BED_SET_TEMP_VP);
+      }
+      else if (recdat.data[0] == 8)
+      {
+        thermalManager.temp_hotend[1].target = PREHEAT_2_TEMP_HOTEND;
+        thermalManager.setTargetHotend(thermalManager.temp_hotend[1].target, 1);
+        RTS_SndData(thermalManager.temp_hotend[1].target, HEAD1_SET_TEMP_VP);
+        thermalManager.temp_bed.target = PREHEAT_2_TEMP_BED;
+        thermalManager.setTargetBed(thermalManager.temp_bed.target);
+        RTS_SndData(thermalManager.temp_bed.target, BED_SET_TEMP_VP);
+      }
+      break;
+
+    case Heater0TempEnterKey:
+      thermalManager.temp_hotend[0].target = recdat.data[0];
+      thermalManager.setTargetHotend(thermalManager.temp_hotend[0].target, 0);
+      RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+      break;
+
+    case Heater1TempEnterKey:
+      thermalManager.temp_hotend[1].target = recdat.data[0];
+      thermalManager.setTargetHotend(thermalManager.temp_hotend[1].target, 1);
+      RTS_SndData(thermalManager.temp_hotend[1].target, HEAD1_SET_TEMP_VP);
+      break;
+
+    case HotBedTempEnterKey:
+      thermalManager.temp_bed.target = recdat.data[0];
+      thermalManager.setTargetBed(thermalManager.temp_bed.target);
+      RTS_SndData(thermalManager.temp_bed.target, BED_SET_TEMP_VP);
+      break;
+
+    case Heater0LoadEnterKey:
+      Filament0LOAD = ((float)recdat.data[0]) / 10;
+      break;
+
+    case Heater1LoadEnterKey:
+      Filament1LOAD = ((float)recdat.data[0]) / 10;
+      break;
+
+    case AxisPageSelectKey:
+      if(recdat.data[0] == 1)
+      {
+        AxisUnitMode = 1;
+        axis_unit = 10.0;
+        RTS_SndData(ExchangePageBase + 29, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 2)
+      {
+        AxisUnitMode = 2;
+        axis_unit = 1.0;
+        RTS_SndData(ExchangePageBase + 30, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 3)
+      {
+        AxisUnitMode = 3;
+        axis_unit = 0.1;
+        RTS_SndData(ExchangePageBase + 31, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 4)
+      {
+        waitway = 4;
+        AutoHomeIconNum = 0;
+        queue.enqueue_now_P(PSTR("G28"));
+        Update_Time_Value = 0;
+        RTS_SndData(ExchangePageBase + 32, ExchangepageAddr);
+        RTS_SndData(0, MOTOR_FREE_ICON_VP);
+      }
+      break;
+
+    case SettingScreenKey:
+      if(recdat.data[0] == 1)
+      {
+        // Motor Icon
+        RTS_SndData(0, MOTOR_FREE_ICON_VP);
+        // only for prohibiting to receive massage
+        waitway = 6;
+        AutoHomeIconNum = 0;
+        active_extruder = 0;
+        active_extruder_flag = false;
+        active_extruder_font = active_extruder;
+        Update_Time_Value = 0;
+        queue.enqueue_now_P(PSTR("G28"));
+        queue.enqueue_now_P(PSTR("G1 F200 Z0.0"));
+        RTS_SndData(ExchangePageBase + 32, ExchangepageAddr);
+
+        if (active_extruder == 0)
+        {
+          RTS_SndData(0, EXCHANGE_NOZZLE_ICON_VP);
+        }
+        else
+        {
+          RTS_SndData(1, EXCHANGE_NOZZLE_ICON_VP);
+        }
+      }
+      else if(recdat.data[0] == 2)
+      {
+        Filament0LOAD = 10;
+        Filament1LOAD = 10;
+        RTS_SndData(10 * Filament0LOAD, HEAD0_FILAMENT_LOAD_DATA_VP);
+        RTS_SndData(10 * Filament1LOAD, HEAD1_FILAMENT_LOAD_DATA_VP);
+        RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+        thermalManager.setTargetHotend(thermalManager.temp_hotend[0].target, 0);
+        RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+
+        RTS_SndData(thermalManager.temp_hotend[1].celsius, HEAD1_CURRENT_TEMP_VP);
+        thermalManager.setTargetHotend(thermalManager.temp_hotend[1].target, 1);
+        RTS_SndData(thermalManager.temp_hotend[1].target, HEAD1_SET_TEMP_VP);
+
+        delay(2);
+        RTS_SndData(ExchangePageBase + 23, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 3)
+      {
+        if(active_extruder == 0)
+        {
+          RTS_SndData(0, EXCHANGE_NOZZLE_ICON_VP);
+          active_extruder_flag = false;
+        }
+        else if(active_extruder == 1)
+        {
+          RTS_SndData(1, EXCHANGE_NOZZLE_ICON_VP);
+          active_extruder_flag = true;
+        }
+
+        AxisUnitMode = 1;
+        if(active_extruder == 0)
+        {
+          if(TEST(axis_trusted, X_AXIS))
+          {
+            current_position_x0_axis = current_position[X_AXIS];
+          }
+          else
+          {
+            current_position[X_AXIS] = current_position_x0_axis;
+          }
+          RTS_SndData(10 * current_position_x0_axis, AXIS_X_COORD_VP);
+          memset(commandbuf, 0, sizeof(commandbuf));
+          sprintf_P(commandbuf, PSTR("G92.9 X%6.3f"), current_position_x0_axis);
+          queue.enqueue_one_now(commandbuf);
+        }
+        else if(active_extruder == 1)
+        {
+          if(TEST(axis_trusted, X_AXIS))
+          {
+            current_position_x1_axis = current_position[X_AXIS];
+          }
+          else
+          {
+            current_position[X_AXIS] = current_position_x1_axis;
+          }
+          RTS_SndData(10 * current_position_x1_axis, AXIS_X_COORD_VP);
+          memset(commandbuf, 0, sizeof(commandbuf));
+          sprintf_P(commandbuf, PSTR("G92.9 X%6.3f"), current_position_x1_axis);
+          queue.enqueue_one_now(commandbuf);
+        }
+        RTS_SndData(10 * current_position[Y_AXIS], AXIS_Y_COORD_VP);
+        RTS_SndData(10 * current_position[Z_AXIS], AXIS_Z_COORD_VP);
+
+        RTS_SndData(ExchangePageBase + 29, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 4)
+      {
+        RTS_SndData(ExchangePageBase + 35, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 5)
+      {
+        RTS_SndData(CORP_WEBSITE, PRINTER_WEBSITE_TEXT_VP);
+        RTS_SndData(Screen_version, Screen_Version_VP);
+        RTS_SndData(ExchangePageBase + 33, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 6)
+      {
+        queue.enqueue_now_P(PSTR("M84"));
+        RTS_SndData(1, MOTOR_FREE_ICON_VP);
+      }
+      else if (recdat.data[0] == 7)
+      {
+        RTS_SndData(ExchangePageBase + 1, ExchangepageAddr);
+      }
+      break;
+
+    case SettingBackKey:
+      if (recdat.data[0] == 1)
+      {
+        Update_Time_Value = RTS_UPDATE_VALUE;
+        //settings.save();
+        RTS_SndData(ExchangePageBase + 21, ExchangepageAddr);
+      }
+      else if (recdat.data[0] == 2)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          #if ENABLED(HAS_LEVELING)
+            RTS_SndData(ExchangePageBase + 22, ExchangepageAddr);
+          #else
+            RTS_SndData(ExchangePageBase + 21, ExchangepageAddr);
+          #endif
+          queue.enqueue_now_P(PSTR("M420 S1"));
+        }
+      }
+      else if (recdat.data[0] == 3)
+      {
+        memset(commandbuf, 0, sizeof(commandbuf));
+        sprintf_P(commandbuf, PSTR("M218 T1 X%4.1f"), hotend_offset[1].x);
+        queue.enqueue_now_P(commandbuf);
+        delay(5);
+        memset(commandbuf, 0, sizeof(commandbuf));
+        sprintf_P(commandbuf, PSTR("M218 T1 Y%4.1f"), hotend_offset[1].y);
+        queue.enqueue_now_P(commandbuf);
+        delay(5);
+        memset(commandbuf, 0, sizeof(commandbuf));
+        sprintf_P(commandbuf, PSTR("M218 T1 Z%4.1f"), hotend_offset[1].z);
+        queue.enqueue_now_P(commandbuf);
+        //settings.save();
+      }
+      break;
+
+    case BedLevelFunKey:
+      if (recdat.data[0] == 1)
+      {
+         
+        waitway = 6;
+        if((active_extruder == 1) || (!TEST(axis_trusted, X_AXIS)) || (!TEST(axis_trusted, Y_AXIS)))
+        {
+          AutoHomeIconNum = 0;
+          active_extruder = 0;
+          active_extruder_flag = false;
+          active_extruder_font = active_extruder;
+          queue.enqueue_now_P(PSTR("G28"));
+          RTS_SndData(ExchangePageBase + 32, ExchangepageAddr);
+        }
+        else
+        {
+          queue.enqueue_now_P(PSTR("G28 Z0"));
+        }
+        queue.enqueue_now_P(PSTR("G1 F200 Z0.0"));
+      }
+      else if (recdat.data[0] == 2)
+      {
+        last_zoffset = zprobe_zoffset;
+        if (WITHIN((zprobe_zoffset + 0.05), Z_PROBE_OFFSET_RANGE_MIN, Z_PROBE_OFFSET_RANGE_MAX))
+        {
+          #if ENABLED(HAS_LEVELING)
+            zprobe_zoffset = (zprobe_zoffset + 0.05);
+            zprobe_zoffset = zprobe_zoffset + 0.00001;
+          #endif
+          babystep.add_mm(Z_AXIS, zprobe_zoffset - last_zoffset);
+          probe.offset.z = zprobe_zoffset;
+        }
+        RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+      }
+      else if (recdat.data[0] == 3)
+      {
+        last_zoffset = zprobe_zoffset;
+        if (WITHIN((zprobe_zoffset - 0.05), Z_PROBE_OFFSET_RANGE_MIN, Z_PROBE_OFFSET_RANGE_MAX))
+        {
+          #if ENABLED(HAS_LEVELING)
+            zprobe_zoffset = (zprobe_zoffset - 0.05);
+            zprobe_zoffset = zprobe_zoffset - 0.00001;
+          #endif
+          babystep.add_mm(Z_AXIS, zprobe_zoffset - last_zoffset);
+          probe.offset.z = zprobe_zoffset;
+        }
+        RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+      }
+      else if (recdat.data[0] == 4)
+      {
+         
+        RTS_SndData(ExchangePageBase + 28, ExchangepageAddr);
+        if(active_extruder == 0)
+        {
+          RTS_SndData(0, EXCHANGE_NOZZLE_ICON_VP);
+        }
+        else if(active_extruder == 1)
+        {
+          RTS_SndData(1, EXCHANGE_NOZZLE_ICON_VP);
+        }
+        queue.enqueue_now_P(PSTR("M420 S0"));
+      }
+      else if (recdat.data[0] == 5)
+      {
+        #if ENABLED(BLTOUCH)
+          waitway = 3;
+          RTS_SndData(1, AUTO_BED_LEVEL_ICON_VP);
+          RTS_SndData(ExchangePageBase + 38, ExchangepageAddr);
+          queue.enqueue_now_P(PSTR("G29"));
+           
+        #endif
+      }
+      if (recdat.data[0] == 6)
+      {
+        // Assitant Level ,  Centre 1
+        if(!planner.has_blocks_queued())
+        {
+          waitway = 4;
+          queue.enqueue_now_P(PSTR("G1 F600 Z3"));
+          queue.enqueue_now_P(PSTR("G1 X150 Y150 F3000"));
+          queue.enqueue_now_P(PSTR("G1 F600 Z0"));
+          waitway = 0;
+        }
+      }
+      else if (recdat.data[0] == 7)
+      {
+        // Assitant Level , Front Left 2
+        if(!planner.has_blocks_queued())
+        {
+          waitway = 4;
+          queue.enqueue_now_P(PSTR("G1 F600 Z3"));
+          queue.enqueue_now_P(PSTR("G1 X30 Y30 F3000"));
+          queue.enqueue_now_P(PSTR("G1 F600 Z0"));
+          waitway = 0;
+        }
+      }
+      else if (recdat.data[0] == 8)
+      {
+        // Assitant Level , Front Right 3
+        if(!planner.has_blocks_queued())
+        {
+          waitway = 4;
+          queue.enqueue_now_P(PSTR("G1 F600 Z3"));
+          queue.enqueue_now_P(PSTR("G1 X275 Y30 F3000"));
+          queue.enqueue_now_P(PSTR("G1 F600 Z0"));
+          waitway = 0;
+        }
+      }
+      else if (recdat.data[0] == 9)
+      {
+        // Assitant Level , Back Right 4
+        if(!planner.has_blocks_queued())
+        {
+          waitway = 4;
+          queue.enqueue_now_P(PSTR("G1 F600 Z3"));
+          queue.enqueue_now_P(PSTR("G1 X275 Y275 F3000"));
+          queue.enqueue_now_P(PSTR("G1 F600 Z0"));
+          waitway = 0;
+        }
+      }
+      else if (recdat.data[0] == 10)
+      {
+        // Assitant Level , Back Left 5
+        if(!planner.has_blocks_queued())
+        {   
+          waitway = 4;
+          queue.enqueue_now_P(PSTR("G1 F600 Z3"));
+          queue.enqueue_now_P(PSTR("G1 X30 Y275 F3000"));
+          queue.enqueue_now_P(PSTR("G1 F600 Z0"));
+          waitway = 0;
+        }
+      }
+       else if (recdat.data[0] == 11)
+      { 
+        waitway = 3;
+        RTS_SndData(ExchangePageBase + 40, ExchangepageAddr);
+        queue.enqueue_now_P(PSTR("G28 X"));
+        queue.enqueue_now_P(PSTR("G34"));
+        Update_Time_Value = 0;
+        waitway = 0;
+      }
+      RTS_SndData(0, MOTOR_FREE_ICON_VP);
+      break;
+
+    case XaxismoveKey:
+    if(!planner.has_blocks_queued())
+      {
+        waitway = 4;
+        if(active_extruder == 0)
+        {
+          active_extruder_flag = false;
+        }
+        else if(active_extruder == 1)
+        {
+          active_extruder_flag = true;
+        }
+
+        if(active_extruder == 1)
+        {
+          if(recdat.data[0] >= 32768)
+          {
+            current_position_x1_axis = ((float)recdat.data[0] - 65536) / 10;
+          }
+          else
+          {
+            current_position_x1_axis = ((float)recdat.data[0]) / 10;
+          }
+
+          if(current_position_x1_axis > X2_MAX_POS)
+          {
+            current_position_x1_axis = X2_MAX_POS;
+          }
+          else if((TEST(axis_trusted, X_AXIS)) && (current_position_x1_axis < (current_position_x0_axis - X_MIN_POS)))
+          {
+            current_position_x1_axis = current_position_x0_axis - X_MIN_POS;
+          }
+          else if(current_position_x1_axis < (X_MIN_POS - X_MIN_POS))
+          {
+            current_position_x1_axis = X_MIN_POS - X_MIN_POS;
+          }
+          current_position[X_AXIS] = current_position_x1_axis;
+        }
+        else if(active_extruder == 0)
+        {
+          if(recdat.data[0] >= 32768)
+          {
+            current_position_x0_axis = ((float)recdat.data[0] - 65536) / 10;
+          }
+          else
+          {
+            current_position_x0_axis = ((float)recdat.data[0]) / 10;
+          }
+
+          if(current_position_x0_axis < X_MIN_POS)
+          {
+            current_position_x0_axis = X_MIN_POS;
+          }
+          else if((TEST(axis_trusted, X_AXIS)) && (current_position_x0_axis > (current_position_x1_axis + X_MIN_POS)))
+          {
+            current_position_x0_axis = current_position_x1_axis + X_MIN_POS;
+          }
+          else if(current_position_x0_axis > (X2_MAX_POS + X_MIN_POS))
+          {
+            current_position_x0_axis = X2_MAX_POS + X_MIN_POS;
+          }
+          current_position[X_AXIS] = current_position_x0_axis;
+        }
+        RTS_line_to_current(X_AXIS);
+        RTS_SndData(10 * current_position[X_AXIS], AXIS_X_COORD_VP);
+        RTS_SndData(0, MOTOR_FREE_ICON_VP);
+        waitway = 0;
+      }
+      break;
+
+    case YaxismoveKey:
+    if(!planner.has_blocks_queued())
+      {
+        float y_min, y_max;
+        waitway = 4;
+        y_min = Y_MIN_POS;
+        y_max = Y_MAX_POS;
+        current_position[Y_AXIS] = ((float)recdat.data[0]) / 10;
+        if (current_position[Y_AXIS] < y_min)
+        {
+          current_position[Y_AXIS] = y_min;
+        }
+        else if (current_position[Y_AXIS] > y_max)
+        {
+          current_position[Y_AXIS] = y_max;
+        }
+        RTS_line_to_current(Y_AXIS);
+        RTS_SndData(10 * current_position[Y_AXIS], AXIS_Y_COORD_VP);
+        RTS_SndData(0, MOTOR_FREE_ICON_VP);
+        waitway = 0;
+      }
+      break;
+    
+    case ZaxismoveKey:
+      if(!planner.has_blocks_queued())
+      {
+        float z_min, z_max;
+        waitway = 4;
+        z_min = Z_MIN_POS;
+        z_max = Z_MAX_POS;
+        current_position[Z_AXIS] = ((float)recdat.data[0]) / 10;
+        if (current_position[Z_AXIS] < z_min)
+        {
+          current_position[Z_AXIS] = z_min;
+        }
+        else if (current_position[Z_AXIS] > z_max)
+        {
+          current_position[Z_AXIS] = z_max;
+        }
+        RTS_line_to_current(Z_AXIS);
+        RTS_SndData(10 * current_position[Z_AXIS], AXIS_Z_COORD_VP);
+        RTS_SndData(0, MOTOR_FREE_ICON_VP);
+        waitway = 0;
+      }
+      break;
+    case SelectExtruderKey:
+      if(recdat.data[0] == 1)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          if(active_extruder == 0)
+          {
+            queue.enqueue_now_P(PSTR("G28 X"));
+            queue.enqueue_now_P(PSTR("T1"));
+            active_extruder = 1;
+            active_extruder_flag = true;
+            active_extruder_font = active_extruder;
+
+            RTS_SndData(10 * X2_MAX_POS, AXIS_X_COORD_VP);
+            RTS_SndData(1, EXCHANGE_NOZZLE_ICON_VP);
+          }
+          else if(active_extruder == 1)
+          {
+            queue.enqueue_now_P(PSTR("G28 X"));
+            queue.enqueue_now_P(PSTR("T0"));
+            active_extruder = 0;
+            active_extruder_flag = false;
+            active_extruder_font = active_extruder;
+
+            RTS_SndData(10 * X_MIN_POS, AXIS_X_COORD_VP);
+            RTS_SndData(0, EXCHANGE_NOZZLE_ICON_VP);
+          }
+        }
+      }
+      else if (recdat.data[0] == 2)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          waitway = 4;
+          if(active_extruder == 0)
+          {
+            queue.enqueue_now_P(PSTR("G28 X"));
+            queue.enqueue_now_P(PSTR("T1"));
+            active_extruder_flag = true;
+            active_extruder = 1;
+            active_extruder_font = active_extruder;
+            RTS_SndData(1, EXCHANGE_NOZZLE_ICON_VP);
+            RTS_SndData(10 * current_position[X_AXIS], AXIS_X_COORD_VP);
+          }
+          else if(active_extruder == 1)
+          {
+            queue.enqueue_now_P(PSTR("G28 X"));
+            queue.enqueue_now_P(PSTR("T0"));
+            active_extruder_flag = false;
+            active_extruder = 0;
+            active_extruder_font = active_extruder;
+            RTS_SndData(0, EXCHANGE_NOZZLE_ICON_VP);
+            RTS_SndData(10 * current_position[X_AXIS], AXIS_X_COORD_VP);
+          }
+          RTS_SndData(0, MOTOR_FREE_ICON_VP);
+          waitway = 0;
+        }
+      }
+      break;
+
+    case FilamentLoadKey:
+      if(recdat.data[0] == 1)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          #if ENABLED(CHECKFILEMENT)
+            if(0 == READ(CHECKFILEMENT0_PIN))
+            {
+              RTS_SndData(ExchangePageBase + 20, ExchangepageAddr);
+            }
+          #endif
+          current_position[E_AXIS] -= Filament0LOAD;
+          active_extruder = 0;
+          queue.enqueue_now_P(PSTR("T0"));
+
+          if(thermalManager.temp_hotend[0].celsius < (ChangeFilament0Temp - 5))
+          {
+            RTS_SndData((int)ChangeFilament0Temp, CHANGE_FILAMENT0_TEMP_VP);
+            RTS_SndData(ExchangePageBase + 24, ExchangepageAddr);
+          }
+          else
+          {
+            RTS_line_to_current(E_AXIS);
+            RTS_SndData(10 * Filament0LOAD, HEAD0_FILAMENT_LOAD_DATA_VP);
+          }
+        }
+      }
+      else if(recdat.data[0] == 2)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          #if ENABLED(CHECKFILEMENT)
+            if(0 == READ(CHECKFILEMENT0_PIN))
+            {
+              RTS_SndData(ExchangePageBase + 20, ExchangepageAddr);
+            }
+          #endif
+          current_position[E_AXIS] += Filament0LOAD;
+          active_extruder = 0;
+          queue.enqueue_now_P(PSTR("T0"));
+
+          if(thermalManager.temp_hotend[0].celsius < (ChangeFilament0Temp - 5))
+          {
+            RTS_SndData((int)ChangeFilament0Temp, CHANGE_FILAMENT0_TEMP_VP);
+            RTS_SndData(ExchangePageBase + 24, ExchangepageAddr);
+          }
+          else
+          {
+            RTS_line_to_current(E_AXIS);
+            RTS_SndData(10 * Filament0LOAD, HEAD0_FILAMENT_LOAD_DATA_VP);
+          }
+        }
+      }
+      else if(recdat.data[0] == 3)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          #if ENABLED(CHECKFILEMENT)
+            if(0 == READ(CHECKFILEMENT1_PIN))
+            {
+              RTS_SndData(ExchangePageBase + 20, ExchangepageAddr);
+            }
+          #endif
+          current_position[E_AXIS] -= Filament1LOAD;
+          active_extruder = 1;
+          queue.enqueue_now_P(PSTR("T1"));
+
+          if (thermalManager.temp_hotend[1].celsius < (ChangeFilament1Temp - 5))
+          {
+            RTS_SndData((int)ChangeFilament1Temp, CHANGE_FILAMENT1_TEMP_VP);
+
+            RTS_SndData(ExchangePageBase + 25, ExchangepageAddr);
+          }
+          else
+          {
+            RTS_line_to_current(E_AXIS);
+            RTS_SndData(10 * Filament1LOAD, HEAD1_FILAMENT_LOAD_DATA_VP);
+          }
+        }
+      }
+      else if(recdat.data[0] == 4)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          #if ENABLED(CHECKFILEMENT)
+            if(0 == READ(CHECKFILEMENT1_PIN))
+            {
+              RTS_SndData(ExchangePageBase + 20, ExchangepageAddr);
+            }
+          #endif
+          current_position[E_AXIS] += Filament1LOAD;
+          active_extruder = 1;
+          queue.enqueue_now_P(PSTR("T1"));
+
+          if(thermalManager.temp_hotend[1].celsius < (ChangeFilament1Temp - 5))
+          {
+            RTS_SndData((int)ChangeFilament1Temp, CHANGE_FILAMENT1_TEMP_VP);
+            RTS_SndData(ExchangePageBase + 25, ExchangepageAddr);
+          }
+          else
+          {
+            RTS_line_to_current(E_AXIS);
+            RTS_SndData(10 * Filament1LOAD, HEAD1_FILAMENT_LOAD_DATA_VP);
+          }
+        }
+      }
+      else if(recdat.data[0] == 5)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+          thermalManager.setTargetHotend(ChangeFilament0Temp, 0);
+          RTS_SndData(ChangeFilament0Temp, HEAD0_SET_TEMP_VP);
+          RTS_SndData(ExchangePageBase + 26, ExchangepageAddr);
+          heatway = 1;
+        }
+      }
+      else if(recdat.data[0] == 6)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          Filament0LOAD = 10;
+          Filament1LOAD = 10;
+          RTS_SndData(10 * Filament0LOAD, HEAD0_FILAMENT_LOAD_DATA_VP);
+          RTS_SndData(10 * Filament1LOAD, HEAD1_FILAMENT_LOAD_DATA_VP);
+          RTS_SndData(ExchangePageBase + 23, ExchangepageAddr);
+        }
+      }
+      else if(recdat.data[0] == 7)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          RTS_SndData(thermalManager.temp_hotend[1].celsius, HEAD1_CURRENT_TEMP_VP);
+          thermalManager.setTargetHotend(ChangeFilament1Temp, 1);
+          RTS_SndData(ChangeFilament1Temp, HEAD1_SET_TEMP_VP);
+          RTS_SndData(ExchangePageBase + 26, ExchangepageAddr);
+          heatway = 2;
+        }
+      }
+      else if (recdat.data[0] == 8)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          Filament0LOAD = 10;
+          Filament1LOAD = 10;
+          RTS_SndData(10 * Filament0LOAD, HEAD0_FILAMENT_LOAD_DATA_VP);
+          RTS_SndData(10 * Filament1LOAD, HEAD1_FILAMENT_LOAD_DATA_VP);
+          RTS_SndData(ExchangePageBase + 23, ExchangepageAddr);
+        }
+      }
+      else if(recdat.data[0] == 0xF1)
+      {
+        if(!planner.has_blocks_queued())
+        {
+          thermalManager.temp_hotend[0].target = 0;
+          thermalManager.temp_hotend[1].target = 0;
+          RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+          RTS_SndData(thermalManager.temp_hotend[1].target, HEAD1_SET_TEMP_VP);
+          RTS_SndData(ExchangePageBase + 23, ExchangepageAddr);
+          Filament0LOAD = 10;
+          Filament1LOAD = 10;
+          RTS_SndData(10 * Filament0LOAD, HEAD0_FILAMENT_LOAD_DATA_VP);
+          RTS_SndData(10 * Filament1LOAD, HEAD1_FILAMENT_LOAD_DATA_VP);
+          break;
+        }
+      }
+      else if(recdat.data[0] == 0xF0)
+      {
+        break;
+      }
+      break;
+
+    case FilamentCheckKey:
+      if (recdat.data[0] == 1)
+      {
+        #if ENABLED(CHECKFILEMENT)
+          if((0 == READ(CHECKFILEMENT0_PIN)) && (active_extruder == 0))
+          {
+            RTS_SndData(ExchangePageBase + 20, ExchangepageAddr);
+          }
+          else if((0 == READ(CHECKFILEMENT1_PIN)) && (active_extruder == 1))
+          {
+            RTS_SndData(ExchangePageBase + 20, ExchangepageAddr);
+          }
+          else
+          {
+            RTS_SndData(ExchangePageBase + 23, ExchangepageAddr);
+          }
+        #endif
+      }
+      else if (recdat.data[0] == 2)
+      {
+        RTS_SndData(ExchangePageBase + 21, ExchangepageAddr);
+        Filament0LOAD = 10;
+        Filament1LOAD = 10;
+      }
+      break;
+
+    case PowerContinuePrintKey:
+      if (recdat.data[0] == 1)
+      {
+        #if ENABLED(DUAL_X_CARRIAGE)
+          save_dual_x_carriage_mode = dualXPrintingModeStatus;
+          switch(save_dual_x_carriage_mode)
+          {
+            case 1:
+              queue.enqueue_now_P(PSTR("M605 S1"));
+              break;
+            case 2:
+              queue.enqueue_now_P(PSTR("M605 S2"));
+              break;
+            case 3:
+              queue.enqueue_now_P(PSTR("M605 S2 X68 R0"));
+              queue.enqueue_now_P(PSTR("M605 S3"));
+              break;
+            default:
+              queue.enqueue_now_P(PSTR("M605 S0"));
+              break;
+          }
+        #endif
+        if (recovery.info.recovery_flag)
+        {
+          power_off_type_yes = 1;
+          Update_Time_Value = 0;
+          RTS_SndData(ExchangePageBase + 10, ExchangepageAddr);
+          // recovery.resume();
+          queue.enqueue_now_P(PSTR("M1000"));
+
+          PoweroffContinue = true;
+          sdcard_pause_check = true;
+          zprobe_zoffset = probe.offset.z;
+          RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+          RTS_SndData(feedrate_percentage, PRINT_SPEED_RATE_VP);
+          PrintFlag = 2;
+        }
+      }
+      else if (recdat.data[0] == 2)
+      {
+        Update_Time_Value = RTS_UPDATE_VALUE;
+        #if ENABLED(DUAL_X_CARRIAGE)
+          extruder_duplication_enabled = false;
+          dual_x_carriage_mode = DEFAULT_DUAL_X_CARRIAGE_MODE;
+          active_extruder = 0;
+        #endif
+        RTS_SndData(ExchangePageBase + 1, ExchangepageAddr);
+
+        PoweroffContinue = false;
+        sdcard_pause_check = false;
+        queue.clear();
+        quickstop_stepper();
+        print_job_timer.stop();
+        thermalManager.disable_all_heaters();
+        print_job_timer.reset();
+
+        if(CardReader::flag.mounted)
+        {
+          #if ENABLED(SDSUPPORT) && ENABLED(POWER_LOSS_RECOVERY)
+            card.removeJobRecoveryFile();
+            recovery.info.valid_head = 0;
+            recovery.info.valid_foot = 0;
+            recovery.close();
+          #endif
+        }
+
+        wait_for_heatup = wait_for_user = false;
+        sd_printing_autopause = false;
+        PrintFlag = 0;
+      }
+      break;
+
+    case SelectFileKey:
+      if (RTS_SD_Detected())
+      {
+        if (recdat.data[0] > CardRecbuf.Filesum)
+        {
+          break;
+        }
+
+        CardRecbuf.recordcount = recdat.data[0] - 1;
+        for (int j = 0; j < 10; j ++)
+        {
+          RTS_SndData(0, SELECT_FILE_TEXT_VP + j);
+          RTS_SndData(0, PRINT_FILE_TEXT_VP + j);
+        }
+        RTS_SndData(CardRecbuf.Cardshowfilename[CardRecbuf.recordcount], SELECT_FILE_TEXT_VP);
+        RTS_SndData(CardRecbuf.Cardshowfilename[CardRecbuf.recordcount], PRINT_FILE_TEXT_VP);
+        delay(2);
+        for(int j = 1;j <= CardRecbuf.Filesum;j ++)
+        {
+          RTS_SndData((unsigned long)0xA514, FilenameNature + j * 16);
+        }
+        RTS_SndData((unsigned long)0x073F, FilenameNature + recdat.data[0] * 16);
+        RTS_SndData(1, FILE1_SELECT_ICON_VP + (recdat.data[0] - 1));
+      }
+      break;
+
+    case PrintFileKey:
+    if (recdat.data[0] == 1)
+      {
+        if((0 != dualXPrintingModeStatus) && (4 != dualXPrintingModeStatus))
+        {
+          RTS_SndData(dualXPrintingModeStatus, SELECT_MODE_ICON_VP);        
+        }
+        else if(4 == dualXPrintingModeStatus)
+        {
+          RTS_SndData(5, SELECT_MODE_ICON_VP);  
+        }
+        else
+        {
+          RTS_SndData(4, SELECT_MODE_ICON_VP);
+        }
+        RTS_SndData(CardRecbuf.Cardshowfilename[CardRecbuf.recordcount], PRINT_FILE_TEXT_VP);
+        RTS_SndData(ExchangePageBase + 56, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 2)
+      {
+        RTS_SndData(ExchangePageBase + 3, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 3)
+      {
+        RTS_SndData(ExchangePageBase + 2, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 4)
+      {
+        RTS_SndData(ExchangePageBase + 4, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 5)
+      {
+        RTS_SndData(ExchangePageBase + 3, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 6)
+      {
+        RTS_SndData(ExchangePageBase + 5, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 7)
+      {
+        RTS_SndData(ExchangePageBase + 4, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 8)
+      {
+        RTS_SndData(ExchangePageBase + 6, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 9)
+      {
+        RTS_SndData(ExchangePageBase + 5, ExchangepageAddr);
+      }
+      else if(recdat.data[0] == 10)
+      {
+        RTS_SndData(ExchangePageBase + 1, ExchangepageAddr);
+      }
+      else if ((recdat.data[0] == 11) && RTS_SD_Detected())
+      {
+        if (CardRecbuf.recordcount < 0)
+        {
+          break;
+        }
+
+        char cmd[30];
+        char *c;
+        sprintf_P(cmd, PSTR("M23 %s"), CardRecbuf.Cardfilename[CardRecbuf.recordcount]);
+        for (c = &cmd[4]; *c; c++)
+          *c = tolower(*c);
+
+        memset(cmdbuf, 0, sizeof(cmdbuf));
+        strcpy(cmdbuf, cmd);
+        FilenamesCount = CardRecbuf.recordcount;
+
+        save_dual_x_carriage_mode = dualXPrintingModeStatus;
+        switch(save_dual_x_carriage_mode)
+        {
+          case 1:
+            queue.enqueue_now_P(PSTR("M605 S1"));
+            break;
+          case 2:
+            queue.enqueue_now_P(PSTR("M605 S2"));
+            break;
+          case 3:
+            queue.enqueue_now_P(PSTR("M605 S2 X68 R0"));
+            queue.enqueue_now_P(PSTR("M605 S3"));
+            break;
+          default:
+            queue.enqueue_now_P(PSTR("M605 S0"));
+            break;
+        }
+        #if ENABLED(CHECKFILEMENT)
+          if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+            sdcard_pause_check = false;
+            break;
+          }
+          else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+            sdcard_pause_check = false;
+            break;
+          }
+          else if((4 != save_dual_x_carriage_mode) && (0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+          {
+            RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+            sdcard_pause_check = false;
+            break;
+          }
+        #endif
+        queue.enqueue_one_now(cmd);
+        delay(20);
+        queue.enqueue_now_P(PSTR("M24"));
+        // clean screen.
+        for (int j = 0; j < 20; j ++)
+        {
+          RTS_SndData(0, PRINT_FILE_TEXT_VP + j);
+        }
+
+        RTS_SndData(CardRecbuf.Cardshowfilename[CardRecbuf.recordcount], PRINT_FILE_TEXT_VP);
+
+        delay(2);
+
+        #if ENABLED(BABYSTEPPING)
+          RTS_SndData(0, AUTO_BED_LEVEL_ZOFFSET_VP);
+        #endif
+        feedrate_percentage = 100;
+        RTS_SndData(feedrate_percentage, PRINT_SPEED_RATE_VP);
+        zprobe_zoffset = last_zoffset;
+        RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+        PoweroffContinue = true;
+        RTS_SndData(ExchangePageBase + 10, ExchangepageAddr);
+        Update_Time_Value = 0;
+        PrintFlag = 2;
+        change_page_number = 11;
+      }
+      else if(recdat.data[0] == 12)
+      {
+        RTS_SndData(ExchangePageBase + 1, ExchangepageAddr);
+      }
+      break;
+
+    case PrintSelectModeKey:
+      if (recdat.data[0] == 1)
+      {
+        RTS_SndData(1, TWO_COLOR_MODE_ICON_VP);
+        RTS_SndData(0, COPY_MODE_ICON_VP);
+        RTS_SndData(0, MIRROR_MODE_ICON_VP);
+        RTS_SndData(0, SINGLE_MODE_ICON_VP);
+        dualXPrintingModeStatus = 1;
+        RTS_SndData(1, PRINT_MODE_ICON_VP);
+        RTS_SndData(1, SELECT_MODE_ICON_VP);
+      }
+      else if (recdat.data[0] == 2)
+      {
+        RTS_SndData(1, COPY_MODE_ICON_VP);
+        RTS_SndData(0, TWO_COLOR_MODE_ICON_VP);
+        RTS_SndData(0, MIRROR_MODE_ICON_VP);
+        RTS_SndData(0, SINGLE_MODE_ICON_VP);
+        dualXPrintingModeStatus = 2;
+        RTS_SndData(2, PRINT_MODE_ICON_VP);
+        RTS_SndData(2, SELECT_MODE_ICON_VP);
+      }
+      else if (recdat.data[0] == 3)
+      {
+        RTS_SndData(1, MIRROR_MODE_ICON_VP);
+        RTS_SndData(0, TWO_COLOR_MODE_ICON_VP);
+        RTS_SndData(0, COPY_MODE_ICON_VP);
+        RTS_SndData(0, SINGLE_MODE_ICON_VP);
+        dualXPrintingModeStatus = 3;
+        RTS_SndData(3, PRINT_MODE_ICON_VP);
+        RTS_SndData(3, SELECT_MODE_ICON_VP);
+      }
+      else if (recdat.data[0] == 4)
+      {
+        RTS_SndData(0, MIRROR_MODE_ICON_VP);
+        RTS_SndData(0, TWO_COLOR_MODE_ICON_VP);
+        RTS_SndData(0, COPY_MODE_ICON_VP);
+        RTS_SndData(1, SINGLE_MODE_ICON_VP);
+        dualXPrintingModeStatus = 0;
+        RTS_SndData(4, PRINT_MODE_ICON_VP);
+        RTS_SndData(4, SELECT_MODE_ICON_VP);
+      }
+      else if (recdat.data[0] == 5)
+      {
+        RTS_SndData(0, MIRROR_MODE_ICON_VP);
+        RTS_SndData(0, TWO_COLOR_MODE_ICON_VP);
+        RTS_SndData(0, COPY_MODE_ICON_VP);
+        RTS_SndData(2, SINGLE_MODE_ICON_VP);
+        dualXPrintingModeStatus = 4;
+        RTS_SndData(5, PRINT_MODE_ICON_VP);
+        RTS_SndData(5, SELECT_MODE_ICON_VP);
+      }
+      else if (recdat.data[0] == 6)
+      {
+        settings.save();
+        rtscheck.RTS_SndData(ExchangePageBase + 1, ExchangepageAddr);
+      }
+      break;
+
+    case StoreMemoryKey:
+      if(recdat.data[0] == 0xF1)
+      {
+        settings.init_eeprom();
+        #if ENABLED(AUTO_BED_LEVELING_BILINEAR)
+          bool zig = false;
+          int8_t inStart, inStop, inInc, showcount;
+          showcount = 0;
+          for (int y = 0; y < GRID_MAX_POINTS_Y; y++)
+          {
+            // away from origin
+            if (zig)
+            {
+              inStart = 0;
+              inStop = GRID_MAX_POINTS_X;
+              inInc = 1;
+            }
+            else
+            {
+              // towards origin
+              inStart = GRID_MAX_POINTS_X - 1;
+              inStop = -1;
+              inInc = -1;
+            }
+            zig ^= true;
+            for (int x = inStart; x != inStop; x += inInc)
+            {
+              RTS_SndData(z_values[x][y] * 1000, AUTO_BED_LEVEL_1POINT_VP + showcount * 2);
+              showcount++;
+            }
+          }
+          queue.enqueue_now_P(PSTR("M420 S1"));
+        #endif
+        zprobe_zoffset = 0;
+        last_zoffset = 0;
+        RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+        rtscheck.RTS_SndData(ExchangePageBase + 21, ExchangepageAddr);
+        RTS_SndData((hotend_offset[1].x - X2_MAX_POS) * 100, TWO_EXTRUDER_HOTEND_XOFFSET_VP);
+        RTS_SndData(hotend_offset[1].y * 100, TWO_EXTRUDER_HOTEND_YOFFSET_VP);
+        RTS_SndData(hotend_offset[1].z * 100, TWO_EXTRUDER_HOTEND_ZOFFSET_VP);
+      }
+      else if (recdat.data[0] == 0xF0)
+      {
+        memset(commandbuf, 0, sizeof(commandbuf));
+        sprintf_P(commandbuf, PSTR("M218 T1 X%4.1f"), hotend_offset[1].x);
+        queue.enqueue_now_P(commandbuf);
+        sprintf_P(commandbuf, PSTR("M218 T1 Y%4.1f"), hotend_offset[1].y);
+        queue.enqueue_now_P(commandbuf);
+        sprintf_P(commandbuf, PSTR("M218 T1 Z%4.1f"), hotend_offset[1].z);
+        queue.enqueue_now_P(commandbuf);
+        //settings.save();
+        rtscheck.RTS_SndData(ExchangePageBase + 35, ExchangepageAddr);
+      }
+      break;
+
+    case XhotendOffsetKey:
+      if (recdat.data[0] >= 32768)
+      {
+        hotend_offset[1].x = (recdat.data[0] - 65536) / 100.0;
+        hotend_offset[1].x = hotend_offset[1].x - 0.00001 + X2_MAX_POS;
+      }
+      else
+      {
+        hotend_offset[1].x = (recdat.data[0]) / 100.0;
+        hotend_offset[1].x = hotend_offset[1].x + 0.00001 + X2_MAX_POS;
+      }
+
+      RTS_SndData((hotend_offset[1].x - X2_MAX_POS)* 100, TWO_EXTRUDER_HOTEND_XOFFSET_VP);
+      break;
+
+    case YhotendOffsetKey:
+      if (recdat.data[0] >= 32768)
+      {
+        hotend_offset[1].y = (recdat.data[0] - 65536) / 100.0;
+        hotend_offset[1].y = hotend_offset[1].y - 0.00001;
+      }
+      else
+      {
+        hotend_offset[1].y = (recdat.data[0]) / 100.0;
+        hotend_offset[1].y = hotend_offset[1].y + 0.00001;
+      }
+
+      RTS_SndData(hotend_offset[1].y * 100, TWO_EXTRUDER_HOTEND_YOFFSET_VP);
+      break;
+
+    case SaveEEPROM:
+      if (recdat.data[0] == 1)
+      {
+        settings.save();
+      }
+    break;
+    case ChangePageKey:
+      if ((change_page_number == 36) || (change_page_number == 76))
+      {
+        break;
+      }
+      else if(change_page_number == 11)
+      {
+        RTS_SndData(change_page_number + ExchangePageBase, ExchangepageAddr);
+        if((0 != dualXPrintingModeStatus) && (4 != dualXPrintingModeStatus))
+        {
+          RTS_SndData(dualXPrintingModeStatus, PRINT_MODE_ICON_VP);        
+        }
+        else if(4 == dualXPrintingModeStatus)
+        {
+          RTS_SndData(5, PRINT_MODE_ICON_VP);
+        }
+        else
+        {
+          RTS_SndData(4, PRINT_MODE_ICON_VP);
+        }
+      }
+      else if(change_page_number == 12)
+      {
+        RTS_SndData(change_page_number + ExchangePageBase, ExchangepageAddr);
+        if((0 != dualXPrintingModeStatus) && (4 != dualXPrintingModeStatus))
+        {
+          RTS_SndData(dualXPrintingModeStatus, PRINT_MODE_ICON_VP);        
+        }
+        else if(4 == dualXPrintingModeStatus)
+        {
+          RTS_SndData(5, PRINT_MODE_ICON_VP);
+        }
+        else
+        {
+          RTS_SndData(4, PRINT_MODE_ICON_VP);
+        }
+      }
+      else
+      {
+        RTS_SndData(change_page_number + ExchangePageBase, ExchangepageAddr);
+        change_page_number = 1;
+      }
+      if((0 != dualXPrintingModeStatus) && (4 != dualXPrintingModeStatus))
+      {
+        RTS_SndData(dualXPrintingModeStatus, SELECT_MODE_ICON_VP);        
+      }
+      else if(4 == dualXPrintingModeStatus)
+      {
+        RTS_SndData(5, SELECT_MODE_ICON_VP);
+      }
+      else
+      {
+        RTS_SndData(4, SELECT_MODE_ICON_VP);
+      }
+
+      for (int i = 0; i < MaxFileNumber; i ++)
+      {
+        for (int j = 0; j < 20; j ++)
+        {
+          RTS_SndData(0, FILE1_TEXT_VP + i * 20 + j);
+        }
+      }
+
+      for (int i = 0; i < CardRecbuf.Filesum; i++)
+      {
+        for (int j = 0; j < 20; j++)
+        {
+          RTS_SndData(0, CardRecbuf.addr[i] + j);
+        }
+        RTS_SndData((unsigned long)0xA514, FilenameNature + (i + 1) * 16);
+      }
+
+      for (int j = 0; j < 20; j ++)
+      {
+        // clean screen.
+        RTS_SndData(0, PRINT_FILE_TEXT_VP + j);
+        // clean filename
+        RTS_SndData(0, SELECT_FILE_TEXT_VP + j);
+      }
+      // clean filename Icon
+      for (int j = 0; j < 20; j ++)
+      {
+        RTS_SndData(10, FILE1_SELECT_ICON_VP + j);
+      }
+
+      RTS_SndData(CardRecbuf.Cardshowfilename[CardRecbuf.recordcount], PRINT_FILE_TEXT_VP);
+
+      // represents to update file list
+      if (CardUpdate && lcd_sd_status && IS_SD_INSERTED())
+      {
+        for (uint16_t i = 0; i < CardRecbuf.Filesum; i++)
+        {
+          delay(3);
+          RTS_SndData(CardRecbuf.Cardshowfilename[i], CardRecbuf.addr[i]);
+          RTS_SndData((unsigned long)0xA514, FilenameNature + (i + 1) * 16);
+          RTS_SndData(0, FILE1_SELECT_ICON_VP + i);
+        }
+      }
+
+      char sizeBuf[20];
+      sprintf(sizeBuf, "%d X %d X %d", X_MAX_POS - 2, Y_MAX_POS - 2, Z_MAX_POS);
+      RTS_SndData(MACVERSION, PRINTER_MACHINE_TEXT_VP);
+      RTS_SndData(SOFTVERSION, PRINTER_VERSION_TEXT_VP);
+      RTS_SndData(sizeBuf, PRINTER_PRINTSIZE_TEXT_VP);
+
+      RTS_SndData(CORP_WEBSITE, PRINTER_WEBSITE_TEXT_VP);
+      RTS_SndData(Screen_version, Screen_Version_VP);
+
+      if (thermalManager.fan_speed[0] == 0)
+      {
+        RTS_SndData(1, HEAD0_FAN_ICON_VP);
+      }
+      else
+      {
+        RTS_SndData(0, HEAD0_FAN_ICON_VP);
+      }
+      if (thermalManager.fan_speed[1] == 0)
+      {
+        RTS_SndData(1, HEAD1_FAN_ICON_VP);
+      }
+      else
+      {
+        RTS_SndData(0, HEAD1_FAN_ICON_VP);
+      }
+      Percentrecord = card.percentDone() + 1;
+      if (Percentrecord <= 100)
+      {
+        rtscheck.RTS_SndData((unsigned char)Percentrecord, PRINT_PROCESS_ICON_VP);
+      }
+      rtscheck.RTS_SndData((unsigned char)card.percentDone(), PRINT_PROCESS_VP);
+
+      RTS_SndData(probe.offset.z * 100, AUTO_BED_LEVEL_ZOFFSET_VP);
+
+      RTS_SndData(feedrate_percentage, PRINT_SPEED_RATE_VP);
+      RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+      RTS_SndData(thermalManager.temp_hotend[1].target, HEAD1_SET_TEMP_VP);
+      RTS_SndData(thermalManager.temp_bed.target, BED_SET_TEMP_VP);
+
+      
+      break;
+
+    default:
+      break;
+  }
+  memset(&recdat, 0, sizeof(recdat));
+  recdat.head[0] = FHONE;
+  recdat.head[1] = FHTWO;
+}
+
+void EachMomentUpdate()
+{
+  millis_t ms = millis();
+  if(ms > next_rts_update_ms)
+  {
+    // print the file before the power is off.
+    if((power_off_type_yes == 0) && lcd_sd_status && (recovery.info.recovery_flag == true))
+    {
+      rtscheck.RTS_SndData(ExchangePageBase, ExchangepageAddr);
+      if(startprogress < 100)
+      {
+        rtscheck.RTS_SndData(startprogress, START1_PROCESS_ICON_VP);
+      }
+      delay(30);
+      if((startprogress += 1) > 100)
+      {
+        rtscheck.RTS_SndData(StartSoundSet, SoundAddr);
+        power_off_type_yes = 1;
+        for(uint16_t i = 0;i < CardRecbuf.Filesum;i ++) 
+        {
+          if(!strcmp(CardRecbuf.Cardfilename[i], &recovery.info.sd_filename[1]))
+          {
+            rtscheck.RTS_SndData(CardRecbuf.Cardshowfilename[i], PRINT_FILE_TEXT_VP);
+            rtscheck.RTS_SndData(ExchangePageBase + 36, ExchangepageAddr);
+            break;
+          }
+        }
+        StartFlag = 1;
+      }
+      return;
+    }
+    else if((power_off_type_yes == 0) && (recovery.info.recovery_flag == false))
+    {
+      rtscheck.RTS_SndData(ExchangePageBase, ExchangepageAddr);
+      if(startprogress < 100)
+      {
+        rtscheck.RTS_SndData(startprogress, START1_PROCESS_ICON_VP);
+      }
+      delay(30);
+      if((startprogress += 1) > 100)
+      {
+        rtscheck.RTS_SndData(StartSoundSet, SoundAddr);
+        power_off_type_yes = 1;
+        Update_Time_Value = RTS_UPDATE_VALUE;
+        rtscheck.RTS_SndData(ExchangePageBase + 1, ExchangepageAddr);
+        change_page_number = 1;
+      }
+      return;
+    }
+    else
+    {
+      // need to optimize
+      if(recovery.info.print_job_elapsed != 0)
+      {
+        duration_t elapsed = print_job_timer.duration();
+        static unsigned char last_cardpercentValue = 100;
+        rtscheck.RTS_SndData(elapsed.value / 3600, PRINT_TIME_HOUR_VP);
+        rtscheck.RTS_SndData((elapsed.value % 3600) / 60, PRINT_TIME_MIN_VP);
+
+        if(card.isPrinting() && (last_cardpercentValue != card.percentDone()))
+        {
+          if((unsigned char) card.percentDone() > 0)
+          {
+            Percentrecord = card.percentDone();
+            if(Percentrecord <= 100)
+            {
+              rtscheck.RTS_SndData((unsigned char)Percentrecord, PRINT_PROCESS_ICON_VP);
+            }
+            // Estimate remaining time every 20 seconds
+            static millis_t next_remain_time_update = 0;
+            if(ELAPSED(ms, next_remain_time_update))
+            {
+              if((0 == save_dual_x_carriage_mode) && (thermalManager.temp_hotend[0].celsius >= (thermalManager.temp_hotend[0].target - 5)))
+              {
+                remain_time = elapsed.value / (Percentrecord * 0.01f) - elapsed.value;
+                next_remain_time_update += 20 * 1000UL;
+                rtscheck.RTS_SndData(remain_time / 3600, PRINT_SURPLUS_TIME_HOUR_VP);
+                rtscheck.RTS_SndData((remain_time % 3600) / 60, PRINT_SURPLUS_TIME_MIN_VP);
+              }
+              else if((0 != save_dual_x_carriage_mode) && (thermalManager.temp_hotend[0].celsius >= (thermalManager.temp_hotend[0].target - 5)) && (thermalManager.temp_hotend[1].celsius >= (thermalManager.temp_hotend[1].target - 5)))
+              {
+                remain_time = elapsed.value / (Percentrecord * 0.01f) - elapsed.value;
+                next_remain_time_update += 20 * 1000UL;
+                rtscheck.RTS_SndData(remain_time / 3600, PRINT_SURPLUS_TIME_HOUR_VP);
+                rtscheck.RTS_SndData((remain_time % 3600) / 60, PRINT_SURPLUS_TIME_MIN_VP);
+              }
+            }
+          }
+          else
+          {
+            rtscheck.RTS_SndData(0, PRINT_PROCESS_ICON_VP);
+            rtscheck.RTS_SndData(0, PRINT_SURPLUS_TIME_HOUR_VP);
+            rtscheck.RTS_SndData(0, PRINT_SURPLUS_TIME_MIN_VP);
+          }
+          rtscheck.RTS_SndData((unsigned char)card.percentDone(), PRINT_PROCESS_VP);
+          last_cardpercentValue = card.percentDone();
+          rtscheck.RTS_SndData(10 * current_position[Z_AXIS], AXIS_Z_COORD_VP);
+        }
+      }
+
+      if(pause_action_flag && (false == sdcard_pause_check) && printingIsPaused() && !planner.has_blocks_queued())
+      {
+        pause_action_flag = false;
+        if((1 == active_extruder) && (1 == save_dual_x_carriage_mode))
+        {
+          queue.enqueue_now_P(PSTR("G0 F3000 X362 Y0"));
+        }
+        else
+        {
+          queue.enqueue_now_P(PSTR("G0 F3000 X-62 Y0"));
+        }
+      }
+
+      rtscheck.RTS_SndData(thermalManager.temp_hotend[0].celsius, HEAD0_CURRENT_TEMP_VP);
+      rtscheck.RTS_SndData(thermalManager.temp_hotend[1].celsius, HEAD1_CURRENT_TEMP_VP);
+      rtscheck.RTS_SndData(thermalManager.temp_bed.celsius, BED_CURRENT_TEMP_VP);
+
+      if((last_target_temperature[0] != thermalManager.temp_hotend[0].target) || (last_target_temperature[1] != thermalManager.temp_hotend[1].target) || (last_target_temperature_bed != thermalManager.temp_bed.target))
+      {
+        thermalManager.setTargetHotend(thermalManager.temp_hotend[0].target, 0);
+        thermalManager.setTargetHotend(thermalManager.temp_hotend[1].target, 1);
+        thermalManager.setTargetBed(thermalManager.temp_bed.target);
+        rtscheck.RTS_SndData(thermalManager.temp_hotend[0].target, HEAD0_SET_TEMP_VP);
+        rtscheck.RTS_SndData(thermalManager.temp_hotend[1].target, HEAD1_SET_TEMP_VP);
+        rtscheck.RTS_SndData(thermalManager.temp_bed.target, BED_SET_TEMP_VP);
+        last_target_temperature[0] = thermalManager.temp_hotend[0].target;
+        last_target_temperature[1] = thermalManager.temp_hotend[1].target;
+        last_target_temperature_bed = thermalManager.temp_bed.target;
+      }
+
+      if((thermalManager.temp_hotend[0].celsius >= thermalManager.temp_hotend[0].target) && (heatway == 1))
+      {
+        rtscheck.RTS_SndData(ExchangePageBase + 23, ExchangepageAddr);
+        heatway = 0;
+        rtscheck.RTS_SndData(10 * Filament0LOAD, HEAD0_FILAMENT_LOAD_DATA_VP);
+      }
+      else if((thermalManager.temp_hotend[1].celsius >= thermalManager.temp_hotend[1].target) && (heatway == 2))
+      {
+        rtscheck.RTS_SndData(ExchangePageBase + 23, ExchangepageAddr);
+        heatway = 0;
+        rtscheck.RTS_SndData(10 * Filament1LOAD, HEAD1_FILAMENT_LOAD_DATA_VP);
+      }
+
+      #if ENABLED(CHECKFILEMENT)
+        if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+        {
+          rtscheck.RTS_SndData(0, CHANGE_FILAMENT_ICON_VP);
+        }
+        else if((0 == save_dual_x_carriage_mode) && (1 == READ(CHECKFILEMENT0_PIN)))
+        {
+          rtscheck.RTS_SndData(1, CHANGE_FILAMENT_ICON_VP);
+        }
+        else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+        {
+          rtscheck.RTS_SndData(0, CHANGE_FILAMENT_ICON_VP);
+        }
+        else if((4 == save_dual_x_carriage_mode) && (1 == READ(CHECKFILEMENT1_PIN)))
+        {
+          rtscheck.RTS_SndData(1, CHANGE_FILAMENT_ICON_VP);
+        }
+        else if((0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+        {
+          rtscheck.RTS_SndData(0, CHANGE_FILAMENT_ICON_VP);
+        }
+        else if((0 != save_dual_x_carriage_mode) && (1 == READ(CHECKFILEMENT0_PIN)) && (1 == READ(CHECKFILEMENT1_PIN)))
+        {
+          rtscheck.RTS_SndData(1, CHANGE_FILAMENT_ICON_VP);
+        }
+      #endif
+
+      rtscheck.RTS_SndData(AutoHomeIconNum ++, AUTO_HOME_DISPLAY_ICON_VP);
+      if (AutoHomeIconNum > 8)
+      {
+        AutoHomeIconNum = 0;
+      }
+    }
+    next_rts_update_ms = ms + RTS_UPDATE_INTERVAL + Update_Time_Value;
+  }
+}
+
+// looping at the loop function
+void RTSUpdate()
+{
+  // Check the status of card
+  rtscheck.RTS_SDCardUpate();
+
+	sd_printing = IS_SD_PRINTING();
+	card_insert_st = IS_SD_INSERTED() ;
+
+	if((card_insert_st == false) && (sd_printing == true)){
+		rtscheck.RTS_SndData(ExchangePageBase + 46, ExchangepageAddr);	
+		rtscheck.RTS_SndData(0, CHANGE_SDCARD_ICON_VP);
+		/* 暂停打印，使得喷头可以回到零点 */
+		card.pauseSDPrint();
+		print_job_timer.pause();
+    pause_action_flag = true;
+    sdcard_pause_check = false;
+
+	}
+
+	/* 更新拔卡和插卡提示图标 */
+	if(last_card_insert_st != card_insert_st){
+		/* 当前页面显示为拔卡提示页面，但卡已经插入了，更新插卡图标 */
+		rtscheck.RTS_SndData((int)card_insert_st, CHANGE_SDCARD_ICON_VP);
+		last_card_insert_st = card_insert_st;
+	}
+
+
+  #if ENABLED(CHECKFILEMENT)
+    // checking filement status during printing
+    if((true == card.isPrinting()) && (true == PoweroffContinue))
+    {
+      if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+      {
+        Checkfilenum ++;
+        delay(5);
+      }
+      else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+      {
+        Checkfilenum ++;
+        delay(5);
+      }
+      else if((4 != save_dual_x_carriage_mode) && (0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+      {
+        Checkfilenum ++;
+        delay(5);
+      }
+      else
+      {
+        delay(5);
+        if((0 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT0_PIN)))
+        {
+          Checkfilenum ++;
+        }
+        else if((4 == save_dual_x_carriage_mode) && (0 == READ(CHECKFILEMENT1_PIN)))
+        {
+          Checkfilenum ++;
+        }
+        else if((4 != save_dual_x_carriage_mode) && (0 != save_dual_x_carriage_mode) && ((0 == READ(CHECKFILEMENT0_PIN)) || (0 == READ(CHECKFILEMENT1_PIN))))
+        {
+          Checkfilenum ++;
+        }
+        else
+        {
+          Checkfilenum = 0;
+        }
+      }
+      if(Checkfilenum > 10)
+      {
+        rtscheck.RTS_SndData(Beep, SoundAddr);
+        pause_z = current_position[Z_AXIS];
+        pause_e = current_position[E_AXIS] - 5;
+        if((0 == save_dual_x_carriage_mode) && (thermalManager.temp_hotend[0].celsius <= (thermalManager.temp_hotend[0].target - 5)))
+        {
+          rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          card.pauseSDPrint();
+          print_job_timer.pause();
+
+          pause_action_flag = true;
+          Update_Time_Value = 0;
+          sdcard_pause_check = false;
+          PrintFlag = 1;
+        }
+        else if((4 == save_dual_x_carriage_mode) && (thermalManager.temp_hotend[1].celsius <= (thermalManager.temp_hotend[1].target - 5)))
+        {
+          rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          card.pauseSDPrint();
+          print_job_timer.pause();
+
+          pause_action_flag = true;
+          Update_Time_Value = 0;
+          sdcard_pause_check = false;
+          PrintFlag = 1;
+        }
+        else if((0 != save_dual_x_carriage_mode) && (4 != save_dual_x_carriage_mode) && ((thermalManager.temp_hotend[0].celsius <= (thermalManager.temp_hotend[0].target - 5)) || (thermalManager.temp_hotend[1].celsius <= (thermalManager.temp_hotend[1].target - 5))))
+        {
+          rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          card.pauseSDPrint();
+          print_job_timer.pause();
+
+          pause_action_flag = true;
+          Checkfilenum = 0;
+          Update_Time_Value = 0;
+          sdcard_pause_check = false;
+          print_preheat_check = true;
+          PrintFlag = 1;
+        }
+        else if(thermalManager.temp_bed.celsius <= thermalManager.temp_bed.target)
+        {
+          rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          card.pauseSDPrint();
+          print_job_timer.pause();
+
+          pause_action_flag = true;
+          Checkfilenum = 0;
+          Update_Time_Value = 0;
+          sdcard_pause_check = false;
+          print_preheat_check = true;
+          PrintFlag = 1;
+        }
+        else if((!TEST(axis_trusted, X_AXIS)) || (!TEST(axis_trusted, Y_AXIS)) || (!TEST(axis_trusted, Z_AXIS)))
+        {
+          rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+          card.pauseSDPrint();
+          print_job_timer.pause();
+
+          pause_action_flag = true;
+          Checkfilenum = 0;
+          Update_Time_Value = 0;
+          sdcard_pause_check = false;
+          print_preheat_check = true;
+          PrintFlag = 1;
+        }
+        else
+        {
+          rtscheck.RTS_SndData(ExchangePageBase + 40, ExchangepageAddr);
+          waitway = 5;
+
+          #if ENABLED(POWER_LOSS_RECOVERY)
+            if (recovery.enabled)
+            {
+              recovery.save(true, false);
+            }
+          #endif
+          card.pauseSDPrint();
+          print_job_timer.pause();
+
+          pause_action_flag = true;
+          Checkfilenum = 0;
+          Update_Time_Value = 0;
+          sdcard_pause_check = false;
+          PrintFlag = 1;
+        }
+      }
+    }
+  #endif
+
+  
+  EachMomentUpdate();
+  // wait to receive massage and response
+  while(rtscheck.RTS_RecData() > 0)
+  {
+    rtscheck.RTS_HandleData();
+  }
+}
+
+void RTS_PauseMoveAxisPage()
+{
+  if(waitway == 1)
+  {
+    rtscheck.RTS_SndData(ExchangePageBase + 12, ExchangepageAddr);
+    waitway = 0;
+  }
+  else if(waitway == 5)
+  {
+    rtscheck.RTS_SndData(ExchangePageBase + 39, ExchangepageAddr);
+    waitway = 0;
+  }
+}
+
+void RTS_AutoBedLevelPage()
+{
+  if(waitway == 3)
+  {
+    rtscheck.RTS_SndData(ExchangePageBase + 22, ExchangepageAddr);
+    waitway = 0;
+  }
+}
+
+void RTS_MoveAxisHoming()
+{
+  if(waitway == 4)
+  {
+    rtscheck.RTS_SndData(ExchangePageBase + 29 + (AxisUnitMode - 1), ExchangepageAddr);
+    waitway = 0;
+  }
+  else if(waitway == 6)
+  {
+    #if ENABLED(HAS_LEVELING)
+      rtscheck.RTS_SndData(ExchangePageBase + 22, ExchangepageAddr);
+    #else
+      rtscheck.RTS_SndData(ExchangePageBase + 28, ExchangepageAddr);
+    #endif
+    waitway = 0;
+  }
+  else if(waitway == 7)
+  {
+    // Click Print finish
+    rtscheck.RTS_SndData(ExchangePageBase + 1, ExchangepageAddr);
+    waitway = 0;
+  }
+  if(active_extruder == 0)
+  {
+    rtscheck.RTS_SndData(0, EXCHANGE_NOZZLE_ICON_VP);
+  }
+  else
+  {
+    rtscheck.RTS_SndData(1, EXCHANGE_NOZZLE_ICON_VP);
+  }
+
+  rtscheck.RTS_SndData(10*current_position[X_AXIS], AXIS_X_COORD_VP);
+  rtscheck.RTS_SndData(10*current_position[Y_AXIS], AXIS_Y_COORD_VP);
+  rtscheck.RTS_SndData(10*current_position[Z_AXIS], AXIS_Z_COORD_VP);
+}
+
+#endif
diff --git a/Marlin/src/lcd/e3v2/creality/LCD_RTS.h b/Marlin/src/lcd/e3v2/creality/LCD_RTS.h
new file mode 100644
index 0000000000..8b4c50e386
--- /dev/null
+++ b/Marlin/src/lcd/e3v2/creality/LCD_RTS.h
@@ -0,0 +1,307 @@
+#ifndef LCD_RTS_H
+#define LCD_RTS_H
+
+#include "string.h"
+#include <Arduino.h>
+
+#include "../../../inc/MarlinConfig.h"
+
+extern int power_off_type_yes;
+
+/*********************************/
+#define FHONE   (0x5A)
+#define FHTWO   (0xA5)
+#define FHLENG  (0x06)
+#define TEXTBYTELEN     20
+#define MaxFileNumber   20
+
+#define FileNum             MaxFileNumber
+#define FileNameLen         TEXTBYTELEN
+#define RTS_UPDATE_INTERVAL 2000
+#define RTS_UPDATE_VALUE    (2 * RTS_UPDATE_INTERVAL)
+
+#define SizeofDatabuf       26
+
+/*************Register and Variable addr*****************/
+#define RegAddr_W   0x80
+#define RegAddr_R   0x81
+#define VarAddr_W   0x82
+#define VarAddr_R   0x83
+#define ExchangePageBase    ((unsigned long)0x5A010000)
+#define StartSoundSet       ((unsigned long)0x060480A0)
+#define FONT_EEPROM         0
+
+/*variable addr*/
+#define ExchangepageAddr      0x0084
+#define SoundAddr             0x00A0
+
+#define START1_PROCESS_ICON_VP             0x1000
+#define PRINT_SPEED_RATE_VP                0x1006
+#define PRINT_PROCESS_ICON_VP              0x100E
+#define PRINT_TIME_HOUR_VP                 0x1010
+#define PRINT_TIME_MIN_VP                  0x1012
+#define PRINT_PROCESS_VP                   0x1016
+#define HEAD0_FAN_ICON_VP                  0x101E
+#define HEAD1_FAN_ICON_VP                  0x101F
+#define CHANGE_FILAMENT0_TEMP_VP           0x1020
+#define CHANGE_FILAMENT1_TEMP_VP           0x1022
+#define AUTO_BED_LEVEL_ZOFFSET_VP          0x1026
+
+#define HEAD0_SET_TEMP_VP                  0x1034
+#define HEAD0_CURRENT_TEMP_VP              0x1036
+#define HEAD1_SET_TEMP_VP                  0x1038
+#define HEAD1_CURRENT_TEMP_VP              0x1052
+#define BED_SET_TEMP_VP                    0x103A
+#define BED_CURRENT_TEMP_VP                0x103C
+#define AUTO_HOME_DISPLAY_ICON_VP          0x1042
+#define AXIS_X_COORD_VP                    0x1048
+#define AXIS_Y_COORD_VP                    0x104A
+#define AXIS_Z_COORD_VP                    0x104C
+#define HEAD0_FILAMENT_LOAD_DATA_VP        0x1054
+#define HEAD1_FILAMENT_LOAD_DATA_VP        0x1058
+#define PRINTER_MACHINE_TEXT_VP            0x1060
+#define PRINTER_VERSION_TEXT_VP            0x106A
+#define PRINTER_PRINTSIZE_TEXT_VP          0x1074
+#define PRINTER_WEBSITE_TEXT_VP            0x107E
+#define AUTO_BED_LEVEL_ICON_VP             0x108D
+#define CHANGE_FILAMENT_ICON_VP            0x108E
+#define TWO_EXTRUDER_HOTEND_XOFFSET_VP     0x1092
+#define TWO_EXTRUDER_HOTEND_YOFFSET_VP     0x1094
+#define TWO_EXTRUDER_HOTEND_ZOFFSET_VP     0x1096
+
+#define AUTO_BED_LEVEL_1POINT_VP           0x1100
+// #define AUTO_BED_LEVEL_2POINT_VP           0x1102
+// #define AUTO_BED_LEVEL_3POINT_VP           0x1104
+// #define AUTO_BED_LEVEL_4POINT_VP           0x1106
+// #define AUTO_BED_LEVEL_5POINT_VP           0x1108
+// #define AUTO_BED_LEVEL_6POINT_VP           0x110A
+// #define AUTO_BED_LEVEL_7POINT_VP           0x110C
+// #define AUTO_BED_LEVEL_8POINT_VP           0x110E
+// #define AUTO_BED_LEVEL_9POINT_VP           0x1110
+// #define AUTO_BED_LEVEL_10POINT_VP          0x1112
+// #define AUTO_BED_LEVEL_11POINT_VP          0x1114
+// #define AUTO_BED_LEVEL_12POINT_VP          0x1116
+// #define AUTO_BED_LEVEL_13POINT_VP          0x1118
+// #define AUTO_BED_LEVEL_14POINT_VP          0x111A
+// #define AUTO_BED_LEVEL_15POINT_VP          0x111C
+// #define AUTO_BED_LEVEL_16POINT_VP          0x111E
+
+#define PRINT_SURPLUS_TIME_HOUR_VP         0x1162
+#define PRINT_SURPLUS_TIME_MIN_VP          0x1164
+#define SELECT_MODE_ICON_VP                0x1166
+#define CHANGE_SDCARD_ICON_VP              0x1168
+
+#define MOTOR_FREE_ICON_VP                 0x1200
+#define FILE1_SELECT_ICON_VP               0x1221
+#define FILE2_SELECT_ICON_VP               0x1222
+#define FILE3_SELECT_ICON_VP               0x1223
+#define FILE4_SELECT_ICON_VP               0x1224
+#define FILE5_SELECT_ICON_VP               0x1225
+#define FILE6_SELECT_ICON_VP               0x1226
+#define FILE7_SELECT_ICON_VP               0x1227
+#define FILE8_SELECT_ICON_VP               0x1228
+#define FILE9_SELECT_ICON_VP               0x1229
+#define FILE10_SELECT_ICON_VP              0x122A
+#define FILE11_SELECT_ICON_VP              0x122B
+#define FILE12_SELECT_ICON_VP              0x122C
+#define FILE13_SELECT_ICON_VP              0x122D
+#define FILE14_SELECT_ICON_VP              0x122E
+#define FILE15_SELECT_ICON_VP              0x122F
+#define FILE16_SELECT_ICON_VP              0x1230
+#define FILE17_SELECT_ICON_VP              0x1231
+#define FILE18_SELECT_ICON_VP              0x1232
+#define FILE19_SELECT_ICON_VP              0x1233
+#define FILE20_SELECT_ICON_VP              0x1234
+
+#define FILE1_TEXT_VP                      0x200A
+#define FILE2_TEXT_VP                      0x201E
+#define FILE3_TEXT_VP                      0x2032
+#define FILE4_TEXT_VP                      0x2046
+#define FILE5_TEXT_VP                      0x205A
+#define FILE6_TEXT_VP                      0x206E
+#define FILE7_TEXT_VP                      0x2082
+#define FILE8_TEXT_VP                      0x2096
+#define FILE9_TEXT_VP                      0x20AA
+#define FILE10_TEXT_VP                     0x20BE
+#define FILE11_TEXT_VP                     0x20D2
+#define FILE12_TEXT_VP                     0x20E6
+#define FILE13_TEXT_VP                     0x20FA
+#define FILE14_TEXT_VP                     0x210E
+#define FILE15_TEXT_VP                     0x2122
+#define FILE16_TEXT_VP                     0x2136
+#define FILE17_TEXT_VP                     0x214A
+#define FILE18_TEXT_VP                     0x215E
+#define FILE19_TEXT_VP                     0x2172
+#define FILE20_TEXT_VP                     0x2186
+
+#define SELECT_FILE_TEXT_VP                0x219A
+#define TWO_COLOR_MODE_ICON_VP             0x21B8
+#define COPY_MODE_ICON_VP                  0x21B9
+#define MIRROR_MODE_ICON_VP                0x21BA
+#define SINGLE_MODE_ICON_VP                0x21BB
+#define EXCHANGE_NOZZLE_ICON_VP            0x21BC
+#define PRINT_MODE_ICON_VP                 0x21BD
+#define PRINT_FILE_TEXT_VP                 0x21C0
+#define Screen_Version_VP                  0X2200
+#define FilenameNature                     0x6003
+#define	Beep       					               ((unsigned long)0x02AF0100)
+/************struct**************/
+typedef struct DataBuf
+{
+  unsigned char len;
+  unsigned char head[2];
+  unsigned char command;
+  unsigned long addr;
+  unsigned long bytelen;
+  unsigned short data[32];
+  unsigned char reserv[4];
+} DB;
+
+typedef struct CardRecord
+{
+  int recordcount;
+  int Filesum;
+  unsigned long addr[FileNum];
+  char Cardshowfilename[FileNum][FileNameLen];
+  char Cardfilename[FileNum][FileNameLen];
+}CRec;
+
+class RTSUI
+{
+  public:
+  #if ENABLED(LCD_BED_LEVELING) && EITHER(PROBE_MANUALLY, MESH_BED_LEVELING)
+    static bool wait_for_bl_move;
+  #else
+    static constexpr bool wait_for_bl_move = false;
+  #endif
+};
+
+extern RTSUI rtsui;
+
+class RTSSHOW
+{
+  public:
+    RTSSHOW();
+    int RTS_RecData();
+    void RTS_SDCardInit(void);
+    bool RTS_SD_Detected(void);
+    void RTS_SDCardUpate(void);
+    void RTS_SndData(void);
+    void RTS_SndData(const String &, unsigned long, unsigned char = VarAddr_W);
+    void RTS_SndData(const char[], unsigned long, unsigned char = VarAddr_W);
+    void RTS_SndData(char, unsigned long, unsigned char = VarAddr_W);
+    void RTS_SndData(unsigned char*, unsigned long, unsigned char = VarAddr_W);
+    void RTS_SndData(int, unsigned long, unsigned char = VarAddr_W);
+    void RTS_SndData(float, unsigned long, unsigned char = VarAddr_W);
+    void RTS_SndData(unsigned int,unsigned long, unsigned char = VarAddr_W);
+    void RTS_SndData(long,unsigned long, unsigned char = VarAddr_W);
+    void RTS_SndData(unsigned long,unsigned long, unsigned char = VarAddr_W);
+    void RTS_SDcard_Stop();
+    void RTS_HandleData();
+    void RTS_Init();
+
+    DB recdat;
+    DB snddat;
+  private:
+    unsigned char databuf[SizeofDatabuf];
+};
+
+extern RTSSHOW rtscheck;
+
+enum PROC_COM
+{
+  MainPageKey = 0,
+  AdjustmentKey,
+  PrintSpeedKey,
+  StopPrintKey,
+  PausePrintKey,
+  ResumePrintKey,
+  ZOffsetKey,
+  TempScreenKey,
+  CoolScreenKey,
+  Heater0TempEnterKey,
+  Heater1TempEnterKey,
+  HotBedTempEnterKey,
+  SettingScreenKey,
+  SettingBackKey,
+  BedLevelFunKey,
+  AxisPageSelectKey,
+  XaxismoveKey,
+  YaxismoveKey,
+  ZaxismoveKey,
+  SelectExtruderKey,
+  Heater0LoadEnterKey,
+  FilamentLoadKey,
+  Heater1LoadEnterKey,
+  SelectLanguageKey,
+  FilamentCheckKey,
+  PowerContinuePrintKey,
+  PrintSelectModeKey,
+  XhotendOffsetKey,
+  YhotendOffsetKey,
+  ZhotendOffsetKey,
+  StoreMemoryKey,
+  PrintFileKey,
+  SelectFileKey,
+  SaveEEPROM,
+  ChangePageKey
+};
+
+const unsigned long Addrbuf[] = 
+{
+  0x1002,
+  0x1004,
+  0x1006,
+  0x1008,
+  0x100A,
+  0x100C,
+  0x1026,
+  0x1030,
+  0x1032,
+  0x1034,
+  0x1038,
+  0x103A,
+  0x103E,
+  0x1040,
+  0x1044,
+  0x1046,
+  0x1048,
+  0x104A,
+  0x104C,
+  0x104E,
+  0x1054,
+  0x1056,
+  0x1058,
+  0x105C,
+  0x105E,
+  0x105F,
+  0x1090,
+  0x1092,
+  0x1094,
+  0x1096,
+  0x1098,
+  0x2198,
+  0x2199,
+  0X2202,
+  0x110E,
+  0
+};
+
+extern void RTSUpdate();
+extern void RTSInit();
+
+extern uint8_t active_extruder_font;
+extern uint8_t dualXPrintingModeStatus;
+extern int Update_Time_Value;
+extern bool PoweroffContinue;
+extern bool sdcard_pause_check;
+
+extern char save_dual_x_carriage_mode;
+extern float current_position_x0_axis;
+extern float current_position_x1_axis;
+
+void RTS_PauseMoveAxisPage();
+void RTS_AutoBedLevelPage();
+void RTS_MoveAxisHoming();
+
+#endif
```

Comment out some code, include LCD_RTS header (but don't use it?)

```diff
diff --git a/Marlin/src/lcd/marlinui.cpp b/Marlin/src/lcd/marlinui.cpp
index 5fcbff870e..6fb5115d55 100644
--- a/Marlin/src/lcd/marlinui.cpp
+++ b/Marlin/src/lcd/marlinui.cpp
@@ -50,6 +50,8 @@ MarlinUI ui;
   #include "e3v2/creality/dwin.h"
 #elif ENABLED(DWIN_CREALITY_LCD_ENHANCED)
   #include "e3v2/enhanced/dwin.h"
+#elif ENABLED(RTS_AVAILABLE)
+  #include "e3v2/creality/LCD_RTS.h"
 #elif ENABLED(DWIN_CREALITY_LCD_JYERSUI)
   #include "e3v2/jyersui/dwin.h"
 #endif
@@ -111,7 +113,7 @@ constexpr uint8_t epps = ENCODER_PULSES_PER_STEP;
   void MarlinUI::set_brightness(const uint8_t value) {
     backlight = !!value;
     if (backlight) brightness = constrain(value, LCD_BRIGHTNESS_MIN, LCD_BRIGHTNESS_MAX);
-    _set_brightness();
+    //_set_brightness();
   }
 #endif
 
@@ -1496,7 +1498,7 @@ constexpr uint8_t epps = ENCODER_PULSES_PER_STEP;
     #endif
 
     TERN_(EXTENSIBLE_UI, ExtUI::onStatusChanged(status_message));
-    TERN_(HAS_DWIN_E3V2_BASIC, DWIN_StatusChanged(status_message));
+    //TERN_(HAS_DWIN_E3V2_BASIC, DWIN_StatusChanged(status_message));
     TERN_(DWIN_CREALITY_LCD_JYERSUI, CrealityDWIN.Update_Status(status_message));
   }
 
@@ -1691,7 +1693,7 @@ constexpr uint8_t epps = ENCODER_PULSES_PER_STEP;
       init_lcd(); // Revive a noisy shared SPI LCD
     #endif
 
-    refresh();
+    //refresh();
 
     #if HAS_WIRED_LCD || defined(LED_BACKLIGHT_TIMEOUT)
       const millis_t ms = millis();
```

Tell touch screen if homing fails.

```diff
diff --git a/Marlin/src/module/endstops.cpp b/Marlin/src/module/endstops.cpp
index d29fd3ecb3..5f6a2c69ce 100644
--- a/Marlin/src/module/endstops.cpp
+++ b/Marlin/src/module/endstops.cpp
@@ -30,6 +30,9 @@
 #include "../sd/cardreader.h"
 #include "temperature.h"
 #include "../lcd/marlinui.h"
+#if ENABLED(RTS_AVAILABLE)
+  #include "../lcd/e3v2/creality/LCD_RTS.h"
+#endif
 
 #if ENABLED(ENDSTOP_INTERRUPTS_FEATURE)
   #include HAL_PATH(../HAL, endstop_interrupts.h)
@@ -384,7 +387,12 @@ void Endstops::not_homing() {
   // If the last move failed to trigger an endstop, call kill
   void Endstops::validate_homing_move() {
     if (trigger_state()) hit_on_purpose();
-    else kill(GET_TEXT(MSG_KILL_HOMING_FAILED));
+    else{ 
+      
+      rtscheck.RTS_SndData(ExchangePageBase + 55, ExchangepageAddr);
+
+      kill(GET_TEXT(MSG_KILL_HOMING_FAILED));
+    }
   }
 #endif
 
```

- add special case for extruder 2 single mode
  - set soft endstop min and max x to 0 and dual_max_x
- don't use normal extruder 2 code in mode 4 and mode 1
  - I wonder why
- custom code in mirroed mode or duplicate code:
  - `set_duplication_enabled(false); // Clear stale duplication state`
  - if duplicate: `new_pos.x = x0_pos + duplicate_extruder_x_offset;`
  - else (mirro?): `new_pos.x = _MIN(X_BED_SIZE - x0_pos, X2_MAX_POS);`

```diff
diff --git a/Marlin/src/module/motion.cpp b/Marlin/src/module/motion.cpp
index a56831c214..dd5b251473 100644
--- a/Marlin/src/module/motion.cpp
+++ b/Marlin/src/module/motion.cpp
@@ -55,6 +55,10 @@
   #include "../lcd/marlinui.h"
 #endif
 
+#if ENABLED(RTS_AVAILABLE)
+  #include "../lcd/e3v2/creality/LCD_RTS.h"
+#endif
+
 #if HAS_FILAMENT_SENSOR
   #include "../feature/runout.h"
 #endif
@@ -713,7 +717,7 @@ void restore_feedrate_and_scaling() {
         // In Dual X mode hotend_offset[X] is T1's home position
         const float dual_max_x = _MAX(hotend_offset[1].x, X2_MAX_POS);
 
-        if (new_tool_index != 0) {
+        if ((new_tool_index != 0) && (save_dual_x_carriage_mode != 4) && (save_dual_x_carriage_mode != 1)) {
           // T1 can move from X2_MIN_POS to X2_MAX_POS or X2 home position (whichever is larger)
           soft_endstop.min.x = X2_MIN_POS;
           soft_endstop.max.x = dual_max_x;
@@ -724,6 +728,10 @@ void restore_feedrate_and_scaling() {
           soft_endstop.min.x = X1_MIN_POS;
           soft_endstop.max.x = _MIN(X1_MAX_POS, dual_max_x - duplicate_extruder_x_offset);
         }
+        else if ((save_dual_x_carriage_mode == 4) || (save_dual_x_carriage_mode == 1)) {
+          soft_endstop.min.x = 0;
+          soft_endstop.max.x = dual_max_x;
+        }
         else {
           // In other modes, T0 can move from X1_MIN_POS to X1_MAX_POS
           soft_endstop.min.x = X1_MIN_POS;
@@ -1193,7 +1201,9 @@ FORCE_INLINE void segment_idle(millis_t &next_idle_ms) {
         case DXC_MIRRORED_MODE:
         case DXC_DUPLICATION_MODE:
           if (active_extruder == 0) {
+            set_duplication_enabled(false); // Clear stale duplication state
             // Restore planner to parked head (T1) X position
+            float x0_pos = current_position.x;
             xyze_pos_t pos_now = current_position;
             pos_now.x = inactive_extruder_x;
             planner.set_position_mm(pos_now);
@@ -1201,7 +1211,9 @@ FORCE_INLINE void segment_idle(millis_t &next_idle_ms) {
             // Keep the same X or add the duplication X offset
             xyze_pos_t new_pos = pos_now;
             if (dual_x_carriage_mode == DXC_DUPLICATION_MODE)
-              new_pos.x += duplicate_extruder_x_offset;
+              new_pos.x = x0_pos + duplicate_extruder_x_offset;
+            else
+              new_pos.x = _MIN(X_BED_SIZE - x0_pos, X2_MAX_POS);
 
             // Move duplicate extruder into the correct position
             if (DEBUGGING(LEVELING)) DEBUG_ECHOLNPGM("Set planner X", inactive_extruder_x, " ... Line to X", new_pos.x);
```
added this:
```cpp
  FORCE_INLINE bool dxc_is_parked() { return dual_x_carriage_mode >= DXC_AUTO_PARK_MODE; }
```

I saw that used earlier, didn't realized it was custom too.


```diff
diff --git a/Marlin/src/module/motion.h b/Marlin/src/module/motion.h
index 50df5675e6..b9885c95fb 100644
--- a/Marlin/src/module/motion.h
+++ b/Marlin/src/module/motion.h
@@ -586,7 +586,7 @@ void home_if_needed(const bool keeplev=false);
   extern bool idex_mirrored_mode;                   // Used in mode 3
 
   FORCE_INLINE bool idex_is_duplicating() { return dual_x_carriage_mode >= DXC_DUPLICATION_MODE; }
-
+  FORCE_INLINE bool dxc_is_parked() { return dual_x_carriage_mode >= DXC_AUTO_PARK_MODE; }
   float x_home_pos(const uint8_t extruder);
 
   #define TOOL_X_HOME_DIR(T) ((T) ? X2_HOME_DIR : X_HOME_DIR)
```

You might call patches to this file "a change in plans".

They made some modifications to `ENABLE_ONE_E`, especially for duplicate mode.
They commented them too, which is nice. In English even.

I don't actually know what's going on though. I'll have to read more of planner.cpp to understand the context.


```diff
diff --git a/Marlin/src/module/planner.cpp b/Marlin/src/module/planner.cpp
index c6edfb835e..ff26eff690 100644
--- a/Marlin/src/module/planner.cpp
+++ b/Marlin/src/module/planner.cpp
@@ -2211,19 +2211,15 @@ bool Planner::_populate_block(block_t * const block, bool split_move,
           if (g_uc_extruder_last_move[i]) g_uc_extruder_last_move[i]--;
 
         #define E_STEPPER_INDEX(E) TERN(SWITCHING_EXTRUDER, (E) / 2, E)
+        #define _IS_DUPE(N) TERN0(HAS_DUPLICATION_MODE, (extruder_duplication_enabled && TERN1(MULTI_NOZZLE_DUPLICATION, TEST(duplication_e_mask, N))))
 
         #define ENABLE_ONE_E(N) do{ \
-          if (E_STEPPER_INDEX(extruder) == N) { \
-            stepper.ENABLE_EXTRUDER(N); \
-            g_uc_extruder_last_move[N] = (BLOCK_BUFFER_SIZE) * 2; \
-            if ((N) == 0 && TERN0(HAS_DUPLICATION_MODE, extruder_duplication_enabled)) \
-              stepper.ENABLE_EXTRUDER(1); \
-          } \
-          else if (!g_uc_extruder_last_move[N]) { \
-            stepper.DISABLE_EXTRUDER(N); \
-            if ((N) == 0 && TERN0(HAS_DUPLICATION_MODE, extruder_duplication_enabled)) \
-              stepper.DISABLE_EXTRUDER(1); \
+          if (N == E_STEPPER_INDEX(extruder) || _IS_DUPE(N)) {    /* N is 'extruder', or N is duplicating */ \
+            stepper.ENABLE_EXTRUDER(N);                           /* Enable the relevant E stepper... */ \
+            g_uc_extruder_last_move[N] = (BLOCK_BUFFER_SIZE) * 2; /* ...and reset its counter */ \
           } \
+          else if (!g_uc_extruder_last_move[N])                   /* Counter expired since last E stepper enable */ \
+            stepper.DISABLE_EXTRUDER(N);                          /* Disable the E stepper */ \
         }while(0);
 
       #else
```

settings:
- import the touch screen header
- replace some Ender-3 V2 DWIN with SV04 DWIN
- add variables for dual printing and active extruder font
- modify runout sensor code
- modify config echo code
  - they used SERIAL_ERROR_MSG for some of it, even though it's just a status. I bet that would so it would stand out for debugging, and they forgot to turn it back down.


```diff
diff --git a/Marlin/src/module/settings.cpp b/Marlin/src/module/settings.cpp
index a4b8bb19e6..75a09d9d4d 100644
--- a/Marlin/src/module/settings.cpp
+++ b/Marlin/src/module/settings.cpp
@@ -77,6 +77,10 @@
   #include "../lcd/e3v2/jyersui/dwin.h"
 #endif
 
+#if ENABLED(RTS_AVAILABLE)
+  #include "../lcd/e3v2/creality/LCD_RTS.h"
+#endif
+
 #if HAS_SERVOS
   #include "servo.h"
 #endif
@@ -442,8 +446,10 @@ typedef struct SettingsDataStruct {
   #endif
 
   //
-  // Ender-3 V2 DWIN
+  // SV04 DWIN
   //
+  uint8_t dualXPrintingModeStatus;
+  uint8_t active_extruder_font;
   #if ENABLED(DWIN_CREALITY_LCD_ENHANCED)
     uint8_t dwin_data[eeprom_data_size];
   #elif ENABLED(DWIN_CREALITY_LCD_JYERSUI)
@@ -718,16 +724,16 @@ void MarlinSettings::postprocess() {
       #if HAS_FILAMENT_SENSOR
         const bool &runout_sensor_enabled = runout.enabled;
       #else
-        constexpr int8_t runout_sensor_enabled = -1;
+        constexpr bool runout_sensor_enabled = true;
       #endif
-      _FIELD_TEST(runout_sensor_enabled);
-      EEPROM_WRITE(runout_sensor_enabled);
 
       #if HAS_FILAMENT_RUNOUT_DISTANCE
         const float &runout_distance_mm = runout.runout_distance();
       #else
         constexpr float runout_distance_mm = 0;
       #endif
+      _FIELD_TEST(runout_sensor_enabled);
+      EEPROM_WRITE(runout_sensor_enabled);
       EEPROM_WRITE(runout_distance_mm);
     }
 
@@ -1384,7 +1390,8 @@ void MarlinSettings::postprocess() {
     #if CASELIGHT_USES_BRIGHTNESS
       EEPROM_WRITE(caselight.brightness);
     #endif
-
+    EEPROM_WRITE(dualXPrintingModeStatus);
+    EEPROM_WRITE(active_extruder_font);
     //
     // Password feature
     //
@@ -1582,14 +1589,18 @@ void MarlinSettings::postprocess() {
       // Filament Runout Sensor
       //
       {
-        int8_t runout_sensor_enabled;
+         #if HAS_FILAMENT_SENSOR
+          const bool &runout_sensor_enabled = runout.enabled;
+        #else
+          bool runout_sensor_enabled;
+        #endif
         _FIELD_TEST(runout_sensor_enabled);
         EEPROM_READ(runout_sensor_enabled);
         #if HAS_FILAMENT_SENSOR
           runout.enabled = runout_sensor_enabled < 0 ? FIL_RUNOUT_ENABLED_DEFAULT : runout_sensor_enabled;
         #endif
 
-        TERN_(HAS_FILAMENT_SENSOR, if (runout.enabled) runout.reset());
+        //TERN_(HAS_FILAMENT_SENSOR, if (runout.enabled) runout.reset());
 
         float runout_distance_mm;
         EEPROM_READ(runout_distance_mm);
@@ -2350,7 +2361,17 @@ void MarlinSettings::postprocess() {
         ui.set_language(ui_language);
       }
       #endif
+      if((dualXPrintingModeStatus != 0) && (dualXPrintingModeStatus != 1) && (dualXPrintingModeStatus != 2) && (dualXPrintingModeStatus != 3) && (dualXPrintingModeStatus != 4))
+      {
+        dualXPrintingModeStatus = 0;
+      }
+      EEPROM_READ(dualXPrintingModeStatus);
 
+      if((active_extruder_font != 0) && (active_extruder_font != 1))
+      {
+        active_extruder_font = 0;
+      }
+      EEPROM_READ(active_extruder_font);
       //
       // Validate Final Size and CRC
       //
@@ -3298,7 +3319,13 @@ void MarlinSettings::reset() {
       CONFIG_ECHO_START(); SERIAL_ECHO_SP(2); gcode.M553_report();
       CONFIG_ECHO_START(); SERIAL_ECHO_SP(2); gcode.M554_report();
     #endif
+    CONFIG_ECHO_HEADING("0:Single 1:Two-color 2:Copy 3:Mirror Dual X Printing Mode Status:");
+    CONFIG_ECHO_START();
+    SERIAL_ERROR_MSG("  M605 S", int(dualXPrintingModeStatus));
 
+    CONFIG_ECHO_HEADING("0:extruder0 1:extruder1 active extruder font:");
+    CONFIG_ECHO_START();
+    SERIAL_ERROR_MSG("  T", int(active_extruder_font));
     TERN_(HAS_MULTI_LANGUAGE, gcode.M414_report(forReplay));
   }
 
```

send tempature to touch screen

```diff
diff --git a/Marlin/src/module/temperature.cpp b/Marlin/src/module/temperature.cpp
index d59ebe5695..bfe1ea5d52 100644
--- a/Marlin/src/module/temperature.cpp
+++ b/Marlin/src/module/temperature.cpp
@@ -49,6 +49,8 @@
   #include "../lcd/e3v2/creality/dwin.h"
 #elif ENABLED(DWIN_CREALITY_LCD_ENHANCED)
   #include "../lcd/e3v2/enhanced/dwin.h"
+#elif ENABLED(RTS_AVAILABLE)
+  #include "../lcd/e3v2/creality/LCD_RTS.h"
 #endif
 
 #if ENABLED(EXTENSIBLE_UI)
@@ -731,11 +733,26 @@ volatile bool Temperature::raw_temps_ready = false;
                 if (current_temp > watch_temp_target) heated = true;  // - Flag if target temperature reached
               }
               else if (ELAPSED(ms, temp_change_ms))                   // Watch timer expired
+              {
+                #if ENABLED(RTS_AVAILABLE)
+                 
+                    rtscheck.RTS_SndData(ExchangePageBase + 53, ExchangepageAddr);
+                  
+                #endif
                  _temp_error(heater_id, str_t_heating_failed, GET_TEXT(MSG_HEATING_FAILED_LCD));
               }
+               
+            }
             else if (current_temp < target - (MAX_OVERSHOOT_PID_AUTOTUNE)) // Heated, then temperature fell too far?
+            {
+              #if ENABLED(RTS_AVAILABLE)
+                
+                    rtscheck.RTS_SndData(ExchangePageBase + 52, ExchangepageAddr);
+                  
+              #endif
               _temp_error(heater_id, str_t_thermal_runaway, GET_TEXT(MSG_THERMAL_RUNAWAY));
             }
+          }
         #endif
       } // every 2 seconds
 
@@ -747,6 +764,11 @@ volatile bool Temperature::raw_temps_ready = false;
         TERN_(DWIN_CREALITY_LCD, DWIN_Popup_Temperature(0));
         TERN_(DWIN_CREALITY_LCD_ENHANCED, DWIN_PidTuning(PID_TUNING_TIMEOUT));
         TERN_(EXTENSIBLE_UI, ExtUI::onPidTuning(ExtUI::result_t::PID_TUNING_TIMEOUT));
+        #if ENABLED(RTS_AVAILABLE)
+          
+            rtscheck.RTS_SndData(ExchangePageBase + 53, ExchangepageAddr);
+          
+        #endif
         SERIAL_ECHOLNPGM(STR_PID_TIMEOUT);
         break;
       }
@@ -806,9 +828,8 @@ volatile bool Temperature::raw_temps_ready = false;
 
       // Run HAL idle tasks
       TERN_(HAL_IDLETASK, HAL_idletask());
-
       // Run UI update
-      TERN(HAS_DWIN_E3V2_BASIC, DWIN_Update(), ui.update());
+      TERN(HAS_DWIN_E3V2_BASIC, RTSUpdate(), ui.update());
     }
     wait_for_heatup = false;
 
@@ -1029,14 +1050,18 @@ void Temperature::_temp_error(const heater_id_t heater_id, PGM_P const serial_ms
 
 void Temperature::max_temp_error(const heater_id_t heater_id) {
   #if HAS_DWIN_E3V2_BASIC && (HAS_HOTEND || HAS_HEATED_BED)
-    DWIN_Popup_Temperature(1);
+    
+      rtscheck.RTS_SndData(ExchangePageBase + 54, ExchangepageAddr);
+    
   #endif
   _temp_error(heater_id, PSTR(STR_T_MAXTEMP), GET_TEXT(MSG_ERR_MAXTEMP));
 }
 
 void Temperature::min_temp_error(const heater_id_t heater_id) {
   #if HAS_DWIN_E3V2_BASIC && (HAS_HOTEND || HAS_HEATED_BED)
-    DWIN_Popup_Temperature(0);
+    
+      rtscheck.RTS_SndData(ExchangePageBase + 54, ExchangepageAddr);
+    
   #endif
   _temp_error(heater_id, PSTR(STR_T_MINTEMP), GET_TEXT(MSG_ERR_MINTEMP));
 }
@@ -1325,7 +1350,15 @@ void Temperature::manage_heater() {
 
     HOTEND_LOOP() {
       #if ENABLED(THERMAL_PROTECTION_HOTENDS)
-        if (degHotend(e) > temp_range[e].maxtemp) max_temp_error((heater_id_t)e);
+        if (degHotend(e) > temp_range[e].maxtemp)
+        {
+          #if ENABLED(RTS_AVAILABLE)
+            
+              rtscheck.RTS_SndData(ExchangePageBase + 54, ExchangepageAddr);
+            
+          #endif
+           max_temp_error((heater_id_t)e);
+        }
       #endif
 
       TERN_(HEATER_IDLE_HANDLER, heater_idle[e].update(ms));
@@ -1343,7 +1376,11 @@ void Temperature::manage_heater() {
           if (watch_hotend[e].check(degHotend(e)))  // Increased enough?
             start_watching_hotend(e);               // If temp reached, turn off elapsed check
           else {
-            TERN_(HAS_DWIN_E3V2_BASIC, DWIN_Popup_Temperature(0));
+            #if ENABLED(RTS_AVAILABLE)
+              
+                rtscheck.RTS_SndData(ExchangePageBase + 53, ExchangepageAddr);
+              
+            #endif
             _temp_error((heater_id_t)e, str_t_heating_failed, GET_TEXT(MSG_HEATING_FAILED_LCD));
           }
         }
@@ -1377,7 +1414,15 @@ void Temperature::manage_heater() {
   #if HAS_HEATED_BED
 
     #if ENABLED(THERMAL_PROTECTION_BED)
-      if (degBed() > BED_MAXTEMP) max_temp_error(H_BED);
+      if (degBed() > BED_MAXTEMP)
+      {
+        #if ENABLED(RTS_AVAILABLE)
+          
+            rtscheck.RTS_SndData(ExchangePageBase + 54, ExchangepageAddr);
+          
+        #endif
+        max_temp_error(H_BED);
+      } 
     #endif
 
     #if WATCH_BED
@@ -1386,7 +1431,11 @@ void Temperature::manage_heater() {
         if (watch_bed.check(degBed()))          // Increased enough?
           start_watching_bed();                 // If temp reached, turn off elapsed check
         else {
-          TERN_(HAS_DWIN_E3V2_BASIC, DWIN_Popup_Temperature(0));
+          #if ENABLED(RTS_AVAILABLE)
+           
+              rtscheck.RTS_SndData(ExchangePageBase + 53, ExchangepageAddr);
+           
+          #endif
           _temp_error(H_BED, str_t_heating_failed, GET_TEXT(MSG_HEATING_FAILED_LCD));
         }
       }
@@ -2596,7 +2645,11 @@ void Temperature::init() {
         state = TRRunaway;
 
       case TRRunaway:
-        TERN_(HAS_DWIN_E3V2_BASIC, DWIN_Popup_Temperature(0));
+        #if ENABLED(RTS_AVAILABLE)
+         
+            rtscheck.RTS_SndData(ExchangePageBase + 52, ExchangepageAddr);
+          
+        #endif
         _temp_error(heater_id, str_t_thermal_runaway, GET_TEXT(MSG_THERMAL_RUNAWAY));
     }
   }
@@ -3728,9 +3781,10 @@ void Temperature::isr() {
       if (wait_for_heatup) {
         wait_for_heatup = false;
         #if HAS_DWIN_E3V2_BASIC
-          HMI_flag.heat_flag = 0;
-          duration_t elapsed = print_job_timer.duration();  // print timer
-          dwin_heat_time = elapsed.value;
+          Update_Time_Value = RTS_UPDATE_VALUE;
+          
+            rtscheck.RTS_SndData(ExchangePageBase + 11, ExchangepageAddr);
+          
         #endif
         ui.reset_status();
         TERN_(PRINTER_EVENT_LEDS, printerEventLEDs.onHeatingDone());

```

I read somewhere that they customized this file instead of adding or modifying the one for the actual board they're using.  Looks like it might be true based on the beginning:

```diff
- * Creality 4.2.x (STM32F103RET6) board pin assignments
+ * CREALITY (STM32F103) board pin assignments
/* ... */
#define DEFAULT_MACHINE_NAME "Creality V5.2.1"
```

Important file that defines all the pins. But I wonder if there's a better way to do it than how they did it.

```diff
diff --git a/Marlin/src/pins/stm32f1/pins_CREALITY_V4.h b/Marlin/src/pins/stm32f1/pins_CREALITY_V4.h
index c60d4dc2ba..25f0aec972 100644
--- a/Marlin/src/pins/stm32f1/pins_CREALITY_V4.h
+++ b/Marlin/src/pins/stm32f1/pins_CREALITY_V4.h
@@ -1,9 +1,9 @@
 /**
  * Marlin 3D Printer Firmware
- * Copyright (c) 2020 MarlinFirmware [https://github.com/MarlinFirmware/Marlin]
+ * Copyright (C) 2016 MarlinFirmware [https://github.com/MarlinFirmware/Marlin]
  *
  * Based on Sprinter and grbl.
- * Copyright (c) 2011 Camiel Gubbels / Erik van der Zalm
+ * Copyright (C) 2011 Camiel Gubbels / Erik van der Zalm
  *
  * This program is free software: you can redistribute it and/or modify
  * it under the terms of the GNU General Public License as published by
@@ -19,142 +19,178 @@
  * along with this program.  If not, see <https://www.gnu.org/licenses/>.
  *
  */
-#pragma once
 
 /**
- * Creality 4.2.x (STM32F103RET6) board pin assignments
+ * CREALITY (STM32F103) board pin assignments
  */
 
 #include "env_validate.h"
 
-#if HOTENDS > 1 || E_STEPPERS > 1
-  #error "Creality V4 only supports one hotend / E-stepper. Comment out this line to continue."
+#if HOTENDS > 2 || E_STEPPERS > 2
+  #error "CREALITY supports up to 2 hotends / E-steppers. Comment out this line to continue."
 #endif
 
 #ifndef BOARD_INFO_NAME
   #define BOARD_INFO_NAME      "Creality V4"
 #endif
 #ifndef DEFAULT_MACHINE_NAME
-  #define DEFAULT_MACHINE_NAME "Ender 3 V2"
+  #define DEFAULT_MACHINE_NAME "Creality V5.2.1"
 #endif
 
-#define BOARD_NO_NATIVE_USB
-
 //
 // EEPROM
 //
 #if NO_EEPROM_SELECTED
-  #define IIC_BL24CXX_EEPROM                      // EEPROM on I2C-0
-  //#define SDCARD_EEPROM_EMULATION
-#endif
+  // FLASH
+  //#define FLASH_EEPROM_EMULATION
 
+  // I2C
+  #define IIC_BL24CXX_EEPROM                      // EEPROM on I2C-0 used only for display settings
   #if ENABLED(IIC_BL24CXX_EEPROM)
-  #define IIC_EEPROM_SDA                    PA11
-  #define IIC_EEPROM_SCL                    PA12
+    #define IIC_EEPROM_SDA       PC2
+    #define IIC_EEPROM_SCL       PC3
     #define MARLIN_EEPROM_SIZE             0x800  // 2Kb (24C16)
-#elif ENABLED(SDCARD_EEPROM_EMULATION)
+  #else
+    #define SDCARD_EEPROM_EMULATION               // SD EEPROM until all EEPROM is BL24CXX
     #define MARLIN_EEPROM_SIZE             0x800  // 2Kb
   #endif
 
+  // SPI
+  //#define SPI_EEPROM                            // EEPROM on SPI-0
+  //#define SPI_CHAN_EEPROM1  ?
+  //#define SPI_EEPROM1_CS    ?
+
+  // 2K EEPROM
+  //#define SPI_EEPROM2_CS    ?
+
+  // 32Mb FLASH
+  //#define SPI_FLASH_CS      ?
+#endif
+
 //
 // Servos
 //
-#ifndef SERVO0_PIN
-  #ifndef HAS_PIN_27_BOARD
-    #define SERVO0_PIN                      PB0   // BLTouch OUT
+#ifdef BLTOUCH
+  #define SERVO0_PIN       PD13  // BLTouch OUT
+  #define Z_MIN_PIN        PD12
 #else
-    #define SERVO0_PIN                      PC6
-  #endif
+  #define Z_MIN_PIN        PE1
 #endif
 
 //
 // Limit Switches
 //
-#define X_STOP_PIN                          PA5
-#define Y_STOP_PIN                          PA6
-#define Z_STOP_PIN                          PA7
-
-#ifndef Z_MIN_PROBE_PIN
-  #define Z_MIN_PROBE_PIN                   PB1   // BLTouch IN
-#endif
+#define X_MIN_PIN          PD10
+#define X_MAX_PIN          PE15
+#define Y_MIN_PIN          PE0
+// #define Z_MIN_PIN       PE1
+#define Z2_MIN_PIN         PE2
 
 //
 // Filament Runout Sensor
 //
 #ifndef FIL_RUNOUT_PIN
-  #define FIL_RUNOUT_PIN                    PA4   // "Pulled-high"
+  #define FIL_RUNOUT_PIN                  PA6   // "Pulled-high"
 #endif
 
+#define CHECKFILEMENT0_PIN                PE5
+#define CHECKFILEMENT1_PIN                PE6
+
 //
 // Steppers
 //
+#define X_ENABLE_PIN                        PC7
 #ifndef X_STEP_PIN
-  #define X_STEP_PIN                        PC2
+  #define X_STEP_PIN                        PD15
 #endif
 #ifndef X_DIR_PIN
-  #define X_DIR_PIN                         PB9
+  #define X_DIR_PIN                         PD14
 #endif
-#define X_ENABLE_PIN                        PC3   // Shared
 
+#define X2_ENABLE_PIN                       PE11
+#ifndef X2_STEP_PIN
+  #define X2_STEP_PIN                       PE9
+#endif
+#ifndef X2_DIR_PIN
+  #define X2_DIR_PIN                        PE8
+#endif
+
+#define Y_ENABLE_PIN                        PB9
 #ifndef Y_STEP_PIN
-  #define Y_STEP_PIN                        PB8
+  #define Y_STEP_PIN                        PB7
 #endif
 #ifndef Y_DIR_PIN
-  #define Y_DIR_PIN                         PB7
+  #define Y_DIR_PIN                         PB6
 #endif
-#define Y_ENABLE_PIN                X_ENABLE_PIN
 
+#define Z_ENABLE_PIN                        PB5
 #ifndef Z_STEP_PIN
-  #define Z_STEP_PIN                        PB6
+  #define Z_STEP_PIN                        PB3
 #endif
 #ifndef Z_DIR_PIN
-  #define Z_DIR_PIN                         PB5
+  #define Z_DIR_PIN                         PD7
 #endif
-#define Z_ENABLE_PIN                X_ENABLE_PIN
 
+#define Z2_ENABLE_PIN                       PC5
+#ifndef Z2_STEP_PIN
+  #define Z2_STEP_PIN                       PA7
+#endif
+#ifndef Z2_DIR_PIN
+  #define Z2_DIR_PIN                        PA6
+#endif
+
+#define E0_ENABLE_PIN                       PD4
 #ifndef E0_STEP_PIN
-  #define E0_STEP_PIN                       PB4
+  #define E0_STEP_PIN                       PD1
 #endif
 #ifndef E0_DIR_PIN
-  #define E0_DIR_PIN                        PB3
+  #define E0_DIR_PIN                        PD0
+#endif
+
+#define E1_ENABLE_PIN                       PE7
+#ifndef E1_STEP_PIN
+  #define E1_STEP_PIN                       PB1
+#endif
+#ifndef E1_DIR_PIN
+  #define E1_DIR_PIN                        PB0
 #endif
-#define E0_ENABLE_PIN               X_ENABLE_PIN
 
 //
 // Release PB4 (Y_ENABLE_PIN) from JTAG NRST role
 //
-#define DISABLE_DEBUG
+#define DISABLE_JTAG
 
 //
 // Temperature Sensors
 //
-#define TEMP_0_PIN                          PC5   // TH1
-#define TEMP_BED_PIN                        PC4   // TB1
+#define TEMP_0_PIN                          PA4   // TH0
+#define TEMP_1_PIN                          PA5   // TH1
+#define TEMP_BED_PIN                        PA3   // TB1
 
 //
 // Heaters / Fans
 //
-#define HEATER_0_PIN                        PA1   // HEATER1
+#define HEATER_0_PIN                        PA1   // HEATER0
+#define HEATER_1_PIN                        PA0   // HEATER1
 #define HEATER_BED_PIN                      PA2   // HOT BED
 
-#ifndef FAN_PIN
-  #define FAN_PIN                           PA0   // FAN
-#endif
-#if PIN_EXISTS(FAN)
+#define FAN_PIN                             PB14   // FAN
+#define FAN1_PIN                            PB12   // FAN
 #define FAN_SOFT_PWM
-#endif
 
 //
 // SD Card
 //
-#define SD_DETECT_PIN                       PC7
+#define SD_DETECT_PIN                       PA8
 #define SDCARD_CONNECTION                ONBOARD
 #define ONBOARD_SPI_DEVICE                     1
-#define ONBOARD_SD_CS_PIN                   PA4   // SDSS
+#define ONBOARD_SD_CS_PIN                   PC11   // SDSS
 #define SDIO_SUPPORT
 #define NO_SD_HOST_DRIVE                           // This board's SD is only seen by the printer
 
-#if ENABLED(CR10_STOCKDISPLAY)
+#if ENABLED(CR10_STOCKDISPLAY) && NONE(RET6_12864_LCD, VET6_12864_LCD)
+  #error "Define RET6_12864_LCD or VET6_12864_LCD to select pins for CR10_STOCKDISPLAY with the Creality V4 controller."
+#endif
 
 #if ENABLED(RET6_12864_LCD)
 
@@ -167,9 +203,7 @@
   #define BTN_EN1                           PB10
   #define BTN_EN2                           PB14
 
-    #ifndef HAS_PIN_27_BOARD
   #define BEEPER_PIN                        PC6
-    #endif
 
 #elif ENABLED(VET6_12864_LCD)
 
@@ -182,11 +216,7 @@
   #define BTN_EN1                           PB10
   #define BTN_EN2                           PA6
 
-  #else
-    #error "Define RET6_12864_LCD or VET6_12864_LCD to select pins for CR10_STOCKDISPLAY with the Creality V4 controller."
-  #endif
-
-#elif EITHER(HAS_DWIN_E3V2, IS_DWIN_MARLINUI)
+#elif ENABLED(DWIN_CREALITY_LCD)
 
   // RET6 DWIN ENCODER LCD
   #define BTN_ENC                           PB14
@@ -196,6 +226,7 @@
   //#define LCD_LED_PIN                     PB2
   #ifndef BEEPER_PIN
     #define BEEPER_PIN                      PB13
+    #undef SPEAKER
   #endif
 
 #elif ENABLED(DWIN_VET6_CREALITY_LCD)
```

Touch screen stuff.
- comment out a ui.refresh()
- comment out a TERM_

DUAL_X_CARRIAGE stuff:
- looks like it sets some default on power off or recovery>


```diff
diff --git a/Marlin/src/sd/cardreader.cpp b/Marlin/src/sd/cardreader.cpp
index 9f8aac634b..c8a034dd80 100644
--- a/Marlin/src/sd/cardreader.cpp
+++ b/Marlin/src/sd/cardreader.cpp
@@ -35,6 +35,8 @@
   #include "../lcd/e3v2/creality/dwin.h"
 #elif ENABLED(DWIN_CREALITY_LCD_ENHANCED)
   #include "../lcd/e3v2/enhanced/dwin.h"
+#elif ENABLED(RTS_AVAILABLE)
+  #include "../lcd/e3v2/creality/LCD_RTS.h"
 #endif
 
 #include "../module/planner.h"        // for synchronize
@@ -460,7 +462,7 @@ void CardReader::mount() {
       ui.set_status_P(GET_TEXT(MSG_SD_INIT_FAIL), -1);
   #endif
 
-  ui.refresh();
+  //ui.refresh();
 }
 
 /**
@@ -566,7 +568,7 @@ void CardReader::startOrResumeFilePrinting() {
 //
 void CardReader::endFilePrintNow(TERN_(SD_RESORT, const bool re_sort/*=false*/)) {
   TERN_(ADVANCED_PAUSE_FEATURE, did_pause_print = 0);
-  TERN_(HAS_DWIN_E3V2_BASIC, HMI_flag.print_finish = flag.sdprinting);
+  //TERN_(HAS_DWIN_E3V2_BASIC, HMI_flag.print_finish = flag.sdprinting);
   flag.abort_sd_printing = false;
   if (isFileOpen()) file.close();
   TERN_(SD_RESORT, if (re_sort) presort());
@@ -1287,6 +1289,15 @@ void CardReader::fileHasFinished() {
       startOrResumeFilePrinting();
       return;
     }
+    else
+    {
+      #if ENABLED(DUAL_X_CARRIAGE)
+        extruder_duplication_enabled = false;
+        dual_x_carriage_mode = DEFAULT_DUAL_X_CARRIAGE_MODE;
+        active_extruder = 0;
+      #endif
+      PoweroffContinue = false;
+    }
   #endif
 
   endFilePrintNow(TERN_(SD_RESORT, true));
@@ -1323,6 +1334,7 @@ void CardReader::fileHasFinished() {
     if (jobRecoverFileExists()) {
       recovery.init();
       removeFile(recovery.filename);
+      recovery.info.recovery_flag = false;
       #if ENABLED(DEBUG_POWER_LOSS_RECOVERY)
         SERIAL_ECHOPGM("Power-loss file delete");
         SERIAL_ECHOPGM_P(jobRecoverFileExists() ? PSTR(" failed.\n") : PSTR("d.\n"));
```
They rewrote the README. I'm omitting the diff.

They deleted `ecx.S`.

```diff

diff --git a/buildroot/share/PlatformIO/scripts/exc.S b/buildroot/share/PlatformIO/scripts/exc.S
deleted file mode 100644
index 1db462bb23..0000000000
```

Removed date form firemare PROGNAME for PlatformIO.


```diff
diff --git a/buildroot/share/PlatformIO/scripts/random-bin.py b/buildroot/share/PlatformIO/scripts/random-bin.py
index c03b863448..5d5a648a32 100644
--- a/buildroot/share/PlatformIO/scripts/random-bin.py
+++ b/buildroot/share/PlatformIO/scripts/random-bin.py
@@ -6,4 +6,4 @@ Import("env")
 
 from datetime import datetime
 
-env['PROGNAME'] = datetime.now().strftime("firmware-%Y%m%d-%H%M%S")
+env['PROGNAME'] = datetime.now().strftime("firmware")
```

Delete another asm file. I'm guessing this just aren't used and either cause them problems, or are just new and didn't get `git add`ed on upgrading.

I'm omitting a couple more delete .S files.

```diff
diff --git a/buildroot/share/PlatformIO/variants/marlin_CHITU_F103/wirish/start.S b/buildroot/share/PlatformIO/variants/marlin_CHITU_F103/wirish/start.S
deleted file mode 100755
index 8b181aa993..0000000000
```

features.ini. Looks like they just adde the touch screen.

```diff
diff --git a/ini/features.ini b/ini/features.ini
index f54b645f85..e8f07f6e3d 100644
--- a/ini/features.ini
+++ b/ini/features.ini
@@ -46,6 +46,7 @@ SOFT_I2C_EEPROM                        = SlowSoftI2CMaster, SlowSoftWire=https:/
 SPI_EEPROM                             = src_filter=+<src/HAL/shared/eeprom_if_spi.cpp>
 HAS_DWIN_E3V2|IS_DWIN_MARLINUI         = src_filter=+<src/lcd/e3v2/common>
 DWIN_CREALITY_LCD                      = src_filter=+<src/lcd/e3v2/creality>
+RTS_AVAILABLE                          = src_filter=+<src/lcd/e3v2/creality>
 DWIN_CREALITY_LCD_ENHANCED             = src_filter=+<src/lcd/e3v2/enhanced>
 DWIN_CREALITY_LCD_JYERSUI              = src_filter=+<src/lcd/e3v2/jyersui>
 IS_DWIN_MARLINUI                       = src_filter=+<src/lcd/e3v2/marlinui>
```

slight upgrade to something related to the board.

```diff
diff --git a/ini/stm32f1-maple.ini b/ini/stm32f1-maple.ini
index 00ba93aa63..d764c138fe 100644
--- a/ini/stm32f1-maple.ini
+++ b/ini/stm32f1-maple.ini
@@ -23,7 +23,7 @@
 # HAL/STM32F1 Common Environment values
 #
 [common_stm32f1]
-platform          = ststm32@~12.1
+platform          = ststm32@~12.1.1
 board_build.core  = maple
 build_flags       = !python Marlin/src/HAL/STM32F1/build_flags.py
   ${common.build_flags} -DARDUINO_ARCH_STM32 -DMAPLE_STM32F1
```

more board configuration, changed the variant from RE to VE

```diff
diff --git a/ini/stm32f1.ini b/ini/stm32f1.ini
index da7cedd3a3..1b9d4ff821 100644
--- a/ini/stm32f1.ini
+++ b/ini/stm32f1.ini
@@ -114,14 +114,13 @@ debug_tool                  = stlink
 # Creality (STM32F103RET6)
 #
 [env:STM32F103RET6_creality]
-platform                    = ${common_stm32.platform}
+platform                    = ${common_stm32f1.platform}
 extends                     = stm32_variant
-board                       = genericSTM32F103RE
-board_build.variant         = MARLIN_F103Rx
+board                       = genericSTM32F103VE
+board_build.variant         = MARLIN_F103Vx
 board_build.offset          = 0x7000
 board_upload.offset_address = 0x08007000
 build_flags                 = ${stm32_variant.build_flags}
-                              -DMCU_STM32F103RE -DHAL_SD_MODULE_ENABLED
                               -DSS_TIMER=4 -DTIMER_SERVO=TIM5
                               -DENABLE_HWSERIAL3 -DTRANSFER_CLOCK_DIV=8
 build_unflags               = ${stm32_variant.build_unflags}
```

Set the default for platformio.ini to the board they use.

```diff
diff --git a/platformio.ini b/platformio.ini
index 106e454d10..0e75a369c4 100644
--- a/platformio.ini
+++ b/platformio.ini
@@ -13,7 +13,7 @@
 [platformio]
 src_dir      = Marlin
 boards_dir   = buildroot/share/PlatformIO/boards
-default_envs = mega2560
+default_envs = STM32F103RET6_creality
 include_dir  = Marlin
 extra_configs =
     ini/avr.ini
```
