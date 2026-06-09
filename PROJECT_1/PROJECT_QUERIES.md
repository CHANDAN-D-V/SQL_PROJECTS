

 # Club Member Info

 ## AN SQL DATA CLEANING PROJECT

1. Check for duplicate entries and remove them.
2. Remove extra spaces and/or other invalid characters.
3. Separate or combine values as needed.
4. Ensure that certain values (age, dates...) are within certain range.
5. Check for outliers.
6. Correct incorrect spelling or inputted data.
7. Adding new and relevant rows or columns to the new dataset.
8. Check for null or empty values.

Lets take a look at the first few rows to examine the data in its original form
````sql
select *from club_member_info limit 10;
````
-- RESULT :

<img width="1286" height="261" alt="FINAL CMI" src="https://github.com/user-attachments/assets/66ac859e-7e9f-44ca-a329-4070b1765a9c" />




--------------------------------------------------------------------------------------------------------------------------------------------------------

**Let's create a temp table where we can manipulate and restructure the data without altering the original**  

````sql
DROP TABLE IF EXISTS cleaned_club_member_info;

CREATE TABLE cleaned_club_member_info (
    member_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    age INT,
    marital_status VARCHAR(255),
    member_email VARCHAR(255),
    phone VARCHAR(255),
    street_address VARCHAR(255),
    city VARCHAR(255),
    state VARCHAR(255),
    occupation VARCHAR(255),
    membership_date CHAR(15)
) AS
SELECT
````    
- Some of the names have extra spaces and special characters. 
Trim access whitespace, remove special characters and convert to lowercase.

- In this particular dataset, special characters only occur in the first name that can be removed using a simple regex
    
    ````sql
	
    LOWER(TRIM(REGEXP_REPLACE(SUBSTRING_INDEX(TRIM(ï»¿full_name), ' ', 1), '[^a-zA-Z0-9]', ''))) AS first_name,
	
    ````
    
- Some last names have multiple words ('de palma' or 'de la cruz').
- Convert the string to an array to calculate its length and use a  case statement to find entries with those particular types of surnames.
  
   ````sql
   CASE
        WHEN LENGTH(TRIM(REPLACE(ï»¿full_name, ' ', ''))) - LENGTH(REPLACE(TRIM(ï»¿full_name), ' ', '')) = 2 THEN CONCAT(SUBSTRING_INDEX(SUBSTRING_INDEX(ï»¿full_name, ' ', -2), ' ', 1), ' ', SUBSTRING_INDEX(ï»¿full_name, ' ', -1))
        WHEN LENGTH(TRIM(REPLACE(ï»¿full_name, ' ', ''))) - LENGTH(REPLACE(TRIM(ï»¿full_name), ' ', '')) = 3 THEN CONCAT(SUBSTRING_INDEX(SUBSTRING_INDEX(ï»¿full_name, ' ', -3), ' ', 1), ' ', SUBSTRING_INDEX(SUBSTRING_INDEX(ï»¿full_name, ' ', -2), ' ', 1), ' ', SUBSTRING_INDEX(ï»¿full_name, ' ', -1))
        ELSE SUBSTRING_INDEX(ï»¿full_name, ' ', -1)
    END AS last_name,
````   
````
- During data entry, some ages have an additional digit at the end. Remove the last digit when a 3 digit age value occurs
- Check if value is empty. If empty `''` then change value to `NULL`.
- First cast the integer to a string and test the character length.
- If condition is true, cast the integer to text, extract first 2 digits and cast back to numeric type.

    ````sql
      CASE
           WHEN age = '' THEN NULL
           WHEN LENGTH(age) = 3 THEN CAST(LEFT(age, 2) AS UNSIGNED)
           ELSE age
      END AS age,
    ````
   
   
   
- Trim whitespace from maritial_status column and if empty, ensure it's of null type
  ````sql
  CASE
        WHEN TRIM(marital_status) = '' THEN NULL
        ELSE TRIM(marital_status)
    END AS marital_status,
````
````  
  
- Email addresses are necessary and this dataset contains valid email addresses. Since email addresses are case insensitive, convert to lowercase and trim off any whitespace.

 ````sql
  
    LOWER(TRIM(email)) AS member_email,

````
  
  
  - Trim whitespace from phone column and if empty or incomplete, ensure it's of null type
  ````sql  
    CASE
        WHEN TRIM(phone) = '' THEN NULL
        WHEN LENGTH(TRIM(phone)) < 12 THEN NULL
        ELSE TRIM(phone)
    END AS phone,
````
  
  
  - Members must have a full address for billing purposes. However, many members can live in the same household so address cannot be unique.
  - Convert to lowercase, trim off any whitespace and split the full address to individual street address, city, and state.
   ````sql
    LOWER(TRIM(SUBSTRING_INDEX(full_address, ',', 1))) AS street_address,
    LOWER(TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(full_address, ',', 2), ',', -1))) AS city,
    LOWER(TRIM(SUBSTRING_INDEX(full_address, ',', -1))) AS state,
````
  
  
  - Some job titles define a level in roman numerals (I, II, III, IV). Convert levels to numbers and add a descriptor (ex. Level 3).
  - Trim whitespace from the job title, rename to occupation and if empty convert to null type.
 ````sql   
    CASE
        WHEN TRIM(LOWER(job_title)) = '' THEN NULL
        ELSE 
            CASE
                WHEN LOWER(SUBSTRING_INDEX(job_title, ' ', -1)) = 'i' THEN REPLACE(LOWER(job_title), ' i', ', level 1')
                WHEN LOWER(SUBSTRING_INDEX(job_title, ' ', -1)) = 'ii' THEN REPLACE(LOWER(job_title), ' ii', ', level 2')
                WHEN LOWER(SUBSTRING_INDEX(job_title, ' ', -1)) = 'iii' THEN REPLACE(LOWER(job_title), ' iii', ', level 3')
                WHEN LOWER(SUBSTRING_INDEX(job_title, ' ', -1)) = 'iv' THEN REPLACE(LOWER(job_title), ' iv', ', level 4')
                ELSE LOWER(TRIM(job_title))
            END 
    END AS occupation,
    
   
    
     CASE
        WHEN TRIM(membership_date) = '' THEN NULL
        ELSE TRIM(membership_date)
    END AS membership_date
    
   
   FROM club_member_info;
  ````  
**RESULT :**

<img width="1492" height="327" alt="CLEANED TABLE BEFORE DATE UPDATE" src="https://github.com/user-attachments/assets/7acd39f6-024a-43a1-ab42-01f120906fec" />

    
    
  ---------------------------------------------------------------

- UPDATING THE CORRECT DATE FORMAT AND CHANGING THE DATATYPE
````sql
SET SQL_SAFE_UPDATES = 0 ;

UPDATE  cleaned_club_member_info SET membership_date = STR_TO_DATE(membership_date, '%d-%m-%Y' );

ALTER TABLE cleaned_club_member_info MODIFY COLUMN membership_date  DATE ;

````
   - A few members show membership_date year in the 1900's. Change the year into the 2000's.
````sql  
UPDATE cleaned_club_member_info
SET membership_date = DATE_ADD(membership_date, INTERVAL 100 YEAR)
WHERE YEAR(membership_date) < 2000;
```` 
 -----------------------------------------------------------------------------------------------------------------------------------------------


**Let's take a look at our final cleaned table data.**
````sql    
    
select *from cleaned_club_member_info
limit 10;
````
- RESULT :

  <img width="1487" height="292" alt="3 DATES UPDATED" src="https://github.com/user-attachments/assets/001df905-aae0-4f18-8fd6-47c1219f0c6e" />




---------------------------------------------------------------------------------------------------------------------------------------------------------

**DELETING DUPLICATE ENTRIES**


- Now that the data is cleaned, lets look for any duplicate entries.
- What is the record count?
````sql
SELECT count(*) AS record_count 
FROM cleaned_club_member_info;
````
-- Results:

<img width="167" height="65" alt="4 records count" src="https://github.com/user-attachments/assets/d131d30d-0a17-4b18-8283-c5fb47e64125" />



--------------------------------------------------------------------------



- All members must have a unique email address to join. Lets try to find duplicate entries.

````sql
SELECT member_email, count(member_email)
FROM cleaned_club_member_info
GROUP BY member_email
HAVING count(member_email) > 1;
````
-- Results: 10 duplicate entries.

<img width="422" height="252" alt="5 duplicate entries" src="https://github.com/user-attachments/assets/ef5d5440-7d13-4d87-ac85-4a330f80789b" />



------------------------------------------------------------------------

- Lets delete duplicate entries.
````sql
SET SQL_SAFE_UPDATES=0;

DELETE c1
FROM cleaned_club_member_info c1
JOIN cleaned_club_member_info c2 
ON c1.member_email = c2.member_email 
AND c1.member_id < c2.member_id;
````
- Let's Check the record count after deletion
````sql
SELECT COUNT(*) AS new_record_count 
FROM cleaned_club_member_info;
````
-- Results:

<img width="222" height="80" alt="6 new R C" src="https://github.com/user-attachments/assets/b5317430-44c8-4c73-a39a-cd18ff4692ba" />




-------------------------------------------------------------------------------------------------------------------------------------------------------------

**CORRECTIONS IN MARITAL STATUS**
 
 
- What is the record count where marial_status is null?    
````sql
SELECT count(*) AS null_record_count 
FROM cleaned_club_member_info
WHERE marital_status IS null;	
````
-- Results:

<img width="202" height="72" alt="7 NULL MARITAL STATUS" src="https://github.com/user-attachments/assets/0deece53-50b7-44a8-934c-35d3d756da4d" />

	

--------------------------------------------------------------------------

- What are the different maritial statuses?
````sql
SELECT marital_status , count(*) AS new_record_count 
FROM cleaned_club_member_info
GROUP BY marital_status;
````
-- Results:

<img width="300" height="182" alt="8 DIFF MARITAL STATUS W-N" src="https://github.com/user-attachments/assets/94c9e986-5210-48df-9063-5cdeb19f6f7f" />


--------------------------------------------------------------------------
    
- As we can see, we have a spelling error for 4 records.  Let's update the record and correct the error.    
````sql
UPDATE cleaned_club_member_info
SET marital_status = 'divorced'
WHERE marital_status = 'divored';
````
- Lets check the records
````sql
SELECT marital_status, count(*) AS new_record_count 
FROM cleaned_club_member_info
GROUP BY marital_status;
````
-- Results:

<img width="275" height="136" alt="9 CORRECTED DIVORCED" src="https://github.com/user-attachments/assets/c488b461-7c84-47cd-9bdd-a62605cb9824" />



---------------------------------------------------------------------------------------------------------------------------------

**CORRECTION IN STATE NAMES**

                                                            
- WE ALSO HAVE A FEW SPELLING MISTAKES IN STATE NAMES .
````sql
SELECT state
FROM cleaned_club_member_info
GROUP BY state;
    
UPDATE cleaned_club_member_info
SET state = 'kansas'
WHERE state = 'kansus';

UPDATE cleaned_club_member_info
SET state = 'district of columbia'
WHERE state = 'districts of columbia';

UPDATE cleaned_club_member_info
SET state = 'north carolina'
WHERE state = 'northcarolina';

UPDATE cleaned_club_member_info
SET state = 'california'
WHERE state = 'kalifornia';

UPDATE cleaned_club_member_info
SET state = 'texas'
WHERE state = 'tejas';

UPDATE cleaned_club_member_info
SET state = 'texas'
WHERE state = 'tej+f823as';

UPDATE cleaned_club_member_info
SET state = 'tennessee'
WHERE state = 'tennesseeee';

UPDATE cleaned_club_member_info
SET state = 'new york'
WHERE state = 'newyork';

UPDATE cleaned_club_member_info
SET state = 'puerto rico'
WHERE state = ' puerto rico';

SELECT count(DISTINCT state)
FROM cleaned_club_member_info;
	
SELECT state
FROM cleaned_club_member_info
GROUP BY state;
````
-----------------------------------------------------------------------------------------------------------------------------------------

### WE HAVE SUCCESSFULLY CLEANED THE DATA


**ORIGINAL TABLE** 
````sql
select *from club_member_info
limit 10;
````
- RESULT :

<img width="1286" height="261" alt="FINAL CMI" src="https://github.com/user-attachments/assets/200224a5-8603-44b9-824d-da0a74fb4eef" />




**CLEANED TABLE**
 ````sql                                                              
select *from cleaned_club_member_info
limit 10;
````
- RESULT :

<img width="1506" height="282" alt="FINAL CLEANED CMI" src="https://github.com/user-attachments/assets/083a1dd8-7dcc-45a8-a32f-675fae0d96fd" />





---------------------------------------------------------------------------------------------------------------------------------------------------------------
