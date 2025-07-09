A leap second is a one-second adjustment applied to Coordinated Universal Time (UTC), 
Leap seconds are decided by the International Earth Rotation and Reference Systems Service (IERS), about six months in advance, to keep UTC within 0.9 seconds of the UT1 variant of universal time.
Leap seconds can be applied at the end of any month.  They are announced in IERS's Bulletin C, every 6 months.
A leap second can be "positive", adding a 61th second to the last minute of the month, or "negative", making the last minute of the month 59 seconds long.  So far, only "positive"
leap seconds have been announced.  
The most consistent list of leap seconds is found at the Internet Assigned Numbers Authority (IANA), at [https://data.iana.org/time-zones/data/leap-seconds.list](https://data.iana.org/time-zones/data/leap-seconds.list).
Leap seconds are expressed there as pairs of NTP time and DTAI, the first in seconds since 1900 and the second is (TAI-UTC), where TAI is the International Atomic Time, which is not corrected for leap seconds.  Below is
the line for the latest leap second, just before 1 January 2017:
```
3692217600      37      # 1 Jan 2017
```
This conversion from NTP to UTC can be done using standard UTC libraries, for example:

```python
>>> from obspy.core import UTCDateTime
>>> UTCDateTime(1900,1,1)+3692217600
UTCDateTime(2017, 1, 1, 0, 0)
```

OBS dataloggers do not receive leap-second information during deployment, and most are not built to integrate leap seconds, even if they had prior knowledge.
Timing from OBS dataloggers covering periods containing leapseconds must therefore be post-processed to include the leap-second.

# Inserting a positive leapsecond using msmod

Currently, inserting a positive leapsecond into a miniSEED file using `msmod` involves subtracting one second from all records after the record containg the leapsecond, and setting the "positive leapsecond" flag in the record containing the leap second:


```
msmod --timeshift -1 -ts 2017,01,00,00,00.999999
msmod –-actflags ‘4,1’ –tsc 2017,001,00,00,00.999999 –tec 2017,001,00,00,00.999999
```

It would be simpler and less prone to errors to simply specify a positive leap second at a given date and let msmod take care of the details:

```
msmod -lsp 2017,001,00,00,00
```

or

```
msmod -lsp 3692217600
```

The former requires translating the Month-Day in the leapseconds.list file into a day of year.
The latter eliminates interpretation of the NTP time value, but may allow for more copying error (an error check could be to make sure that it corresponds to the first second of a new month)

# Inserting a negative leapsecond using msmod

Currently,  a negative leapsecond can be inserted into a miniSEED file as follows:

```
msmod --timeshift +1 -ts 2016,366,23,59,58.999999
msmod –-actflags ‘5,1’ –tsc 2016,366,23,59,58.999999 –tec 2016,366,23,59,58.999999
```

This could be replaced by:

```
msmod -lsn 2017,001,00,00,00
```

or

```
msmod -lsn 3692217600
```


# Other comments/modifications
The flags ``-tsc`` and ``-tec`` were initially added to ``msmod`` to allow one to specify a record containing a time value, specifically for the leap second correction.  If the leap second is handled otherwise,
these flags might be able to disappear, and/or a flag ``-ts`` could select the single record containing the given time.
