# Case Study - SAS Programming
As a part of SAS Programming Training I have completed this Case Study.
# Background Information
You are a SAS programmer with six months of experience who is in charge of creating basic reports and maintaining SAS programs. Recently, you completed a SAS Programming course, and your supervisor has given you your first SAS programming project.

# Business Problem
Your first project is to prepare and analyze Transportation Security Administration (TSA) Airport Claims data from 2002 through 2017. The TSA is an agency of the United States Department of Homeland Security that has authority over the security of the traveling public. A claim is filed if you are injured or your property is lost or damaged during the screening process at an airport.

To complete your project, you follow your supervisor's requirements, which is provided in the document. Here is what you need to do:
- Prepare the data
- Create one PDF  report that analyzes the overall data as well as the data for a dynamically specified state.

To prepare and analyze the data, you follow the requirements given to you by your supervisor.

# Requirements
- The final data should be in the permanent library tsa.
- Entirely duplicated records need to be removed from the data set.
- All "missing" and "-" values in the columns Claim_Type, Claim_Site, and Disposition must be changed to Unknown.
- Values in the columns Claim_Type, Claim_Site, and Disposition must follow the requirements in the data layout.
- All StateName values should be in proper case.
- All State values should be in uppercase.
- You create a new column named Date_Issues with a value of Needs Review to indicate that a row has a date issue. Date issues consist of the following:
           - a missing value for Incident_Date or Date_Received 
       an Incident_Date or Date_Received value out of the predefined year range of 2002 through 2017
       an Incident_Date value that occurs after the Date_Received value
