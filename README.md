# The-power-of-plots-challeng

Observations
Capomulin was effective in reducing the tumor volume after about 30 days of treatment. bu, the tumor started to increase

Capomulin and Ramicane were the most effective among other drugs in reducing tumor volume

As the mouse wieght increases, the tumor average tumor volume increases


# supress warnings
import warnings
warnings.filterwarnings('ignore')


# Dependencies

import matplotlib.pyplot as plt
import pandas as pd
import scipy.stats as st
import numpy as np
from scipy.stats import linregress

# find the data files path
mouse_data = "data/Mouse_metadata.csv"
study_results = "data/Study_results.csv"

# Read both files

Mouse_data_df = pd.read_csv(mouse_data)
Study_data_df = pd.read_csv(study_results)

Mouse_data_df.head()

Study_data_df.head()

# combine both files into a single data frame using a left join
combined_study = pd.merge(Study_data_df, Mouse_data_df, on = "Mouse ID", how ="left")
combined_study

# confirm the data frame is complete
combined_study.count()
combined_study.dtypes

# Checking the number of mice.
len(combined_study["Mouse ID"].unique())

# Getting the duplicate mice by ID number that shows up for Mouse ID and Timepoint. 
miceID_and_Timpoint = combined_study[["Mouse ID", "Timepoint"]]
Duplicated = miceID_and_Timpoint[miceID_and_Timpoint.duplicated()== True]
Duplicated["Mouse ID"]

# Optional: Get all the data for the duplicate mouse ID. 
Duplicated_miceID_data = combined_study.loc[combined_study["Mouse ID"]== "g989"]
Duplicated_miceID_data 

# Create a clean DataFrame by dropping the duplicate mouse by its ID.
combined_study_cleaned = combined_study.loc[combined_study["Mouse ID"] != "g989"]
combined_study_cleaned.head()

# Checking the number of mice in the clean DataFrame.
len(combined_study_cleaned["Mouse ID"].unique())

combined_study_cleaned.dtypes

# Summary Statistics #

# Generate a summary statistics table of mean, median, variance, standard deviation, and SEM of the tumor volume for each regimen

# Use groupby and summary statistical methods to calculate the following properties of each drug regimen: 
# mean, median, variance, standard deviation, and SEM of the tumor volume. 


Mean_Tumor_Volume = combined_study_cleaned.groupby(["Drug Regimen"]).mean()["Tumor Volume (mm3)"]
Mean_Tumor_Volume



Median_Tumor_Volume = combined_study_cleaned.groupby(["Drug Regimen"]).median()["Tumor Volume (mm3)"]


Tumor_Volume_Variance = combined_study_cleaned.groupby(["Drug Regimen"]).var()["Tumor Volume (mm3)"]

Tumor_Volume_StdDev = combined_study_cleaned.groupby(["Drug Regimen"]).std()["Tumor Volume (mm3)"]


Tumor_Volume_Std_Err = combined_study_cleaned.groupby(["Drug Regimen"]).sem()["Tumor Volume (mm3)"]

# Assemble the resulting series into a single summary dataframe.
Summary_Table = pd.DataFrame({"Mean Tumor Volume": Mean_Tumor_Volume,
                             "Median Tumor Volume":Median_Tumor_Volume,
                             "Tumor Volume Variance": Tumor_Volume_Variance,
                             "Tumor_Volume_StdDev": Tumor_Volume_StdDev,
                             "Tumor Volume Std. Err.": Tumor_Volume_Std_Err})
Summary_Table


# Using the aggregation method, produce the same summary statistics in a single line
Sammry_table = combined_study_cleaned.groupby(["Drug Regimen"]).agg(["mean", "median", "var", "std","sem" ])["Tumor Volume (mm3)"]
Sammry_table

# Bar and Pie Charts#
# Generate a bar plot showing the total number of timepoints for all mice tested for each drug regimen using Pandas.

#find the value count of drug regimens
DrugRegiment_count = combined_study_cleaned["Drug Regimen"].value_counts()


DrugRegiment_count.plot(kind = "bar", figsize =(10,4),xlabel = "Drug Regimen", ylabel =" Number of Mice Tested")
plt.show()

# Generate a bar plot showing the total number of timepoints for all mice tested for each drug regimen using pyplot.


 
plt.bar(DrugRegiment_count.index.values ,DrugRegiment_count.values , color = 'blue', alpha =1, align ='center')
plt.title(" Drug Regiment Vs. Number of Mice Tested")
plt.xlabel("Drug Regimen")
plt.ylabel("Number of Mice Tested")
plt.xticks(rotation ="vertical")
plt.tight_layout()

# Generate a pie plot showing the distribution of female versus male mice using Pandas

male_Female_mice = combined_study_cleaned.groupby(["Sex"]).count()




male_Female_mice.plot(kind ="pie", y= "Mouse ID" , autopct="%1.1f%%", title = "Sex", colors = ["red", "blue"], shadow = True, startangle = 140, legend =False)
plt.ylabel("")

# Generate a pie plot showing the distribution of female versus male mice using pyplot

# filter the male mic
Male_mice_filtered = combined_study_cleaned.loc[combined_study_cleaned["Sex"] == "Male"]

# find the the count of male mice
Male_mice_count= Male_mice_filtered["Sex"].count()

# filter the Female mic
Female_mice_filtered = combined_study_cleaned.loc[combined_study_cleaned["Sex"] == "Female"]

# find the the count of male mice
Female_mice_count= Female_mice_filtered["Sex"].count()

Label = [ "Female", "Male"]
sizes = [Female_mice_count, Male_mice_count]
Colors = [ "red","blue"]

Explode = (0,0)

# create the pie charts based on the values above
plt.pie( sizes, explode = Explode, labels = Label, colors = Colors, autopct= "%1.1f%%", shadow = True, startangle =140)
plt.axis("equal")
plt.title("Sex")

# Quartiles, Outliers and Boxplots #

# Calculate the final tumor volume of each mouse across four of the treatment regimens:  
# Capomulin, Ramicane, Infubinol, and Ceftamin

# Start by getting the last (greatest) timepoint for each mouse


# Merge this group df with the original dataframe to get the tumor volume at the last timepoint

Final_tumor_volume_df =combined_study_cleaned.groupby("Mouse ID")["Timepoint"].max()
Final_tumor_volume_df = Final_tumor_volume_df.reset_index()
Final_tumor_volume


# Merge this group df with the original dataframe to get the tumor volume at the last timepoint
Max_data_merged = pd.merge(Final_tumor_volume_df,combined_study_cleaned, on=["Mouse ID","Timepoint"], how ="left")
Max_data_merged.head()

# Calculate the final tumor volume of each mouse across four of the treatment regimens:  
# Capomulin, Ramicane, Infubinol, and Ceftamin
Treatment_list = ["Capomulin", "Ramicane", "Infubinol", "Ceftamin"]

# create empty list for the final tumor volume

Tumor_volume = []

# filter the drugs regimen using loc function
for treatment in Treatment_list:
    Final_Tumor_Volume = Max_data_merged.loc[Max_data_merged["Drug Regimen"]==treatment, "Tumor Volume (mm3)"]
    
    # add subset
    
    Tumor_volume.append(Final_Tumor_Volume)
    
    # Determine the outliers
    
    quartiles = Final_Tumor_Volume.quantile([.25,.5,.75])
    lowerq = quartiles[0.25]
    upperq = quartiles[0.75]
    iqr = upperq-lowerq

    lower_bound = lowerq - (1.5*iqr)
    upper_bound = upperq + (1.5*iqr)
    
    Outliers =  Final_Tumor_Volume.loc[ (Final_Tumor_Volume > upper_bound)|(Final_Tumor_Volume < lower_bound)]
 
   
    print(f"{treatment}:\n The potential outlliers :\n {Outliers}\n")

# Generate a box plot of the final tumor volume of each mouse across four regimens of interest
out = dict(markerfacecolor = "r", markersize = 10)
plt.boxplot(Tumor_volume, labels= Treatment_list, flierprops=out)
plt.xlabel("Drug Regimen")
plt.ylabel("Final Tumor Volume (mm3)")
plt.show()

# Generate a line plot of tumor volume vs. time point for a mouse treated with Capomulin

mice = combined_study_cleaned.loc[combined_study_cleaned["Mouse ID"] == "l509"]

capomulin_timepoint_vs_tumorVolume_plot = plt.plot(mice["Timepoint"],mice["Tumor Volume (mm3)"], color ="blue")
plt.title("capomulin treatement of mouse I509")
plt.xlabel("Timepoint(days)")
plt.ylabel("Tumor Volume (mm3)")

# Generate a scatter plot of average tumor volume vs. mouse weight for the Capomulin regimen

Capomulin_filtered_DF = combined_study_cleaned.loc[combined_study_cleaned["Drug Regimen"] == "Capomulin"]

aver_tumorVol_Wegiht = Capomulin_filtered_DF .groupby(["Mouse ID"])["Tumor Volume (mm3)", "Weight (g)"].mean()


aver_tumorVol_Wegiht.plot(kind ="scatter", x = "Weight (g)", y= "Tumor Volume (mm3)")

# Calculate the correlation coefficient and linear regression model 
# for mouse weight and average tumor volume for the Capomulin regimen

Mouse_weight = aver_tumorVol_Wegiht["Weight (g)"]
Ave_Tumor = aver_tumorVol_Wegiht["Tumor Volume (mm3)"]
correlation = round(st.pearsonr(Mouse_weight,Ave_Tumor)[0],2)
print(f"The correlation between Mouse weight and average tumor size is {correlation}")

# Calculate the linear regression model 

Mouse_weight = aver_tumorVol_Wegiht["Weight (g)"]
Ave_Tumor = aver_tumorVol_Wegiht["Tumor Volume (mm3)"]
(slope, intercept, rvalue, pvalue, stderr) = linregress(Mouse_weight, Ave_Tumor)
regress_values = Mouse_weight * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(Mouse_weight,Ave_Tumor)
plt.plot(Mouse_weight,regress_values,"r-")
plt.annotate(line_eq,(0,50),fontsize=15,color="red")
plt.xlabel('Weight (g)')
plt.ylabel('Average Tumor Volume (mm3)')
print(f"The r-squared is: {rvalue**2}")
plt.show()


The end_Hassan Mohamed