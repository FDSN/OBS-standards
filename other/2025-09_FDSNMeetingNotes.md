# 2025-09 FDSN Meeting at IASPEI Symposium, Lisbon

Wayne presented the marine seismology proposal document at the FDSN workgroup meetings:
first at the WG2 (data exchange), then at WG5 (portable instrumentation).

## Feedbacks/comments/corrections received.

### msmod

The clock drift correction software has been written, but needs to be validated.
The correction was submitted to the project before a major branch for miniSEED3 was created by Chad Trabant,
so validation should proceed in three steps:
1. Validation of the "old" branch (Chad requested some updates to better separate the new code from the rest).
   This will provide a miniSEED2 "reference" version
2. Addition of the new code to the new branch (by Chad)
3. Validation of the new branch with the new code (Wayne)

Chad Trabant also plans to integrate leap-second correction into msmod.
Rather than having the user specify the leap seconds, they will specify from what date on the leap seconds are
not corrected and the code will automatically apply subsequent leapseconds.
This will require a code update if there are new leapseconds in the future, but this does
not appear to be very likely.

### WG2 meeting

- It will be difficult, if not impossible, to use <Channel> <StartDate> and <EndDate> to indicate the extent of "good" data,
  while delivering data outside of these bounds, as many data centers require that the data be within the <Channel> date boundaries.

- Chad and Javier prefer that we propose new in StationXML Elements rather than either JSON-encoded strings or marine-specific namespaces.
  They do not mind if these elements only apply to marine seismology data.

- Chad thinks that the sample rate should be as accurate as possible, including any clock drift.
  This will be possible in miniSEED3

- The list of relevel times we proposed in the new "Leveler" element is more appropriate in the miniSEED data than in StationXML.

### WG5 meeting

- There was a discussion about whether there should be an "Annotation" stnadard separate from miniSEED and StationXML (for indicating where
  data has problems, for example.  CTrabant feels that adding miniSEED after the data collection is not sustainable.
- It was not clear if GFZ's requested instrumentation id was to be a general addition, or their desire to have elements that only they use.
  If the former, they need to specify what is different from the various "id" elements that exist already
  If the latter, this is not an FDSN problem but rather one of the softwares that read and write StationXML (obspy has been updated
  to be a better citizen).
- There is already [a recommended mapping](https://docs.fdsn.org/projects/miniseed3/en/latest/appendix.html#miniseed-2-4-fixed-section-data-header-fsdh)
  from "R"D"Q"M" data quality codes to DataSequence numbers.  In this case, DataSequenceNumber should be 2 for NTC and 3+ for TC.
- No solution was found for tagging Resampled data.  Should be discusssed/resolved in WG2.

Overall, some of the Action Group recommendations can be proposed/integrated into miniSEED and StationXML, whereas others
cannot.

Wayne proposed to finish with two documents:
1. proposals for changes/documentation in the FDSN StationXML/miniSEED/SourceIdentifier webpages
2. A document on other issues in marine seismology data/metadata creation

AStrollo recommends that marine-specific information be available on the FDSN documentation sites, with links to the relevant locations.
  

## Propositions to address these

Wayne will update the document and create one issue per change.  The issues are:
- Remove specification of <Channel><StartDate><Enddate> to indicate where the data are good quality: all data saved must
  fit within the <Station> and <Channel> data bounds
- Simplify description of leap year correction to match new msmod functionality (link to current method if anyone is desparate?)
- Start documentation for each category (StationXML/miniSEED/SourceIdentifiers) with recommended modification to standards or docuentation,
  end with marine-specific recommentations and reminders.
- Propose a StationXML <Leveler> element, based on Equipment
- Remove references to CONTINUOUS and GEOPHYSICAL types
- Remove reminder about deployments in lakes (already documented)
- change miniSEED recommendations for specifying NOT CLOCK CORRECTED (NCC) versus CLOCK CORRECTED (CC) data from "FDSN-> DataQuality"
  to the DataSequence Number (with a reference to the correspondance)
- No recommendata for RESAMPLED data: ask WG2 to work on this.
- Remove recommendation for files with both corrected and not-corrected data? (Earthscope just keeps highest level, archives lower levels)
- Propose a StationXML <ClockDrift> element, based on Equipment?
