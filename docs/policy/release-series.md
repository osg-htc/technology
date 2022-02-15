
OSG Software Release Series Support Policy
==========================================

This document describes the OSG policy for managing its software releases. Use this policy to help plan when to perform
major OSG software updates at your site.

OSG software releases are organized into __release series__, with the intent that software updates within a series do
not take long to perform, cause significant downtime, or break dependent software.
New series can be more disruptive, allowing OSG to add, substantially change, and remove software components.
Releases are assigned versions, such as "OSG 3.2.1", where the first two numbers designate the release series and the
third number increments within the series.
Changes to the first number are infrequent and indicate sweeping changes to the way in which OSG software is
distributed.

OSG supports at most two concurrent release series, __current__ and __previous__, where the goal is to begin a new
release series about every 18 months.
Once a new series starts, OSG will support the previous series for at least 12 months and will announce its end-of-life
date at least 6 months in advance.
During the first 6 months of a series, OSG will endeavor to apply backward-compatible changes to the previous series as
well; afterward, OSG will apply only critical bug and security fixes.
When support ends for a release series, it means that OSG no longer updates the software, fixes issues, or troubleshoots
installations for releases within the series.
The plan is to maintain interoperability between supported series, but there is no guarantee that unsupported series
will continue to function in OSG.

Files for release series older than current or previous will be removed from the OSG software repositories no earlier
than when support ends for the previous release.
For example, files for OSG 3.2 will be removed no earlier than when support ends for OSG 3.3 in May 2018.

OSG Operations will handle deviations from this policy, in consultation with OSG Technology and stakeholders.

Life-cycle Dates
----------------

Support ends at the end of the month of the following dates unless otherwise specified:

| Release Series | Initial Release | End of Regular Support | End of Critical Bug/Security Support |
|:--------------:|-----------------|------------------------|--------------------------------------|
| 3.6            | Februrary 2021  | Not set                | Not set                              |
| 3.5            | August 2019     | August 2021            | 1 May 2022                           |
| 3.4            | June 2017       | February 2020          | November 2020                        |
| 3.3            | August 2015     | December 2017          | May 2018                             |
| 3.2            | November 2013   | February 2016          | August 2016                          |
| 3.1            | April 2012      | October 2014           | April 2015                           |
