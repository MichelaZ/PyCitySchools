# PyCitySchools
#
##Purpose:
The school board has notified Maria and her supervisor that the students_complete.csv file shows evidence of academic dishonesty; specifically, reading and math grades for Thomas High School ninth graders appear to have been altered. Although the school board does not know the full extent of the academic dishonesty, they want to uphold state-testing standards and have turned to Maria for help. She has asked you to replace the math and reading scores for Thomas High School with NaNs while keeping the rest of the data intact. Once you’ve replaced the math and reading scores, Maria would like you to repeat the school district analysis that you did in this module and write up a report to describe how these changes affected the overall analysis.

## Methods: Deliverable 1 - Replacing the 9th grade reading and math scores at Thomas High School with NaN.
I Imported dependencies and loaded the data for the schools and the students into data frames. I cleaned the student names using formulas tested during the module. Normally I import all my dependencies in the first cell, so that’s what I’ve done here. The starter code wanted NumPy imported in the second cell, so it’s in the 2nd cell in the jupyter notebook.
```
import pandas as pd
import numpy as np

school_data_to_load = "Resources/schools_complete.csv"
student_data_to_load = "Resources/students_complete.csv"

school_data_df = pd.read_csv(school_data_to_load)
student_data_df = pd.read_csv(student_data_to_load)

prefixes_suffixes = ["Dr. ", "Mr. ","Ms. ", "Mrs. ", "Miss ", " MD", " DDS", " DVM", " PhD"]
for word in prefixes_suffixes:
    student_data_df["student_name"] = student_data_df["student_name"].str.replace(word,"")
student_data_df.head(10)
```
![Cleaned student data.](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/student_data_df.png)

 I used the  loc method to find the reading and math scores from the 9th grade at Thomas High School in the student_data_df and replaced them with NaNs. 
```
student_data_df.loc[(student_data_df["grade"]== "9th") & (student_data_df["school_name"]=="Thomas High School") , "reading_score"] = np.nan

student_data_df.loc[(student_data_df["grade"]== "9th") & (student_data_df["school_name"]=="Thomas High School") , "math_score"] = np.nan
```
![Student data with NaNs](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/student_data_df_NaNs.png)

## Methods: Deliverable 2 - School District Analysis
#### District Summary:
1. I merged the data from the student_data_df and school_data_df together into one DataFrame.
```
school_data_complete_df = pd.merge(student_data_df, school_data_df, how="left", on=["school_name", "school_name"])
```
![Merged data frame](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/school_data_complete_df.png)

2. I set variables for the number of schools, students, budget total, and average reading/math scores. 
```
school_count = len(school_data_complete_df["school_name"].unique())
student_count = school_data_complete_df["Student ID"].count()
total_budget = school_data_df["budget"].sum()
average_reading_score = school_data_complete_df["reading_score"].mean()
average_math_score = school_data_complete_df["math_score"].mean()
```
3. I created a variable to get the number of ninth graders at Thomas High School. I subtracted those students from the total count and stored the new count  in a new variable to use for more accurate calculations of student test results.
```
ninth_grade_thomas = student_data_df.loc[(student_data_df["grade"]== "9th")&(student_data_df["school_name"]=="Thomas High School")].count()['Student ID']

student_count = school_data_complete_df["Student ID"].count()

new_student_count = student_count - ninth_grade_thomas

```
4. I calculated the count and percentage of students who passed math or reading as well as math and reading.
```
passing_math_count = school_data_complete_df[(school_data_complete_df["math_score"] >= 70)].count()["student_name"]
passing_reading_count = school_data_complete_df[(school_data_complete_df["reading_score"] >= 70)].count()["student_name"]

passing_math_percentage = passing_math_count / new_student_count * 100
passing_reading_percentage = passing_reading_count/ new_student_count * 100

passing_math_reading = school_data_complete_df[(school_data_complete_df["math_score"] >= 70)& (school_data_complete_df["reading_score"] >= 70)]

overall_passing_math_reading_count = passing_math_reading["student_name"].count()
overall_passing_percentage = overall_passing_math_reading_count / new_student_count * 100
```
5. Then I created a new dataframe to summarize the totals of all the schools in the district and formatted it.
```
# Create a DataFrame
district_summary_df = pd.DataFrame(
          [{"Total Schools": school_count, 
          "Total Students": student_count, 
          "Total Budget": total_budget,
          "Average Math Score": average_math_score, 
          "Average Reading Score": average_reading_score,
          "% Passing Math": passing_math_percentage,
         "% Passing Reading": passing_reading_percentage,
        "% Overall Passing": overall_passing_percentage}])



district_summary_df["Total Students"] = district_summary_df["Total Students"].map("{:,}".format)
district_summary_df["Total Budget"] = district_summary_df["Total Budget"].map("${:,.2f}".format)
district_summary_df["Average Math Score"] = district_summary_df["Average Math Score"].map("{:.1f}".format)
district_summary_df["Average Reading Score"] = district_summary_df["Average Reading Score"].map("{:.1f}".format)
district_summary_df["% Passing Math"] = district_summary_df["% Passing Math"].map("{:.1f}".format)
district_summary_df["% Passing Reading"] = district_summary_df["% Passing Reading"].map("{:.1f}".format)
district_summary_df["% Overall Passing"] = district_summary_df["% Overall Passing"].map("{:.1f}".format)
```
![District total summary](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/district_summary_df.png)

#### School Summary:
-  In the module I got the types, student count, total school budget,  per capita spending, average reading/math scores, percentage of students who passed math, percentage of students who passed reading, and percentage of students who passed math and reading and used them to make a DataFrame that summarizes the points for each school in the district. The starter code did this for me and I kept the code to avoid any naming discrepancies later on. 
```
per_school_types = school_data_df.set_index(["school_name"])["type"]
per_school_counts = school_data_complete_df["school_name"].value_counts()
per_school_budget = school_data_complete_df.groupby(["school_name"]).mean()["budget"]
per_school_capita = per_school_budget / per_school_counts

per_school_math = school_data_complete_df.groupby(["school_name"]).mean()["math_score"]
per_school_reading = school_data_complete_df.groupby(["school_name"]).mean()["reading_score"]

per_school_passing_math = school_data_complete_df[(school_data_complete_df["math_score"] >= 70)]
per_school_passing_reading = school_data_complete_df[(school_data_complete_df["reading_score"] >= 70)]

per_school_passing_math = per_school_passing_math.groupby(["school_name"]).count()["student_name"]
per_school_passing_reading = per_school_passing_reading.groupby(["school_name"]).count()["student_name"]

per_school_passing_math = per_school_passing_math / per_school_counts * 100
per_school_passing_reading = per_school_passing_reading / per_school_counts * 100

per_passing_math_reading = school_data_complete_df[(school_data_complete_df["reading_score"] >= 70)
                                               & (school_data_complete_df["math_score"] >= 70)]

per_passing_math_reading = per_passing_math_reading.groupby(["school_name"]).count()["student_name"]

per_overall_passing_percentage = per_passing_math_reading / per_school_counts * 100

per_school_summary_df = pd.DataFrame({
    "School Type": per_school_types,
    "Total Students": per_school_counts,
    "Total School Budget": per_school_budget,
    "Per Student Budget": per_school_capita,
    "Average Math Score": per_school_math,
    "Average Reading Score": per_school_reading,
    "% Passing Math": per_school_passing_math,
    "% Passing Reading": per_school_passing_reading,
    "% Overall Passing": per_overall_passing_percentage})

per_school_summary_df["Total School Budget"] = per_school_summary_df["Total School Budget"].map("${:,.2f}".format)
per_school_summary_df["Per Student Budget"] = per_school_summary_df["Per Student Budget"].map("${:,.2f}".format)
```
![Original per school summary](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/per_school_summary_df_unadjustedTHS.png)
6. The per_school_summary_df is currently using the test score percentages with the NaNs for Thomas High School ninth graders. So the passing values aren’t counting any nineth graders, but the total student counts are making the percentages seem lower than they should be. To fix this I used the .loc method to count the students in each grade 10-12 for THS and then created another variable with their sum.
```
THS_grade_10 = student_data_df.loc[(student_data_df["grade"]== "10th")&(student_data_df["school_name"]=="Thomas High School")].count()['Student ID']
THS_grade_11 = student_data_df.loc[(student_data_df["grade"]== "11th")&(student_data_df["school_name"]=="Thomas High School")].count()['Student ID']
THS_grade_12 = student_data_df.loc[(student_data_df["grade"]== "12th")&(student_data_df["school_name"]=="Thomas High School")].count()['Student ID']

THS_student_count = THS_grade_10 + THS_grade_11 + THS_grade_12
```
7. Then I calculated the passing data for THS.
```
THS_passing_math_count = school_data_complete_df.loc[(school_data_complete_df["school_name"]=="Thomas High School") & (school_data_complete_df["math_score"] >= 70)].count()["student_name"]

THS_passing_reading_count = school_data_complete_df.loc[(school_data_complete_df["school_name"]=="Thomas High School") & (school_data_complete_df["reading_score"] >= 70)].count()["student_name"]

THS_passing_math_reading = school_data_complete_df.loc[(school_data_complete_df["school_name"]=="Thomas High School") & (school_data_complete_df["math_score"] >= 70) & (school_data_complete_df["reading_score"] >= 70)]
```
8. 
```

THS_UC_math_count = school_data_complete_df.loc[(school_data_complete_df["grade"]!= "9th")
                                                   &(school_data_complete_df["school_name"]=="Thomas High School")
                                                &(school_data_complete_df["math_score"] >= 70)].count()['Student ID']
THS_UC_math_percentage = THS_UC_math_count / THS_student_count * 100

# Step 10. Calculate the percentage of 10th-12th grade students passing reading from Thomas High School.
THS_UC_reading_count = school_data_complete_df.loc[(school_data_complete_df["grade"]!= "9th")
                                                   &(school_data_complete_df["school_name"]=="Thomas High School")
                                                &(school_data_complete_df["reading_score"] >= 70)].count()['Student ID']
THS_UC_reading_percentage = THS_UC_reading_count / THS_student_count * 100
```
9.

percent_passing_10th = school_data_complete_df.loc[(school_data_complete_df["math_score"] >= 70) 
                                            & (school_data_complete_df["reading_score"] >= 70)
                                           &(school_data_complete_df["school_name"]=="Thomas High School")
                                            & (school_data_complete_df["grade"]=="10th")]
percent_passing_11th = school_data_complete_df.loc[(school_data_complete_df["math_score"] >= 70) 
                                            & (school_data_complete_df["reading_score"] >= 70)
                                           &(school_data_complete_df["school_name"]=="Thomas High School")
                                            & (school_data_complete_df["grade"]=="11th")]
percent_passing_12 = school_data_complete_df.loc[(school_data_complete_df["math_score"] >= 70) 
                                            & (school_data_complete_df["reading_score"] >= 70)
                                           &(school_data_complete_df["school_name"]=="Thomas High School")
                                            & (school_data_complete_df["grade"]=="12th")]

THS_overall_pass_count = THS_passing_math_reading["student_name"].count()

THS_overall_pass_percentage =  THS_overall_pass_count / THS_student_count *100
```
10.
```
per_school_summary_df.loc[["Thomas High School"],['% Passing Math']] = THS_UC_math_percentage

per_school_summary_df.loc[["Thomas High School"],['% Passing Reading']] = THS_UC_reading_percentage

per_school_summary_df.loc[["Thomas High School"],['% Overall Passing']] = THS_overall_pass_percentage

```
![Per School summary with corrected THS data.](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/per_school_summary_df.png)
#### High and Low Performing Schools
11.
```
per_school_summary_df.sort_values(["% Overall Passing"], ascending=False).head(5)
```

![Per school top 5]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/per_school_summary_df_top_5.png)
12. 
```
per_school_summary_df.sort_values(["% Overall Passing"], ascending=True).head(5)

```
![per school bottom 5](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/per_school_summary_df_bottom_5.png)

#### Math and Reading Scores by Grade

13.
```
grade_9 = school_data_complete_df[(school_data_complete_df["grade"] == "9th")]
grade_10 = school_data_complete_df[(school_data_complete_df["grade"] == "10th")]
grade_11 = school_data_complete_df[(school_data_complete_df["grade"] == "11th")]
grade_12 = school_data_complete_df[(school_data_complete_df["grade"] == "12th")]

grade_9_math_scores = grade_9.groupby(["school_name"]).mean()["math_score"]
grade_10_math_scores = grade_10.groupby(["school_name"]).mean()["math_score"]
grade_11_math_scores = grade_11.groupby(["school_name"]).mean()["math_score"]
grade_12_math_scores = grade_12.groupby(["school_name"]).mean()["math_score"]

grade_9_reading_scores = grade_9.groupby(["school_name"]).mean()["reading_score"]
grade_10_reading_scores = grade_10.groupby(["school_name"]).mean()["reading_score"]
grade_11_reading_scores = grade_11.groupby(["school_name"]).mean()["reading_score"]
grade_12_reading_scores = grade_12.groupby(["school_name"]).mean()["reading_score"]
```
14. 
```
math_scores_by_grade = pd.DataFrame({
               "9th": grade_9_math_scores,
               "10th": grade_10_math_scores,
               "11th": grade_11_math_scores,
               "12th": grade_12_math_scores})
reading_scores_by_grade = pd.DataFrame({
               "9th": grade_9_reading_scores,
               "10th": grade_10_reading_scores,
               "11th": grade_11_reading_scores,
               "12th": grade_12_reading_scores})

math_scores_by_grade["9th"] = math_scores_by_grade["9th"].map("{:.1f}".format)
math_scores_by_grade["10th"] = math_scores_by_grade["10th"].map("{:.1f}".format)
math_scores_by_grade["11th"] = math_scores_by_grade["11th"].map("{:.1f}".format)
math_scores_by_grade["12th"] = math_scores_by_grade["12th"].map("{:.1f}".format)

reading_scores_by_grade["9th"] = reading_scores_by_grade["9th"].map("{:,.1f}".format)
reading_scores_by_grade["10th"] = reading_scores_by_grade["10th"].map("{:,.1f}".format)
reading_scores_by_grade["11th"] = reading_scores_by_grade["11th"].map("{:,.1f}".format)
reading_scores_by_grade["12th"] = reading_scores_by_grade["12th"].map("{:,.1f}".format)

math_scores_by_grade.index.name = None
reading_scores_by_grade.index.name = None
```
![Math Scores by grade](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/math_scores_by_grade.png)
![Reading scores by grade](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/reading_scores_by_grade.png
#### Scores by School Spending:
15.
```
# Establish the spending bins and group names.
spending_bins = [0, 585, 630, 645, 675]
# Categorize spending based on the bins.
group_names = ["<$584", "$585-629", "$630-644", "$645-675"]
per_school_summary_df["Spending Ranges (Per Student)"] = pd.cut(per_school_capita, spending_bins, labels=group_names)

# Calculate averages for the desired columns. 
spending_math_scores = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Math Score"]

spending_reading_scores = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Reading Score"]

spending_passing_math = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Math"]

spending_passing_reading = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Reading"]

overall_passing_spending = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Overall Passing"]

# Create the DataFrame
spending_summary_df = pd.DataFrame({
          "Average Math Score" : spending_math_scores,
          "Average Reading Score": spending_reading_scores,
          "% Passing Math": spending_passing_math,
          "% Passing Reading": spending_passing_reading,
          "% Overall Passing": overall_passing_spending})


# Format the DataFrame 
spending_summary_df["Average Math Score"] = spending_summary_df["Average Math Score"].map("{:.1f}".format)

spending_summary_df["Average Reading Score"] = spending_summary_df["Average Reading Score"].map("{:.1f}".format)

spending_summary_df["% Passing Math"] = spending_summary_df["% Passing Math"].map("{:.0f}".format)

spending_summary_df["% Passing Reading"] = spending_summary_df["% Passing Reading"].map("{:.0f}".format)

spending_summary_df["% Overall Passing"] = spending_summary_df["% Overall Passing"].map("{:.0f}".format)
```
![Spending Summary]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/school_spending_df.png)
#### Scores by School Size

```
# Establish the bins.
size_bins = [0, 999, 1999, 5000]
group_names = ["Small (<1000)", "Medium (1000-1999)", "Large (2000-5000)"]

# Categorize size based on the bins.
per_school_summary_df["School Size"] = pd.cut(per_school_summary_df["Total Students"], size_bins, labels=group_names)

# Calculate averages for the desired columns.
size_math_scores = per_school_summary_df.groupby(["School Size"]).mean()["Average Math Score"]

size_reading_scores = per_school_summary_df.groupby(["School Size"]).mean()["Average Reading Score"]

size_passing_math = per_school_summary_df.groupby(["School Size"]).mean()["% Passing Math"]

size_passing_reading = per_school_summary_df.groupby(["School Size"]).mean()["% Passing Reading"]

size_overall_passing = per_school_summary_df.groupby(["School Size"]).mean()["% Overall Passing"]

# Assemble into DataFrame. 
size_summary_df = pd.DataFrame({
          "Average Math Score" : size_math_scores,
          "Average Reading Score": size_reading_scores,
          "% Passing Math": size_passing_math,
          "% Passing Reading": size_passing_reading,
          "% Overall Passing": size_overall_passing})
# Format the DataFrame  
size_summary_df["Average Math Score"] = size_summary_df["Average Math Score"].map("{:.1f}".format)

size_summary_df["Average Reading Score"] = size_summary_df["Average Reading Score"].map("{:.1f}".format)

size_summary_df["% Passing Math"] = size_summary_df["% Passing Math"].map("{:.0f}".format)

size_summary_df["% Passing Reading"] = size_summary_df["% Passing Reading"].map("{:.0f}".format)

size_summary_df["% Overall Passing"] = size_summary_df["% Overall Passing"].map("{:.0f}".format)

size_summary_df
![school size df]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/school_size_df.png)

#### Scores by School Type

```
# Calculate averages for the desired columns. 
type_math_scores = per_school_summary_df.groupby(["School Type"]).mean()["Average Math Score"]

type_reading_scores = per_school_summary_df.groupby(["School Type"]).mean()["Average Reading Score"]

type_passing_math = per_school_summary_df.groupby(["School Type"]).mean()["% Passing Math"]

type_passing_reading = per_school_summary_df.groupby(["School Type"]).mean()["% Passing Reading"]

type_overall_passing = per_school_summary_df.groupby(["School Type"]).mean()["% Overall Passing"]

# Assemble into DataFrame. 
type_summary_df = pd.DataFrame({
          "Average Math Score" : type_math_scores,
          "Average Reading Score": type_reading_scores,
          "% Passing Math": type_passing_math,
          "% Passing Reading": type_passing_reading,
          "% Overall Passing": type_overall_passing})
# Format the DataFrame 
# Formatting school type data
type_summary_df["Average Math Score"] = type_summary_df["Average Math Score"].map("{:.1f}".format)

type_summary_df["Average Reading Score"] = type_summary_df["Average Reading Score"].map("{:.1f}".format)

type_summary_df["% Passing Math"] = type_summary_df["% Passing Math"].map("{:.0f}".format)

type_summary_df["% Passing Reading"] = type_summary_df["% Passing Reading"].map("{:.0f}".format)

type_summary_df["% Overall Passing"] = type_summary_df["% Overall Passing"].map("{:.0f}".format)
```
![School type df](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/school_type_dg.png)


## Results:

https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/THS_UC_math_percentage.png


![This is an image]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/THS_passing_math_reading.png)
![This is an image](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/THS_overall_pass_percentage.png)
![This is an image](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/THS_UC_reading_percentage.png)
![This is an image]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/THS_UC_math_percentage.png)

