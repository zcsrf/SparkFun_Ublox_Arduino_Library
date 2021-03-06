Forked from SparkFun u-blox Arduino Library
===========================================================

U-blox makes some incredible GPS receivers covering everything from low-cost, highly configurable modules such as the SAM-M8Q all the way up to the surveyor grade ZED-F9P with precision of the diameter of a dime. This library focuses on configuration and control of u-blox devices over I2C (called DDC by u-blox) and Serial. The UBX protocol is supported over both I2C and serial, and is a much easier and lighterweight interface to a GPS module. Stop parsing NMEA data! And simply ask for the datums you need.

Max (400kHz) I2C Support
-------------------

To achieve 400kHz I2C speed please be sure to remove all pull-ups on the I2C bus. Most, if not all, u-blox modules include pull ups on the I2C lines (sometimes called DDC in their manuals). Cut all I2C pull up jumpers and/or remove them from peripheral boards. Otherwise, various data glitches can occur. See issues [38](https://github.com/sparkfun/SparkFun_Ublox_Arduino_Library/issues/38) and [40](https://github.com/sparkfun/SparkFun_Ublox_Arduino_Library/issues/40) for more information. If possible, run the I2C bus at 100kHz.

-------------------

Thanks to:

* [trycoon](https://github.com/sparkfun/SparkFun_Ublox_Arduino_Library/pull/7) for fixing the lack of I2C buffer length defines.
* [tve](https://github.com/tve) for building out serial additions and examples.
* [Redstoned](https://github.com/Redstoned) and [davidallenmann](https://github.com/davidallenmann) for adding PVT date and time.
* [wittend](https://forum.sparkfun.com/viewtopic.php?t=49874) for pointing out the RTCM print bug.
* Big thanks to [PaulZC](https://github.com/PaulZC) for implementing the combined key ValSet method, geofence functions, better saveConfig handling, as well as a bunch of small fixes.
* [RollieRowland](https://github.com/RollieRowland) for adding HPPOSLLH (High Precision Geodetic Position).
* [tedder](https://github.com/tedder) for moving iTOW to PVT instead of HPPOS and comment cleanup.
* [grexjmo](https://github.com/grexjmo) for pushing for a better NMEA sentence configuration method.
* [averywallis](https://github.com/averywallis) for adding good comments to the various constants.
* [blazczak](https://github.com/blazczak) and [geeksville](https://github.com/geeksville) for adding support for the series 6 and 7 modules.
* [bjorn@unsurv](https://github.com/unsurv) for adding powerOff and powerOffWithInterrupt.

Repository Contents
-------------------

* **/examples** - Example sketches for the library (.ino). Run these from the Arduino IDE.
* **/src** - Source files for the library (.cpp, .h).
* **keywords.txt** - Keywords from this library that will be highlighted in the Arduino IDE.
* **library.properties** - General library properties for the Arduino package manager.

Documentation
--------------

* **[Installing an Arduino Library Guide](https://learn.sparkfun.com/tutorials/installing-an-arduino-library)** - Basic information on how to install an Arduino library.

Polling vs. Auto-Reporting
--------------------------

This library supports two modes of operation for getting navigation information with the `getPVT`
function (based on the `UBX_NAV_PVT` protocol packet): polling and auto-reporting.

The standard method is for the sketch to call `getPVT` (or one of the `getLatitude`, `getLongitude`,
etc. methods) when it needs a fresh navigation solution. At that point the library sends a request
to the GPS to produce a fresh solution. The GPS then waits until the next measurement occurs (e.g.
once per second or as set using `setNavigationFrequency`) and then sends the fresh data.
The advantage of this method is that the data received is always fresh, the downside is that getPVT
can block until the next measurement is made by the GPS, e.g. up to 1 second if the nav frequency is
set to one second.

An alternate method can be chosen using `setAutoPVT(true)` which instructs the GPS to send the
navigation information (`UBX_NAV_PVT` packet) as soon as it is produced. This is the way the older
NMEA navigation data has been used for years. The sketch continues to call `getPVT` as before but
under the hood the library returns the data of the last solution received from the GPS, which may be
a bit out of date (how much depends on the `setNavigationFrequency` value).

The advantage of this method is that getPVT does not block: it returns true if new data is available
and false otherwise. The disadvantages are that the data may be a bit old and that buffering for
these spontaneus `UBX_NAV_PVT` packets is required (100 bytes each). When using Serial the buffering
is an issue because the std serial buffer is 32 or 64 bytes long depending on Arduino version. When
using I2C the buffering is not an issue because the GPS device has at least 1KB of internal buffering
(possibly as large as 4KB).

As an example, assume that the GPS is set to produce 5 navigation
solutions per second and that the sketch only calls getPVT once a second, then the GPS will queue 5
packets in its internal buffer (about 500 bytes) and the library will read those when getPVT is
called, update its internal copy of the nav data 5 times, and return `true` to the sketch. The
sketch calls `getLatitude`, etc. and retrieve the data of the most recent of those 5 packets.

License Information
-------------------

This product is _**open source**_!

Various bits of the code have different licenses applied. Anything SparkFun wrote is beerware; if you see me (or any other SparkFun employee) at the local, and you've found our code helpful, please buy us a round!

Please use, reuse, and modify these files as you see fit. Please maintain attribution to SparkFun Electronics and release anything derivative under the same license.

Distributed as-is; no warranty is given.

- Your friends at SparkFun.
