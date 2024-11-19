# Patient_appointment_attendance

## Table of content

- [Project Overview](#project-overview)
- [Data source](#data-source)
- [Tools](#tools)
- [Data cleaning](#data-cleaning)
- [Exploratory data analysis](#exploratory-data-analysis)
- [Data analysis](#data-analysis)
- [Results and findings](#results-and-findings)
- [Recomendations](#recomendations)
  
### Project Overview
I used SQL to clean and preprocess the patient appointment dataset from Kaggle, which contains over 100,000 patient appointments. Used advanced SQL queries to segment patients based on their no-show rates, age, gender and disease type. Segmented data by age groups to uncover attendance rates by life stage.Used Power BI to visualize the no-show trends and provide recommendations to improve patient attendance at appointments.

<img width="630" alt="Patient_appointment_dashboard" src="https://github.com/user-attachments/assets/d808cf3e-cee0-462a-8af4-c59f24cbb9d9">

### Data source
The secondary data used for this project it the 'Medical_Appointment_No-Shows.csv'. It contains detailed information on patients attendance to the scheduled appointments.

### Tools
- Excel - Data exploration
- My SQL - Data cleaning and analysis
- Power Bi - Data visualization

### Data cleaning
In this project, I worked with the Patient Appointment No-Shows dataset to ensure it was ready for analysis. The dataset required extensive cleaning to correct inconsistencies in column names, data types, and values. Below is a summary of the key steps I followed:
- Initial Exploration: Retrieved the first few rows and checked data types using SQL queries (SELECT *, DESCRIBE healthcare), identifying issues with column names and types.
- Column Name and Data Type Corrections: Used ALTER TABLE queries to rename columns for clarity and update data types where needed (e.g., converting appointmentday to DATE).
    - ```sql
      ALTER TABLE healthcare CHANGE COLUMN patientid patient_id BIGINT
      ALTER TABLE healthcare CHANGE COLUMN Appointmentday appointment_day DATE
      ALTER TABLE healthcare CHANGE COLUMN Age age INT
      ALTER TABLE healthcare CHANGE COLUMN Neighbourhood location VARCHAR(30)
      ALTER TABLE healthcare CHANGE COLUMN SCHOLARSHIP scholarship VARCHAR(10)
      ALTER TABLE healthcare CHANGE COLUMN hipertension hypertension VARCHAR(30)
      ALTER TABLE healthcare CHANGE COLUMN diabetes diabetes VARCHAR(10)
      ALTER TABLE healthcare CHANGE COLUMN alcoholism alcoholism VARCHAR(30)
      ALTER TABLE healthcare CHANGE COLUMN handcap handicap VARCHAR(30)
      ALTER TABLE healthcare CHANGE COLUMN sms_received sms_received VARCHAR(30)
      ALTER TABLE healthcare CHANGE COLUMN showed_up showed_up VARCHAR(30)
      ALTER TABLE healthcare CHANGE COLUMN `date.diff`date_diff int

- Boolean Values Update: Columns like scholarship, hypertension, diabetes, alcoholism, handicap, sms_received, and showed_up had values stored as TRUE or FALSE. I updated these to 1 (TRUE) and 0 (FALSE) using SQL UPDATE queries.
    - ```sql
      SET SQL_SAFE_UPDATES = 0;
      UPDATE healthcare 
      SET scholarship = CASE 
      WHEN scholarship= "TRUE" THEN 1
      ELSE 0
      END 
      UPDATE healthcare 
      SET hypertension = CASE 
      WHEN hypertension= "TRUE" THEN 1
      ELSE 0
      END 
      UPDATE healthcare 
      SET diabetes = CASE 
      WHEN diabetes= "TRUE" THEN 1
      ELSE 0
      END 
      UPDATE healthcare 
      SET alcoholism = CASE 
      WHEN alcoholism= "TRUE" THEN 1
      ELSE 0
      END 
      UPDATE healthcare 
      SET handicap = CASE 
      WHEN handicap= "TRUE" THEN 1
      ELSE 0
      END 
      UPDATE healthcare 
      SET showed_up = CASE 
      WHEN showed_up= "TRUE" THEN 1
      ELSE 0
      END 
      UPDATE healthcare 
      SET sms_received = CASE 
      WHEN sms_received= "TRUE" THEN 1
      ELSE 0
      END 

- Null and Blank Value Check: Ran queries to ensure there were no missing (NULL) or blank values across key columns (e.g., patient_id, gender, appointment_id).
   - Patient_id
       - ```sql
         SELECT 
         COUNT(*) - COUNT(patient_id) AS patient_id_null_count,
         SUM(CASE WHEN TRIM(patient_id) = '' THEN 1 ELSE 0 END) AS patient_id_blank_count
         FROM healthcare;
  
    - Gender
      - ```sql
        SELECT 
        COUNT(*) - COUNT(gender) AS gender_null_count,
        SUM(CASE WHEN TRIM(gender) = '' THEN 1 ELSE 0 END) AS gender_blank_count
        FROM healthcare;
     
  - Appointment ID
     - ```sql
       SELECT 
       COUNT(*) - COUNT(appointment_id) AS appointment_id_null_count,
       SUM(CASE WHEN TRIM(appointment_id) = '' THEN 1 ELSE 0 END) AS appointment_id_blank_count
       FROM healthcare;


- Outlier Removal: Identified and removed 7 records where patient ages were greater than 100, considering them outliers.
   - ```sql
     SELECT COUNT(*) FROM healthcare
     WHERE AGE >100;
     
     SET SQL_SAFE_UPDATES = 0;
     DELETE FROM healthcare 
     WHERE age> 100;

- Duplicate Handling: While duplicates in the patient_id column were expected (since a patient can have multiple appointments), I verified the appointment_id column was unique.
   - ```sql
     SELECT patient_id, COUNT(patient_id) AS appointment_count
     FROM healthcare_new1
     GROUP BY patient_id
     HAVING COUNT(patient_id) > 1
     ORDER BY patient_id DESC;
     ##Checking for duplicates in the appointment_id column resulted in 0 duplicates.
     ## Hence I could use the appointment_id column as a unique identifier.

- Anomalies in Gender and Age: For patients with inconsistent gender or age entries, I imputed the most frequent value using the mode for each patient.
   - Gender
      - ```sql
        WITH gender_mode AS (SELECT patient_id, gender, ROW_NUMBER() 
        OVER (PARTITION BY patient_id ORDER BY COUNT(*) DESC) AS rn
        FROM healthcare
        GROUP BY patient_id, gender
        )
        UPDATE healthcare AS h
        SET gender = (SELECT gm.gender
        FROM gender_mode AS gm
        WHERE gm.patient_id = h.patient_id AND gm.rn = 1
        )
        WHERE     h.gender != (SELECT gm.gender
        FROM gender_mode AS gm
        WHERE gm.patient_id = h.patient_id AND gm.rn = 1
        );

Through these steps, I ensured the dataset was clean, consistent, and ready for further analysis, leading to more reliable insights into patient attendance patterns.

### Exploratory data analysis
EDA involved exploring the medical appointment dataset to answer key questions such as: 
- What percentage of patients did not show up for their appointments?
- How do no-show rates differ by age-group,gender and chronic disease?
- How often do patients revist the hospital ?

### Data analysis
Here are the interesting queries that I used in my data analysis process.
- To answer the first question: What percentage of patients did not show up for their appointments?
 - ```sql
    SELECT showed_up,COUNT(*) AS total_count, COUNT(*) * 100.0 / (SELECT COUNT(*) FROM healthcare) AS
    percentage
    FROM  healthcare
    GROUP BY showed_up;
- For the second question: How do no-show rates differ by age-group,gender and chronic disease?
   - Codes are below:
   - AGE-GROUP
     -  ```SQL
         SELECT showed_up,COUNT(showed_up) AS show_count,
         (COUNT(showed_up)/(SELECT COUNT(*) FROM healthcare))*100 AS percent_show,
         (CASE WHEN age BETWEEN 0 AND 18 THEN 'children'
         WHEN age BETWEEN 19 AND 35 THEN 'young_adults'
         WHEN age BETWEEN 36 AND 45 THEN 'adults'
         WHEN age BETWEEN 46 AND 55 THEN 'middle_aged_adults'
         WHEN age BETWEEN 56 AND 70 THEN 'seniors'
         WHEN age >70 THEN 'elderly'
         ELSE NULL
         END) AS age_category FROM healthcare
         GROUP BY showed_up, age_category
         ORDER BY showed_up
  - GENDER
      - ```SQL
          SELECT gender,showed_up,COUNT(*) AS total_appointments,
          (COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY gender)) AS percentage
          FROM  healthcare
          GROUP BY gender, showed_up;
  - CHRONIC DISEASE
    - Diabetes
        - ```sql
          SELECT diabetes,showed_up,COUNT(*) AS showed_count,
          (COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY diabetes)) AS rate_effect
          FROM healthcare
          GROUP BY diabetes, showed_up;
    
    - Hypertension
        - ```sql
          SELECT hypertension,showed_up,COUNT(*) AS showed_count,
          (COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY diabetes)) AS rate_effect
          FROM healthcare
          GROUP BY hypertension, showed_up;
- To answer the third question: How often do patients revist the hospital based on thier age group ?
  - ```sql
     SELECT  CASE
     WHEN age BETWEEN 0 AND 18 THEN 'children'
	   WHEN age BETWEEN 19 AND 35 THEN 'young_adults'
	   WHEN age BETWEEN 36 AND 45 THEN 'adults'
	   WHEN age BETWEEN 46 AND 55 THEN 'middle_aged_adult'
	   WHEN age BETWEEN 56 AND 70 THEN 'seniors'
	   WHEN age > 70 THEN 'elderly'
     END AS age_category,
     COUNT(appointment_id) AS appointment_count
     FROM healthcare
     GROUP BY age_category
     ORDER BY appointment_count DESC;

### Results and findings
- Analysis of the patient appointment dataset shows that women have a high tendency of responding positively to appointment compared to men. 91,122 women who showed up for the appointment make up 86% of the total population that showed up. These findings show that women have a high utilization rate to healthcare compared to men.
- Attendance by age category
  - Highest Attendance: Young adults have the highest attendance rate at 18%, closely followed by children at 17.77%.
  - Moderate Attendance: Seniors (14.71%) and middle-aged adults (12.41%) show moderate attendance.
  - Lower Attendance: Adults (11.38%) and elderly patients (5.42%) have the lowest attendance rates.
- Appointment distribution across different age categories show that:
   - Young Adults (24,385) and Children (24,098) have the highest number of appointments, closely followed by Seniors (19,539).
   - Middle-Aged Adults (16,541) and Adults (15,226) have moderate appointment counts.
   - Elderly (7,191) have the lowest number of appointments among all groups.
     
### Recomendations
- From the above findings its clear that the elderly are having a challenge with showing up for appointments. This could be due to challenges in transportation to the health centres. Implementation of sevices like transportation to the elderly might improve the attendance of this group.
- Promote preventive care for young adults and children: Through wellness programs encourage the use of preventive measures among the young adults and children.
- Implement mobile clinic for the elderly. This could be very helpful for the elderly who find it difficult to access the hospitals due to distance. 
