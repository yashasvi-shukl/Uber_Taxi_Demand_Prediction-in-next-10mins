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
          1. Ex1: \_ \_ \_ x =>ceil(x/4), ceil(x/4), ceil(x/4), ceil(x/4)
          2. Ex2: \_ \_ x => ceil(x/3), ceil(x/3), ceil(x/3)
      - Case 2:(values missing in middle)
          1. Ex1: x \_ \_ y => ceil((x+y)/4), ceil((x+y)/4), ceil((x+y)/4), ceil((x+y)/4)
          2. Ex2: x \_ \_ \_ y => ceil((x+y)/5), ceil((x+y)/5), ceil((x+y)/5), ceil((x+y)/5), ceil((x+y)/5)
      - Case 3:(values missing at the end)
          1. Ex1: x \_ \_ \_ => ceil(x/4), ceil(x/4), ceil(x/4), ceil(x/4)
          2. Ex2: x \_ => ceil(x/2), ceil(x/2)
      </br>
      <h5>Smoothing Vs Filling</h5>
![image](https://user-images.githubusercontent.com/22805226/160099426-6375110f-47c1-4410-b2e8-b307d5c06324.png)
</br>
      <h6>why we choose these methods and which method is used for which data?</h6>
      Ans: consider we have data of some month in 2015 jan 1st, 10 _ _ _ _ 20, i.e there are 10 pickups that are happened in 1st 10min intravel, 0 pickups happened in 2nd 10mins intravel, 0 pickups happened in 3rd & 4th 10min intravel and 20 pickups happened in 5th 10min intravel. In fill_missing method we replace these values like 10, 0, 0, 0, 20 whereas in smoothing method we replace these values with average as 6,6,6,6,6 ((10+20)/5) if you can check the number of pickups that are happened in the first 50min are same in both cases, but if you can observe that we looking at the future values. when you are using smoothing we are looking at the future number of pickups which might cause a data leakage. So we use smoothing for jan 2015th data since it acts as our training data and we use simple fill_misssing method for 2016th data.
        
# Modeling
## Baseline Models:
  Now within modelling, in order to forecast the pickup densities for the months of Jan, Feb and March of 2016 for which I'm using multiple models with two variations:</br>
  - **Using Ratios** of the 2016 data to the 2015 data i.e  **Rt=P2016t/P2015t**
  - **Using Previous known values** of the 2016 data itself to predict the future values
  
  1. **Simple Moving Averages:**
  The First Model used is the Moving Averages Model which uses the previous n values in order to predict the next value.
      - **Using Ratio Values -  Rt=(Rt−1+Rt−2+Rt−3....Rt−n)/n** <br>
      The best window-size obtained after hyperparameter tuning is 3.
      - Next we use the Moving averages of the 2016 values itself (i.e., on previous data) to predict the future value using </br>
      **Pt=(Pt−1+Pt−2+Pt−3....Pt−n)/n** </br>
        The best window-size obtained after hyperparameter tuning is 1.
   2. **Weighted Moving Averages:**
        The Moving Avergaes Model used gave equal importance to all the values in the window used, but we know intuitively that the future is more likely to be similar to the latest values and less similar to the older values. Weighted Averages converts this analogy into a mathematical relationship giving the highest weight while computing the averages to the latest previous value and decreasing weights to the subsequent older ones.
        - **Weighted Moving Averages using Ratio Values -  Rt=(N∗Rt−1+(N−1)∗Rt−2+(N−2)∗Rt−3....1∗Rt−n)/(N∗(N+1)/2)**
        The best window-size obtained after hyperparameter tuning is 5.
        
        - **using Previous Values - Pt=(N∗Pt−1+(N−1)∗Pt−2+(N−2)∗Pt−3....1∗Pt−n)/(N∗(N+1)/2)** 
        The optimal window-size obatained after hyperparameter tuning is 5.
  3. **Exponential Weighted Moving Average:**
      <p>Through weighted averaged we have satisfied the analogy of giving higher weights to the latest value and decreasing weights to the subsequent ones but we still do not know which is the correct weighting scheme as there are infinetly many possibilities in which we can assign weights in a non-increasing order and tune the the hyperparameter window-size. To simplify this process we use Exponential Moving Averages which is a more logical way towards assigning weights and at the same time also using an optimal window-size.
              In exponential moving averages we use a single hyperparameter alpha  (α)  which is a value between 0 & 1 and based on the value of the hyperparameter alpha the weights and the window sizes are configured. </p>
              
        **R′t=α∗Rt−1+(1−α)∗R′t−1**  where alpha range from 0 to 1
</br>
===========================================================================================
</br>
<h3>Comparison between baseline models</h3>
Plese Note:- Below comparisons is made using Jan 2015 and Jan 2016 only
</br></br>

![image](https://user-images.githubusercontent.com/22805226/160114942-f0bee07a-ceeb-48eb-bcaf-cfe184260f99.png)
</br>

# Regression Models
1. <h2>Train Test Split:</h2>
    <p>Before we start predictions using the tree based regression models we take 3 months of 2016 pickup data and split it such that for every region we have 70% data in train and 30% in test, ordered date-wise for every region. As we have observed Exponential Weighted Moving Average is working best of all with window-size of 5. To Get the predictions of exponential moving averages to be used as a feature in cumulative form.
  
      I have computed 8 features for every data point that starts from 50th min of the day
      1. cluster center lattitude
      2. cluster center longitude
      3. day of the week 
      4. f_t_1: number of pickups that are happened previous t-1th 10min intravel
      5. f_t_2: number of pickups that are happened previous t-2th 10min intravel
      6. f_t_3: number of pickups that are happened previous t-3th 10min intravel
      7. f_t_4: number of pickups that are happened previous t-4th 10min intravel
      8. f_t_5: number of pickups that are happened previous t-5th 10min intravel

2. Model trained are: 
    - Linear Regression
    - Random Forest
    - XGBoost

# Result:

![image](https://user-images.githubusercontent.com/22805226/160117758-4ec5bc60-a196-4d0c-8bdf-37ad642a95d7.png)

      - XGBoost working best of all other models.
      - Random Forest seems be overfitting somewhat as compared to other models. (Difference between train and test MAPE score)
