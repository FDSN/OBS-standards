

# Project for clock correction in msmod

We would like to add clock correction as an `msmod` option.  Data providers could use this to correct instrument times with a known drift, as is the case for ocean-bottom seismometers where the instrument-reference offset is measured before and after deployment.

## Command line  help

with `-h`

```
--cc CCFILENAME      # Clock correction parameters.  Type '-H' for details
```

with `-H`, same as above plus, at the end of the help:
```
The clock correction file format is (do not indent):
  type: {keyword} {parameters}
  # Reference Time   Instrument Time
  {reference_time_0} {instrument_time_0}
  {reference_time_1} {instrument_time_1}
  ...

There must be at least 2 time lines: there is no upper limit.
The times in each column must be monotonically increasing and the range
of {instrument_time}s must cover the time range in the input miniSEED file.
Time format is yyyy-mm-ddTHH:MM:SS(.FFFFF)Z
The comment line (starting with '#' in column 1) is optional and has no effect on processing
Other comment lines, starting with '#' are allowed
Possible {keyword} {parameters} are:
   piecewise_linear (no parameters): linear interpolation between each time line
   cubic_spline (no parameters): cubic spline interpolation between each time line
   polynomial a0 a1 a2 a3...: corrected_time = instrument_time + a0 + a1*dT + a2*dT^2 + ...
                              where dTime = instrument_time - instrument_time_0
                              and time line values are used to validate corrected_times
An output named {CCFILENAME}.log contains

```
  
## File format example

```
type: cubic_spline
# Reference time         Instrument time
2023-01-01T10:01:00Z     2023-01-01T10:01:00.001Z
2023-02-01T00:00:00Z     2023-02-01T00:00:00.012Z
2023-03-01T00:00:00Z     2023-03-01T00:00:00.503Z
2023-04-01T13:12:00Z     2023-04-01T13:12:01.234Z
```

## Algorithm

- Verify that all data times are included in the "Instrument time" bounds
- Verify that each time column is monotonically increasing
- If polynomial correction, verify that it produces the indicated "reference_times"
  when applied to the corresponding "instrument_times"
- For each record in the input file
	- Calculate the time correction neeeded, by the selected method
	- Apply this to the Record Start Time field
	- Put this value in the Time Correction field
	- Set Activity Flag "Time Correction Applied" Bit

### piecewise_linear algorithm
corrected_time = instrument_time + dT*(reference_time[n+1]-reference_times[n])/(instrument_times[n+1]-instrument_times[n])

where dT = (instrument_time - instrument_times[n])
where n is chosen such that instrument_times[n]<=instrument_time <= instrument_times[n+1].
This leaves an ambiguity at instrument_time = instrument_times[k], choose n=k if k=0, n=k-1 otherwise

### cubic spline algorithm
corrected_time = instrument_time + CubicSpline(reference_times-instrument_times,
                                               instrument_times-instrument_times[0],
					       bc_type='natural')(dT)
(using SciPy's CubicSpline algorithm, find equivalent in your language)

### polynomial algorithm
corrected_time = instrument_time + a0 + a1*dT + a2*dT^2 + a3*dT^3...


## Warnings:

No  | Problem                         | Action
--- | ------------------------------- | --------------------------------
1   | A More than 0.5-sample change in the offset between two records | WARNING with record number and time information

## Errors:

No  | Problem                         | Action
--- | ------------------------------- | --------------------------------
1   | Badly formatted input file      | ERROR with first bad line #
2a  | Non-increasing reference times  | ERROR with first bad line #
2b  | Non-increasing instrument times | ERROR with first bad line #
3a  | Data starts before first instrument time | ERROR with suggested modifications
3b  | Data ends after last instrument time | ERROR with suggested modifications
4   | Time Correction or Time Correction Applied Field already set in data | ERROR with first record # (or time)
5   | Log file exists                 | ERROR with "log file {name} exists"
6   | Polynomial output does not match a time line | ERROR with suggested message

Flagging part "a" does not prevent checking part "b"

### Suggested error messages

#### Error 3a:

```
Data starts before first instrument time (by 3600 seconds).
To correct, assuming the same drift as the first segment, prepend:
   2023-01-01T09:01:00Z     2023-01-01T09:01:00.001Z
To correct, assuming no drift until the first segment, prepend:
   2023-01-01T09:01:00Z     2023-01-01T09:01:00.001Z
```
#### Error 3b:

```
Data ends after first instrument time (by 86400 seconds).
To correct, assuming the same drift as the last segment, append:
   2023-04-02T13:12:00Z     2023-04-02T13:12:01.257Z
To correct, assuming no drift after the last segment, append:
   2023-04-02T13:12:00Z     2023-04-02T13:12:01.234Z
```

#### Error 6:

```
Polynomial does not generate reference corrected times:
REFERENCE_TIME |  INSTRUMENT_TIME | CORRECTED_TIME  | ERROR (s)
-------------- | ---------------- | --------------- | -------------
yyyy-mm-ddTHH:MM:SS.FFFF | yyyy-mm-ddTHH:MM:SS.FFFF | yyyy-mm-ddTHH:MM:SS.FFFF | X.XXXX
```

# TEST FILES
We have provided:
- a miniSEED file, with one sample per second from 2022-01-01 to
2023-01-01
- 3 clock correction files, one for each type
- 3 "log" files.

`msmod infile -cc {CLOCK_CORRECTION_FILE}` should produce the same log files (and implicit miniseed file) as the provided log files.
