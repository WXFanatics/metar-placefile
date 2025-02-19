## This is a fork of https://github.com/ktrue/metar-placefile

I will most likely create and modify the placefiles with my personal touches.

Modified Placefiles:

[Temperatures Only](https://github.com/WXFanatics/metar-placefile/blob/main/metar-t.php) - Temperatures, color coded. Commented out windbarbs. Centered temperatures on location

<img src="https://github.com/WXFanatics/metar-placefile/assets/96398274/4e377ca8-89cb-47e2-97e6-199558c54906" />


------------------

[Temperature and Heat Index](https://github.com/WXFanatics/metar-placefile/blob/main/metar-thi.php) - Commented out windbarbs. Added color coding. Centered temperatures on location. Set heat index >=103 to show with temperatures. This allows the danger classification of heat indexes to be displayed with the temperatures. More info here: https://www.weather.gov/ama/heatindex

<img src="https://github.com/WXFanatics/metar-placefile/assets/96398274/2d9c3e9d-e533-4b8a-a06e-07c9d7b3d6d5" />

# metar-placefile
## GRLevelX placefile generator for METAR data from aviationweather.gov

## Purpose:

This script set gets METAR data from aviationweather.gov and formats a placefile for GRLevelX software
to display icons for weather conditions/sky conditions, wind barbs for wind direction/speed, and
mouse-over popups with text for the current METAR report from the station.


Two scripts are to run via cron to gather the data routinely (*get-metar-metadata.php*, *get-aviation-metars.php*).  

The *metar-placefile.php* script is to be accessed by including the website URL in the GRLevelX placefile manager window.

## Scripts:

### *get-metar-metadata.php*

This script reads the **stations.txt** from aviationweather.gov and merges optional updates from
**new_station_data.txt** (a comma delimited CSV file) to produce *metar-metadata-inc.php* which
is used by the *get-aviation-metars.php* program for all the descriptive info about a METAR site.

It should be run daily by cron .. the source file doesn't change very often.
If the **stations.txt** file is not available, the cache file **metar-location-raw.txt** will be used instead


### *get-aviation-metars.php*

This script reads the **metar.cache.csv** from aviationweather.gov and creates the 
*aviation-metars-data-inc.php* file which contains the parsed and formatted weather
data for each reporting METAR station. 

This program requires the following files:
-  *metar-cond-iconcodes-inc.php* (for weather code to iconnumber lookup)
-  *metar-metadata-inc.php* (for details about the METAR ICAO produced by *get-metar-metadata.php*)

It should be run by cron every 5 or 10 minutes to keep the data current.  Keep in mind that
many METAR sites report only once per hour so loading more often won't result in 'new' data.

### *metar-placefile.php*

This script generates a GRLevelX placefile from the aviation-metars-data-inc.php 
file on demand by a GRLevel3 instance.  It will return METAR icons with popup info and wind barb
for each METAR within 300 miles of the current radar selected in GRLevel3.
It requires the following files:

-   *metar-cond-iconcodes-inc.php* (for weather code to iconnumber lookup)
-   *aviation-metars-data-inc.php* (produced by *get-aviation-metars.php* for the current METAR data)
   

The script uses 2 icon files:  *windbarbs_75_new.png*, *cloudcover_new.png*

If you run the script for debugging in a browser, add `?version=1.5&dpi=96&lat={latitude}&lon={longitude}` to
the *metar-placefile.php* URL so it knows what to select for display.

### *metar-cond-iconcodes-inc.php*

This file contains the `pick_cond_icon()` function and lookup tables to determine the iconnumber to
display from the *cloudcover_new.png* file.  It is included in the *get-aviation-metars.php* and
*metar-placefile.php*.  The *get-aviation-metars.php* output will report any code not found (with the raw METAR)
to enable adding any missing codes.

Additional documentation is in each script for further modification convenience.

## Installation

Change the `date_default_timezone_set('America/Los_Angeles');` to your timezone in
*metar-placefile.php*, and *get-metar-metadata.php*; 
and `$ourTZ = 'America/Los_Angeles';` in *get-aviation-metars.php*
to your timezone if needed.

Upload the following files in a directory under the document root of your website.  (We used 'placefiles' in the examples below)

- *get-metar-metadata.php*
- *get-aviation-metars.php*
- *metar-placefile.php*
- *metar-cond-iconcodes-inc.php*
- *cloudcover_new.png*
- *windbarbs_75_new.png*
  
Set up cron to run *get-metar-metadata.php* like:
```
1 1 * * * cd $HOME/public_html/placefiles;php -q get-metar-metadata.php > metadata-status.txt
```
be sure to change the public_html/placefiles to the directory where you installed the scripts.

Run the script once (to generate data) by `https://your.website.com/placefiles/get-metar-metadata.php`

Set up cron to run *get-aviation-metars.php* like:

```
*/10 * * * * cd $HOME/public_html/placefiles;php -q get-aviation-metars.php > metar-status.txt
```
be sure to change the public_html/placefiles to the directory where you installed the scripts.

Run the script once (to generate data) by `https://your.website.com/placefiles/get-aviation-metars.php`

Then you can test the metar-placefile script by using your browser to go to
`https://your.website.com/placefiles/metar-placefile.php?dpi=96&lat=37.0&lon=-122.0`

If that returns a placefile, then add your placefile URL into the GRLevelX placefile
manager window.

Sample *metar-placefile.php* output:
```
; placefile with conditions generated by metar-placefile.php V1.01 - 01-Jul-2023 - webmaster@saratoga-weather.org
; Generated on Sat, 01 Jul 2023 19:47:39 +0000
;
Title: METAR Surface Observations - Sat, 01 Jul 2023 19:47:39 +0000 
Refresh: 7
Color: 200 200 255
Font: 1, 12, 1, Arial
IconFile: 1, 43, 68, 29, 67, windbarbs_75_new.png
IconFile: 2, 15, 15, 8, 8, cloudcover_new.png
Threshold: 999

; generate K06C Schaumburg, Illinois, USA at 41.98,-88.1 at 25 miles N 
Object: 41.98,-88.1
Threshold: 999
Icon: 0,0,260,1,1
Icon: 0,0,000,2,1,"Schaumburg, Illinois, USA (41.98,-88.1 elev 797 ft)\n----------------------------------------------------------\nK06C 011935Z AUTO 26003KT 10SM OVC120 27/23 A2984 RMK AO2 T02670232\n----------------------------------------------------------\nOBStime: 01-Jul-2023 12:35pm PDT (19:35Z)\nTemp: 80F (27C)\nDewPt: 74F (23C)\nHumid: 81%\nHeatIdx: 84F (29C)\nWind: W at 3 mph(6 km/h)\nVisib: 10.0 sm (16 km)\nCond: Overcast\nSky: Overcast 12000 ft\nAltim: 29.84 inHg (1010.5 hPa)\nWXcodes: OVC\n----------------------------------------------------------"
Text: -17, 13, 1, 80
Text: -17, -13, 1, 74
Text: -27, 0, 1, 10.0
Text: 17, 13, 1, 1011
Text: 17, -13, 1, K06C
End:

; generate K1H2 Effingham, Illinois, USA at 39.07,-88.53 at 178 miles S 
Object: 39.07,-88.53
Threshold: 999
Icon: 0,0,170,1,1
Icon: 0,0,000,2,3,"Effingham, Illinois, USA (39.07,-88.53 elev 571 ft)\n----------------------------------------------------------\nK1H2 011935Z AUTO 17003KT 10SM SCT028 29/23 A2988 RMK AO2 T02930231\n----------------------------------------------------------\nOBStime: 01-Jul-2023 12:35pm PDT (19:35Z)\nTemp: 85F (29C)\nDewPt: 74F (23C)\nHumid: 69%\nHeatIdx: 92F (33C)\nWind: S at 3 mph(6 km/h)\nVisib: 10.0 sm (16 km)\nCond: Partly Cloudy\nSky: Partly Cloudy 2800 ft\nAltim: 29.88 inHg (1011.8 hPa)\nWXcodes: SCT\n----------------------------------------------------------"
Text: -17, 13, 1, 85
Text: -17, -13, 1, 74
Text: -27, 0, 1, 10.0
Text: 17, 13, 1, 1012
Text: 17, -13, 1, K1H2
End:
```


## Acknowledgements

Special thanks to Mike Davis, W1ARN of the National Weather Service, Nashville TN office
for the *windbarbs_75_new.png* and *cloudcover_new.png* icon sheets,
the METAR weather conditions to icon code mapping, 
preliminary placefile output example, 
and for his testing/feedback during development.   


Some code in *get-aviation-metars.php* and *metar-cond-iconcodes-inc.php* are
adapted from https://github.com/pear/Services_Weather/blob/trunk/Weather/Common.php
by Alexander Wirtz with Copyright (c) 2005-2011 and is used under the
permitted redistribution instructions. 

## Sample screen capture

![sample-output](https://github.com/ktrue/metar-placefile/assets/17507343/4fbe3e21-72e1-40b1-80a3-1a9a235e3a62)
