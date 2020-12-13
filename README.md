- EXAMPLE MAP HERE
# Overview

## Context

## Interesting

# Approach
## Algorithms
#### Multiple Linear Regression

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
2. Data Joining
3. Data 


## External tools and packages
In this project, I further developed my knowledge of R to create my data pipeline. I chose R for this task because I am using it in another course this semester, and there are two very convienent packages authored by Kyle Walker:
* [tidycensus](https://walker-data.com/tidycensus/)
* [tigris](https://github.com/walkerke/tigris)
These packages were instrumental in gathering Census data. Tidycensus is a convenient alternative to [data.census.gov](data.census.gov) and tigris is a similar package for spatial data from the US Census. 
Another software essential to this project was ESRI's ArcGIS Pro. I used ArcGIS to tabulate raster image pixels by census tracts. This converted raster image data to counts of pixels as a CSV file.

# Results
Prediction map here

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
- Input data again?
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


# References
- https://bookdown.org/ripberjt/qrmbook/introduction-to-multiple-regression.html
  * This resource was used to better understand the multiple linear regression problem as well as to serve as an example (in R)
- https://github.com/runawayhorse001/CheatSheet/blob/master/cheatSheet_pyspark.pdf
- https://medium.com/@talentorigin/creating-apache-spark-cluster-on-google-cloud-platform-gcp-9a5d8b9c0ffb
- https://guides.github.com/features/mastering-markdown/

Thank you.
