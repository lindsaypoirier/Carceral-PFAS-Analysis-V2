# Carceral PFAS Analysis

## About

The code and data in this repo calculates the number, percentages, and populations of US carceral facilities in the same HUC-12 watershed boundary and, as a proxy for hydrologic flow, at a lower elevation than point sources with likely PFAS contamination.

## Contributors

<!-- ALL-CONTRIBUTORS-LIST:START -->
| Contributions | Name |
| ----: | :---- |
| [🔢](# "Data") [💻](# "Code") [🤔](# "Ideas and Planning")| [Lindsay Poirier](#) |
| [🔢](# "Data") [🔬](# "Research") [🤔](# "Ideas and Planning") | [Derrick Salvatore](#) |
| [🔬](# "Research")  [🤔](# "Ideas and Planning") | [Phil Brown](#) |
| [🔬](# "Research") [🤔](# "Ideas and Planning") | [Alissa Cordner](#) |
| [🔬](# "Research")  [🤔](# "Ideas and Planning") | [Kira Mok](#) |
| [📋](# "Organizer")[🔢](# "Data")[🔬](# "Research")[🤔](# "Ideas and Planning") | [Nicholas Shapiro](https://github.com/shapironick) |

<!-- ALL-CONTRIBUTORS-LIST:END -->

(For a key to the contribution emoji or more info on this format, check out ["All Contributors."](https://allcontributors.org/docs/en/emoji-key))

## Definitions and Data Sources

* Department of Homeland Security. "Prison Boundaries." Homeland Infrastructure Foundation Level Data, July 7, 2020. https://hifld-geoplatform.opendata.arcgis.com/datasets/geoplatform::prison-boundaries/about
* Federal Aviation Administration. "Part 139 Status Certification List." United States Department of Transportation, August 18, 2021. https://www.faa.gov/airports/airport_safety/part139_cert/
* Environmental Protection Agency. "Clean Watersheds Needs Survey Data and Reports." 2012. https://ordspub.epa.gov/ords/cwns2012:; Facilities that discharge effluent to a deep well or did not produce discharge were excluded.
* Environmental Protection Agency. "Facility Registry Service." https://www.epa.gov/frs; Facilities filtered by corresponding NAICS code (see table below)
* Office of the Assistant Secretary of Defense for Sustainment. "Geospatial Information for U.S. Military Installations, Ranges, and Training Areas." Defense Installations Spatial Data Infrastructure (DISDI),  2020. https://www.acq.osd.mil/eie/BSI/BEI_DISDI.html
* Environmental Protection Agency. "Base Realignment and Closure Act (BRAC) Sites." https://www.epa.gov/fedfacts/base-realignment-and-closure-act-brac-sites; Facilities that overlapped geospatially with U.S. Military Installations, Ranges, and Training Area were excluded. 
* Department of Homeland Security. "Formerly Used Defense Sites (FUDS) Public Properties." Homeland Infrastructure Foundation Level Data, August 21, 2021. https://hifld-geoplatform.opendata.arcgis.com/datasets/geoplatform::formerly-used-defense-sites-fuds-public-property-boundaries/about 

### NAICS Codes Included

|NAICS Code  |Label  |
|:-----|:-------------------|
| 313 | Textile Mills |
| 322 | Paper manufacturing |
| 323 | Printing and related support activities |
| 324 | Petroleum and coal products manufacturing |
| 3255 | Chemical manufacturing |
| 32591 | Printing ink manufacturing |
| 3328 | Metal coating, engraving, heat treating, and allied activities |
| 3344 | Semiconductor and other electronic component manufacturing	 |
| 562212 | Landfills |

## Repo Architecture

There are two R Markdown files in this repo:

1. get_HUC_elevation.Rmd
  * Imports source datasets and transforms them to a common CRS (4269)
  * Calculates the total facilities under consideration each category
  * Determines which watershed boundary (HUC-12) each facility in the source datasets geo-spatially intersect with
  * Determines the elevation of each facility in the source datasets via the USGS Elevation Point Query Service
  * Saves the resulting data as .gpkg files for use in HUC_analysis.Rmd
  
2. HUC_analysis.Rmd
  * Loads the .gpkg files created in get_HUC_elevation.Rmd
  * Joins carceral facility data to data representing the locations of each potential point source based on a shared HUC-12 code
  * Filters to carceral facilities at a lower elevation than a potential point source as a proxy for hydrological flow direction
  * Calculates the number of unique facilities in the same HUC-12 watershed boundary and at a lower elevation than each point source and a combination of point sources, along with the number of incarcerated people potentially impacted
  * Presents increasingly conservative calculations by filtering the values down depending on whether facilities have been designated as not closed and whether they have been marked as having accurate geo-coding
  * Produces the tables presented in the manuscript
  
All of the .gpkg files created for this project are available in the huc_elevation_datasets folder in this repo, so the code in HUC_analysis.Rmd can be run without running the code in get_HUC_elevation.Rmd. Note that get_HUC_elevation.Rmd involves a number of API calls to the USGS Elevation Point Query Service. 

## Notes

* There were a number of facilities in the EPA's Facility Registry Service that either did not pass a geo-coding quality assurance test or were geo-coded based on an approximated location with increasingly less specificity. The EPA assigned an accuracy score to each facility (in meters) as an expression of the degree of accuracy of the recorded coordinates. Lower accuracy scores represented more accurately geo-coded coordinates. Scores were assigned first based on the specificity of the collection method. For instance, coordinates determined by street addresses were considered more accurate than those determined by intersection, and coordinates determined by intersections were considered more accurate than those determined by zip codes. After this, QA checks were run to make sure the derived coordinates were in the same zip code, city, county, and state as the reported facility. Accuracy scores were increased (representing less accuracy) when a facility failed a QA check. We selected 1000 meters or less as a threshold in order to exclude facilities that failed quality assurance checks and to exclude facilities that were geo-coded based on a location with less specificity than a street with only one block. For more information see: [https://www.epa.gov/system/files/documents/2021-08/facility-registry-service-best-pick-processing-v-2.0.pdf](https://www.epa.gov/system/files/documents/2021-08/facility-registry-service-best-pick-processing-v-2.0.pdf)
* As of 2020, US facilities were required to report certain PFAS emissions via the Toxic Release Inventory (TRI). This past year 39 facilities reported PFAS emissions to the TRI. When we looked into these facilities, and the changes to the results of the analysis were negligible. We opted to exclude this category based on concerns that the novelty of this reporting requirement was resulting in dramatic under-estimations of the total emitted PFAS. 
* Carceral facilities reported by HIFLD are missing a number of private and ICE facilities. Population data is also missing for about 29% of these facilities. 
* When analyzing the "Clean Watersheds Needs Survey Data and Reports" we included both waste water treatment plants and storm water sites based on a 2008 [study](https://www.sciencedirect.com/science/article/pii/S0045653507014865) that an urban river had "a higher load of PFAAs from storm water runoff than from WWTPs." The runoff levels were 2–11 higher than WWTPs. 

## Copyrights

All code in this repo is licensed with a GNU General Public License 3.0. Please see the [license](https://github.com/Critical-Data-Analysis-Group/Carceral-PFAS-Analysis/blob/main/LICENSE) file for details.

All written materials are licensed with a Creative Commons Attribution-ShareAlike 3.0 Unported (CC BY-SA 3.0). Please see this [license](https://creativecommons.org/licenses/by-sa/3.0/) for details.

## Have Questions?

Contact [Lindsay Poirier](mailto:lpoirier@smith.edu)

