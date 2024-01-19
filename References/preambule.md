# Preamble
Ocean bottom seismometers (OBS) provide seismological access to the 70% of the
earth’s surface that is covered by water.
For full usage, they should be archived in and distributed from FDSN
compatible seismological databases.

standards.md lists OBS-specific standards and “best practices” for using
the stationXML and miniSEED formats to make OBS data clear,
easy to use and intercompatible.

OBS clocks generally have a non-negligible drift because of the lack of GPS
signal at the seafloor.
The resulting time offsets must be corrected or at least indicated in any data
archived at data centers.
OBS time bases are generally chosen to have small and first- degree linear
drift.
Their drift is calculated by synchronizing the instrument clock to GPS before
the deployment and then comparing the instrument clock to GPS after the deployment.
If the instrument clock cannot be compared to GPS at the end of the experiment,
the drift can be calculated a posteriori by calculating the noise correlation
between this instrument and another synchronized instrument over the length
of the experiment.

Information about the existence of linear clock drift, its value if measured
(linear at least, non-linear if possible) and its probable range if not measured,
should be provided in the data and metadata.
