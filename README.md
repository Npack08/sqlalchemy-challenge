# sqlalchemy-challenge
%matplotlib inline
from matplotlib import style
style.use('fivethirtyeight')
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import datetime as dt
Reflect Tables into SQLAlchemy ORM
# Python SQL toolkit and Object Relational Mapper
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func
# create engine to hawaii.sqlite
engine = create_engine("sqlite:///Resources/hawaii.sqlite")
# reflect an existing database into a new model
Base = automap_base()
Base.prepare(autoload_with=engine)

# reflect the tables

Base.classes.keys()
['measurement', 'station']
# View all of the classes that automap found
Base.classes.keys()
['measurement', 'station']
# Save references to each table
Measurement = Base.classes.measurement
Station = Base.classes.station
# Create our session (link) from Python to the DB
session = Session(engine)
Exploratory Precipitation Analysis
# Find the most recent date in the data set.

session.query(Measurement.date).order_by(Measurement.date.desc()).first()
('2017-08-23',)
# Design a query to retrieve the last 12 months of precipitation data and plot the results. 
# Starting from the most recent data point in the database. 
recent_date = session.query(Measurement.date).order_by(Measurement.date.desc()).first() 

# Calculate the date one year from the last date in data set.
year_ago = dt.date(2017,8,23) - dt.timedelta(days=365)
year_ago
# Perform a query to retrieve the data and precipitation scores
prcp_data = session.query(Measurement.date, Measurement.prcp).\
    filter(Measurement.date >= year_ago).all()
prcp_data

# Save the query results as a Pandas DataFrame and set the index to the date column
prcp_df = pd.DataFrame(prcp_data, columns = ['date', 'precipitation'])
prcp_df['date'] = prcp_df['date'].astype("datetime64")
prcp_df.set_index('date', inplace=True)
prcp_df
# Sort the dataframe by date
prcp_df.sort_index(inplace=True)


# Use Pandas Plotting with Matplotlib to plot the data
prcp_df.plot()
plt.xlabel('date')
plt.ylabel('inches')
plt.tight_layout()
plt.show()

# Use Pandas to calcualte the summary statistics for the precipitation data
prcp_stat= prcp_df['precipitation'].describe()
prcp_stat
count    2021.000000
mean        0.177279
std         0.461190
min         0.000000
25%         0.000000
50%         0.020000
75%         0.130000
max         6.700000
Name: precipitation, dtype: float64
Exploratory Station Analysis
# Design a query to calculate the total number stations in the dataset
total_stations = session.query(func.count(Station.station)).all()
total_stations
[(9,)]
# Design a query to find the most active stations (i.e. what stations have the most rows?)
# List the stations and the counts in descending order.
most_active_stations= session.query(Measurement.station, func.count(Measurement.station))\
.group_by(Measurement.station)\
.order_by(func.count(Measurement.station).desc()).all()
most_active_stations
[('USC00519281', 2772),
 ('USC00519397', 2724),
 ('USC00513117', 2709),
 ('USC00519523', 2669),
 ('USC00516128', 2612),
 ('USC00514830', 2202),
 ('USC00511918', 1979),
 ('USC00517948', 1372),
 ('USC00518838', 511)]
# Using the most active station id from the previous query, calculate the lowest, highest, and average temperature.
station_stat = session.query(func.max(Measurement.tobs), func.min(Measurement.tobs), func.avg(Measurement.tobs)).\
filter(Measurement.station=='USC00519281').all()
print(station_stat)
[(85.0, 54.0, 71.66378066378067)]
# Using the most active station id
# Query the last 12 months of temperature observation data for this station and plot the results as a histogram
station_obs = session.query(Measurement.tobs).\
filter(Measurement.station =='USC00519281').\
filter(Measurement.date >= year_ago).all()

station_data_list = [tobs[0] for tobs in station_obs]

plt.hist(station_data_list, bins =12, label='tobs')
plt.xlabel('Temperature')
plt.ylabel('Frequency')
plt.legend()
plt.show()

Close session
# Close Session
session.close()
