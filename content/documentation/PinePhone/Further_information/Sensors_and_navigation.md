---
title: "Sensors and navigation"
draft: false
menu:
  docs:
    title:
    parent: "PinePhone/Further_information"
    identifier: "PinePhone/Further_information/Sensors_and_navigation"
    weight:
aliases:
  - /wiki/PinePhone_Sensors_and_Navigation
---

{{% docs/construction %}}

## Overview

The PinePhone contains various components that enable or aid orientation and navigation tasks.

* Multi-GNSS Receiver as part of the cellular modem (GPS/Galileo/GLONASS/BeiDou)
* IMU (Inertial Measurement Unit), combined Accelerometer and Gyroscope sensor (MPU6050)
* Magnetometer (LIS3MDL or AF8133J)

{{< figure src="/documentation/images/Pp_sensors_block.png" caption="PinePhone navigation sensors" width="500" >}}

## Hardware

### Quectel EG25G Modem / GNSS Receiver

### Invensense/TDK MPU6050 IMU

Key features:

* Digital-output X-, Y-, and Z-Axis angular rate sensors (gyroscopes) with selectable range of ±250, ±500, ±1000, and ±2000°/sec
* Digital-output triple-axis accelerometer with a programmable full scale range of ±2g, ±4g, ±8g and ±16g
* 400 kHz Fast Mode I2C for communicating with all registers

### ST LIS3MDL Magnetometer/Compass

Key features:

* ±4/±8/±12/±16 gauss selectable magnetic full scale range
* 16-bit data output
* ...

### VTC AF8133J Magnetometer/Compass

Key features:

* ±12/±22 gauss selectable magnetic full scale range
* 400Hz max. update rate
* 16-bit data output

### Sensor reference frames

The sensors are mounted on the PinePhone’s mainboard in different positions and orientations. For the purpose of orientation and navigation it is important to know the relationship between the sensor coordinate frames and the body frame of the phone.

{{< figure src="/documentation/images/pp_coordinates.png" caption="PinePhone sensor coordinates systems (red) and body coordinate system (cyan)" width="500" >}}

As a basis for software development, a common body coordinate system has to be established along with it’s relation to a world coordinate system. "iio-sensor-proxy expects the (sensor) readings to match those defined in the Linux IIO documentation, the Windows Integrating Motion and Orientation Sensors whitepaper, and the Android sensor documentation"[ref]. To transform the sensor coordinate systems into body coordinates, 'mount matrices' have to be defined.

TODO: explain body reference choices, north-east-down world coordinates

## Software

### Drivers

TODO: i2cdev, linux-iio, support matrix

* EG25G: provides either UART or USB based interfaces for NMEA data
  * /dev/ttyUSB2: AT command interface
  * /dev/ttyUSB1: default NMEA data output
* MPU6050:inv_mpu6050, inv_mpu6050_i2c, industrialio

```
iio:device2: mpu6050 (buffer capable)
	9 channels found:
		accel_x:  (input, index: 0, format: be:S16/16>>0)
		6 channel-specific attributes found:
			attr  0: calibbias value: -2102
			attr  1: matrix value: 0, 0, 0; 0, 0, 0; 0, 0, 0
			attr  2: mount_matrix value: 0, 1, 0; -1, 0, 0; 0, 0, -1
			attr  3: raw value: 912
			attr  4: scale value: 0.000598
			attr  5: scale_available value: 0.000598 0.001196 0.002392 0.004785
		accel_y:  (input, index: 1, format: be:S16/16>>0)
		6 channel-specific attributes found:
			attr  0: calibbias value: 941
			attr  1: matrix value: 0, 0, 0; 0, 0, 0; 0, 0, 0
			attr  2: mount_matrix value: 0, 1, 0; -1, 0, 0; 0, 0, -1
			attr  3: raw value: 516
			attr  4: scale value: 0.000598
			attr  5: scale_available value: 0.000598 0.001196 0.002392 0.004785
		accel_z:  (input, index: 2, format: be:S16/16>>0)
		6 channel-specific attributes found:
			attr  0: calibbias value: 1242
			attr  1: matrix value: 0, 0, 0; 0, 0, 0; 0, 0, 0
			attr  2: mount_matrix value: 0, 1, 0; -1, 0, 0; 0, 0, -1
			attr  3: raw value: 15860
			attr  4: scale value: 0.000598
			attr  5: scale_available value: 0.000598 0.001196 0.002392 0.004785
		temp:  (input)
		3 channel-specific attributes found:
			attr  0: offset value: 12420
			attr  1: raw value: -1073
			attr  2: scale value: 2.941176
		anglvel_x:  (input, index: 4, format: be:S16/16>>0)
		5 channel-specific attributes found:
			attr  0: calibbias value: 0
			attr  1: mount_matrix value: 0, 1, 0; -1, 0, 0; 0, 0, -1
			attr  2: raw value: -32
			attr  3: scale value: 0.001064724
			attr  4: scale_available value: 0.000133090 0.000266181 0.000532362 0.001064724
		anglvel_y:  (input, index: 5, format: be:S16/16>>0)
		5 channel-specific attributes found:
			attr  0: calibbias value: 0
			attr  1: mount_matrix value: 0, 1, 0; -1, 0, 0; 0, 0, -1
			attr  2: raw value: 4
			attr  3: scale value: 0.001064724
			attr  4: scale_available value: 0.000133090 0.000266181 0.000532362 0.001064724
		anglvel_z:  (input, index: 6, format: be:S16/16>>0)
		5 channel-specific attributes found:
			attr  0: calibbias value: 0
			attr  1: mount_matrix value: 0, 1, 0; -1, 0, 0; 0, 0, -1
			attr  2: raw value: 0
			attr  3: scale value: 0.001064724
			attr  4: scale_available value: 0.000133090 0.000266181 0.000532362 0.001064724
		timestamp:  (input, index: 7, format: le:S64/64>>0)
		gyro:  (input, WARN:iio_channel_get_type()=UNKNOWN)
		1 channel-specific attributes found:
			attr  0: matrix value: 0, 0, 0; 0, 0, 0; 0, 0, 0
	3 device-specific attributes found:
			attr  0: current_timestamp_clock value: realtime
			attr  1: sampling_frequency value: 50
			attr  2: sampling_frequency_available value: 10 20 50 100 200 500
	2 buffer-specific attributes found:
			attr  0: data_available value: 0
			attr  1: watermark value: 1
	Current trigger: trigger1(mpu6050-dev2)
```
* LIS3MDL: st_sensors, st_sensors_i2c, st_magn, st_magn_i2c, industrialio

```
iio:device1: lis3mdl (buffer capable)
	4 channels found:
		magn_x:  (input, index: 0, format: le:S16/16>>0)
		3 channel-specific attributes found:
			attr  0: raw value: 6766
			attr  1: scale value: 0.000146
			attr  2: scale_available value: 0.000146 0.000292 0.000438 0.000584
		magn_y:  (input, index: 1, format: le:S16/16>>0)
		3 channel-specific attributes found:
			attr  0: raw value: -2046
			attr  1: scale value: 0.000146
			attr  2: scale_available value: 0.000146 0.000292 0.000438 0.000584
		magn_z:  (input, index: 2, format: le:S16/16>>0)
		3 channel-specific attributes found:
			attr  0: raw value: 12726
			attr  1: scale value: 0.000146
			attr  2: scale_available value: 0.000146 0.000292 0.000438 0.000584
		timestamp:  (input, index: 3, format: le:S64/64>>0)
	3 device-specific attributes found:
			attr  0: current_timestamp_clock value: realtime
			attr  1: sampling_frequency value: 1
			attr  2: sampling_frequency_available value: 1 2 3 5 10 20 40 80
	2 buffer-specific attributes found:
			attr  0: data_available value: 0
			attr  1: watermark value: 1
	Current trigger: trigger0(lis3mdl-trigger)
```

### Userspace services

TODO: ...

* Modem Manager
* ofono
* gpsd
* iio-sensor-proxy
* geoclue

### Tools/Utilities

TODO: minicom, gpsmon, monitor-sensor, iio-utils...

### Applications

TODO: pure-maps, navit, ...

## Development

TODO: Open issues, tasks, projects

### Open issues

* AGPS integration
  * how to enable/disable the service?
  * update triggers?
    * network connectivity?
    * agps data valid?
* Raw magnetometer handling
  * Issue: Support polling/non-compensated magnetometers in iio-sensor-proxy
    * https://gitlab.freedesktop.org/hadess/iio-sensor-proxy/-/issues/310
    * https://gitlab.freedesktop.org/hadess/iio-sensor-proxy/-/merge_requests/316 (needs more work)
  * Issue: raw magnetometer calibration implementation
    * https://appelsiini.net/2018/calibrate-magnetometer/
* GPS/Compass App
  * TODO: specification

### Frameworks/APIs/...

* Modem Manager location API
  * https://www.freedesktop.org/software/ModemManager/api/latest/gdbus-org.freedesktop.ModemManager1.Modem.Location.html
* oFono location API (experimental?)
  * https://git.kernel.org/pub/scm/network/ofono/ofono.git/tree/doc/location-reporting-api.txt
* geoclue API
  * https://www.freedesktop.org/software/geoclue/docs/gdbus-org.freedesktop.GeoClue2.Location.html
  * https://www.freedesktop.org/software/geoclue/docs/libgeoclue/GClueLocation.html
