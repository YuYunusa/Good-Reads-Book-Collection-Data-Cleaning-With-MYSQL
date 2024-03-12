# Good-Reads-Book-Collection-Data-Cleaning-With-MYSQL

This is a breakdown of how I resolved issues such as incorrect formatting, empty rows, missing values, and incorrect values using MYSQL during the data cleaning phase of my project [Goodreads Book Analysis](https://www.kaggle.com/datasets/cristaliss/ultimate-book-collection-top-100-books-up-to-2023).

## LOADING THE DATA

In this phase, I had to import the data into MYSQL, but first I had to create the database.

```sql
-- Create the 'goodreads' database
CREATE DATABASE goodreads;
USE goodreads
```

After that, I disabled 'Autocommit' to avoid automatically saving changes, and disabled 'Safe updates' in order to make changes to the table if needed.

```sql
-- Disable autocommit and safe updates for this session
SET autocommit = 0;
SET sql_safe_updates = 0;
```

I also set up a 'Rollback' function to basically undo any unwanted changes made to the dataset.

``` sql 
rollback;
```


Using the 'Create Table' function, I created the columns for the Goodreads dataset. Aside from the 'book_no' which was the primary key, every other column was set to the TEXT datatype in order to safely & quickly import the dataset.

``` sql
-- Create the 'goodreads_data' table
CREATE TABLE goodreads_data (
    Book_no INT,
    Book TEXT,
    Series TEXT,
    Release_number TEXT,
    Author TEXT,
    Description TEXT,
    Num_Pages TEXT,
    Format TEXT,
    Genres TEXT,
    Publication_Date TEXT,
    Rating TEXT,
    Number_of_voters TEXT,
    PRIMARY KEY (Book_no)
);
```

The dataset was then loaded into the database through the LOAD DATA INFILE method, which is faster than using the import wizard option.

``` sql
-- Load data from CSV file into the 'goodreads_data' table
LOAD DATA INFILE '\\ProgramData\\MySQL\\MySQL Server 8.0\\Uploads\\goodreads_top100_from1980to2023_v2 (1).CSV'
INTO TABLE goodreads.goodreads_data
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES;
```
## WRONG DATE FORMAT

The 'publication_date' column had date values in the wrong format, so new columns 'pub_date' & 'years' were created to insert the corrected date format and to extract the year of the date respectively.

``` sql
-- Add new date columns and populate them based on different date formats
ALTER TABLE goodreads_data
ADD COLUMN Pub_date DATE AFTER Publication_Date,
ADD COLUMN Years VARCHAR(10) AFTER Pub_Date;

-- Update 'pub_date' column based on the 'Publication_Date' column
UPDATE goodreads_data
SET pub_date = CASE
    WHEN Publication_Date LIKE '%-%' THEN date_format(str_to_date(Publication_Date, '%d-%M-%Y'), '%y-%m-%d')
    ELSE NULL
END;
```

## INCORRECT VALUES, NULL & EMPTY ROWS

The 'Release_number' & 'format' column had INCORRECT VALUES that were unwanted, the rows having these INCORRECT VALUES were set to NULL.

``` sql
-- Update 'Release_number' to NULL for specific cases
UPDATE goodreads_data
SET Release_number = NULL
WHERE Release_number IN ('_-%');

-- Update 'Release_number' to NULL for specific patterns
UPDATE goodreads_data
SET Release_number = NULL
WHERE Release_number LIKE '%,%';

-- Update 'Release_number' to NULL for specific patterns
UPDATE goodreads_data
SET Release_number = NULL
WHERE Release_number LIKE '%.%';

-- Update 'Release_number' to NULL for specific patterns
UPDATE goodreads_data
SET Release_number = NULL
WHERE Release_number LIKE '_A';

-- Update 'format' to NULL for specific patterns
UPDATE goodreads_data
SET format = NULL
WHERE format LIKE '% pages';
```

Other columns with similar issues were fixed by using the REPLACE function which led to having EMPTY ROWS in some of the columns, which were then set to NULL.

``` sql
-- Replace single quotes in 'Genres' column
UPDATE goodreads_data
SET Genres = replace(Genres, '''', '');

-- Replace single quotes and brackets in 'format' column
UPDATE goodreads_data
SET format = replace(replace(replace(format, '''', ''), '[', ''), ']', '');

-- Replace unwanted text in 'Rating' column
UPDATE goodreads_data
SET Rating = replace(Rating, 'real', '');


-- Set 'Num_Pages' to NULL where it's empty
UPDATE goodreads_data
SET Num_Pages = NULL
WHERE Num_Pages = '';

-- Set 'Series' to NULL where it's empty
UPDATE goodreads_data
SET Series = NULL
WHERE Series = '';

-- Set 'Release_number' to NULL where it's empty
UPDATE goodreads_data
SET Release_number = NULL
WHERE Release_number = '';

-- Set 'Format' to NULL where it's empty
UPDATE goodreads_data
SET Format = NULL
WHERE Format = '';

-- Set 'Genres' to NULL where it's empty
UPDATE goodreads_data
SET Genres = NULL
WHERE Genres = '';

-- Set 'Rating' to NULL where it's empty
UPDATE goodreads_data
SET Rating = NULL
WHERE Rating = '';
```

The correctly formatted date values in the 'pub_date' column still had INCORRECT VALUES with some of the dates turning a date of "1966-01-01" into "2066-01-01", these issues were addressed and the year of the 'pub_date' column was extracted into the 'years' column successfully.

``` sql
-- Correcting date values for specific years
UPDATE goodreads_data
SET Pub_date = '1968-01-01'
where Pub_date = '2068-01-01';

UPDATE goodreads_data
SET Pub_date = '1967-01-01'
where Pub_date = '2067-01-01';

UPDATE goodreads_data
SET Pub_date = '1966-01-01'
where Pub_date = '2066-01-01';

UPDATE goodreads_data
SET Pub_date = '1963-01-01'
where Pub_date = '2063-01-01';

UPDATE goodreads_data
SET Pub_date = '1963-01-01'
where Pub_date = '2063-01-01';

UPDATE goodreads_data
SET Pub_date = '1960-01-01'
where Pub_date = '2060-01-01';

UPDATE goodreads_data
SET Pub_date = '1956-11-01'
where Pub_date = '2056-11-01';

UPDATE goodreads_data
SET Pub_date = '1955-01-01'
where Pub_date = '2055-01-01';

UPDATE goodreads_data
SET Pub_date = '1951-01-01'
where Pub_date = '2051-01-01';

UPDATE goodreads_data
SET Pub_date = '1946-01-01'
where Pub_date = '2046-01-01';

-- Fill up the 'Years' column with the year from 'Pub_date'
UPDATE goodreads_data
SET Years = YEAR(Pub_date);
```

## MISSING VALUES

The 'Author' column had a single MISSING VALUE for one of the books, after doing research and found the author related to the book, the changes were made.

``` sql
-- Update specific record's author
UPDATE goodreads_data
SET Author = 'Isabel Allende'
WHERE Book = 'The Infinite Plan';
```

A similar solution was made for the 'pub_date' column that had a single MISSING VALUE.

``` sql
-- Correcting the publication date for a specific record
UPDATE goodreads_data
SET Pub_date = '2020-07-31'
WHERE Series = 'VanWest' AND BOOK = 'The Present';
```

The 'ratings' column on the other hand had multiple MISSING VALUES, so rather than leaving them empty, they were filled up with the average rating of the values in the same column.

``` sql
-- Fill up empty rating rows with an average rating
UPDATE goodreads_data
SET rating = 4.02 
WHERE rating IS NULL;
```

The 'format' column also had multiple MISSING VALUES, research was made to correct this but unfortunately.

``` sql
-- fill up empty format rows with the correct format
-- note: books that had their format as 'pages' were left empty
UPDATE goodreads_data
SET format = 'Paperback' 
WHERE book = 'The Green Book';

UPDATE goodreads_data
SET format = 'Hardcover' 
WHERE book = 'Officer Buckle and Gloria';

UPDATE goodreads_data
SET format = 'kindle Edition' 
WHERE book = 'The Pairing';

UPDATE goodreads_data
SET format = 'pages' 
WHERE format IS NULL;
```

## FIXING DATA TYPE

After the cleaning was completed and cross-checked, the columns were  assigned their correct DATA TYPE 

``` sql
-- Correct the datatype of each column
ALTER TABLE goodreads_data
MODIFY COLUMN Book TEXT,
MODIFY COLUMN Series VARCHAR(100),
MODIFY COLUMN Release_number INT,
MODIFY COLUMN Author VARCHAR(50),
MODIFY COLUMN Description TEXT,
MODIFY COLUMN Num_Pages INT,
MODIFY COLUMN Format VARCHAR(50),
MODIFY COLUMN Genres TEXT,
MODIFY COLUMN Rating DOUBLE,
MODIFY COLUMN Number_of_voters INT;
```

Using the COMMIT function, the changes made to the table were saved.

``` sql
Commit;
```

NOTE: Even after cleaning the dataset, there were still null values, most of them were necessary for their specific columns as they were not meant to have values while the remaining were missing values. The missing values had little to no effect on the insights that were found, hence they were left null.



