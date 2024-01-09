**-- using big query to clean data**

select *
from `active-cove-387220.NashvilleHousing.Nashville_Property

-- **advisable to change date format but our format needed no change
**
**--populate PropertyAddress data
-- this will help us identify those rows which had null as their property address 
--  we will allign them with rows that have same parcel id as such rows have matching property address**

SELECT
  A.ParcelID,
  A.PropertyAddress,
  B.ParcelID,
  B.PropertyAddress, COALESCE(A.PropertyAddress, B.PropertyAddress)
FROM
  `active-cove-387220.NashvilleHousing.Nashville_Property` A
JOIN
  `active-cove-387220.NashvilleHousing.Nashville_Property` B
ON
  A.ParcelID = B.ParcelID
  AND A.UniqueID_ <> B.UniqueID_
WHERE
  A.PropertyAddress IS null
**-- this helped us identify all those property addresses in the table which had a null value, thereby useless.**

-- In order to replace such null values we created a **temp table** and the addresses that shared the same parcel ID was used 

CREATE OR REPLACE TABLE
  `momoproject-391120.nashville.A` AS
SELECT
  A.*
  EXCEPT(PropertyAddress),
  IFNULL(A.PropertyAddress,B.PropertyAddress) AS PropertyAddress
FROM
  `momoproject-391120.nashville.nashville_property` A
JOIN
  `momoproject-391120.nashville.nashville_property` B
ON
  A.ParcelID = B.ParcelID
  AND A.UniqueID_ != B.UniqueID_
WHERE
  A.PropertyAddress IS NULL



**-- Breaking down Property Address into address ans city**
-- **city**

UPDATE `active-cove-387220.NashvilleHousing.Nashville_Property`
SET PropertySplitCity = SUBSTR(PropertyAddress,STRPOS(PropertyAddress,',') +1)
WHERE STRPOS(PropertyAddress,',') >0 ;

-**-street address**
UPDATE `active-cove-387220.NashvilleHousing.Nashville_Property`
SET PropertySplitAddress = SUBSTR(PropertyAddress,1, STRPOS(PropertyAddress, ','))
WHERE STRPOS(PropertyAddress,',') >0

**Breaking down owner Address into; STREET ADDRESS, CITY, STATE**
SELECT
  OwnerAddress,
  SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(2)] AS part3,
  SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(1)] AS PART2,
  SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(0)] AS PART1
FROM
  `active-cove-387220.NashvilleHousing.Nashville_Property`

**Adding Columns**

Alter Table
`active-cove-387220.NashvilleHousing.Nashville_Property`
ADD COLUMN SplitOwnerAddress  STRING;


Alter Table
`active-cove-387220.NashvilleHousing.Nashville_Property`
ADD COLUMN SplitOwnerCity  STRING;

Alter Table
`active-cove-387220.NashvilleHousing.Nashville_Property`
ADD COLUMN SplitOwnerState  STRING:

**Inputting Values in columns
SplitOwnerAddress,
SplitOwnerCity,
SplitOwnerState**

UPDATE
  `active-cove-387220.NashvilleHousing.Nashville_Property`
SET
  OwnerSplitAddress = SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(0)]
  where OwnerAddress is not null

UPDATE
  `active-cove-387220.NashvilleHousing.Nashville_Property`
SET
  OwnerSplitCity = SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(1)]
  where OwnerAddress is not null

UPDATE
  `active-cove-387220.NashvilleHousing.Nashville_Property`
SET
  OwnerSplitState = SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(2)]
  where OwnerAddress is not null


**Categorizing SoldAsVacant to "yes" & "No"**

SELECT
  DISTINCT(SoldAsVacant),
  COUNT (SoldAsVacant)
FROM
  `active-cove-387220.NashvilleHousing.Nashville_Property`
GROUP BY
  SoldAsVacant
ORDER BY
  SoldAsVacant

**Adjusting Sold as Vacant To "YES, OR "NO"**

SELECT
  SoldAsVacant,
  CASE
    WHEN SoldAsVacant ='Y' THEN 'YES'
    WHEN SoldAsVacant ='N' THEN 'NO'
    WHEN SoldAsVacant = 'No' THEN 'NO'
    WHEN SoldAsVacant = 'Yes' THEN 'YES'
  ELSE
  SoldAsVacant
END
FROM
  `active-cove-387220.NashvilleHousing.Nashville_Property`

**Updating the "sold as vacant" column**
UPDATE
  `active-cove-387220.NashvilleHousing.Nashville_Property`
SET
  SoldAsVacant =
  CASE
    WHEN SoldAsVacant = 'Y' THEN 'YES'
    WHEN SoldAsVacant ='N' THEN 'NO'
    WHEN SoldAsVacant = 'No' THEN 'NO'
    WHEN SoldAsVacant = 'Yes' THEN 'YES'
  ELSE
  SoldAsVacant
END
WHERE
  SoldAsVacant IN('Y',
    'N',
    'No',
    'Yes')


**Query to delete duplicate rows, identified by UniqueID_**

creating temp table to identify rows with duplicate values
  Create TEMP TABLE TempToDelete AS
SELECT
  UniqueID_
FROM (
  SELECT
    UniqueID_,
    ROW_NUMBER() OVER
  (PARTITION BY
    ParcelID,
    PropertyAddress,
    SalePrice,
    SaleDate,
    LegalReference
  ORDER BY
    UniqueID_ ) AS ROW_NUM
FROM
  `active-cove-387220.NashvilleHousing.Nashville_Property`)
  WHERE ROW_NUM > 1;
**-- deleting duplicates from original table**
  DELETE FROM `active-cove-387220.NashvilleHousing.Nashville_Property`
  WHERE UniqueID_ IN (SELECT UniqueID_
  from TempToDelete);

 **--finally deleting the temp table**

 drop table TempToDelete

**IN order to check if the above query went through**

SELECT
  *
FROM (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference ORDER BY UniqueID_ ) AS ROW_NUM
  FROM
    `active-cove-387220.NashvilleHousing.Nashville_Property`)
WHERE
  ROW_NUM >1
**
**Deleting Columns that are not needed****

ALTER TABLE `active-cove-387220.NashvilleHousing.Nashville_Property` 
Drop Column
SaleDate

ALTER TABLE `active-cove-387220.NashvilleHousing.Nashville_Property` 
Drop Column
OwnerAddress

ALTER TABLE `active-cove-387220.NashvilleHousing.Nashville_Property` 
Drop Column
PropertyAddress

ALTER TABLE `active-cove-387220.NashvilleHousing.Nashville_Property` 
Drop Column
TaxDistrict


