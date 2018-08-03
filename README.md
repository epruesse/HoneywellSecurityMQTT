# HoneywellSecurityMQTT

This project is based on jhaines0's HoneywellSecurity project but instead of being intended for integration with SmartThings, it is designed to report status via MQTT.  This allows it to easily be used with other systems such as Home Assistant.


## Features
 - Decodes data from sensors based on Honeywell's 345MHz system.  This includes rebrands such as 2GIG, Vivint, etc.
 - Requires no per-sensor configuration
 - Decodes sensor status such as tamper and low battery
 - Reports alarm and sensor status to an MQTT broker
 - Watchdog with reporting in case of receiver failure
 - Checks for sensors failing to report in


## Requirements
 - RTL-SDR USB adapter; commonly available on Amazon
 - rtlsdr library
 - mosquittopp library
 - gcc

## Installation
### Dependencies
On a Debian-based system, something like this should work:
```
  sudo apt-get install build-essential librtlsdr-dev rtl-sdr libmosquittopp-dev
```

To avoid having to run as root, you can add the following rule to a file in `/etc/udev/rules.d`:
```
  SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="2838", MODE:="0660", GROUP:="audio"
```

Then add the desired user to the `audio` group.
If you plugged in the RTL-SDR before installing rtl-sdr, you probably will need to do something like `sudo rmmod rtl2832 dvb_usb_rtl28xxu` then remove and reinstall the adapter.

### Configuration
Modify `mqtt_config.h` to specify the host, port, username, and password of your MQTT broker.  If `""` is used for the username or password, then an anonymous login is attempted.

### Building
```
  cd src
  ./build.sh
```

### Running
  `./honeywell`

### Home Assistant example
```yaml

sensor:
  - platform: mqtt
    name: 345MHz RX Fault
    state_topic: "/security/sensors345/rx_status"
    payload_on: "FAILED"
    payload_off: "OK"
    device_class: safety
  - platform: mqtt
    name: Front Door Status
    state_topic: "/security/sensors345/732804/status"
binary_sensor:
  - platform: mqtt
    name: Front Door
    state_topic: "/security/sensors345/732804/alarm"
    payload_on: "ALARM"
    payload_off: "OK"
    device_class: opening

```

## Notes
 - The alarm loop bit will vary depending on sensor type and installation.  To handle this without requiring manual configuration HoneywellSecurityMQTT will need to receive at least one non-triggered packet from each sensor.  This can be accomplished by letting it run for at least ~90 minutes to allow pulse checks to arrive from each sensor.  This does mean that if the first packet received is due to an event (e.g. open door), it will not be detected.  Subsequent detections will work, however.  A future improvement is to save the learned devices and their behavior so that future executions of the program will not require this learning period.

## Future Work
 - Cache learned sensor data
 - Fix broker indication of unexpected termination
