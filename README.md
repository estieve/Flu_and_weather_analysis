# Flu_and_weather_analysis

## Compiling the Data

To complete this project I had to collect and combine data from multiple sources. There are two primary pieces: 1) the data on flu infections, and 2) the weather data. For the weather data I accessed the API from the National Weather Service (https://www.ncdc.noaa.gov/cdo-web/). The code in my "Compile_Data" notebook was used to pull temperature and precipitation data for a span ranging from 2009-2019. 

### Weather Data
The NOAA api only allows for 365 day of data in a single pull with one attribute, to get around this I built a series of loops to get three attributes per year and then combine the 10 year span into a single dataset. I created the tool in such a way that I could repeat the process easily with different stations to create a compiled list for more than one location. Unfortunately there are both gaps in the datasets where no records are available for specific days, and some outliers where egregious values are present such as precipitation totals greater than 400 inches and temperatures higher than 390°! An example of this is seen below from the Malta weather data.

![WeatherMal_outliers](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/WeatherMal_outliers.PNG)

These are things that I had to deal with in the weather dataset before I could move forward with adding the flu data. To do so, I removed the egregious values by making then Nan's and reseting exceptionally low values to a specified minimum (0 for precipitation and the location's lowest seasonal temperature for temp). The Nan's I then reset to a mean value since there was a wide distribution of Nan data and filling forward created high plateau peaks of data. 

![WeatherMal_corrected](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/WeatherMal_corrected.PNG)

The New York weather had to be dealt with in a slightly different way. The problem there was that we had significant outliers prior to 2018, if I addressed it in the same way as the other locations we would see something that looked like this:

![NY_precip_prob](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/NY_precip_prob.PNG)

This appeared to me to be a scaling issue with the data so rather than removing high values I scaled records above 1 inch to align better. I did this by creating a new column that was the log of the precipitation values and used the records in this value to replace corresponding records greater than 1 inch. While this isn't perfect it provides something that more closely mirrors what we might expect.

![NY_precip_corre](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/NY_precip_corre.PNG)

The final step involved prepping the data for merging with the weekly flu data, to do this I converted our dataframe of daily records into a new dataframe that calculated weekly averages for our attributes. I then converted the precipitation data to a weekly total because I felt that better represents the amount of precipitation for a given week.

### Flu Data
The influenza data was collected from two different sources, the WHO FluNet (https://www.who.int/influenza/gisrs_laboratory/flunet/en/)surveillance system and from the New York State Department of Health (https://health.data.ny.gov/Health/Influenza-Laboratory-Confirmed-Cases-By-County-Beg/jr8b-6gh6/data). These were extracted as csv files directly from the websites and I chose data for New York City, Luxembourg, and Malta. All of the associated files are in the repo as LUX_Flu, Mal_Flu, and NY_FLU. This data required less cleaning and primarily consisted of removing extraneous columns to get down to the core factors which were end date for the specified week and the count for number of positive flu cases.

### Flu and Weather
I created an output dataframe for each location that included the key weather and flu data joined on the date column. With that completed I then joined them together into a single dataframe titled "flu_weather". I exported this as a csv to use in my data analysis.

## EDA
Below are some time-series correlations from each location between the features and the target (flu count). Here I was just looking for notional relationships to see if anything stood out.

### Precipitation
![Precip_NY](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Precip_NY.PNG)
![Precip_lux](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Precip_lux.PNG)
![Precip_mal](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Precip_mal.PNG)

### Max Temp
![Max_temp_NY](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Max_temp_NY.PNG)
![Max_temp_lux](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Max_temp_lux.PNG)
![Max_temp_mal](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Max_temp_mal.PNG)

### Distribution of Flu Counts
Next we take a look at the distribution of the data relative to the flu counts.

![Flu_unbalanced](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Flu_unbalanced.PNG)

Not surprisingly this is a highly unbalanced dataset. To correct this I created three categories based on the value in the counts column. The categories are defined as follows:
  
  Category 0 = 0
  
  Category 1 = 1-99
  
  Category 2 = 100+

![Flu_balanced](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Flu_balanced.PNG)
  
This looks much better!

### Correlation Matrix

![corr_plot](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/corr_plot.PNG)

## Data Analysis

Before beginning any analysis I set up the test and training data and then utilized a standard scaler to scale the feature data between -1 and 1. Next my plan was to run various machine learning algorithms to test for the best options. In this situation I will be running classifiers with decision trees, K-nearest neighbors, and neural networks.

### Decision Tree
I started with a decision tree to help identify importance features just to test my assumptions that percipitation and max temperature are important features.

    totPrecip:  0.39
    
    avgTmax: .. 0.35

    avgTmin: .. 0.27

By visualizing the decision tree we can check for overfitting.

![tree_unpruned](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/tree_unpruned.png)

There are a lot of branches in this tree so I ran it through pruning to fine tune the parameters. I did this through a series of efforts to test the optimal values for:

    Max_depth
    min_samples_split
    min_samples_leaf
    max_leaf_nodes
    min_impurity_decrease
    
The values for the optimal solution turned out to be:

    pruned_model = RandomForestClassifier(max_depth=3,
                                          min_samples_split=30,
                                          min_samples_leaf=10,
                                          max_leaf_nodes=4,
                                          min_impurity_decrease=0.02,
                                          random_state=35)
                                          
The results of our model:

      [[  0  89   1]
       [  0 138   3]
       [  0  55  33]]
      Accuracy Score : 0.5360501567398119

                    precision    recall  f1-score   support

             0.0       0.00      0.00      0.00        90
             1.0       0.49      0.98      0.65       141
             2.0       0.89      0.38      0.53        88

        accuracy                           0.54       319
       macro avg       0.46      0.45      0.39       319
    weighted avg       0.46      0.54      0.43       319

![dec_tree](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/dec_tree.png)

Then I ran a random forest model with the same parameters and received the following results:

        [[  0  87   3]
         [  0 133   8]
         [  0  34  54]]
        Accuracy Score : 0.5862068965517241

                   precision    recall  f1-score   support

             0.0       0.00      0.00      0.00        90
             1.0       0.52      0.94      0.67       141
             2.0       0.83      0.61      0.71        88

        accuracy                           0.59       319
       macro avg       0.45      0.52      0.46       319
    weighted avg       0.46      0.59      0.49       319


### KNN

Next I tested utilizing K-nearest neighbors. To start I tested against different values for k between 2 and 19.

![corr_plot](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/KNN_neighbors.PNG)

k=5 proved to be our optimal option so I ran the following model on the data:

    model = KNeighborsClassifier(n_neighbors=5, n_jobs=-1)

Our confusion matrix and accuracy scores:

     [[51 39  0]
      [36 99  6]
      [ 1 29 58]]
     Accuracy Score : 0.6520376175548589
     
                    precision    recall  f1-score   support

              0.0       0.58      0.57      0.57        90
              1.0       0.59      0.70      0.64       141
              2.0       0.91      0.66      0.76        88

         accuracy                           0.65       319
        macro avg       0.69      0.64      0.66       319
     weighted avg       0.68      0.65      0.66       319


### Neural Network
The last assessment utilizes a neural network. I ran tuning against the MLP Classifier utilizing the following parameters:

    'max_iter':[100, 500, 1000],
    'hidden_layer_sizes': [(5,), (10,), (20,)],
    'activation': ['relu', 'tanh'],
    'solver': ['sgd', 'adam'],
    'alpha': [0.0001, 0.01, 0.05],
    'learning_rate': ['constant', 'invscaling', 'adaptive'],

The optimal parameters achieved for my model were:

    {'activation': 'tanh', 'alpha': 0.05, 'hidden_layer_sizes': (20,), 'learning_rate': 'constant', 'max_iter': 1000, 'solver': 'sgd'}
    
Here is the final model:

    model = MLPClassifier(hidden_layer_sizes=(20,), 
                      activation='tanh', 
                      alpha=0.05, 
                      learning_rate='constant', 
                      solver='sgd', 
                      max_iter=1000)
                      
Our confusion matrix and accuracy scores:

     [[ 16  73   1]
      [ 20 108  13]
      [  1  22  65]]
     Accuracy Score : 0.5924764890282131
     
     
     Results on the test set:
                    precision    recall  f1-score   support

              0.0       0.43      0.18      0.25        90
              1.0       0.53      0.77      0.63       141
              2.0       0.82      0.74      0.78        88

         accuracy                           0.59       319
        macro avg       0.60      0.56      0.55       319
     weighted avg       0.58      0.59      0.56       319
