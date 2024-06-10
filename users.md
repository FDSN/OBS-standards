# Information for marine seismology data users

## Preamble
Ocean bottom seismometers (OBS) provide seismological access to the 70% of the earth’s surface that is covered by water. This document provides information about specificities of ocean bottom seismometer data that users may need to take into account in order to correctly use the data and fully exploit the signals.

The [Metadata and Data](./users.md#metadata-and-data) section contains subsection titles whose content would be derived from the [standards](./standards.md) document.

## Metadata and Data

- Station names for repeated deployments
- Metadata
  - Azimuth
  - Dip
  - Clock corrections
  - Leap seconds
- Data
  - Data quality flags
  - Pressure sensor data


## OBS-specific Software
There are several publically available software modules that can be useful for confirming or correcting data and metadata. If you calculate better clock drift or sensor
orientation values than those provided in the data center,  inform the data center of the new values and the method you used to calculate them.

### Descriptions

#### Creating metadata

[obsinfo](https://obsinfo.readthedocs.io) is a tool for creating StationXML files that include OBS-specific information

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
BRUIT-FM Toolbox   | https://gitlab.ifremer.fr/anr-bruitfm/bruit-fm-toolbox | Improved windowing selection and data stacking for more accurate transfer function calculation
Station localization | obsloc?  |
Active seismic data  |          |
