# PycitySchools
## Purpose:

The PyCity School Board wants to analyze the reading and math testing results for the district. During the module I created an analysis, however they believe that there may have been scholastic dishonesty in the ninth grade class of Thomas High School. The purpose of this project is to remove those results from the data and preserve the rest of the data to repeat the analysis as well as provide an explanation of how these changes effect the data.

## Deliverable 1: Replacing the 9th grade reading and math scores at Thomas High School with NaN:

Just dropping the ninth grade THS students would make the total students incorrect. I don’t want to replace the data with a different value, because it will throw off my averages. Instead I converted all the test results for THS 9th graders to NaNs so that they would be ignored in my .mean functions.

### Deliverable 1: Methods

1. I Imported dependencies and loaded the data for the schools and the students into data frames. I cleaned the student names using formulas tested during the module. Normally I import all my dependencies in the first cell, so that’s what I’ve done here. The starter code wanted NumPy imported in the second cell, so it’s in the 2nd cell in the jupyter notebook.
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

2.  I used the  loc method to find the reading and math scores from the 9th grade at Thomas High School in the student_data_df and replaced them with NaNs. 
```
student_data_df.loc[(student_data_df["grade"]== "9th") & (student_data_df["school_name"]=="Thomas High School") , "reading_score"] = np.nan

student_data_df.loc[(student_data_df["grade"]== "9th") & (student_data_df["school_name"]=="Thomas High School") , "math_score"] = np.nan
```

![Student data with NaNs](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/student_data_df_NaNs.png)


## Deliverable 2: School District Analysis

Now that all of the THS ninth grader’s test results have been converted to NaNs they need to be removed from anywhere they may affect our data and analysis.
### District Summary:

The district summary in the original analysis just used a count of the students to for it’s % passing calculations. This may have some effect on the results, so for the % Passing Math, % Passing Reading, and % Overall Passing I removed the THS ninth graders from the total student count.
#### District Summary: Methods

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
#### District Summary: Results

![District total summary](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/district_summary_df.png)

There is not a significant difference in the percent of students who passed math and reading or who passed over all when we drop the THS 9th graders from the student count. This makes sense they make up about 1% of the overall population of students. 

|	| % Passing Math | % Passing Reading |% Overall Passing|
| --- | --- | --- | ---- |
| w/THS 9th | 75.0 | 85.8 | 65.2 |
| w/o THS 9th | 74.8 |	85.7 | 64.9 |
|Difference|0.18  | 0.11 | 0.27 |


### School Summary:

Even though removing the ninth graders from our count didn’t have a large effect on the overall data for the district, once we look deeper into the data it may have a larger effect. For example, although the THS freshmen only made up 1% of the district they make up 28% of THS students, so we can assume it influence the conclusions we can make from the data in our school summary. To combat this I removed the THS 9th graders from the THS total students and then calculated new percentages to make the per_school_summary_df which will be used to analyze the per capita spending, size and type.
#### School Summary: Methods

-  In the module I got the types, student count, total school budget,  per capita spending, average reading/math scores, percentage of students who passed math, percentage of students who passed reading, and percentage of students who passed math and reading and used them to make a DataFrame that summarizes the points for each school in the district. The starter code did this for me and I kept the code to avoid any naming discrepancies later. 
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

per_passing_math_reading = school_data_complete_df[(school_data_complete_df["reading_score"] >= 70) & (school_data_complete_df["math_score"] >= 70)]

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

6. The per_school_summary_df is currently using the test score percentages with the NaNs for Thomas High School ninth graders. The passing values aren’t counting any nineth graders, but the total student counts are making the percentages seem lower than they should be. To fix this I used the .loc method to count the students in each grade 10-12 for THS and then created another variable with their sum.
```
THS_grade_10 = student_data_df.loc[(student_data_df["grade"]== "10th")&(student_data_df["school_name"]=="Thomas High School")].count()['Student ID']
THS_grade_11 = student_data_df.loc[(student_data_df["grade"]== "11th")&(student_data_df["school_name"]=="Thomas High School")].count()['Student ID']
THS_grade_12 = student_data_df.loc[(student_data_df["grade"]== "12th")&(student_data_df["school_name"]=="Thomas High School")].count()['Student ID']

THS_student_count = THS_grade_10 + THS_grade_11 + THS_grade_12
```
7. Then I gathered the names for the students who passed math, reading, and math & reading using .loc. Then I used the .count method to get the counts for them, I should have done the .loc and .count in the same variable. 
```
THS_passing_math = school_data_complete_df.loc[(school_data_complete_df["school_name"]=="Thomas High School") & (school_data_complete_df["math_score"] >= 70)]
THS_passing_reading = school_data_complete_df.loc[(school_data_complete_df["school_name"]=="Thomas High School") & (school_data_complete_df["reading_score"] >= 70)]
THS_passing_math_reading = school_data_complete_df.loc[(school_data_complete_df["school_name"]=="Thomas High School") & (school_data_complete_df["math_score"] >= 70) & (school_data_complete_df["reading_score"] >= 70)]

THS_UC_math_count = THS_passing_math["student_name"].count()
THS_UC_reading_count = THS_passing_reading["student_name"].count()
THS_overall_pass_count = THS_passing_math_reading["student_name"].count()
```
8. I calculated the percenages based of the counts in step 7.
```
THS_UC_math_percentage = THS_UC_math_count / THS_student_count * 100
```
![Math Passing Percent]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/THS_UC_math_percentage.png)
```
THS_UC_reading_percentage = THS_UC_reading_count / THS_student_count * 100
```
![Reading Pass Percentage](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/THS_UC_reading_percentage.png)
```
THS_overall_pass_percentage = THS_overall_pass_count / THS_student_count *100
```
![Overall Passing Percentage](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/THS_overall_pass_percentage.png)

9. Then I replaced the Thomas Highschool % Passing Math, % Passing Reading, and % Overall Passing with the adjusted values using .loc.
```
per_school_summary_df.loc[["Thomas High School"],['% Passing Math']] = THS_UC_math_percentage

per_school_summary_df.loc[["Thomas High School"],['% Passing Reading']] = THS_UC_reading_percentage
per_school_summary_df.loc[["Thomas High School"],['% Overall Passing']] = THS_overall_pass_percentage
```
![Per School summary with corrected THS data.](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/per_school_summary_df.png)

*I found the top five and bottom five schools using the sort_values function.*
__Top Five Schools:__
```
Top_5 = per_school_summary_df.sort_values(["% Overall Passing"], ascending=False).head(5)
```
![Per school top 5]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/per_school_summary_df_top_5.png)
__Bottom Five Schools:__
```
Bottom_5 = per_school_summary_df.sort_values(["% Overall Passing"], ascending=True).head(5)
```
![per school bottom 5](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/per_school_summary_df_bottom_5.png)

#### School Summary: Results 

Removing the ninth graders from THS had a large effect on the passing percent for the school(See the table below). With the ninth graders left in the data they would rank 9th for overall passing %, but adjusted they are in 2nd place and follows the trend that charter schools in this district preform better than traditional public schools.

|	| % Passing Math | % Passing Reading | % Overall Passing |
| --- | --- | --- | --- |
|w/THS 9th|	67 |	70 |	65 |
|w/o THS 9th|	93 |	97 |	91 |
|Difference|	26 |	27 |	26 |

### Math and Reading Scores by Grade

I created a series of data from each grade then calculated the average of the math and reading scores for each series. Then I loaded them into a new data frame which I removed the index from and formatted before displaying.
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
#### Math and Reading Scores by Grade: Results

![Math Scores by grade](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/math_scores_by_grade.png)
![Reading scores by grade](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/reading_scores_by_grade.png)

The reading_scores_by_grade and math_scores_by_grade use the school_data_complete_df. They are not affected by replacing the ninth grader scores for THS, because they are grouped by both the nineth graders and the schools. There is little difference between grades from the same school. This means any outliers might indicate a problem with the data. 

If you wanted to look at the average grades for all the students in the district by grade, the code would have to be modified, because it may be somewhat affected by the missing values. This also probably wouldn’t be very useful as all the values would probably be very close to the averages from the district summary.

### Scores by School Spending:

To create a dataframe of school data based on spending I created bins  that would have a relatively equal number of schools in each bin and named them. Then I used the .cut method to divide the per capita spending by my bins. I used the group by method to get the averages for each variable. Then I created and formatted the spending ranges per student dataframe.
```
spending_bins = [0, 585, 630, 645, 675]
group_names = ["<$584", "$585-629", "$630-644", "$645-675"]
per_school_summary_df["Spending Ranges (Per Student)"] = pd.cut(per_school_capita, spending_bins, labels=group_names)

spending_math_scores = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Math Score"]
spending_reading_scores = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Reading Score"]
spending_passing_math = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Math"]
spending_passing_reading = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Reading"]
overall_passing_spending = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Overall Passing"]

spending_summary_df = pd.DataFrame({
          "Average Math Score" : spending_math_scores,
          "Average Reading Score": spending_reading_scores,
          "% Passing Math": spending_passing_math,
          "% Passing Reading": spending_passing_reading,
          "% Overall Passing": overall_passing_spending})

spending_summary_df["Average Math Score"] = spending_summary_df["Average Math Score"].map("{:.1f}".format)
spending_summary_df["Average Reading Score"] = spending_summary_df["Average Reading Score"].map("{:.1f}".format)
spending_summary_df["% Passing Math"] = spending_summary_df["% Passing Math"].map("{:.0f}".format)
spending_summary_df["% Passing Reading"] = spending_summary_df["% Passing Reading"].map("{:.0f}".format)
spending_summary_df["% Overall Passing"] = spending_summary_df["% Overall Passing"].map("{:.0f}".format)
```
#### Scores by School Spending: Results

![Spending Summary]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/school_spending_df.png)
THS is in the middle of per capita spending so the scores very greatly. If they make up a significant portion of the total count the THS ninth graders would lower the percentage results.

### Scores by School Size

To create a dataframe of school data based on size I created bins  that would have a relatively equal number of schools in each bin and named them. Then I used the .cut method to divide total students for my bins. I used the group by method to get the averages for each variable. Then I created and formatted the dataframe.
```
size_bins = [0, 999, 1999, 5000]
group_names = ["Small (<1000)", "Medium (1000-1999)", "Large (2000-5000)"]

per_school_summary_df["School Size"] = pd.cut(per_school_summary_df["Total Students"], size_bins, labels=group_names)

size_math_scores = per_school_summary_df.groupby(["School Size"]).mean()["Average Math Score"]
size_reading_scores = per_school_summary_df.groupby(["School Size"]).mean()["Average Reading Score"]
size_passing_math = per_school_summary_df.groupby(["School Size"]).mean()["% Passing Math"]
size_passing_reading = per_school_summary_df.groupby(["School Size"]).mean()["% Passing Reading"]
size_overall_passing = per_school_summary_df.groupby(["School Size"]).mean()["% Overall Passing"]

size_summary_df = pd.DataFrame({
          "Average Math Score" : size_math_scores,
          "Average Reading Score": size_reading_scores,
          "% Passing Math": size_passing_math,
          "% Passing Reading": size_passing_reading,
          "% Overall Passing": size_overall_passing})

size_summary_df["Average Math Score"] = size_summary_df["Average Math Score"].map("{:.1f}".format)
size_summary_df["Average Reading Score"] = size_summary_df["Average Reading Score"].map("{:.1f}".format)
size_summary_df["% Passing Math"] = size_summary_df["% Passing Math"].map("{:.0f}".format)
size_summary_df["% Passing Reading"] = size_summary_df["% Passing Reading"].map("{:.0f}".format)
size_summary_df["% Overall Passing"] = size_summary_df["% Overall Passing"].map("{:.0f}".format)
```
#### School Size: Results
![school size df]( https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/school_size_df.png)

If they make up a significant portion of the total count the THS ninth graders would lower the passing precents of medium school

### Scores by School Type

To find the averages by type I used the groupby method to get the averages for each variable. Then I assembled them into a dataframe and formatted it.
```
type_math_scores = per_school_summary_df.groupby(["School Type"]).mean()["Average Math Score"]
type_reading_scores = per_school_summary_df.groupby(["School Type"]).mean()["Average Reading Score"]
type_passing_math = per_school_summary_df.groupby(["School Type"]).mean()["% Passing Math"]
type_passing_reading = per_school_summary_df.groupby(["School Type"]).mean()["% Passing Reading"]
type_overall_passing = per_school_summary_df.groupby(["School Type"]).mean()["% Overall Passing"]

type_summary_df = pd.DataFrame({
          "Average Math Score" : type_math_scores,
          "Average Reading Score": type_reading_scores,
          "% Passing Math": type_passing_math,
          "% Passing Reading": type_passing_reading,
          "% Overall Passing": type_overall_passing})

type_summary_df["Average Math Score"] = type_summary_df["Average Math Score"].map("{:.1f}".format)
type_summary_df["Average Reading Score"] = type_summary_df["Average Reading Score"].map("{:.1f}".format)
type_summary_df["% Passing Math"] = type_summary_df["% Passing Math"].map("{:.0f}".format)
type_summary_df["% Passing Reading"] = type_summary_df["% Passing Reading"].map("{:.0f}".format)
type_summary_df["% Overall Passing"] = type_summary_df["% Overall Passing"].map("{:.0f}".format)
```
#### Scores by School Type: Results
![School type df](https://github.com/MichelaZ/PyCitySchools/blob/main/Resources/school_type_dg.png)

If they make up a significant portion of the total count the THS ninth graders in the data have lowered the passing percentage for charter schools.

## Results Summary:
- For the district summary the THS ninth graders do not make up a significant portion of the total student count so removing them does not have a statistically significant change in the % Passing Math, % Passing Reading, and % Overall Passing results.
- Removing THS ninth graders from the school summary changes the data significantly, because they make up a significant portion of the THS students.
- Removing the NaNs for the by grades summaries is unnecessary because there is little difference between grades from the same school. This report should be used to detect outliers.
- For the school by type, per capita spending and size dataframes if the THS ninth graders make up a significant portion of the total count of the students the % Passing Math, % Passing Reading, and % Overall Passing results will be lower than if they are removed.

## Conclusions:
- Students preform slightly worse in math than reading.
- Students from the same school’s performance is not significantly different from grade to grade.
- Charter schools outperformed the traditional schools. 
- Large schools tend to preform significantly worse than small or medium schools which preform about the same.
- Per capita spending is inversely related to performance.

Charter schools do not typically perform better or worse than traditional public school, but research does seem to suggest that in disadvantaged populations there is performance improvement in charter schools. The PyCity results seem to support this. Charter schools are probably performing better than public schools because of the following:

- Even though traditional schools have a higher per capita budget, the % of the budget they are spending on teachers and educational materials is probably lower than that of charter schools.
    - *Charter school performance had an inverse correlation with per capita spending.* Compared to charter schools, traditional public schools may have higher budgets because they are required to provide students with additional service such as transportation, food, and support services (National Conference of State Legislatures). It would be great to see a breakdown of the budgets to confirm this.
- The traditional public schools are over crowded.
    -  *Medium and small school preform about the same, but large schools tend to preform significantly* (Condition of America's Public School Facilities: 1999). Large schools tend to be in urban areas with higher percentages of minorities and higher cost of living. They are more likely to be overcrowded which is correlated with poorer facilities and poor performance. *Only one Charter school is a large school.* This is probably the, because charter school are not forced to take students once they are at capacity; enrollment is either first come first serve or by lottery. So their infrastructure is built to handle the large number of students. Public schools, however, must take students even if they are over crowded.

__Reference:__

National Conference of State Legislatures. *Charter Schools Research and Report.* https://www.ncsl.org/research/education/charter-schools-research-and-report.aspx

Condition of America's Public School Facilities: 1999, *National Center for Education Statistics.* https://nces.ed.gov/surveys/frss/publications/2000032/index.asp?sectionid=8

__Author’s Notes:__

- The comments have been removed from the code in the methods sections to reduce redundancies.
- There may be some differences between the method section and the PyCitySchool_Challenge file to improve legibility.
