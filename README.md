An ESPHome script to record the speed data from the LDTR20 24mm radar sensor. 

These, according to the blurb, are commonly used in speed warning signs and out of the box only record inbound speeds.

These devices can operate on 12 volts and datasheets are difficult to find. 

However using the datasheet for LDTR04 I discovered you can write to the device and tell it to provide bi-directional data.

The code creates an esphome device with two sensors (outbound and inbound). It records the peak speed for each direction in 30 second intervals.

