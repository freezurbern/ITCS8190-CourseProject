TODO EXAMPLE MAP HERE

# Overview


* **Context**
I chose to use my course project as an opportunity to learn more about Apache Spark and statistics and their interaction with geospatial research. I am a Ph.D. student in the Department of Geography at UNC Charlotte taking an advanced computer science course (ITCS 8190) because I want to bring computer science methods to Geographic Information Science (GIS). My background before GIS was web development as a computer science undergraduate at UNC Charlotte. I felt this project would be an easy introduction to the intersection of cloud computing and GIS, but was surprised and overwhelmed by the complexity involved. While I have learned a weighty tome of information about cloud computing this semester, I still have a lot of work ahead to understand the entire scope of this exciting topic.

* **Interesting**
The most interesting aspect of this project is the distributed training and prediction of a multiple linear regression model using Apache Spark. Also of interest is the data preparation incorporating land cover and demographic data. 

# Approach
In this course project, I used multiple linear regression to model urban growth. The model formula is:
```
Urban = Beta_0 + Beta_1*roadDens + Beta_2*popDens + Beta_3*barren + Beta_4*water + Beta_5*nature + Beta_6*agric
Where
  Urban  = Dependent variable, percent of census tract's land cover urban
  Beta_0 = Y-intercept coefficient
  Beta_{1..6} = Coefficients
  roadDens = (Length of roads in meters / area of tract in square meters)
  popDens = (ACS Estimated population / area of tract in square meters)
  barren = (NLCD Barren landcover pixels / total pixels in census tract)
  water = (NLCD Open Water landcover pixels / total pixels in census tract)
  nature = ((Forest + Shrub + Grassland + Wetlands) landcover pixels / total pixels in census tract)
  agric = ((Pasture + Crops) landcover pixels / total pixels in census tract)
```

## Frameworks
I used the Google Cloud Platform's Dataproc service to host a Hadoop cluster. This service has 
[multiple images](https://cloud.google.com/dataproc/docs/concepts/versioning/overview)
for cluster VM nodes. I chose Dataproc 1.5, which includes Ubuntu 18.04 LTS, Hadoop 2.10, and Spark 2.4. It also runs Python 3, which was my primary motivation for choosing this image.

# Data sources
* [US Census American Community Survey](https://www.census.gov/programs-surveys/acs)
* [US Census TIGER/LINE Shapefiles](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html)
* [National Land Cover Database](https://www.mrlc.gov/data/nlcd-land-cover-conus-all-years)

# Data Pipeline / Tasks
1. Download Census data for years 2011, 2013, 2016
   * Population Estimate from American Community Survey (5-Year)
   * TIGER/LINE Roads
   * Save to disk as ESRI Shapefiles
2. Calculate road density per census tract
   1. ST_INTERSECTION: Cut road lines with tract polygon edges
   2. Calculate lengths of road line segments
   3. Calculate area of census tracts
   4. Group By tract ID and sum all segment lengths
   5. Divide summed lengths by tract areas to get density
   * (Therefore, tract size no longer matters)
3. Calculate population density per census tract
   1. Left Join road density data frame with population data frame (to re-use census tract area column)
   2. Divide population estimate by tract area
   3. Rename columns for convenience
4. Write population and road densities to CSV
5. Test Linear Regression in R
   1. By suggestion of the instructor, it was very useful to have known-valid values to validate my implementation later in Apache Spark.
6. Upload data to Hadoop HDFS
   * ```freezurbern@cluster-95c6-m:/home/ubuntu$ hadoop fs -put lcpr2011.csv /user/root/lcpr2011.csv```
   * ```freezurbern@cluster-95c6-m:/home/ubuntu$ hadoop fs -put lcpr2013.csv /user/root/lcpr2013.csv```
   * For Google Dataproc, upload the files following their [tutorial](https://cloud.google.com/compute/docs/instances/transfer-files?hl=en#transferbrowser)

# Apache Spark - Steps Implemented
1. Data Loading
   1. After uploading to HDFS, using spark to read CSV
   2. Cleanup data columns and rename
3. Compute multiple linear regression coefficients
   1. Based on my Assignment 2 code
   2. Store the coefficients on HDFS for later use
2. Distributed prediction
   1. Loading new data from HDFS
   2. Loading coefficients from HDFS
   3. Adding coefficients as columns
   4. Computing dependent variable
3. Data Export
   1. Saving computed prediction to HDFS
4. Searching and Plotting
   1. TODO

## External tools and packages
In this project, I further developed my knowledge of R to create my data pipeline. I chose R for this task because I am using it in another course this semester, and there are two very convienent packages authored by Kyle Walker:
* [tidycensus](https://walker-data.com/tidycensus/)
* [tigris](https://github.com/walkerke/tigris)
These packages were instrumental in gathering Census data. Tidycensus is a convenient alternative to [data.census.gov](data.census.gov) and tigris is a similar package for spatial data from the US Census. 
Another software essential to this project was ESRI's ArcGIS Pro. I used ArcGIS to tabulate raster image pixels by census tracts. This converted raster image data to counts of pixels as a CSV file.

# Results
TODO Prediction map here

# Performance Evaluation
I evaluated the performance of my linear regression model first by manually calculating a prediction:
```python
betas = [0.9828573112718250737174230, 0.6322003963821229977071425, 4.0829155870278501794246040,
         0.0187037366133609578300323,-0.9753240641474236749530746,-0.9734866562156101466030123,
         -0.9760581738234358484262998]
ex_feature = ['01003010500',
              0.0093421030,
              0.0002470766835,
              0.01558030241,
              0.00715161422,
              0.501277074,
              0.08065999183,
              0.41091132]

ex_y = ex_feature[7]
ex_x = ex_feature[1:8]
calc = beta[0] + (beta[1]*ex_x[0]) + (beta[2]*ex_x[1]) + (beta[3]*ex_x[2]) + (beta[4]*ex_x[3]) + (beta[5]*ex_x[4]) + (beta[6]*ex_x[5]) 
print(calc)
# 0.41637306722808143
print(calc - ex_y)
# 0.005461747228081404             
```
The result of this calculation is approximately +0.005, which shows my model overestimates the percent-urban by 0.5%. I find this result adequate but not inspiring, because 0.05% of a census tract (average size in my dataset is 2,436 acres) is **121 acres**. 

- RMSE?
- Map visual inspection?
- Matching census urban areas?

# Aspects Desired
## Definitely will do

:heavy_check_mark: **Loading my data into Apache Spark**
 
:heavy_check_mark: **Use the South East region of the United States**

:heavy_check_mark: **Implementing multiple linear regression in Apache Spark**

:heavy_check_mark: **Train a model**

:heavy_check_mark: **Predict urban growth**

:heavy_check_mark: **Export and map the predicted urban growth**

## Likely will do
:x:	**Use data across the entire United States**
  * I did not accomplish this likely goal due to pre-processing time associated with road density. Calculating density for the southeastern United States took atleast 30 minutes depending on which machine was running the R code. With more time, I believe I could expand my analysis.

## Would ideally like to do
:x:	**Import my datasets directly into spark without preprocessing in R or ArcGIS Pro**
  * I did not accomplish this stretch-goal because Apache Spark does not have native support for raster image data, and my knowledge of the platform is not yet advanced enough to implement this myself.

## Unexpected accomplishments
:heavy_check_mark: **Distributed prediction using Spark DataFrames**

:heavy_check_mark: **Running my code on a Google Cloud Platform cluster**

:heavy_check_mark: **Loading data from a Google Cloud Storage Bucket**

:heavy_check_mark: **Documenting projects on GitHub Pages**




# Challenges
Here I will briefly describe the challenges encountered during this project.
**1. Unit of Analysis**
   - My project proposal described my unit of analysis as a raster image pixel. This was ultimately impractical due to Apache Spark's affinity for matricies and tables. With the gracious permission of the instructor, I was able to move to census tracts as my primary unit. Census tracts are given a unique GEOID primary key by the Census which is consistent through time and datasets.
**2. Regression type**
   - My second error in my project proposal was defining my research question and approach. I intended to implement logistic regression on Apache Spark to determine whether an area had experienced urban growth or not. After digging into the datasets, I found my urban variable was continuous between 0.0 and 1.0. I contacted the instructor who again allowed me to modify my project to work around this mistake.

# Lessons Learned
1. My first lesson learned was that much preparation must go into a project proposal. My challenges were a result of too little data exploration and knowledge of methods. In the future, I will consider datasets carefully and properly document the exact columns of each dataset used in my proposed analysis.
2. Data pipelines with multiple tools on multiple machines should be avoided. For this project, I used R Studio in a Docker container on my server to conduct preprocessing of census data and later for processing of ArcGIS output. Census tract polygons were copied from R Studio to ArcGIS Pro on my laptop for tabulation with land cover raster images. Then, the tabulations were copied back to R Studio for additional analysis. This back-and-forth between multiple machines and multiple applications should be avoided if possible. A purely-Spark data pipeline would have been even cleaner.
3. Native libraries are a convenient black box with nice outputs. By implementing regression by-hand, I learned exactly what is going on behind the scenes of native functions. I also learned how much work is put into them to give useful outputs such as r-squared or RMSE.
4. TODO

# References
- https://bookdown.org/ripberjt/qrmbook/introduction-to-multiple-regression.html
  * This resource was used to better understand the multiple linear regression problem as well as to serve as an example (in R)
- https://github.com/runawayhorse001/CheatSheet/blob/master/cheatSheet_pyspark.pdf
- https://medium.com/@talentorigin/creating-apache-spark-cluster-on-google-cloud-platform-gcp-9a5d8b9c0ffb
- https://guides.github.com/features/mastering-markdown/

Thank you.
