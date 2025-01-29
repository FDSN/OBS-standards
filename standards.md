# Marine seismology data/metadata standards

## Introduction

### Nomenclature

This document contains FDSN standards, proposed standards, and recommentations.
Proposed standards are preceded by "We propose".
Recommendations are preceded by "We recommend".
FDSN standards are stated.

### A general rule for adding marine-specific information to StationXML

We propose the following rule:

1) Define the sub-elements that need to be specified
2) Insert into StationXML as a **StationXML-standardized element**
3) Request the addition of the information into StationXML, through FDSN WGII

A **StationXML-standardized element** is an element that obeys the StationXML schema but contains information that is not specified in the schema.
Currently, **StationXML-standardized elements** are encoded as StationXML ``<Comment>``s, with the elements written as JSON-formatted strings, preferably with a marine-specific ``subject``.  For example, if the information concerns "peanuts" and can be expressed as:
```yaml
can_size.ml: 200
num_nuts: 10
brand_name: "Mr Peanut"
nut_weights.g: [1.1, 1.1, 2.2, 2.4, 1.8, 1.4, 1.3, 1.2, 3.0, 2.1]
```

then this would be entered into the StationXML file (at the station or channel level, as appropriate) as:

```xml
<Comment subject=”peanuts”><Value>“{can_size.ml: 200. num_nuts: 10, brand_name: "Mr Peanut", nut_weights.g: [1.1, 1.1, 2.2, 2.4, 1.8, 1.4, 1.3, 1.2, 3.0, 2.1]}”</Value></Comment>
```

## Proposed standards and recommentations

### Marine-specific

#### StationXML

##### Clock drift and leapseconds

**[REC {/}]** There is no dedicated field in StationXML.
Embed this information as JSON-coded strings JSON-coded in Station-level `<Comment>` fields with `subjet="Clock Correction"`.
In the future, a seperate namespace may be created to allow a more specific and structured representation

The structures are shown below in YAML format first, for clarity, then as StationXML Comments.

##### Clock drift

We recommend specifying clock drifts using UTC datetimes, to avoid ambiguity.  The datetimes should be in ISO8601 format, followed by a "Z" to unambiguously specify UTC.  

###### YAML format:

```yaml
drift:
    type: 'piecewise_linear' # 'piecewise_linear', 'cubic_spline' or 'polynomial {a0 a1 a2 ...}'
    instrument: 'Seascan MCXO'
    instrument_nomimal_drift_rate: 1e-8
    reference: 'GPS'
    syncs_instrument_reference:
        - ["2016-09-10T00:00:00Z", "2016-09-10T00:00:00Z"]  
        - ["2017-01-12T00:00:01Z", "2017-01-12T00:00:00.415Z"]  
        - ["2017-07-13T11:25:01Z", "2017-07-13T11:25:00.6189Z"] 
```

###### StationXML format:

```xml
<Comment subject=”Clock Correction”><Value>“{drift: {instrument: Seascan MCXO,
 instrument_nominal_drift_rate: 1e-8, reference: GPS, type: piecewise_linear,
 syncs_instrument_reference: [['2016-09-10T00:00:00Z', '2016-09-10T00:00:00Z'],
 ['2017-01-12T00:00:01Z', '2017-01-12T00:00:00.415Z'], ['2017-07-13T11:25:01Z',
 '2017-07-13T11:25:00.6189Z',]]}}”</Value></Comment>
```

###### Explanation of fields

the `time_base`, `nominal_drift_rate` and `reference` fields are optional.
In the simplest case of a synchronization at the start and end of an experiment, and assuming purely
linear drift, there would only be two items within `syncs_instrument_reference`.

If there is assumed to be clock drift but some or all reference values were not measured, each missing value should be represented by ``~`` (None).

##### Leap seconds

Specified using information from the `leap-seconds.list` file, which is available online at several sites, including https://data.iana.org/time-zones/tzdb/leap-seconds.list.  The user should verify that the "File expires on" date is later than the last instrument channel's end-date.

###### YAML format:

```yaml
leapseconds:
    list_file_entries:
        - line_text: "3692217600      37      # 1 Jan 2017"
          leap_type: '+'
    applied_corrections:
        not_clock_corrected_miniseed: false
        syncs_instrument: true
```

###### StationXML format

```xml
<Comment subject=”Clock Correction”><Value>{list_file_entries: [{
 line_text: '3692217600      37      # 1 Jan 2017', leap_type: '+'}],
 applied_corrections: {syncs_instrument: true,
 not_clock_corrected_miniseed: false}}</Value></Comment>
```

###### Explanation of fields

- `list_file_entries` is an array/list, to allow for more than one leap-second during a deployment.

    - `line_text` should be directly copied from 
    - `leap_type` indicates whether the 2nd number in the list_file line_text ("37" in the above example) is greater than the previous line's value ("+") or less than the previous line's value ("-").  As of June 2024, all leap seconds have been type "+"

- `applied corrections` indicates if leapsecond corrections were applied to input data/parameters:

    - `not_clock_corrected_miniseed` is true if the "NOT CLOCK CORRECTED" miniSEED data integrates the leap second.  For most OBS deployments, this value should be `false` as dataloggers without GPS don't (yet?) have a way to integrate leap seconds.
    - `syncs_instrument`: indicates whether the instrument sync times have been corrected for the leap second(s).  They generally shouldbe, but in some cases (many sync times and/or more than one leap second) this may be better left to an algorithm than to a human operator. 

##### Orientation information
Set the following `<Azimuth>` and `<Dip>` values for the following source/subsource codes:

code       | `<Dip unit="DEGREES>xxx</Dip>`[^1] | `<Azimuth unit="DEGREES" xxx>` | `yyy</Azimuth>`   | Comment
---------- | ------------------------------ | ----------------------------- | ---------------- | ---------------------------
1          | 0.0                     | minusError="180.0" plusError="180.0" | 0.0  |
2          | 0.0                     | minusError="180.0" plusError="180.0" | 90.0 |
3          | 90.0                    |                                      | 0.0  |  For positive voltage for DOWNWARD motion
N          | 0.0                     |                                      | 0.0  |  Azimuth must be within 5° OF 0°
E          | 0.0                     |                                      | 90.0 |  Aziumth must be within 5° OF 90°
Z          | -90.0                   |                                      | 0.0  |  Dip must be within 5° of -90
DH, DG, DO | 90.0                    |                                      | 0.0  | if value *DECREASES* for a pressure increase[^2]
DH, DG, DO | -90.0                   |                                      | 0.0  | if value *INCREASES* for a pressure increase[^2]

[^1]: StationXML/Seed Dip is degrees down from horizontal.  SAC "CMPINC" is StationXML Dip+90 degrees ([Component incident angle (degrees from upward vertical)](https://ds.iris.edu/files/sac-manual/manual/file_format.html)
[^2]: The pressure sensor dip gives the same polarity as the seismometer/geophone for UPGOING waves.  Dip = -90 means that the first break will have the same polarity as a "Z" channel

##### Data completeness
We recommend using Station ``<StartDate>`` and ``<EndDate>`` to specify when the data was supposed to start and end, and Channel ``<StartDate>`` and ``<EndDate>`` to specify when it actually starts and ends for each channel.

We recommend keeping all of the recorded data, including "noisy" or "bad".


##### Leveling system
The instrument's leveling system can affect the quality and absolute values of measureables.  The following elements should be specified:
```yaml
threshold.deg: (float) # deviation from vertical (degrees) which will trigger releveling
accuracy.deg:  (float) # maximum deviation from vertical accepted after relevel
max_relevel_interval.h (float) # longest interval between level checks during the deployment
n_relevels: (int) # number of relevels performed during the deployment
relevels: (list) # list of [date, level_before.deg, level_after.deg] for each relevel
```

We propose implementing this as an "Equipment" element at the appropriate (Station or Channel) level, with ``<Type>Leveler</Type>`` and ``<Description>`` a JSON-encoded string of the above elements.

We will request a specific ``Leveler`` implementation of the ``Equipment`` element, with the above elements added to it.

#### subnetwork files

We recommend using obsinfo ["subnetwork" files](https://obsinfo.readthedocs.io/en/latest/subnetwork.html),
to store essential information about OBS deployments.

obsinfo subnetwork files can be used with the ``obsinfo`` software to generate [FDSN StationXML](http://docs.fdsn.org/projects/stationxml/en/latest/index.html)
files with embedded OBS-specific information, or with other tools to modify
existing StationXML files.

#### miniSEED data

##### Clock drift correction

There are three main possibilities for distributing data are proposed:
1. *"NOT CLOCK CORRECTED"*: No time correction applied.
    - May be preferred by users of long-period data (>10s) because it can be easier to concatenate daily files.
    - We recommend for Indicating:
        - miniSEED2: data quality flag "D"
    - We recommend for Creating:
        - Put time correction in record header field 16 and set field 12 bit 1 to 0.

2. *"CLOCK CORRECTED"*: Indicate the time correction in each record header and apply it.
    - Allows the user to work with time-corrected but otherwise unmodified data
    - We recommend for Indicating:
        - miniSEED 2: data quality flag "Q"
    - We recommend for Creating:
        - Calculate a new time drift for each record
        - miniSEED2: Indicate time correction applied in record header field 16 (“Time Correction” and set field 12, bit 1 (“Activity flag, time correction applied”) to 1.

3. *"RESAMPLED"*: Resample the data at the originally intended rate.
    - "Best of both worlds": data are time-corrected and easy to concatenate/combine with other data
        - Could distort waveforms/spectra (needs to be studied)
    - We recommend for Indicating:
        - miniSEED 2/3: Define another channel code (modified data)
    - We have no recommendations for creating

We highly recommend providing the _"CLOCK CORRECTED"_ data, if possible.

Recommendations if there is no measure of clock drift:
- miniSEED2:
    - Provide data as "D". set bit 7 of data quality flag ("time tag is questionable") to 1.  
    - Add blockette 500, field 10 ("Clock status") indicating that there is an unmeasured drift (for example: "Unmeasured clock drift on Seascan MCXO, expected order = 1e-8")

A new version of ``msmod``, in development, should be able to create *CLOCK CORRECTED* data from *NOT CLOCK CORRECTED data*, using piecewise linear, cubic spline, or polynomial interpolation.

##### Leap seconds
Leap seconds should be corrected in **CLOCK CORRECTED** data and the record containing the leap second should be flagged.

If the leap second is positive (the most common case: 61 seconds in the minute):
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

*A new ``msmod`` option has been proposed to simplify this to one line and one time specification.*

If the leap second is negative (59 seconds in the minute):
- Shift all record times AFTER the leap second forward one second.
- Set activity flag bit 5 to 1 in the header of the record containing the leap second.
- Change `end_sync_instrument` to be one second later than what the instrument
  indicated


### Marine and maybe general, too

#### processing steps
We recommend that processing done on data files (from data download to delivery to the data center)
should be recorded in text-based, structured files.
The [JSON process-steps format](https://github.com/WayneCrawford/sdpchainpy?tab=readme-ov-file#process-stepsjson) is an example.

#### Proposed modifications to existing standards

##### Allow sampling rate to be specified as double precision[^2].

[^2]: is in the miniSEED3 draft

This is the only way to accurately represent OBS clock rates, which are regular but off of the specified sampling rate by a factor of approximately 1e-8 (MCXOs) or 1e-9.5 (CSACs), requiring 27- or 32-bit floating-point mantissas, respectively, to be correctly specified. Single precision floats only have 23-bit mantissas, double precision floats have 52-bit mantissas.

###### In miniSEED3
In the miniSEED3 header, the Data publication version replaces the data quality flags
(can still have flags in “Extra Header Fields” (field 14). This offers a clear hierarchy,
but not a way to specify that one wants uncorrected or corrected data (recommended
RAW=1 could be used for uncorrected data). Could this be put in field 14: extra
header fields? In any case, would have to be searchable using web tools

## Reminder/clarification of existing standards

### Source Identifiers
The following source-subsource codes (see [FDSN Source Identifiers documentation](http://docs.fdsn.org/projects/source-identifiers/en/v1.0/index.html))
should be used for the following types of sensor/data:

code | description
---- | ------------------------------------------------------------
1    | Unoriented seismometer, “N” channel equivalent
2    | Unoriented seismometer “E” channel (+90 degrees from "1")
3    | Seismometer/geophone with inverted vertical channel (positive voltage is down)
DH   | Hydrophone
DG   | Differential pressure gauge
DO   | "Absolute” bottom pressure recorder

### Station names for repeated deployments
The [IASPEI Station Coding Standard](http://www.isc.ac.uk/registries/download/IR_implementation.pdf) recommends that station and/or location codes be changed if the associated sensors are moved far enought to result in a signifcant *teleseismic* travel-time residual discrepancy.  We recommend the same, except that the basis for what is a significant travel-time residual discrepancy should depend on your study.

If you change the station name between deployments, we recommend incrementing the last _N_ characters of the station name, e.g. ``STAA``, ``STAB``, ``STAC``, or ``STA01``, ``STA02``, ``STA03``, etc.  _N_ should depend on the maximum number of deployments you will possibly make and the characters you use in the incrementer (numeric, alphabetic, or alphanumeric). 

### Deployments in lakes
Set the `<WaterLevel>` to the elevation of the lake surface

### Positions
We recommend using the `plusError`, `minusError` and `measurementMethod` attributes to specify uncertainties in Latitude, Longitude and Elevation and how you measured them.

### Standard values that marine seismologists may not know:
Within each `<Channel>`, set `<Type>CONTINUOUS</Type>` and `<Type>GEOPHYSICAL</Type>`





