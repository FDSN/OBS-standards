# Information for marine seismology data users

## Preamble
Ocean bottom seismometers (OBS) provide seismological access to the 70% of the earth’s surface that is covered by water. This document provides information about specificities of ocean bottom seismometer data that users may need to take into account in order to correctly use the data and fully exploit the signals.

The information below on metatdata and data are simplified from the `standards.md` document, which is the reference.

## Metadata

### Azimuth
Mobile marine seismology horizontal channels are often not oriented on recovery, in which case the horizontal channel names are ‘??1’ and ‘??2’ instead of ‘??N’ and ‘??E’. The ‘??1’ channel is oriented -90° with respect to the ‘??2’ channel. If the channels have been oriented, the metadata will provide the azimuths, uncertainty and measurement method. If not, they will often indicate ‘??1’s azimuth as 0+-180° and ‘??2’s azimuth as 90+-180°.

### Dip
Pressure sensor channel dips are given such that the the first (upgoing) P arrival will have the same polarity as that on a vertical seismometer channel.

### Clock corrections
The clock drift and method used to measure it are usually stored as a JSON-formatted station-level comment

### Leap seconds
Mobile marine seismology dataloggers do not automatically integrate leap seconds. If a leap-second has been corrected in the data, there should be a JSON-formatted station-level comment specifying the leapsecond


## Data

### Data quality flags
Mobile marine seismology data usually have a significant clock drift: up to several seconds per year. A data quality of “D” indicates that the data has not been corrected for the clock drift. A data quality of “Q” indicates that it has. Some data centers store both, in which case a standard FDSN webservices request will return the “Q” data quality, if it is available. Researchers interested in low-frequency, multi-day signals and not concerned with second-level timing may prefer to request the “D” quality, which is easier for some software to stitch together for multi-day spectral analysis.

### Pressure sensor data
Most Mobile marine seismology data have at least one pressure sensor channel. The most common pressure sensors are hydrophones (channel code “?DH”), differential pressure gauges (channel code “?DG” in theory, but often saved as “?DH”) and depth sensors (channel code “?DO” in theory, but often saved as “?DH”). The former have a low frequency limit from ~1 second to 100 seconds and effectively no high frequency limit, whereas the last two are more sensitive at low frequencies and usually have a clear high frequency limit. Hydrophone data can act as a proxy for vertical channel data and often have clearer P phase arrivals. Differential and absolulte pressure gauges are useful for measuring the coherent signal between low-frequency ocean waves and seafloor motion, which can then be either exploited for subsurface studies, or removed for clearer earthquake and normal mode signals.
and significant clock drift: up to several seconds per year. A data quality of “D” indicates that the data has not been corrected for the clock drift. A data quality of “Q” indicates that it has. Some data centers store both, in which case a standard FDSN webservices request will return the “Q” data quality, if it is available. Researchers interested in low-frequency, multi-day signals and not concerned with second-level timing may prefer to request the “D” quality, which is easier for some software to stitch together for multi-day spectral analysis.


## Both metadata and data

### Station names for repeated deployments
If OBSs are deployed repeatedly at one site (to make a long series), they often an incrementing alphanumeric character at the end of the station name (i.e., A01A, then A01B then A01C for subsequent deployments at the same approximate site “A01”).


## OBS-specific Software
There are several publically available software modules that can be useful for confirming or correcting data and metadata. If you find better clock drift or sensor
orientation values than those provided in the data center,  inform the data center of the new values and the method you used to calculate them.

### Descriptions

#### Clock drift confirmation
The instrument clock drift (relative to a reference instrument) can be calculated using noise correlation (REF_SITE) or using the observed drift in hypocenter traveltime residuals.

#### Sensor orientation calculation/correction
Determining the horizontal orientation of the instrument using teleseisms and/or local earthquakes. A common method used is the Rayleigh wave polarization method, outlined in the Stachnik et al., 2012 paper[^1]
with code made available on the OBSIP website.

[^1]: Stachnik, J.C., A.F. Sheehan, D.W. Zietlow, Z. Yang, J. Collins, and A. Ferris (2012), Determination of New Zealand Ocean Bottom Seismometer Orientation via Rayleigh- Wave Polarization: Seismol. Res. Lett., 83, 704–712, doi:10.1785/0220110128.

#### Noise removal
Removal of noise caused by infragravity waves (based on pressure measurements) and dynamic tilt (based on correlation with horizontal channel noise).


#### Active seismic data extractor
A program to input continuous data-center-level data and a shot file (in some standardized format) and output SEG-Y files.

### Available software

Subject      |  Site            | Comments
------------ | ---------------- | ---------------
Clock drift  |                  | sara hable's code
Sensor orientation |            | John Scholz's code
"                  |    ppol    | Wayne's ppol code
"                  | http://www.obsip.org/data/obs-horizontal-orientation/ | Stachnik et al. code  
Noise removal      |   ATACR    | matlab version
"                  |   ATACR    | python version
"                  |  tiskitpy  |
Station localization | obsloc?  |
Active seismic data  |          |
