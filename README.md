# SQL-Data-Cleaning
This is an educational project on data cleaning and preparation using SQL. The original database in CSV format is located in the file club_member_info.csv. Here, we will explore the steps that need to be applied to obtain a cleansed version of the dataset.



|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|


## Create a new table

Let's generate a new table where we can manipulate and restructure the data without modifying the original dataset
-- club_member_info definition

```sql
CREATE TABLE club_member_info_cleaned (
	full_name VARCHAR(50),
	age INTEGER,
	martial_status VARCHAR(50),
	email VARCHAR(50),
	phone NVARCHAR(50),
	full_address NVARCHAR(50),
	job_title VARCHAR(50),
	membership_date NVARCHAR(50)
);
```

## Copy all values from original table

```sql
INSERT INTO club_member_info_cleaned
SELECT * FROM club_member_info;
```


## Clean Data
1. In the first step, we should review all the data in the table
```sql
SELECT * 
FROM club_member_info;
```

2. Fix the full_name column — check the result first
```sql
SELECT UPPER(SUBSTRING(LTRIM(full_name),1))
FROM club_member_info;
```

3. Fix, Update and Commit full_name column
Result: I will adjust it so that all letters are capitalized and aligned to the left.
```sql
UPDATE club_member_info
SET full_name = UPPER(SUBSTR(LTRIM(full_name), 1));
```

4. After that, I noticed that the age column had many missing values or outliers, so I fixed the age column.
Result: I will replace the outliers with the most frequently occurring age, which is 40.
```sql
WITH mode_val AS 
(
    SELECT age
    FROM club_member_info
    GROUP BY age
    ORDER BY COUNT(*) DESC
    LIMIT 1
)
UPDATE club_member_info
SET age = (SELECT age FROM mode_val)
WHERE age < 1 OR age > 100 OR age IS NULL;
```

5. After that, I noticed that the martial_status column had many missing values, so I fixed the age column.
Result: I will also replace the null values with the most frequently occurring value.
```sql
WITH update_status AS 
(
    SELECT martial_status as ms 
    FROM club_member_info
    GROUP BY martial_status
    ORDER BY COUNT(*) DESC
    LIMIT 1
)
UPDATE club_member_info
SET  martial_status = (SELECT ms FROM update_status)
WHERE TRIM(martial_status) IS NULL;
```
6. However, to ensure that my results are logically consistent, I ran a query to check that individuals under the age of 16 do not have a marital_status of “married” or “divorced.”
```sql
SELECT martial_status
FROM club_member_info
WHERE age < 16 AND (martial_status IN ('married','divorced'));
```

7. Then I noticed the phone column also had missing values, so I decided to check the results before updating to make sure nothing gets updated incorrectly. For this column, if a value is missing or incorrectly formatted, I’ll set it to 'ERROR' by default.
```sql
SELECT
	CASE 
		WHEN LENGTH(TRIM(phone)) < 12 THEN 'ERROR'
		ELSE phone
	END
FROM club_member_info;
```

8. After confirming the above query returned no issues, I ran the update command.
```sql
UPDATE club_member_info
SET phone = 
	CASE 
		WHEN LENGTH(TRIM(phone)) < 12 THEN 'ERROR'
		ELSE phone
	END;
```

9. Check empty values in job_title column
```sql
SELECT COUNT(job_title)
FROM club_member_info
WHERE LENGTH(TRIM(job_title)) < 1;
```

10. Update job_title column, in this column, if the values are empty, I will default them to 'No Info'.
```sql
UPDATE club_member_info
SET job_title =
	CASE
		WHEN LENGTH(TRIM(job_title)) < 1 THEN 'No Info'
		ELSE job_title
	END;
```
## Result
[Uploading Club_member_info_cleaned.sql…]()
