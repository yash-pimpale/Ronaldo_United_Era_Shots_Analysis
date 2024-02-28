<p align="center">
  <img width="400" height="700" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/CR7.jpeg">
</p>

# Ronaldo's United Era Shots Analysis

## Objective

1. Analyze and gain insights into Cristiano Ronaldo's shot patterns during his tenure with Manchester United from the 1996-1997 season to the 1999-2000 season.
2. Investigate factors such as shot conversion rate, goal distribution, shot types, and performance in different areas of the field.
3. Uncover trends and patterns to better understand Ronaldo's goal-scoring dynamics during this pivotal period of his career.

## Overview of the Dataset

The dataset comprises detailed information about each shot taken by Cristiano Ronaldo during his time with Manchester United spanning from the 1996-1997 season to the 1999-2000 season. The data includes various attributes related to each shot, providing insights into Ronaldo's goal-scoring performance during this crucial period of his career.

Key Features:

- Shot details such as shot location (x, y), shot type, and shot basics.
- Game-related information, including match ID, team name, and home/away status.
- Temporal data, including the date of the game and remaining minutes/seconds when the shot was taken.
- Additional factors like power of the shot, knockout match status, and area of shot.

## Implementation Process

Based on above mentioned details we will now implement the project using Snowflake, AWS S3 and Microsoft Power BI.

### 1. Data Profiling

Performed below tasks as part of data profiling.

1. Column Datatype Validation: Confirm that each column has an appropriate and consistent datatype.

2. Data Value Inspection: Examine the actual values within each column for anomalies, inconsistencies, or unexpected patterns.

3. Missing Values Handling: Identify and address missing values to ensure data completeness.

4. Identify Key Performance Indicators (KPIs): Determine relevant metrics that serve as key performance indicators for analysis (e.g., conversion rate, average distance of the shot, etc).

### 2. Real-Time Data Loading in Snowflake

Leveraged Snowflake's real-time data loading capabilities to seamlessly ingest data from an Amazon S3 bucket into Snowflake tables using Snowpipe.

Below diagram shows the Snowpipe auto-ingest process flow:-

<p align="center">
  <img width="500" height="650" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/S3_to_Snowflake_Process.png">
</p>

1. Whenever the user uploads files to the AWS S3 bucket, the cloud storage notification service triggers a SQS notification to the Snowflake Snowpipe object. 

2. This S3 event notification informs Snowpipe that a new file has been added to the S3 bucket and is ready to be loaded into the Snowflake table. 

3. When Snowpipe receives this trigger, it extracts the data file from the AWS S3 bucket and loads it into the table.

Note: Snowpipe uses file-loading metadata associated with each pipe object to prevent loading the same files again. So, if the same file appears in the S3 bucket, Snowpipe will not reload it.

### Steps

1. Amazon S3 Bucket: Created an S3 bucket on Amazon using an IAM user with appropriate premission. Snowflake requires the following permissions on an S3 bucket and folder to be able to access files in the folder - s3:GetBucketLocation, s3:GetObject, s3:GetObjectVersion, s3:ListBucket.

2. Snowflake Table: Created a Table in Snowflake Datawarehouse as per datatypes identified in Data Profiling steps. Data will be loaded in this table.

3. External Stage: Created an external stage that references the S3 bucket where the data file is stored. External Stage is used to load data which is stored externally (e.g. Amazon S3, Azure blob storage) into tables in a Snowflake database.
  
4. Snowpipe: It is a continuous data ingestion service that automatically loads new data as it arrives in a specified location. It fetches the data files from the external stage and temporarily queue them before loading them into the target table. Provided file format as well while creating the snowpipe.

More details can be found in the SQL file attached.

Log of data being loaded into snowflake using snowpipe:-

<p align="center">
  <img width="600" height="400" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/S3_to_Snowflake_Data_Loaded.png">
</p>

### 3. Data Transformation & Visualization in Microsoft Power BI

Loaded the data into Power BI by creating a connection with Snowflake Datawarehouse.

Created key parameters which are essential for performing calculations and analysis. These parameters allow us to define custom calculations/aggregations based on existing column data. For e.g. 
- Total Goals: Denotes the total number of goals scored by Ronaldo based on "IS_GOAL" column. DAX formula - COUNTROWS(FILTER(RONALDO_YDS_TB_INSERT, RONALDO_YDS_TB_INSERT[IS_GOAL] = 1)).
- Total Shots Taken: It represents the total number of shots taken by Ronaldo. DAX formula - SUMX(RONALDO_YDS_TB_INSERT,1).
- Goal Conversion Rate: Ratio of number of goals scored by total shots taken. DAX formula - [TOTAL_GOALS]/[TOTAL_SHOTS_TAKEN]*100
- Average Distance of Shot: Represents the average distance from the goal post to the location where shot was taken. DAX formula - AVERAGE(RONALDO_YDS_TB_INSERT[DISTANCE_OF_SHOT])

### a. Data Transformation

Below is the list of transformation performed on the dataset.
1. Removed duplicate columns.
2. Extracted opponent team name by split the "HOME/AWAY" column based on "@" and "vs." characters. "@" denotes that Manchester United is playing an "Away" match, while "vs." denotes it is a "Home" match.
3. Extracted latitude and longitude from "LAT/LNG" column based on "," (comma) as a delimitor.
4. Created custom columns which will be used to enhance the visual representation. E.g. "BIN_IS_GOAL", "DISP_IS_AWAY_MATCH".
5. Specified the date format for the "DATE_OF_GAME" column.
6. Removed all the rows with missing values.

Preview of the dataset post transformation:-
<p align="center">
  <img width="600" height="350" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/Dataset_preview.png">
</p>

### b. Data Visualization

Implemented three distinct reports presenting varied statistical insights derived from Cristiano Ronaldo's shot analysis. Employed the page navigation feature to facilitate seamless and swift transitions between different sets of statistics, enhancing the user experience and enabling efficient exploration of the comprehensive shot dataset.

1. Overall Statistics

- This report features key performance indicators (KPIs) such as total goals, total shots, total matches, and conversion rate.
- Included an interactive table detailing goals scored in each season, allowing for dynamic exploration of seasonal goal distribution.
- Implemented a donut chart showcasing the percentage of goals scored in home and away games, providing a visual breakdown of scoring efficiency in different match settings.
- Integrated another interactive donut chart illustrating the percentage of goals scored based on shot basics, offering insights into the effectiveness of various shot techniques.
- Utilized page navigation features to enhance the interactivity of the report, enabling users to seamlessly explore and analyze overall statistics with ease.

<p align="center">
  <img width="600" height="600" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/Overall_Statistics.png">
</p>

2. Position Analysis

- This interactive report focuses on analyzing Cristiano Ronaldo's shot positions and performance against specific opponent teams.
- Incorporated a slicer allowing users to select the opponent team, facilitating dynamic filtering of the report based on the chosen team. A table showcases detailed match-specific information such as Date, Venue (Home or Away), Total Shots Taken, Goals Scored, Knockout Match status, and Game Season for matches played by Manchester United against the selected opponent.
- Created an interactive scatter plot, illustrating Ronaldo's positions on the football field when shots were taken against the selected opponent. Points on the scatter plot are color-coded based on whether the shot resulted in a goal or not. Enhanced the scatter plot with a tooltip displaying crucial details such as location (X, Y), goal status, distance of the shot, shot area, and more.
- KPI cards displays key metrics including Conversion Rate and Average Distance of the shot for quick reference.
- Integrated a line chart at the bottom depicting the timeline of shots, specifically remaining minutes vs. goals scored. Points on the timeline are color-coded based on whether the shot resulted in a goal or not. Enabled drill-down functionality on the timeline chart, clicking on a point reveals a detailed view of remaining seconds vs. goals scored during that specific minute.

<p align="center">
  <img width="600" height="600" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/Position_Analysis.png">
</p>

3. Shot Analysis

- This report helps to efficiently visualize the distribution and performance of Ronaldo's shots based on power and shot area and the importance of conversion rate.
- Implemented a stacked column chart showcasing the conversion rates for the top 5 shot types, providing a detailed comparison of their effectiveness.
- Designed an interactive donut chart displaying the distribution of goals scored based on the power of the shot, offering a visual breakdown of scoring efficiency at different power levels.
- Incorporated a dynamic funnel chart representing the number of goals scored categorized by the area of the shot (Center, Right Side, Left Side, Mid Ground, etc.), offering a visual hierarchy of goal-scoring patterns.
- Introduced KPI cards displaying average distance of the shot and average shots taken per match, providing concise summaries of key aspects of Ronaldo's shooting performance.

<p align="center">
  <img width="600" height="600" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/Shot_Analysis.png">
</p>

### Video Presentation

Below video showcases all the functionalities and capabilities of this Power BI Project. It serves as a comprehensive demonstration of how users can interact with and leverage the report for informed decision-making.

<p align="center">
  <img width="600" height="600" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/Ronaldo_Data_Visualization.gif">
</p>


## Conclusion

This project delivers a comprehensive analysis of Cristiano Ronaldo's shot performance during his Manchester United tenure. The 'Overall Statistics' report offers a macroscopic view, highlighting total goals, shots, matches, and conversion rates. The 'Position Analysis' report delves into match details, shot positions, and timeline dynamics against specific opponents. The 'Shot Analysis' report provides nuanced insights, focusing on top shot types, power distribution, and goal-scoring patterns. Together, these reports offer a holistic understanding, empowering stakeholders with actionable insights into Ronaldo's prolific goal-scoring journey at Manchester United.

## Refrences

S3 to Snowflake Data Ingestion : [Snowflake Website](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-auto-s3)
