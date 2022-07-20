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

# Bar and Pie Charts #
# Generate a bar plot showing the total number of timepoints for all mice tested for each drug regimen using Pandas.

#filter the data frame to the comlumns of interst and set the index to the first Drug regimen
DrugRegiment_SumTotalPoints = combined_study_cleaned.groupby(["Drug Regimen"]).count()["Timepoint"]

# sort the data from highest to lowest for better viewing 
#DrugRegiment_SumTotalPoints_sorted = DrugRegiment_SumTotalPoints.sort_values(ascending = False)

DrugRegiment_SumTotalPoints.plot(kind = "bar", figsize =(10,4), ylabel =" Number of Mice Tested")

# Generate a bar plot showing the total number of timepoints for all mice tested for each drug regimen using pyplot.
xAxis = combined_study_cleaned["Drug Regimen"].unique()

Totalpoints_counts = DrugRegiment_SumTotalPoints
ticks = []
plt.bar(xAxis ,Totalpoints_counts, color = 'blue', alpha =1, align ='center')
plt.title(" Drug Regiment Vs. Number of Mice Tested")
plt.xlabel("Drug Regimen")
plt.ylabel("Number of Mice Tested")
plt.xlim(-1, len(xAxis))

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

 Calculate the final tumor volume of each mouse across four of the treatment regimens:  
# Capomulin, Ramicane, Infubinol, and Ceftamin

# Start by getting the last (greatest) timepoint for each mouse


# Merge this group df with the original dataframe to get the tumor volume at the last timepoint

Final_tumor_volume_df =combined_study_cleaned.groupby("Mouse ID")["Tumor Volume (mm3)", "Timepoint"].max()

Final_tumor_volume_df

# Merge this group df with the original dataframe to get the tumor volume at the last timepoint
Max_data_merged = pd.merge(Final_tumor_volume_df,combined_study_cleaned, on=["Mouse ID","Timepoint"], how ="left")
del Max_data_merged["Tumor Volume (mm3)_y"]
Max_data_merged
Max_data_merged_DF= Max_data_merged.rename(columns = {"Tumor Volume (mm3)_x": "Tumor Volume (mm3)"})
Max_data_merged_DF

# Calculate the final tumor volume of each mouse across four of the treatment regimens:  
# Capomulin, Ramicane, Infubinol, and Ceftamin

# filter the drugs using loc function
Capomulin_filtered= Max_data_merged_DF.loc[Max_data_merged_DF["Drug Regimen"] == "Capomulin","Tumor Volume (mm3)"]

Ramicane_filtered =Max_data_merged_DF.loc[Max_data_merged_DF["Drug Regimen"] == "Ramicane","Tumor Volume (mm3)"]

Infubinol_filtered = Max_data_merged_DF.loc[Max_data_merged_DF["Drug Regimen"] == "Infubinol","Tumor Volume (mm3)"]

Ceftamin_filtered = Max_data_merged_DF.loc[Max_data_merged_DF["Drug Regimen"] == "Ceftamin","Tumor Volume (mm3)"]


Treatment_list = [Capomulin_filtered,Ramicane_filtered ,Infubinol_filtered,Ceftamin_filtered]
UpperBound = []
lowerBound = []
finalvolume = []
for treatment in Treatment_list:
    quartiles = treatment.quantile([.25,.5,.75])
    lowerq = quartiles[0.25]
    upperq = quartiles[0.75]
    iqr = upperq-lowerq

    lower_bound = lowerq - (1.5*iqr)
    upper_bound = upperq + (1.5*iqr)
 
    Finalvol = treatment.max()
    
    finalvolume.append(Finalvol)
    UpperBound.append(upper_bound)
    lowerBound.append(lower_bound)



          
print(f"Capomulin final Tumor volume is{finalvolume[0]:.2f} The potential outlliers are higher than {UpperBound[0]:.2f} and lower than {lowerBound[0]:.2f}")

print(f"Ramicane final Tumor volume is{finalvolume[1]:.2f} The potential outlliers are higher than  {UpperBound[1]:.2f} and lower than{lowerBound[1]:.2f}")
print(f"Infubinol final Tumor volume is {finalvolume[2]:.2f} The potential outlliers are higher than{UpperBound[2]:.2f} and lower than{lowerBound[2]:.2f}")
print(f"Ceftaminfinal Tumor volume is {finalvolume[3]:.2f} The potential outlliers are higher than{UpperBound[3]:.2f} and lower than{lowerBound[3]:.2f}")


# Generate a box plot of the final tumor volume of each mouse across four regimens of interest

Capomulin = Capomulin_filtered
Ramicane = Ramicane_filtered
Infubinol = Infubinol_filtered
Ceftamin = Ceftamin_filtered

data = [Capomulin,Ramicane, Infubinol, Ceftamin]

fig1, ax1 = plt.subplots()
ax1.set_title('Final Tumor Volume of Each Mouse Across Four Regimens of Interest')
ax1.set_ylabel('Final Tumor Volume (mm3)')
ax1.set_xticklabels(['Capomulin', 'Ramicane','Infubinol', 'Ceftamin'])
ax1.boxplot(data)
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
correlation =st.pearsonr(Mouse_weight,Ave_Tumor)
print(f"The correlation between Mouse weight and average tumor size is {round(correlation[0])}")

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


The end_Hassan