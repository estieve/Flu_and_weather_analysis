# Flu_and_weather_analysis

## Compiling the Data

To complete this project I had to collect and combine data from multiple sources. There are two primary pieces: 1) the data on flu infections, and 2) the weather data. For the weather data I accessed the API from the National Weather Service (https://www.ncdc.noaa.gov/cdo-web/). The code in my "Compile_Data" notebook was used to pull temperature and precipitation data for a span ranging from 2009-2019. 

### Weather Data
The NOAA api only allows for 365 day of data in a single pull with one attribute, to get around this I built a series of loops to get three attributes per year and then combine the 10 year span into a single dataset. I created the tool in such a way that I could repeat the process easily with different stations to create a compiled list for more than one location. Unfortunately there are both gaps in the datasets where no records are available for specific days, and some outliers where egregious values are present such as precipitation totals greater than 400 inches and temperatures higher than 390°! An example of this is seen below from the Malta weather data.

![WeatherMal_outliers](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/WeatherMal_outliers.PNG)

These are things that I had to deal with in the weather dataset before I could move forward with adding the flu data. To do so, I removed the egregious values by making then Nan's and reseting exceptionally low values to a specified minimum (0 for precipitation and the location's lowest seasonal temperature for temp). The Nan's for temperature I then reset to a mean value since there was a wide distribution of Nan data and filling forward created high plateau peaks of data. For precipitation I set all values higher than 0.2 inches (the mean daily total) to 0.2 inches. This was not ideal, but setting to the mean value seemed to alter the variance in the data too much and this seemed like a reasonable middle ground.

![WeatherMal_corrected](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/WeatherMal_corrected.PNG)

The New York weather had to be dealt with in a slightly different way. The problem there was that we had significant outliers prior to 2018, if I addressed it in the same way as the other locations we would see something that looked like this:

![NY_precip_prob](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/NY_precip_prob.PNG)

This appeared to me to be a scaling issue with the data so rather than removing high values I scaled records above 0.2 inches (the average daily total for NYC) to align better. I did this by multiplying these values by 0.001. While this isn't perfect it provides something that more closely mirrors what we might expect.

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
  
  Category 0 = 0 cases
  
  Category 1 = 1-99 cases
  
  Category 2 = 100+ cases

![Flu_balanced](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/Flu_balanced.PNG)
  
This looks much better!

### Correlation Matrix

![corr_plot](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/corr_plot.PNG)

## Data Analysis

Before beginning any analysis I set up the test and training data and then utilized a standard scaler to scale the feature data between -1 and 1. Next my plan was to run various machine learning algorithms to test for the best options. In this situation I will be running classifiers with decision trees, K-nearest neighbors, and neural networks.

### Decision Tree
I started with a decision tree to help identify importance features just to test my assumptions that percipitation and max temperature are important features.

    totPrecip:  0.44
    
    avgTmax: .. 0.31

    avgTmin: .. 0.25

By visualizing the decision tree we can check for overfitting.

![tree_unpruned](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/tree_unpruned.png)

There are a lot of branches in this tree so I ran it through pruning to fine tune the parameters. I did this through a series of efforts to test the optimal values for:

    Max_depth
    min_samples_split
    min_samples_leaf
    max_leaf_nodes
    min_impurity_decrease
    
The values for the optimal solution turned out to be:

    pruned_model = tree.DecisionTreeClassifier(criterion='entropy', 
                                                splitter='random', 
                                                max_depth=5, 
                                                min_samples_split=80, 
                                                min_samples_leaf=50, 
                                                max_leaf_nodes=7, 
                                                min_impurity_decrease=0.01, 
                                                random_state=35)
                                          
The results of our model:

        [[ 0 88  5]
         [ 0 90 66]
         [ 0  2 68]]
         Accuracy Score : 0.4952978056426332

                            precision    recall  f1-score   support

                   0.0       0.00      0.00      0.00        93
                   1.0       0.50      0.58      0.54       156
                   2.0       0.49      0.97      0.65        70

              accuracy                           0.50       319
             macro avg       0.33      0.52      0.40       319
          weighted avg       0.35      0.50      0.40       319

![dec_tree](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/dec_tree.png)

Then I ran a random forest model with the same parameters and received the following results:

        [[  0  92   1]
         [  0 125  31]
         [  0  21  49]]
         Accuracy Score : 0.5454545454545454

                        precision    recall  f1-score   support

                 0.0       0.00      0.00      0.00        93
                 1.0       0.53      0.80      0.63       156
                 2.0       0.60      0.70      0.65        70

            accuracy                           0.55       319
           macro avg       0.38      0.50      0.43       319
        weighted avg       0.39      0.55      0.45       319


### KNN

Next I tested utilizing K-nearest neighbors. To start I tested against different values for k between 2 and 19.

![corr_plot](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/KNN_neighbors.PNG)

k=15 proved to be our optimal option so I ran the following model on the data:

    model = KNeighborsClassifier(n_neighbors=5, n_jobs=-1)

Our confusion matrix and accuracy scores:


      [[ 53  38   2]
       [ 29 101  26]
       [  0  13  57]]
       Accuracy Score : 0.6614420062695925
     
                        precision    recall  f1-score   support

                 0.0       0.65      0.57      0.61        93
                 1.0       0.66      0.65      0.66       156
                 2.0       0.67      0.81      0.74        70

            accuracy                           0.66       319
           macro avg       0.66      0.68      0.67       319
        weighted avg       0.66      0.66      0.66       319


### Neural Network
The last assessment utilizes a neural network. I ran tuning against the MLP Classifier utilizing the following parameters:

    'max_iter':[100, 500, 1000],
    'hidden_layer_sizes': [(5,), (10,), (20,)],
    'activation': ['relu', 'tanh'],
    'solver': ['sgd', 'adam'],
    'alpha': [0.0001, 0.01, 0.05],
    'learning_rate': ['constant', 'invscaling', 'adaptive'],

The optimal parameters achieved for my model were:

    {'activation': 'relu', 'alpha': 0.0001, 'hidden_layer_sizes': (20,), 'learning_rate': 'constant', 'max_iter': 500, 'solver': 'adam'}
    
Here is the final model:

    model = MLPClassifier(hidden_layer_sizes=(20,), 
                          activation='relu', 
                          alpha=0.0001, 
                          learning_rate='constant', 
                          solver='adam', 
                          max_iter=500)

Training and validation loss for our model:

![NN_loss](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/NN_loss.PNG)

Our confusion matrix and accuracy scores:

     [[53  38   2]
      [23 110  23]
      [ 0  14  56]]
      Accuracy Score : 0.6865203761755486
     
     
     Results on the test set:
                          precision    recall  f1-score   support

                 0.0       0.70      0.57      0.63        93
                 1.0       0.68      0.71      0.69       156
                 2.0       0.69      0.80      0.74        70

            accuracy                           0.69       319
           macro avg       0.69      0.69      0.69       319
        weighted avg       0.69      0.69      0.68       319
     
## Results and Conclusions

### What did we learn? 
If we look solely at our accuracy scores:

    Decision Tree   Accuracy Score : 0.4952978056426332
    Random Forest   Accuracy Score : 0.5454545454545454
    KNN             Accuracy Score : 0.6614420062695925
    Neural Network  Accuracy Score : 0.6865203761755486
    
K-nearest neighbors and the neural net provided us with the best models, but the accuracy score doesn't tell us the whole story. The purpose of the project was to predict the severity of flu based on weather conditions. The reason we may want to do this type of analysis is to prepare for conditions that may result in a particularly bad season. In this case the most important category is 2, which includes weeks with 100+ positive flu cases. With this in mind we are more likely to be okay with measuring false positives from the other categories which fall into this one. Because of this accuracy is not necessarily our best metric for our purposes, rather we need to look more closely at the classification report. This allows us to see specifically how well each model scored against the specific categories. Below we will look at the results for category 2.

                    precision    recall  f1-score   
    Decision Tree       0.49      0.97      0.65        
    Random Forest       0.60      0.70      0.65       
    KNN                 0.67      0.81      0.74        
    Neural Network      0.69      0.80      0.74        
    
When we look at precision we can see that k-nearest neighbors and the neural net both did well in regards to how many of the records the model placed in category 2 which were actually in that category. This is a good metric for them, but based on our goal this isn't necessarily important since we are okay with some records falling into category 2 that weren't necessarily in that one. Recall is more likely to describe what we are really looking for since this is telling us how many records that truly were category 2 were labeled as such by the model. In this area the decision tree actually did the best at 0.97 while the neural network and knn models still showed very will with scores of 0.80 and 0.81. The problem with the decision tree is how poorly it did on every other measure (accuracy score of 0.495 and precision on category 2 at 0.49). This leads me to look towards the knn and neural net as our best options. The slight edge goes to the neural net on every important category, but the knn still holds fairly well and is worth keeping in consideration for future analysis.

### Challenges and Future Considerations
This effort was far from complete based on the end goal for the analysis. I believe that we have succeeded in finding some correlation between weather and flu infections, but we were not quite comprehensive enough to reach a state where we could confidently predict severity of the flu season. To do so would require a more detailed dataset, specifically in regards to weather tracking. I feel that the three variables we were able to aquire from NOAA are a begninning, but do not contain enough nuance to get us to a final state. Initially my thoughts are that humidity data could prove valuable, we can hint at that with the precipitation information but that really only provides us with a binary 100% or not with regards to moisture in the atmosphere. Another valuable piece of information might be climactic trends, we are dealing with recorded weather data any predictions will need to be based on projections several months out rather than for specific weeks. Incorporating seasonal records into our model could help to refine it, and provide valuable insights when it comes to forcasting the severity of the flu season as a whole. If we can find a way to impute seasonal predictions for moisture, temperatures and weather patterns that can help us target the model to predict what is coming before the flu season starts.

The other problem is reliable data, I really struggled to find consistent weather information. That was apparent in my need to impute a large number of values into the source data. I think there could be more reliable options out there such as Weather Underground, which has an api for pulling data. The problem with that resource is the amount of money I would have to spend to access it, in this scenario it was not worth the return on investment. I view this as a starting point rather than a final product, so my goal was to find relevant data that could drive future data retrieval and assessment.

I could also work on more nuance within the data. By this I mean things such as measuring temperature swings like the difference between the min and max temperature on a given day or for a given week. The challenge with assessments like this is that you can go too far into the nuances while gaining little return; as an example I started with 2 values for precipitation, one was for average daily precipitation for a given week and the other was total precipitation. In the end going that far did not provide me with a significant return and just complicated the data cleaning. In this situation I chose to go with weekly total because that felt more reflective of how precipitation had an impact on a given week. 

When we dive into the flu data I chose arbitrary counts for flu infections to define my categories, what we should really be utilizing is infection rates. I could not find a reliable dataset that provided more than count of confirmed cases. Without rates it can become difficult to analyze whether more than 100 confirmed cases is problematic or not. Obviously for larger populations that is not really a significant threshold. I could have added calculations to my data cleaning to address this, but I would run into the problem of constantly shifting populations year-over-year and that makes things more complicated.

All this being said, I think we have built a starting point. We see a correlation, and this is something that could grow with better datasets and larger distributions of records. For this we are only using three areas chosen because of the availability of the data, what we really need is a complex system of records throughout the world covering multiple regions, climates, and environmental factors. With something like this we could drive towards more nuance in our assessments.
