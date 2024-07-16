

# Project for clock correction in msmod

We would like to add piecewise-linear clock correction as an option to `msmod`.  Data providers could use this to correct instrument times with a known drift, as is the case for ocean-bottom seismometers where the instrument-reference offset is measured before and after deployment.

## Command line addition

```
--cc FILENAME      # Piecewise clock correction. 
                   # Columns are: reference time, instrument time
```
  
## File format

- Comment/header lines, starting with '#'
- Interpolation type ('linear' or 'cubic')
- Timing lines.
    -  At least two
    -  Two columns of libmseed-readable times, ending in 'Z'
	-  Each column must be monotonically increasing.

Example:

```
interpolation: cubic
# Reference time         Instrument time
2023-01-01T10:01:00Z     2023-01-01T10:01:00.001Z
2023-02-01T00:00:00Z     2023-02-01T00:00:00.012Z
2023-03-01T00:00:00Z     2023-03-01T00:00:00.503Z
2023-04-01T13:12:00Z     2023-04-01T13:12:01.234Z
```

## Algorithm

- Verify that all data times are included in the "Instrument time" bounds
- For each record in the input file
	- Calculate the time correction neeeded, by linear interpolation between the surrounding "Instrument time" lines
	- Apply this to the Record Start Time field
	- Put this value in the Time Correction field
	- Set Activity Flag "Time Correction Applied" Bit

## Errors:


No  | Problem                         | Action
--- | ------------------------------- | --------------------------------
1   | Badly formatted input file      | ERROR with first bad line #
2a  | Non-increasing reference times  | ERROR with first bad line #
2b  | Non-increasing instrument times | ERROR with first bad line #
3a  | Data starts before first instrument time | ERROR with suggested modifcations
3b  | Data ends after last instrument time | ERROR with suggested modifications
4   | Time Correction or Time Correction Applied Field already set in data | ERROR with first record # (or time)

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

## QUESTIONS
- Should the order be reference, then instrument (current case) or vice versa?
  In principal, we should always have an instrumen time, but not necessarily a reference time
  (in the case of no synchronization): it might be prettier/clearer to have
  instrument time in the first column
