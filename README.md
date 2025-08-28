
# Housing Data Cleaning and Transformation
![](https://github.com/Roma90527/Data-Cleaning-and-Transformation-SQL/blob/8621aeb42f7d9d74bf19548d4fda9eb66a874a8d/housing%20.jpg#L1)

This project demostrates data cleaning techniques using SQL. The dataset contains housing data with issues such as missing values, duplicate values, date formatting issues. 

## Objective 
This project demonstarte practical SQL skills for data cleaning, including data types standarization, duplicate removal, null handling. The purpose is to showcase my ability to prepare the dataset for analytical tasks using efficient SQL queries.This ensures data accuracy, consistency, and readiness for further reporting. 




# Data Cleaning and Transformation Steps 

## 1. Standarization

The sale_date column originally stored both date and time values in a DATETIME format.To improve the consistency and simplify analysis , the time component was removed and the change column data type to DATE 

**Example: Standarize the Date Format** 
```sql
-- Step 1: Update the column to remove time
UPDATE HousingData
SET SaleDate = CAST(SaleDate AS Date)

-- Step 2: Alter the column to Date data type
ALTER TABLE HousingData
ALTER COLUMN SaleDate Date;
```

**Example: Categorical Value Standarization** 

The SoldAsVacant column contained values recorded as a single letter i.e. Y and N . To improve readability and consistency, these are standarized to full values as Yes and No.

```sql
-- Now Change Y to Yes and N to No in the SoldAsvacant Column
--Now update the column using Case Statement
SELECT SoldAsVacant,
   CASE When SoldASVacant = 'Y' THEN 'Yes'
   When SoldASVacant = 'N' THEN 'No'
   ELSE SoldAsVacant
   END
FROM HousingData;
```
## 2. Handling Null Values

The propertyAddress column contained NULL values for some records. Since each PARCELID uniquely identifies a property , we used a JOIN on the dataset to populate missing address by matching rows with the same PARCELID that already had a valid address. 

```sql
-- We have NUll values in property address so based on parcelId we can populate property address , so for missing data population we use join

SELECT tb1.ParcelID, tb1.PropertyAddress,tb2.ParcelID,tb2.PropertyAddress,ISNULL(tb1.PropertyAddress,tb2.PropertyAddress)
FROM HousingData as tb1
JOIN HousingData as tb2
ON tb1.parcelID = tb2.ParcelID
AND tb1.[UniqueID ] <> tb2.[UniqueID ]
Where tb1.PropertyAddress IS NULL

-- To replace all NUll values, you need to update actual table  
UPDATE tb1
SET PropertyAddress = ISNULL(tb1.Propertyaddress,tb2.PropertyAddress)
FROM HousingData as tb1
JOIN HousingData as tb2
ON tb1.parcelID = tb2.ParcelID
AND tb1.[UniqueID ] <> tb2.[UniqueID ]
Where tb1.PropertyAddress IS NULL;
```
## 3. Remove Duplicate Values
The Dataset contained duplicate records based on the same ParcelID, PropertyAddress, SalePrice, SaleDate, and LegalReference. The duplicates records are identified using ROW_NUMBER() function and removed to ensure each property sale record is unique.
```sql
-- Remove Duplicate Values---

WITH remove_duuplicate_cte AS (
   
     SELECT *,
     ROW_NUMBER() OVER(
     PARTITION BY ParcelId,
                  PropertyAddress,
                  SalePrice,
                  SaleDate,
                  LegalReference
                  ORDER BY UniqueID
                  ) row_num
FROM HousingData
)
SELECT *
FROM remove_duuplicate_cte
WHERE row_num >1 ;

-- delete all duplicate values
DELETE
FROM remove_duuplicate_cte
WHERE row_num >1 ;
```

## 4. Normalizing and Parsing Data
**Method 1:** 
The propert_address column stored complete address in a single field, combining street address and city. To improve datastructure and usablity , this filed is parsed into seperate columns: Address and City.

```sql
-- Breaking out Address into Individulas Columns (Address, City, State)

SELECT PropertyAddress
FROM HousingData;
-- Extract the address
SELECT 
SUBSTRING(PropertyAddress,1, CHARINDEX(',',PropertyAddress)-1) AS Street_Address,
-- CHARINDEX(',',PropertyAddress) this actually tell the location where delimeter occur so in this output delimeter also display, so -1 will remove the delimeter ','

LTRIM(SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1,LEN(PropertyAddress))) AS City
FROM HousingData;

ALTER TABLE HousingData
Add Street_Address Nvarchar(255)

UPDATE HousingData
SET Street_Address = SUBSTRING(PropertyAddress,1, CHARINDEX(',',PropertyAddress)-1)

ALTER TABLE HousingData
Add City Nvarchar(255);

UPDATE HousingData
SET  City = LTRIM(SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1,LEN(PropertyAddress)));

```
**Method 2:** 
The Owner_address column stored complete address in a single field, combining street address, city, and state. To improve datastructure and usablity , this filed is parsed into seperate columns: Address, City and State.
```sql
--- Now split owner Address Using ParseName(string,partnumber) 1=last part, function that work with dot only so replace , with dot------------------------------------
SELECT 
PARSENAME(REPLACE(OwnerAddress,',','.'),3),  -- it will return first part
PARSENAME(REPLACE(OwnerAddress,',','.'),2),  -- it will return second part
PARSENAME(REPLACE(OwnerAddress,',','.'),1)  -- it will return third part
FROM HousingData;

ALTER TABLE HousingData
ADD OwnerSplitAddress Nvarchar(255)

UPDATE HousingData
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress,',','.'),3)

ALTER TABLE HousingData
ADD OwnerSplitCity Nvarchar(255)

UPDATE HousingData
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress,',','.'),2)

ALTER TABLE HousingData
ADD OwnerSplitState Nvarchar(255)

UPDATE HousingData
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress,',','.'),1)
```

 # Deliverables
 1. Dataset file
 2. SQL Script
    
       
