# HADashboard-widgets
Widgets for HADashboard / Appdaemon

This is a graph widget for HADashboard that uses influxdb as data source and plot.ly as the graph engine.

![HADashboard-widgets](https://github.com/tjntomas/HADashboard-widgets/blob/master/img/influx_graph.png?raw=true)
The image shows six different widgets with one or more time series for each widget.

Features:
* Add any number of traces to a graph
* Zoom and scroll
* hide/show individual traces
* Automatically updates when new values are written  to influx/when the entity is updated in Home Assistant
* Can span over any number of columns
* The height can be set for each widget
* Supply your own SQL query to filter and group data

To use it, you need to:
1. Have influxdb installed and configured in Home Assistant, see instructions here:  https://www.home-assistant.io/components/influxdb/
Example configuration for influxdb in configuration.yaml:
````yaml
influxdb:
  host: 127.0.0.1
  include: 
    entities:
    - sensor.neo_coolcam_power_plug_12a_voltage
    - sensor.pi3_cpu_temp
    - sensor.pi3_cpu_usage
    - sensor.cpu_temp_pi2
    - sensor.cpu_usage_pi2
    - sensor.cputemp
 ````
 By specifiying only to log the entities that you actually use, the influxdb size can be kept to a minimum. Once everything is set up, it will take some time before you have enough data in influxdb to make a graph.
 
2. Copy the basegraph folder to your custom_widgets folder
3. Copy the graph.yaml file to your custom_widgets folder
4. Copy influxdb-latest.js and plotly-latest.min.js (from the custom_css folder)  to your custom_css folder
5. Add the following to your variables.yaml file in your custom_css folder for the skin you are using:
````yaml
graph_style: "border-radius: 0px; "background-color: rgba(30,30,30,0.45 );"  # Change to whichever color/opacity you like.
graph_legend_text_color: "#888888"
graph_grid_color: "#888"
graph_influxdb_path: http://<URL_TO_YOUR_INFLUXDB_SERVER>:8086 # Example: 192.168.1.20:8086  or http://www.mydomain.com:8086
graph_influxdb_path_local: "http://<_LOCAL_URL_TO_YOUR_INFLUXDB_SERVER>:8086"
graph_widget_style: "border-bottom-left-radius: 10px;border-bottom-right-radius: §;border-top-left-radius: 10px;border-top-right-radius: 10px;"
graph_trace_colors: "1"  # Set the opacity for the trace colors.
graph_fill_colors: "1"   # Set the opacity for the fill colors.
graph_bar_colors: "1"    # Set the opacity for the bar colors.
graph_bar_multi: "1"     # Leave this as is.
````
6. Define a widget:
````yaml
# Example of the most simple widget definition showing a single entity:
mvp:
    widget_type: graph
    entities:
      - sensor.indoor_temperature  # The entity_id to be plotted.
    influxdb_units: 
      - "°C"   # The unit_of_measurement attribute for the entity.
    titles:
      - "Indoor temperature" # Title of the trace.
    time: 24h   # Time interval to plot. You can use "m" for minutes, "h" for hours, "d" for days and "w" for weeks.
    title: "Temperatures"  # Widget title
    fill: "tozeroy" # options are  "none" | "tozeroy" | "tozerox" | "tonexty" | "tonextx" | "toself" 
    colorIndex: 0   # A number between 0 and 11. 12 colors for the traces are predefined and the colorIndex defines which is used for the first trace. If more than 10 traces/entities are specified, the colors are rotated. 

# A more complex graph showing 4 CPU temperatures.
cpu_temp_g:
    widget_type: graph
    entities:   # You can add any number of entities to the list.
      - sensor.pi3_cpu_temp
      - sensor.cpu_temp_pi2
      - sensor.ubuntu_core_0_temp
      - sensor.ubuntu_core_1_temp
    influxdb_units: # You need to add the unit_of_measurement for the entity_id's here. 
      - "°C"
      - "°C"
      - "°C"
      - "°C"
    titles:   # Specify the title for each trace. 
      - "RPI 3"
      - "VPN"
      - "UBUNTU 0"
      - "UBUNTU 1"
    time: 1h  # Specify time interval for the traces. can be anything that influxdb accepts, i.e. 20m, 2d, 4h, 1w etc.
    samples: 200  # Optional. Used to speed up drawing. If your widget is 200 pixels wide, use 200.
    title: "CPU temperatures"  # The title for the widget.
    fill: "tozeroy"
    max: 75                    # Set the max y-axis. Remove to fit the traces automatically.
    min: 30                    # Set the min y-axis. Remove to fit the traces automatically.
    colorIndex: 0              # Set a color index between 0 and 7.
    height: 324                # Specify the height of the widget in pixels. 
    
# Graph to show the daily power consumption from a Sparsnäs device.
# In this widget, we specify the SQL query explicitly instead of just grabbing the trace from influxdb.
# In this way, we can use all features of the SQL query language that influxdb supports.
power_usage_per_day:
    widget_type: graph
    entities:
      - sensor.sparsnas_energy_consumption_over_time
    time: 24h
    samples: 200
    title: "Daily energy consumption"
    colorIndex: 7
    min: 0
    fill: "tozeroy"  # You can find more fill options at plot.ly 
    shape: "hv"      # You can find more shape options at plot.ly 
    sql: "select difference(last(value)) from kWh where entity_id = 'sparsnas_energy_consumption_over_time' and time > now() - 2w  group by time(1d)"
    type: "bar"
    decimals: 0
#Addional parameters:
    time_zone: "Europe/Stockholm
    locale: "se"
    db_name: "home_assistant"
    ds: 5m   # Used for downsampling long time intervals in the SQL query. Use anything that influxdb accepts, i.e. 5m, 1h, 2d etc.
````

7. Add the widget to your dashboard.yaml file 

Reading long time series from influxdb can be time consuming so to speed things up, there is an option to downsample the data read by adding "ds", short for downsample,  to the widget definition. To speed up the drawing or to make the graph more or less course, "samples" can also be added to the widget definition. 

Here is an example using both ds and samples:
````yaml
mvp:
    widget_type: graph
    entities:
      - sensor.indoor_temperature
    influxdb_units: 
      - "°C"
    titles:
      - "Indoor temperature"
    time: 24h
    title: "Temperatures"
    fill: "tozeroy"
    colorIndex: 0
    ds: 30m  # Now, only one sample for every 30 minutes is read from influxdb.
    samples: 200
````

# Writing custom SQL queries
If you are familiar with the SQL language, you can write your own query. To do this, start influx by typing "influx" in a shell session.
Then select the home assistant database by typing "use home_assistant" and press enter.
Now you can write an SQL query. Try starting with: select * from "°C";  and press enter. The semicolon at the end is required.
The result should be the history for all your temperature device.

When you are happy with your query, just add the query to your widget definition:
````yaml
sql: select * from "°C" where entity-id='sensor.my_temperature_sensor' and value > 20 and time > now() - 1d
````

When you supply a custom SQL query, the entity_id is used by the widget only to listen for state change and update your widget when new data is available for your query.




