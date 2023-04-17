# ext_2G4_channel_multiatt

This is a channel model for ext_2G4_phy_v1

This is a non realistic channel model.
It models NxN independent paths each with a configurable attenuation
This attenuation can also be configured to change over time.

The default atennuation for all paths is set with the command line option
`-at=<attenuation>`.

A file can be used to set the attenuation for some/all of the individual paths.
This file is specified with the command line option `-file=<att_matrix_file>`

This file shall have one separate line per path like:<br>
    `x y : {value|"<timed_attenuation_file>"}`<br>

Where:

* x is the transmitter number, x = 0..N-1
* y is the receiver number     y != x
* N being the number of devices in the simulation
* value is a floating point value in dB
* `<timed_attenuation_file>` : is the path to a `<timed_attenuation_file>` as
  described below which applies to that path.
  Note that this file name shall be provided in between `""`
* Not all paths need to be provided. Those omited will default to the attenuation
  provided from the command line.

The file names can be either absolute or relative to `bin/`

For both files, `#` is treated as a comment mark (anything after a `#` is
discarded)
Empty lines are ignored

Note that the Phy assumes the channel to be pseudo-stationary, that is, that the
channel propagation conditions are static enough during a packet reception, that
is, that the coherence time of the channel is much bigger than the typical
packet length.
Therefore the channel model won't be called to reevaluate conditions during a
packet unless the devices which are transmitting change.


### The `<timed_attenuation_file>`:

This file contains 2 columns, the first column represents time in
microseconds (an integer value), and the second column the attenuation at that given time.
The times shall be in ascending order.<br>
If the simulation time is found in the file, the channel will use the
corresponding attenuation value.<br>
If the simulation time falls in between 2 times in the file, the channel
will interpolate linearly the attenuation.<br>
If the simulation time is before the first time value in the file, the
channel will use the first atttenuation provided in the file.<br>
If the simulation time is after the last time value in the file, the channel
will use the last attenuation provided in the file.<br>

### The `-atextra=<extra_attenuation>` command line option:

You can provide an extra attenuation to be added to all paths, independently of
how their attenuation was originally calculated, with the `-atextra` command line
option.
This `extra_attenuation` value will just be added to all paths.

### An example:

`./bs_2G4_phy_v1 -D=3 -s=Hola -channel=multiatt -argschannel -at=40 -file=paths_att_file.txt`

`paths_att_file.txt`:
```
0 1 : 65
1 0 : 65
0 2 : "/myfolder/att_file.txt"
2 0 : "/myfolder/att_file.txt"
#Note that the paths from 1<->2 are not specified. They will default to the -at command line parameter
```

`att_file.txt`:
```
1000000    60
11000000  120
11000001   60
```

With this configuration, the radio traffic between device:

* device 0 and device 1 will have an attenuation of 65dB
* device 0 and device 2 will have an attenuation of
    * 60dB up to the first second
    * An increasing attenuation between 60->120 dB, between the first second and the 11th. At a rate of 6dB per second
    * A sudden reduction in the attenuation to 60 dB right after, which is maintained thereafter.
