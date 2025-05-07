# OBS-specific Software
There are several publically available software modules that can be useful for confirming or correcting data and metadata. If you calculate better clock drift or sensor
orientation values than those provided in the data center,  inform the data center of the new values and the method you used to calculate them.
It would be best if all of these codes worked on the same data/metadata formats (for example, SeisComp Data Structure and StationXML)

## Descriptions

### Modifying miniseed files
Few (none?) of these tools are OBS-specific, but they can be used for OBS-specific tasks

### Creating metadata

[obsinfo](https://obsinfo.readthedocs.io) is a tool for creating StationXML files that include OBS-specific information

### Clock drift confirmation
The instrument clock drift (relative to a reference instrument) can be calculated using noise correlation (REF_SITE) or using the observed drift in hypocenter traveltime residuals.

### Sensor orientation calculation/correction
Determining the horizontal orientation of the instrument using teleseisms, local earthquakes and/or active seismic sources. A common method used is the Rayleigh wave polarization method, outlined in the Stachnik et al., 2012 paper[^1]
with code made available on the OBSIP website.

[^1]: Stachnik, J.C., A.F. Sheehan, D.W. Zietlow, Z. Yang, J. Collins, and A. Ferris (2012), Determination of New Zealand Ocean Bottom Seismometer Orientation via Rayleigh- Wave Polarization: Seismol. Res. Lett., 83, 704–712, doi:10.1785/0220110128.

### Noise removal
Removal of noise caused by infragravity waves (based on pressure measurements) and dynamic tilt (based on correlation with horizontal channel noise).


### Active seismic data extractor
A program to input continuous data-center-level data and a shot file (in some standardized format) and output SEG-Y files.

## Available software

Subject      |  Software                          | Comments
------------ | ---------------------------------- | ------------
Modify miniseed  |  [msmod](https://github.com/earthscope/msmod) | Application of leapseconds, header modification
"             |  [qedit](https://ncedc.org/qug/software/ucb/) | Linear and piecewise linear time correction, application of leap seconds, header modification
"                |  [GIPP tools](https://git.gfz-potsdam.de/gipp/gipptools) | toolbox, e.g. modify header, cut in pieces, export header to ASCII, etc.
Clock drift  |                                    | sara hable's code?
"            | justCorrel (Hanneman 2014)         |
"            | ObsPy (Bayreuther 2010, Krischer 2015) | 
"            |  [Integrated Seismic Program](https://projectisp.github.io/ISP_tutorial.github.io/) | 
"            | Fortran code after Weemstra (2020) |
Sensor orientation |    [ppol](https://github.com/WayneCrawford/Ppol)  | Based Scholz et al. (2017), plus event location uncertainty
"                  |    [OBSIP orientation](http://www.obsip.org/data/obs-horizontal-orientation/) | Stachnik et al.[^1] code  
"                  | [OrientPy](https://nfsi-canada.github.io/OrientPy/) |
Noise removal      |   [ATACR](https://github.com/helenjanisz/ATaCR) | Automated Tilt and Compliance Removal, matlab version
"                  |   [ATACR](https://nfsi-canada.github.io/OBStools/atacr.html) | python version
"                  |  [tiskitpy](https://tiskitpy.readthedocs.io/latest/) | Transfer-function based and rotation-based, plus synthetic noise generation
"                  | [BRUIT-FM Toolbox](https://gitlab.ifremer.fr/anr-bruitfm/bruit-fm-toolbox) | Improved windowing selection and data stacking for more accurate transfer function calculation. 
"                  | [Compy](https://github.com/MohammadAmin-Aminian/ComPy) |   tiskitpy + periodic glitch removal and data processing automation
"                  | [NoiseCut](https://doi.org/10.5281/zenodo.7339552)     | Harmonic-percussive separation algorithms
Station localization | [OBSrange](https://github.com/jbrussell/OBSrange) | matlab and python
Conversion to active seismic data format  |   | Does not exist, should be fairly simple using obspy
