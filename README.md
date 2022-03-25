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
1. Mean Absolute percentage error.
2. Mean Squared error.

## Data Cleaning:
In this section we will be doing univariate analysis and removing outlier/illegitimate values which may be caused due to some error.
1. Latiude and Longitude: Droping all the rows whose latitude and longitude is out of NewYork. It is inferred from the source https://www.flickr.com/places/info/2459115 that New York is bounded by the location cordinates(lat,long) - (40.5774, -74.15) & (40.9176,-73.7004) so hence any cordinates not within these cordinates are not considered by us as we are only concerned with pickups which originate within New York.
2. Trip Duration: According to NYC Taxi & Limousine Commision Regulations the maximum allowed trip duration in a 24 hour interval is 12 hours. So, any trip duration less than 1 minute or higher than 720min (12hrs) will be dropped.
3. Speed: From the data I can see the 99.9 percentile of cabs have speed lessthan 45.3 miles/hr, however the average is 12.45 miles/hour. Therefore, I will be dropping all the rows where speed is less than 0 miles/hr or greater than 45.3 miles/hr.
4. Trip Distance: Similar to Speed I followed percentile approch and I found around 99.9 percentile values have trip distance less than 22.57. So here I will be dropping all rows with trip distance greater than 23 miles.
5. Total Fare: I calculate percentile and even the 99.9th percentile value doesnt look like an outlier The 99.9 percentile of total fare is 95.55, however 100 percentile is 3950611.6. This signifies there is some extreme outlier which will be dropped.

# Data Preparation:
## 1. Clustering & Segmentation (using KMeans clustering)
- The main objective was to find a optimal min. distance(Which roughly estimates to the radius of a cluster) between the clusters which we got was 40. 
## 2. Time Binning:
I will be segmenting whole period into segment of 10 min. This will be called as bin. As the time provided is in datetime format which I will converting into UNIX timestamp before binning. </br>
Here I will be creating two new features:
  1. pickup_cluster  -  to which cluster it belongs to
  2. pickup_bins  -  to which 10min intravel the trip belongs to
## 3. Smoothing:
It is a possible that there would be few bins which does not have any pickups. To handle this case I will be taking two approaches:
  1. Fill the missing value with 0's
  2. Fill the missing values with the avg values
      - Case 1:(values missing at the start)
          Ex1: \_ \_ \_ x =>ceil(x/4), ceil(x/4), ceil(x/4), ceil(x/4)
          Ex2: \_ \_ x => ceil(x/3), ceil(x/3), ceil(x/3)
      - Case 2:(values missing in middle)
          Ex1: x \_ \_ y => ceil((x+y)/4), ceil((x+y)/4), ceil((x+y)/4), ceil((x+y)/4)
          Ex2: x \_ \_ \_ y => ceil((x+y)/5), ceil((x+y)/5), ceil((x+y)/5), ceil((x+y)/5), ceil((x+y)/5)
      - Case 3:(values missing at the end)
          Ex1: x \_ \_ \_ => ceil(x/4), ceil(x/4), ceil(x/4), ceil(x/4)
          Ex2: x \_ => ceil(x/2), ceil(x/2)
