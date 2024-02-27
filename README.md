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
  <img width="700" height="400" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/S3_to_Snowflake_Data_Loaded.png">
</p>

### 3. Data Transformation & Visualization in Microsoft Power BI

We will now create key parameters which are essential for performing calculations and analysis. These parameters allow us to define custom calculations/aggregations based on existing column data. For e.g. 
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
  <img width="700" height="400" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/Dataset_preview.png">
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

<p align="center">
  <img width="600" height="600" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/Position_Analysis.png">
</p>

3. Shot Analysis

<p align="center">
  <img width="600" height="600" src="https://github.com/yash-pimpale/Ronaldo_United_Era_Shots_Analysis/blob/main/Media/Shot_Analysis.png">
</p>


Create interactive report for each type of role by adding respective filters as mentioned above. Add slicer for qualifier and super 12 stage. Create buttons for each role so that user can navigate to respective page by click on it (control + click). Add table displaying selected players based on criteria and add tooltip on player name to display indetailed details of the player. Add onclick functionality such that when user clicks on the player, its statistics are displayed (otherwise hidden). Select multiple player to check their combined stats. Add 4 separate stacked line chart for performance metrics. 

Create interactive reports in Power BI involves designing visualizations and user-friendly features to help users analyze data effectively. Here's how we can create interactive reports for each type of role and incorporate the specified features:

- Role-Specific Pages: We will create separate report pages for each role i.e. Openers, Anchors, Finishers, All Rounders and Fast Bowlers. Added filters specific to each role, allowing users to select a role of interest. These will help users focus on the desired player criteria for each role.

- Stage Slicers: Included slicer that allow users to filter data based on the qualifier and Super 12 stages, enabling them to compare player performance in different tournament stages.

<p align="center">
  <img width="700" height="400" src="https://github.com/yash-pimpale/T20_World_Cup_2022_Dream_Team_Selection/blob/main/Media/Pages_and_Slicer.png">
</p>

- Role Buttons: Created buttons for each role to enable users to navigate to the respective role page with a simple click (Ctrl + click).

- Opposite Selection Table: Added a table to display selected players who meet the role-specific criteria. This table includes player names, role-specific key statistics and an interactive tooltip. When users hover over a player's name, the tooltip will display detailed player information, such as player's picture, country, batting and bowling statistics (depending upon the role). Also allowed users to select multiple players from the table to compare their combined statistics.

<p align="center">
  <img width="700" height="400" src="https://github.com/yash-pimpale/T20_World_Cup_2022_Dream_Team_Selection/blob/main/Media/Report_Hover_Functionality.png">
</p>

- OnClick Functionality: Incorporated onClick functionality for player names in the selection table. When a user clicks on a player's name, additional key statistics such as Batting Average, Strike Rate, Economy, etc will become visible (otherwise hidden) for that player according to the role.

<p align="center">
  <img width="700" height="400" src="https://github.com/yash-pimpale/T20_World_Cup_2022_Dream_Team_Selection/blob/main/Media/Report_OnClick_Functionality.png">
</p>

- Stacked Line Charts: Created four separate stacked line charts to visualize performance metrics. These charts can include metrics like batting average, strike rate, wickets taken, or any other relevant statistics vs match day, for players in each role.

Below video showcases all the functionalities and capabilities of this Power BI report. It serves as a comprehensive demonstration of how users can interact with and leverage the report for informed decision-making.

<p align="center">
  <img width="700" height="400" src="https://github.com/yash-pimpale/T20_World_Cup_2022_Dream_Team_Selection/blob/main/Media/World_Cup_Dream_Team_Report.gif">
</p>


## Conclusion

This project demonstrates the power of data analysis and visualization in sports selection process. It enables cricket enthusiasts, coaches, and team management to make informed decisions and assemble the most competitive team for a prestigious tournament like the ICC T20 World Cup. The project's structured approach and user-friendly reports empower stakeholders to optimize player selection based on specific criteria and roles, ultimately enhancing the team's chances of success.



Refrences

S3 to Snowflake Data Ingestion : [Snowflake Website](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-auto-s3)
