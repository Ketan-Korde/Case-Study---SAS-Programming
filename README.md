# Case Study - SAS Programming
As a part of SAS Programming Training I have completed this Case Study.
# Background Information
You are a SAS programmer with six months of experience who is in charge of creating basic reports and maintaining SAS programs. Recently, you completed a SAS Programming course, and your supervisor has given you your first SAS programming project.

# Business Problem
Your first project is to prepare and analyze Transportation Security Administration (TSA) Airport Claims data from 2002 through 2017. The TSA is an agency of the United States Department of Homeland Security that has authority over the security of the traveling public. A claim is filed if you are injured or your property is lost or damaged during the screening process at an airport.

# Requirements
To complete your project, you follow your supervisor's requirements, which is provided in the document. Here is what you need to do:
- Prepare the data
- Create one PDF  report that analyzes the overall data as well as the data for a dynamically specified state.

To prepare and analyze the data, you follow the requirements given to you by your supervisor.

# Data Requirements
- The final data should be in the permanent library tsa.
- Entirely duplicated records need to be removed from the data set.
- All "missing" and "-" values in the columns Claim_Type, Claim_Site, and Disposition must be changed to Unknown.
- Values in the columns Claim_Type, Claim_Site, and Disposition must follow the requirements in the data layout.
- All StateName values should be in proper case.
- All State values should be in uppercase.
- You create a new column named Date_Issues with a value of Needs Review to indicate that a row has a date issue. 
- Date issues consist of the following: 
      
      1. Missing value for Incident_Date or Date_Received.
      2. An Incident_Date or Date_Received value out of the predefined year range of 2002 through 2017.
      3. An Incident_Date value that occurs after the Date_Received value.
- Remove the County and City columns.
- Currency should be permanently formatted with a dollar sign and include two decimal points.
- All dates should be permanently formatted in the style 01JAN2000.
- Permanent labels should be assigned columns by replacing the underscores with a space.
- Final data should be sorted in ascending order by Incident_Date.

# Report Requirements
The final single-PDF report needs to exclude all rows with date issues in the analysis and answer the following questions:
          
    1. How many date issues are in the overall data?
    2. How many claims per year of Incident_Date are in the overall data? Be sure to include a plot.
    
A user should be able to dynamically input a specific state value and answer the following questions:
    
    1. What are the frequency values for Claim_Type for the selected state?
    2. What are the frequency values for Claim_Site for the selected state?
    3. What are the frequency values for Disposition for the selected state?
    4. What is the mean, minimum, maximum, and sum of Close_Amount for the selected state? Round to the nearest integer.

# SAS Program
Access Data : Importing CSV file Data into SAS
    
    /* Creating macro variable to assign path */
    
    %LET path=/home/u61989325/Case Study;
    
    /* Creating library named tsa */
    
    LIBNAME tsa "&path/output";

    /* Using VALIDVARNAME options statement to follow SAS naming convention */
    
    OPTIONS VALIDVARNAME=v7;

    /* Importing CSV file data into tsa library AS SAS Dataset */
    
    PROC IMPORT DATAFILE="&path/TSAClaims2002_2017.csv"
                DBMS=CSV
                OUT=tsa.ClaimsImport 
                REPLACE;
     GUESSINGROWS=MAX;
    RUN;
    
Explore Data:
    
    /* Observing first 20 rows of data */
    
    PROC PRINT DATA=tsa.ClaimsImport (OBS=20);
    RUN;

    /* Checking variable types, length, format and informats */
    
    PROC CONTENTS DATA=tsa.ClaimsImport;
    RUN;
    
    /* Exploring variables mentioned in Data Requirements and determining whether any adjustments needed in data */
    
    PROC FREQ DATA=tsa.ClaimsImport;
     TABLES claim_site claim_type Disposition Date_Received Incident_Date / NOCUM NOPERCENT;
     FORMAT Date_Received Incident_Date YEAR4.;
    RUN;
    
Prepare Data:
    
    /* removing duplicates */
    
    PROC SORT DATA=tsa.CLAIMSIMPORT NODUPKEY 
              OUT=tsa.Claims_NoDups;
      BY _ALL_;
    RUN;
    
    /* Sorting the data by Incident_Date */
    
    PROC SORT DATA=tsa.Claims_NoDups;
     BY Incident_Date;
      FORMAT Incident_Date YEAR4.;
    RUN;
    
    /* Preparing Data according to Data Requirements */
   
    DATA tsa.Claims_Cleaned;
     SET tsa.claims_nodups;
     
      *Using Conditional Formatting to do adjustments in the data;
      
     IF claim_site IN ("-"," ") THEN claim_site="Unknown";
     
     IF Disposition IN ("-"," ") THEN Disposition="Unknown";
     ELSE IF Disposition="Closed: Canceled" THEN Disposition="Closed:Canceled";
     ELSE IF Disposition="losed: Contractor Claim" THEN Disposition="Closed:Contractor Claim";
     
     IF Claim_Type IN ("-"," ") THEN Claim_Type="Unknown";
     ELSE IF Claim_Type IN ("Passenger Property Loss/Personal Injur","Passenger Property Loss/Personal Injury")
        THEN Claim_Type="Passenger Property Loss";
     ELSE IF Claim_Type="Property Damage/Personal Injury" THEN Claim_Type="Property Damage";
    
    IF (Incident_Date=. OR Date_Received=. OR 
    YEAR(Incident_Date) < 2002 OR 
    YEAR(Incident_Date) > 2017 OR 
    YEAR(Date_Received) < 2002 OR 
    YEAR(Date_Received) > 2017 OR
    Incident_Date > Date_Received)
    THEN Date_Issues="Needs Review";
    
    *Using funtions to assign cases for observation as menioned in Data Requirements;
     
     State=UPCASE(State);
     StateName=PROPCASE(StateName);
    
    *Giving Proper Lables;
    
    LABEL Airport_Code = "Airport Code"
          Airport_Name = "Airport Name"
          Claim_Number = "Claim Number"
          Claim_Site = "Claim Site"
          Claim_Type = "Claim Type"
          Close_Amount = "Close Amount"
          Date_Issues = "Date Issues"
          Date_Received = "Date Received"
          Incident_Date = "Incident Date"
          Item_Category = "Item Category"
          StateName = "State Name";
     
     *Applying proprer formats to date and amount;
     
     FORMAT Incident_Date Date_Received DATE9. Close_Amount DOLLAR20.2;
     
     *Dropping county and city column;
     
     DROP  County City;
     
    RUN;
    
Analyze and Export Data:
    
    *creating Macro Variable StateName so User can dynamically input specific state value;
    
    %LET StateName=California;
    
    ODS PDF FILE="&path/output/ClaimsReports.pdf"
        STYLE=Analysis;
    
    OPTIONS NODATE; 
    ODS NOPROCTITLE;
    ODS PROCLABEL "Overall Date Issues";
    
    *Calculating frequency of overall Date Issues;
    
    TITLE "Overall Date Issues";
    
    PROC FREQ DATA=tsa.claims_cleaned;
      TABLES Date_Issues / NOCUM NOPERCENT;
    RUN;
    TITLE;
    
    *Calculating Overall Claims by Year and Plotting Graph;
    
    ODS PROCLABEL "Overall Claims by Year";
    ODS GRAPHICS ON;
    
    TITLE "Overall Claims by Year";
    
    PROC FREQ DATA=tsa.claims_cleaned;
     TABLES Incident_Date / NOCUM NOPERCENT PLOTS=FREQPLOT;
      FORMAT Incident_Date YEAR4.;
         WHERE Date_Issues IS MISSING;
    RUN;
    TITLE;
    
    *Calculating frequency for Claim_Type, Claim_Site and Disposition;
    
    ODS PROCLABEL "&StateName Claims Overview";
    TITLE "&StateName Claim Type, Claim Site & Disposition Frequencies";
    
    PROC FREQ DATA=tsa.claims_cleaned;
      TABLES Claim_Type Claim_Site Disposition / NOCUM NOPERCENT;
        WHERE StateName="&StateName" AND Date_Issues IS MISSING;
    RUN;
    TITLE;
    
    *Calculating Statistics for Close_Amount Column;
    
    ODS PROCLABEL "&StateName Close Amount Statistics";
    TITLE "&StateName Close Amount Statistics";
    
    PROC MEANS DATA=tsa.Claims_Cleaned MEAN MIN MAX SUM MAXDEC=0;
     VAR Close_Amount;
      WHERE StateName="&StateName" AND Date_Issues IS MISSING;
    RUN;
    TITLE;
    
    ODS PDF CLOSE;
    ODS PROCTITLE;
    OPTIONS DATE;
    
# Report Requirement Solution

1. How many date issues are in the overall data?
    
      ![App Screenshot](https://github.com/Ketan-Korde/Case-Study---SAS-Programming/blob/a9da932f75beb97304b656616b6591a0b405aebd/Frequency%20of%20Overall%20date%20issue.PNG)

2. How many claims per year of Incident_Date are in the overall data? Include a plot.

![App Screenshot](https://github.com/Ketan-Korde/Case-Study---SAS-Programming/blob/a9da932f75beb97304b656616b6591a0b405aebd/Frequency%20of%20Oveall%20Claims%20per%20year.PNG)

3.  What are the frequency values for Claim_Type, Claim_Site and Disposition for the selected state?
   
       Note: Macro variable is created to dynamically input State Name Value here it macro variable contains value "California".

   ![App Screenshot](https://github.com/Ketan-Korde/Case-Study---SAS-Programming/blob/a9da932f75beb97304b656616b6591a0b405aebd/Frequency%20of%20diferent%20variables.PNG)
   
4. What is the mean, minimum, maximum, and sum of Close_Amount for the selected state?

![App Screenshot](https://github.com/Ketan-Korde/Case-Study---SAS-Programming/blob/a9da932f75beb97304b656616b6591a0b405aebd/Basic%20Summery%20Statistics.PNG)

















