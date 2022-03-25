# NYC Taxi Demand Prediction (in-next-10mins)

## Data Information:

Ge the data from : http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml </br>
The data used in the attached datasets were collected and provided to the NYC Taxi and Limousine Commission (TLC)

The dataset used is of Yellow Taxi (Yellow Medallion Taxicabs).

### Yellow Taxi:
These are the famous NYC yellow taxis that provide transportation exclusively through street-hails. The number of taxicabs is limited by a finite number of medallions issued by the TLC. You access this mode of transportation by standing in the street and hailing an available taxi with your hand. The pickups are not pre-arranged.


##### In the given project I'm considering only the yellow taxis for the time period between Jan - Mar 2015 & Jan - Mar 2016

# ML Problem Formulation
Time-series forecasting and Regression


- To find number of pickups, given location cordinates(latitude and longitude) and time, in the query reigion and surrounding regions.
- To solve the above we would be using data collected in Jan - Mar 2015 to predict the pickups in Jan - Mar 2016.

### Performance metrics:
> Mean Absolute percentage error.
> Mean Squared error.
