---
layout: post
title: "Using K means clustering in Scikit-learn"
description: "Demonstration of K-means clustering on a health data set with sklearn"
date: 2017-10-31
tags: [machine, learning, unsupervised, clustering, scikit, learn, sklearn, liver, health, data, science]
comments: true
---

We will examine an unlabeled dataset of recorded data about livers that are believed to be indicative of liver disorders/liver disease, as well as the frequency of drinks per day per person to create a model. We will use K-means clustering to find interesting groups/clusters within the dataset. We will also use cross validation and ensemble learning to fine-tune the model.

## Exploring the data

We will examine the traits of the liver dataset so we can understand the relationships in the data and understand the shape of the dataset.

We will begin by importing all of the appropriate Python libraries.


```python
import pandas as pd
import sklearn as sk
import numpy as np
import matplotlib as plt
from sklearn.preprocessing import normalize, MinMaxScaler
from sklearn.cluster import KMeans
from sklearn.model_selection import KFold, GridSearchCV
from sklearn.metrics import silhouette_score
```

Now we will load our `csv` file into a Pandas Dataframe so we can manipulate and read the data. We will load the data and preview the data.

The structure of the data is as follows (also found [here](http://archive.ics.uci.edu/ml/machine-learning-databases/liver-disorders/bupa.names))

   1. mcv: mean corpuscular volume
   2. alkphos: alkaline phosphotase
   3. sgpt: alamine aminotransferase
   4. sgot: aspartate aminotransferase
   5. gammagt: gamma-glutamyl transpeptidase
   6. drinks: number of half-pint equivalents of alcoholic beverages drunk per day
   7. selector  field used to split data into two sets
   
It appears that column 7 is arbitrary. For the sake of this exercise, we'll treat more than 5 drinks per day as alcoholism, or some other classifier. 

We can explore the relationship between the data and the number of drinks to see if there's some sort of polynomial relationship or heavy clustering. Right now, we will decide that the number of drinks, whether as some threshold, or some polynomial relationship, is our `y` value


```python
col_names = [
    "mcv",
    "alkphos",
    "sgpt",
    "sgot",
    "gammagt",
    "drinks", 
    "gt5",  # "greater than 5", this is the selector as described above
]
raw_data = pd.read_csv("data/liver_data.csv", header=None, names=col_names)
```

Let's look at the data's attributes. We will start by looking at the first 10 entries.


```python
print(raw_data.head(5))
```

       mcv  alkphos  sgpt  sgot  gammagt  drinks  gt5
    0   85       92    45    27       31     0.0    1
    1   85       64    59    32       23     0.0    2
    2   86       54    33    16       54     0.0    2
    3   91       78    34    24       36     0.0    2
    4   87       70    12    28       10     0.0    2


The selector column is useless for our purposes, so we will delete that column from the dataset. If we want to split the data, we can do it ourselves with a random split or with `scikit-learn`'s K-fold cross validation functions. We will then look at the dataframe's metadata.


```python
del raw_data["gt5"]
raw_data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 345 entries, 0 to 344
    Data columns (total 6 columns):
    mcv        345 non-null int64
    alkphos    345 non-null int64
    sgpt       345 non-null int64
    sgot       345 non-null int64
    gammagt    345 non-null int64
    drinks     345 non-null float64
    dtypes: float64(1), int64(5)
    memory usage: 16.2 KB


We will now normalize the data with `L2` normalization. Having different scales for different features can bias the machine learning model and it is generally better to have normalized data for the sake of computation. 

We will also make the assumption that the traits present in the dataset have a distribution close to the normal distribution amongst the general population.


```python
norm_data = normalize(raw_data, norm="l1", axis=0)
norm_data = pd.DataFrame(columns=raw_data.columns, data=norm_data)

scaler = MinMaxScaler()
m_norm_data = normalize(raw_data, norm="max", axis=0)
m_norm_data = pd.DataFrame(columns=raw_data.columns, data=m_norm_data)  # convert back to a dataframe
norm_data["drinks"] = m_norm_data["drinks"]
```

Scikit-learn returned a `numpy` array, we will convert it back to a Pandas Dataframe and bring back the column headers found in `raw_data`, since they were removed by the normalization function.


```python
norm_data = pd.DataFrame(columns=raw_data.columns, data=norm_data)

print(norm_data.head(5))
print(norm_data.info())
```

            mcv   alkphos      sgpt      sgot   gammagt  drinks
    0  0.002733  0.003817  0.004290  0.003176  0.002347     0.0
    1  0.002733  0.002655  0.005624  0.003764  0.001741     0.0
    2  0.002765  0.002240  0.003146  0.001882  0.004088     0.0
    3  0.002926  0.003236  0.003241  0.002823  0.002726     0.0
    4  0.002797  0.002904  0.001144  0.003293  0.000757     0.0
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 345 entries, 0 to 344
    Data columns (total 6 columns):
    mcv        345 non-null float64
    alkphos    345 non-null float64
    sgpt       345 non-null float64
    sgot       345 non-null float64
    gammagt    345 non-null float64
    drinks     345 non-null float64
    dtypes: float64(6)
    memory usage: 16.2 KB
    None


Let's create a correlation matrix and see the correlation values between each attribute


```python
corr_matrix = norm_data.corr()
print(corr_matrix)
```

                  mcv   alkphos      sgpt      sgot   gammagt    drinks
    mcv      1.000000  0.044103  0.147695  0.187765  0.222314  0.312680
    alkphos  0.044103  1.000000  0.076208  0.146057  0.133140  0.100796
    sgpt     0.147695  0.076208  1.000000  0.739675  0.503435  0.206848
    sgot     0.187765  0.146057  0.739675  1.000000  0.527626  0.279588
    gammagt  0.222314  0.133140  0.503435  0.527626  1.000000  0.341224
    drinks   0.312680  0.100796  0.206848  0.279588  0.341224  1.000000


## Generating a model

We can see that the `drinks` column doesn't have a strong correlation with any of the other data points. Regressions will probably not provide good results. We can try to cluster the data into two different groups with K-means clustering using k-fold cross validation, and see how effectively it divides the dataset into groups. We will try several different hyperparameters using `GridSearchCV` in `scikit-learn` to find the best model via ensemble learning.

We will first configure the cross validation split. We are going to use K-fold cross validation with random shuffling (since the data is sorted by drinks). Since we only have 345 data entries, we will keep `k` small, and set `k=3`. We set the random entropy seed so we yield the same results every time this model is generated.


```python
RAND_STATE=50  # for reproducibility and consistency
folds=3
k_fold = KFold(n_splits=folds, shuffle=True, random_state=RAND_STATE)  # setting generator for k-fold splitting
```

Now we will define the hyperparameters we want iterate through in order to find the best model


```python
# Dictionary of hyperparameters to iterate through
# GridSearchCV will try every combination of these hyperparameters and 
# return the model with the best score via KFold validation
hyperparams = {
    "n_clusters": [2, 3],
    "n_init": [10, 15, 20],
    "max_iter": [100, 200, 300, 400, 500],
    "tol": [.0000001, .000001, .00001, .0001],
}

k_means = KMeans()  # sets jobs equal to number of cores

ensemble = GridSearchCV(
    estimator=k_means, 
    param_grid=hyperparams, 
    cv=k_fold,
    n_jobs=-1
)
```

Now we will fit the estimator to our data employing the methods discussed above.


```python
ensemble.fit(norm_data)
```




    GridSearchCV(cv=KFold(n_splits=3, random_state=50, shuffle=True),
           error_score='raise',
           estimator=KMeans(algorithm='auto', copy_x=True, init='k-means++', max_iter=300,
        n_clusters=8, n_init=10, n_jobs=1, precompute_distances='auto',
        random_state=None, tol=0.0001, verbose=0),
           fit_params=None, iid=True, n_jobs=-1,
           param_grid={'max_iter': [100, 200, 300, 400, 500], 'n_init': [10, 15, 20], 'tol': [1e-07, 1e-06, 1e-05, 0.0001], 'n_clusters': [2, 3]},
           pre_dispatch='2*n_jobs', refit=True, return_train_score=True,
           scoring=None, verbose=0)



Now, we need to evaluate the effectiveness of the clustering model. This is difficult to do because the `score()` doesn't provide a good metric, since there is no "ground truth" for the model, since it's unlabeled. We can look at the silhoutte score, which shows how close the points are to the center of their clusters (tighter clusters will give us a better score, if the data points are very scattered, this indicates that our clusters are too loose). The values for the score range from `[-1, 1]` and a score of `1` is ideal. 


```python
# Generate labels for data with model with raw data, compute score
labels = ensemble.predict(norm_data)
score = silhouette_score(norm_data, labels)
print(score)
print(ensemble.best_params_)
```

    0.67850734737
    {'max_iter': 100, 'n_init': 10, 'tol': 1e-07, 'n_clusters': 3}


Now we have split our data into clusters, without any knowledge of what the data actually signifies--we don't know what these groups actually consitute. We tried the model with 20 clusters, and got a very good score, but that is probably overfitting and not very generalization (since we only have 345 samples), so we shaved off some of the more extreme possibilities with the grid search, slowly paring down the number of groups as to prevent overfitting and to keep the model useful and generalizable. This left us with the parameters listed above.

Without having any labels, we were able to deduce three interesting groups within our data with a fairly high silhouette score.

If we want to turn this into a binary classification (say to diagnose those with normal livers and those that don't), we simply set `n_clusters=2` to try to divide the data into 2 groups. Let's do that below.


```python
# hyperparameters to try out for binary classification with KNN
bin_params = {
    "n_init": [10, 15, 20],
    "max_iter": [100, 200, 300, 400, 500],
    "tol": [.0000001, .000001, .00001, .0001],
}

bin_k_means = KMeans(n_clusters=2)  # set 2 clusters

binary_ensemble = GridSearchCV(
    estimator=bin_k_means, 
    param_grid=bin_params, 
    cv=k_fold,
    n_jobs=-1
)

binary_ensemble.fit(norm_data)  # fit model to data and return best model

# Generate labels for data with model with raw data, compute score
bin_labels = binary_ensemble.predict(norm_data)
bin_score = silhouette_score(norm_data, bin_labels)

# Output score to user
print(bin_score)
print(binary_ensemble.best_params_)
```

    0.639223346286
    {'max_iter': 100, 'tol': 1e-07, 'n_init': 10}


We can see that binary classification has a pretty decent score that isn't too much lower than when we divided the data into three groups.