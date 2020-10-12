# **BlueSky Verification - Methodology**
In the summer of 2020 I was tasked with completing the verification of BlueSky Canada forecasts in quasi-realtime that Tim Atkinson had started several months earlier. The three main parts of the site, which I will describe in detail below, are the Verification Map, Verification Statistics, and the Individual Station Traces. At the moment this is only operational for stations in British Columbia, but the hope is to expand that into Alberta and potentially other provinces if there is interest. 

### **Individual Station Traces**
The individual station verification displays the raw observations of PM2.5  and the BlueSky smoke forecast + background PM2.5 at all stations in British Columbia.

### **Statistics**
These statistics are useful to analyze how well the forecast has done on any given day. There are 5 statistics shown for each day: the mean absolute error, the root mean squared error, the accuracy, the hit rate, and the false alarm rate.

### **Verification Map**
There is a map showing the absolute difference between PM2.5 in the BlueSky Canada forecast and the PM2.5 observed at stations throughout BC. It can be interpreted as how well the smoke forecast did at observation stations for different windows in time and space, currently in BC. 

### **Files** 

#### **Python**
* `bczones.py`
    * splits BC into three zones
    * adds column 'zone' to df_mod_obs_spatial 
    * also adds column 'zone' to df_mod_obs_interp
* `bicubicspline.py` (not used operationally)
    * calculates bicubic spline interpolation
* `bluesky.py`
    * grabs and formats bluesky model data
    * outputs 3d array called 'mod_PM25' = concentrations forecasted by bluesky model 
    * defines starting_year and starting_time of forecast 
    * creates dataframe called DateTime - times in the forecast
* `gridpointaverage.py`
    * calculates grid point average interpolation
    * to be used as bluesky forecast in the individual traces script
* `interpolation_schemes.py`
    * adds grid point average column to the interpolation data dataframe 
        * used in verification map and for calculating statistics
        * add 10 to NinePoint_GridPointAverage column to account for background smoke
    * creates second dataframe called interpolaton_data_for_plots 
        * used in plotting individual station traces  
* `nearestneighbour.py` (not used operationally)
    * calculates nearest neighbour interpolation
* `observations.py`
    * prepares observations from across BC for processing
    * looks at the year that BlueSky data is from, grab station observations from BC webpage
    * if data is from 2018/2019 - import station metadata
    * merge data and metadata if necessary 
* `plot_interpolation.py`
    * renames columns (only way i could figure out to change legend names for plots) 
    * sets up the data frame for plotting
    * plots the indiviudal station traces and outputs to html file
* `run_verification.py`
    * control script
    * defines all command-line flags
    * sets up logging
    * sets up symlink
    * executes plotting files, copies html files into the output directories 
* `showBC.py`
    * combines observation data nad bluesky forecast data
    * function llpad performs time and spatial window padding to forecast data 
* `smoke_category_map`
    * changes formats of columns and prepares dataframe for plotting
    * plot the verification map, write it to html file
* `stats.py`
    * MAE and RMSE are calculated for all stations + bc zones
    * accuracy, hit rate, false alarm rate calculated
    * stats are plotted and written to html files
#### **Bash**  
* `run_verification.sh`
    * runs `run_verification.py` with combinations of time and spatial windows
#### **HTML** 
* `index.html`
    * creates webpage for statistics/individual station traces/verification map
#### **Config**
* `default.cfg`
    * config file



### Example Run from BlueSky2 
1. cd bluesky/verification
2. source venv/bin/activate
3. cd bin
4. ./run_verification.sh 

#### Optional Command-Line Flags
**For operational work and to make it easier to test values or change values, I have defined 11 command-line flags. All of these are optional arguments and have default values:**
* —forecast_start
    * Start time for the forecast you are interested in. Example: 2020080108 (August 1st, 2020 at 8 am)
    * Default value is yesterday.
* —forecast_window
    * Define the length of the forecast that you would like to look at. 
    * Default value is 24 hours
* —time_window_input
    * Time window that you would like to look at (in hours).
    * Default value is 5 hours, but several are run in the bash script that should soon be operational. 
* —spatial_window_input
    * Spatial window that you would like to look at (in degrees).
    * Default value is 0.5 degrees, but several are run in the bash script that should soon be operational. 
* —good_threshold_input
    * Number that defines a good forecast threshold (ug/m^3). 
    * Default value is 50 ug/m^3 to match table 1 above. NOTE: This value is not currently used anywhere but can be easily implemented if anyone wants forecasts to be concretely categorized again.
* —fair_threshold_input
    * Number that defines a fair forecast threshold (ug/m^3). 
    * Default value is 80 ug/m^3 to match table 1 above. NOTE: This value is not currently used anywhere but can be easily implemented if anyone wants forecasts to be concretely categorized again. 
* —smoke_event_threshold_input
    * Value that defines a smoke event (ug/m^3).
    * Default is 10 ug/m^3
* —no_smoke_input
    * Value that defines what counts as smoke (ug/m^3)
    * Default is 0 ug/m^3
    * This value is not currently used as the background smoke values are defined throughout the scripts in different ways. 
* —log_level_string
    * Defines the level of logging you’d like to do.
    * Default is “info”.
* —log_directory_string
    * Change the output location (directory only!) of the log file.
    * Default is “”, but is defined below as /bluesky/verification/log
* —forecast_archive_string 
    * Change the forecast archive directory. 
    * Default is “”, but is defined below as /bluesky/archive/forecast. 

#### **Output**
The output of these runs is saved within /bluesky/archive/verification/forecast_id/forecast_start_time. For example, the output for August 20th, 2020 will be saved in /bluesky/archive/verification/BSC00CA12p/2020082008. The output is several html pages containng the different plots (statistics, traces, map) and they get combined into one webapge (index.html).