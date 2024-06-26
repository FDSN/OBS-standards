# Marine seismology data/metadata standards


## obs-specific information

The StationXML format (v1.2) has no intrinsic elements for storing OBS-specific
information.  This information should be stored in a structured metadata file
with a public-published format.

We propose to use obsinfo ["subnetwork" files](https://obsinfo.readthedocs.io/en/latest/subnetwork.html),
which allow the storage of essential information about deployments without
going into instrument-level details.

These files can be used with obsinfo instrumentation files to generate [FDSN StationXML](http://docs.fdsn.org/projects/stationxml/en/latest/index.html)
files with embedded OBS-specific information, or they can be used to fill in information
in StationXML files generated using other tools.

## processing steps
**[REC {/}]** Processing done on data files (from data download to delivery to the data center)
should be recorded in text-based, structured files.
The JSON process-steps format (LINK) is an example.

## Source Identifiers
**[FDSN]** The following source-subsource codes (see [FDSN Source Identifiers documentation](http://docs.fdsn.org/projects/source-identifiers/en/v1.0/index.html))
should be used for the following types of sensor/data:

code | description
---- | ------------------------------------------------------------
1    | Unoriented seismometer, “N” channel equivalent
2    | Unoriented seismometer “E” channel (+90 degrees from "1")
3    | Seismometer/geophone with inverted vertical channel (positive voltage is down)
DH   | Hydrophone
DG   | Differential pressure gauge
DO   | "Absolute” bottom pressure recorder

## Station names for repeated deployments
**[REC {/}]** If OBSs are deployed repeatedly at one site (to make a long series), use an
incrementing alphanumeric character at the end of the station name (i.e., A01A,
then A01B then A01C for subsequent deployments at the same approximate location).
This may be a *de facto* "standard", but I haven't seen it written down


## Metadata (StationXML)
See StationXML Reference for details of StationXML elements 

### Clock drift

**[REC {/}]** Should be specified using absolute datetimes, to avoid ambiguity.  Possible structures are:

```yaml
time_base: 'Seascan MCXO, 1e-8 nominal drift'
reference: 'GPS'
start_sync_reference: '2015-04-22T09:21:00Z'
start_sync_instrument: '2015-04-22T09:21:00Z'
end_sync_reference: '2016-05-28T22:59:00.1843Z'
end_sync_instrument: '2016-05-28T22:59:02Z'
```

the `time_base` and `reference` fields are optional.

or, for more flexibility :

```yaml
interpolation: 'linear'  # 'linear' or 'cubic'
syncs_reference:
    - '2015-04-22T09:21:00Z'
    - '2016-05-28T22:59:00.1843Z'
syncs_instrument:
    - '2015-04-22T09:21:00Z'
    - '2016-05-28T22:59:02Z'
```

This is embedded in the StationXML file as a JSON-coded string in a `<Comment>` field
In the future, a seperate namespace may be created to allow a more specific and structured representation

Below is an example of the first proposition in a `<Comment>` field:

```xml
<Comment subject=”Clock Drift”>
<Value>“{Linear Clock Correction: {time_base: Seascan MCXO, 1e-8 nominal
drift, reference: GPS, start_sync_reference: 2015-04- 22T09:21:00Z,start_sync_instrument: 0, end_sync_reference: 2016-05- 28T22:59:00.1843Z,end_sync_instrument: 2016-05-28T22:59:02Z}}”</Value> </Comment>
```

If there is assumed to be clock drift but some or all of values were not measured, each missing value should be represented by 'None'

### Leap seconds

Structure is:
```yaml
time: 2016-082T23:59:60Z
type: +
description: Positive leap-second (a 61-second minute)
correction_data:
    - msmod --timeshift -1 -ts 2016,182,23:59:59.999999
    - msmod --actflags ‘4,1’ –tsc 2016,182,23:59:59.999999 –tec 2016,182,23:59:59.999999
correction_end_sync_instrument: subtracted one second from displayed instrument time
```

only `time` and `type` are required

**[REC {/}]** Embedded in a StationXML `<Comment>`.  Possible future namespace element, as for clock drift.  Below is a StationXML example

```xml
<Comment subject=”Leap Second”>
<Value>“{time: 2016-082T23:59:60Z, type: '+', description: 'Positive leap-second (a
61-second minute)', correction_data: ['msmod --timeshift -1 -ts 2016,182,23:59:59.999999', ' msmod –actflags ‘4,1’ –ts 2016,182,23:59:36 –te 2016,183,00:00:36'], correction_end_sync_instrument: subtracted one second from displayed instrument time"</Value>
</Comment>
```

### Deployments in lakes
**[STD {/}]** Set the `<WaterLevel>` to the elevation of the lake surface

### Positions
**[REC {/}]** Use the `plusError`, `minusError` and `measurementUncertainty` attributes to specify uncertainties in Latitude, Longitude and Elevation and how you measured them.

### Orientation information
**[STD {/}]** Set the following `<Azimuth>` and `<Dip>` values for the following source/subsource codes:

code       | `<Dip unit="DEGREES>xxx</Dip>`[^1] | `<Azimuth unit="DEGREES" xxx>` | `yyy</Azimuth>`   | Comment
---------- | ------------------------------ | ----------------------------- | ---------------- | ---------------------------
1          | 0.0                     | minusError="180.0" plusError="180.0" | 0.0  |
2          | 0.0                     | minusError="180.0" plusError="180.0" | 90.0 |
3          | 90.0                    |                                      | 0.0  |
DH, DG, DO | 90.0                    |                                      | 0.0  | if value *DECREASES* for a pressure increase[^2]
DH, DG, DO | -90.0                   |                                      | 0.0  | if value *INCREASES* for a pressure increase[^2]

[^1]: StationXML/Seed Dip is degrees down from horizontal.  SAC analog is +90 degrees ([`CMPINC` : Component incident angle (degrees from upward vertical)](https://ds.iris.edu/files/sac-manual/manual/file_format.html)
[^2]: The pressure sensor dip gives the same polarity as the seismometer/geophone for UPGOING waves.  Dip = -90 means that the first break will have the same polarity as a "Z" channel

### Data completeness
**[REC {/}]** Use Station ``<StartDate>`` and ``<EndDate>`` to specify when the data was supposed to start and end, and Channel ``<StartDate>`` and ``<EndDate>`` to specify when it actually starts and ends for each channel.

### Standard values that marine seismologists may not know:
**[FDSN]** Within each `<Channel>`, set `<Type>CONTINUOUS</Type>` and `<Type>GEOPHYSICAL</Type>`


## Data (miniSEED)

### Clock drift correction

Three main possibilities for distributing data are proposed:
1. *"RAW"*: No time correction applied.
    - May be preferred by users of long-period data (>10s) because it can be easier to concatenate daily files.
3. **[HIGHLY RECOMMENDED {/}]** *"SHIFTED"*: Indicate the time correction in each record header and apply it.
    - Allows the user to work with time-corrected but otherwise unmodified data
5. *"RESAMPLED"*: Resample the data at the originally intended rate.
    - "Best of both worlds": data are time-corrected and easy to concatenate/combine with other data
    - Could distort waveforms/spectra (needs to be studied)

#### Distinguishing data of each type

- *SHIFTED**
    - **[RECOMMENDED {/}]** data quality flag "Q"
    - **[ALTERNATIVE {/}]** location code between 00 and 49
    - **[ALTERNATIVE {/}]** letter in location code?
- *RAW*
    - **[RECOMMENDED {/}]** data quality flag "D"
    - **[ALTERNATIVE {/}]** location code between 50 and 99
    - **[ALTERNATIVE {/}]** letter in location code?
- *RESAMPLED*
    - Different channel code (to be specified)

#### Creating data of each type

- *RAW*
    - Nothing to do.
    - [RECOMMENDED? {/}] Put time correction in record header field 16 and set field 12 bit 1 to 0.
- *SHIFTED**
    - **[REC {/}]** Calculate a new time drift for each record
    - **[STD {/}]** Indicate time correction applied in record header field 16 (“Time
      Correction” and set field 12, bit 1 (“Activity flag, time correction applied”) to 1.
    - If there is no measure of clock drift:
       - Provide data as "D". set bit 7 of data quality flag ("time tag is questionable") to 1.  Add blockette 500, field 10 ("Clock status") indicating that there is an unmeasured drift (for example: "Unmeasured clock drift on Seascan MCXO, expected order = 1e-8")
- *RESAMPLED*
    - Not yet determined

There is a new ``msmod`` option in discussion/development that should take care of creating *SHIFTED** data, using either a linear or cubic spline interpolation (possibility of polynomial fit?).

### Leap seconds
**[STD {/}]** Leap seconds should be corrected in **SHIFTED** data and the record containing the leap second should be flagged.

**[STD {/}]** If the leap second is positive (the most common case: 61 seconds in the minute):
- Shift all record times AFTER the leap second back one second.
- Set activity flag bit 4 to 1 in the header of the record containing the leap second.
- Change `end_sync_instrument` to be one second earlier than what the instrument
  indicated

An example of changing the data using msmod, for a positive leap-second at 23:59:60 on day
182, 2016:

```console
msmod --timeshift -1 -ts 2016,182,23:59:59.999999'
msmod –-actflags ‘4,1’ –tsc 2016,182,23:59:59.999999 –tec 2016,182,23:59:59.999999
```

**[STD {/}]** If the leap second is negative (59 seconds in the minute):
- Shift all record times AFTER the leap second forward one second.
- Set activity flag bit 5 to 1 in the header of the record containing the leap second.
- Change `end_sync_instrument` to be one second later than what the instrument
  indicated


## Proposed modifications to existing standards

### Allow sampling rate to be specified as double precision[^2].

[^2]: is in the miniSEED3 draft

This is the only way to accurately represent OBS clock rates, which are regular but off of the specified sampling rate by a factor of approximately 1e-8 (MCXOs) or 1e-9.5 (CSACs), requiring 27- or 32-bit floating-point mantissas, respectively, to be correctly specified. Single precision floats only have 23-bit mantissas, double precision floats have 52-bit mantissas.

### More data quality flags, with clear hierarchy

#### In miniSEED2
Data quality flags are the only clear way to distinguish between levels of data processing, but the choices are too limited. Additional data qualities that cannot currently be specified are: Data directly translated from another format , or data for which the header values have been changed, but not the data itself. A possible hierarchy would be (new in italics):
- “D” : The state of quality control of the data is Indeterminate
- “T”: Translated Raw Waveform Data from another initial format
- “R”: Raw Waveform Data with no Quality Control (reserved for SEEDlink)
- “H”: Quality controlled Data, processes have been applied only to the headers
- “Q”: Quality controlled Data, some processes have been applied to the data
  (does this mean time-series values)?
- “C”: Quality controlled Data, No processes applied to time-series or header
- “M”: Data center modified, time-series values have not been changed

#### In miniSEED3
In the miniSEED3 header, the Data publication version replaces the data quality flags
(can still have flags in “Extra Header Fields” (field 14). This offers a clear hierarchy,
but not a way to specify that one wants uncorrected or corrected data (recommended
RAW=1 could be used for uncorrected data). Could this be put in field 14: extra
header fields? In any case, would have to be searchable using web tools
