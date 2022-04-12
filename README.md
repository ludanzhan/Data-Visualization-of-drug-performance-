# Drug Performance Visualization with matplotlib
## Background 
The data for this project comes from a study. In this study, 249 mice identified with SCC tumor growth were treated through a variety of drug regimens. Over the course of 45 days, tumor development was observed and measured. The purpose of this study was to compare the performance of Pymaceuticals' drug of interest, Capomulin, versus the other treatment regimens. Goal of this project is to generate all of the tables and figures needed for the technical report of the study. The project also generate a top-level summary of the study results.

## 

## Observations and Insights
### Read data and clean data
```python
# Read the mouse data and the study results
mouse_metadata = pd.read_csv('Resources/Mouse_metadata.csv')
study_results = pd.read_csv('Resources/Study_results.csv')

# Combine the data into a single dataset
combine_df = pd.merge(mouse_metadata, study_results, on="Mouse ID")

# Create a clean DataFrame by dropping the duplicate mouse by its ID.
new_df = combine_df.drop_duplicates(subset=['Mouse ID','Timepoint'], keep ='last')
```

### Summary Statistics
Generate a summary statistics table consisting of the mean, median, variance, standard deviation, and SEM of the tumor volume for each drug regimen.

|Drug Regimen| mean       |	  median   |	variance  |	standard deviation 	|SEM	|
| :---------:|:----------:| :-------:| :---------:| :---:    | :-------:|
|Capomulin 	 |40.675741 	|41.557809 |	24.947764 |	4.994774 |	0.329346|
|Ceftamin 	 |52.591172 	|51.776157 |	39.290177 |	6.268188 |	0.469821|
|Infubinol 	 |52.884795 	|51.820584 |	43.128684 |	6.567243 |	0.492236|
|Ketapril 	 |55.235638 	|53.698743 |	68.553577 |	8.279709 |	0.603860|
|Naftisol 	 |54.331565 	|52.509285 |	66.173479 |	8.134708 |	0.596466|
|Placebo 	   |54.033581 	|52.288934 |	61.168083 |	7.821003 |	0.581331|
|Propriva 	 |52.382993 	|50.783528 |	43.220205 |	6.574208 |	0.526358|
|Ramicane 	 |40.216745 	|40.673236 |	23.486704 |	4.846308 |	0.320955|
|Stelasyn 	 |54.233149 	|52.431737 |	59.450562 |	7.710419 |	0.573111|
|Zoniferol 	 |53.236507 	|51.818479 |	48.533355 |	6.966589 |	0.516398|

Using the _aggregation method_, produce the same summary statistics in a single line

```python
tumor_df = new_df.groupby('Drug Regimen')
tumor_df.agg({"Tumor Volume (mm3)" : ['mean','median','var','std','sem']})
```
### Quartiles, Outliers and Boxplots
Calculate the final tumor volume of each mouse across four of the most promising treatment regimens: Capomulin, Ramicane, Infubinol, and Ceftamin. Using _for loop_ to calculate the quartiles and IQR and quantitatively determine if there are any potential outliers across all four treatment regimens.

```python
  # Put treatments into a list for for loop (and later for plot labels)
  treatments = ['Capomulin','Ramicane','Infubinol','Ceftamin']

  # Create empty list to fill with tumor vol data (for plotting)
  tumorVol = []

  # Calculate the IQR and quantitatively determine if there are any potential outliers. 
  # Locate the rows which contain mice on each drug and get the tumor volumes
  for treatment in treatments:
      drug_seperate = drug_merge[drug_merge["Drug Regimen"] == treatment]
      tumor_seperate = drug_seperate["Tumor Volume (mm3)"]
      tumorVol.append(tumor_seperate)
    
      # add subset 
      quantile = tumor_seperate.quantile([.25,.5,.75])
      lowerq = quantile[0.25]
      upperq = quantile[0.75]
      iqr = upperq-lowerq

      # Determine outliers using upper and lower bounds
      lower_bound = lowerq - (1.5*iqr)
      upper_bound = upperq + (1.5*iqr)
```
* Generate a box plot of the final tumor volume of each mouse across four regimens of interest
![image](https://github.com/ludanzhan/Matplotlib-Challenge/blob/main/Images/boxplot.png)

* Generate a line plot of tumor volume vs. time point for a mouse treated with Capomulin
![image](https://github.com/ludanzhan/Matplotlib-Challenge/blob/main/Images/scatterplot.png)

* Calculate the correlation coefficient and linear regression model between mouse weight and average tumor volume for the Capomulin treatment

```python
  corr=round(st.pearsonr(tumorAvg,weightAvg)[0],2)
```
* Plot the linear regression model on top of the previous scatter plot.
![image](https://github.com/ludanzhan/Matplotlib-Challenge/blob/main/Images/regressionplot.png)

```python
  (slope, intercept, rvalue, pvalue, stderr) = linregress(weightAvg, tumorAvg )
  regress_values = weightAvg * slope + intercept
  line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))

  plt.scatter( weightAvg, tumorAvg )
  plt.plot(weightAvg,regress_values,"r-")

  plt.title('Average Tumor Volume vs. Mouse Weight',fontsize =14)
  plt.ylabel('Average Tumor Volume',fontsize =14)
  plt.xlabel('Average Mouse Weight',fontsize =14)

  plt.tight_layout()
  plt.rcParams["figure.figsize"] = (13,7)
  plt.show()
```



### Graphic Summary
Generate a bar plot and pie chart using both Pandas's _DataFrame.plot()_ and _Matplotlib's pyplot_ that shows the total number of timepoints for all mice tested for each drug regimen throughout the course of the study.
*  **Bar plot** using Pandas's _DataFrame.plot()_ showing the total number of timepoints for all mice tested for each drug regimen using Pandas.
 
 ```python
  drug_df = pd.DataFrame(tumor_df["Timepoint"].sum())
  drug_df.plot(kind = "bar", figsize=(10,5))
  plt.title("Total number of Timepoints For each Drug Regimen",fontsize =15)
  plt.tight_layout()
  plt.savefig("Images/bar_pandas.png",bbox_inches = "tight")
  plt.show()
  ```
  ![image](https://github.com/ludanzhan/Matplotlib-Challenge/blob/main/Images/bar_pandas.png)

* **Bar plot** using _Matplotlib's pyplot_ showing the total number of timepoints for all mice tested for each drug regimen using Pandas.

  ```python
  x_axis = drug_df.index
  y_axis = drug_df["Timepoint"]
  plt.bar(x_axis,y_axis)
  plt.xticks(rotation=45)
  plt.rcParams["figure.figsize"] = (12,5)
  ```
  ![image](https://github.com/ludanzhan/Matplotlib-Challenge/blob/main/Images/bar_matplot.png)

* **Pie Chart** using Pandas's _DataFrame.plot()_ showing the distribution of female versus male mice
![image](https://github.com/ludanzhan/Matplotlib-Challenge/blob/main/Images/pie_pandas.png)

  ```python
    sex = new_df.groupby('Sex')
    sex_df = pd.DataFrame(sex["Sex"].count())
    sex_df.plot(kind = "pie", y = 'Sex', autopct='%1.1f%%')

    plt.tight_layout()
    plt.axis("equal")
    plt.show()
  ```
 * **Pie Chart** using _Matplotlib's pyplot_ showing the distribution of female versus male mice
 ![image](https://github.com/ludanzhan/Matplotlib-Challenge/blob/main/Images/pie_matploy.png)
 
   ```python
    explode = (0.1, 0)
    plt.pie(sex_df["Sex"], explode=explode, labels=['Female','Male'],
        autopct="%1.1f%%", shadow=True, startangle=140)

    plt.tight_layout()
    plt.axis("equal")
    plt.show()
  ```
