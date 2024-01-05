-- using big query to clean data

select *
from `active-cove-387220.NashvilleHousing.Nashville_Property

-- advisable to change date format but our format needed no change

--populate PropertyAddress data
-- this will help us identify those rows which had null as their property address 
--  we will allign them with rows that have same parcel id as such rows have matching property address

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
-- this helped us identify all those property addresses in the table which had a null value, thereby useless.

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

-- BREAKING DOWN OWNER ADDRESS INTO ** HOUSE NO, STREET,CITY**
SELECT
  OwnerAddress,
  SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(2)] AS part3,
  SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(1)] AS PART2,
  SPLIT(REPLACE(OwnerAddress,',','.'),'.') [SAFE_OFFSET(0)] AS PART1
FROM
  `active-cove-387220.NashvilleHousing.Nashville_Property`

-- Breaking down Property Address into address ans city
-- **city**

UPDATE `active-cove-387220.NashvilleHousing.Nashville_Property`
SET PropertySplitCity = SUBSTR(PropertyAddress,STRPOS(PropertyAddress,',') +1)
WHERE STRPOS(PropertyAddress,',') >0 ;

-**-street address**
UPDATE `active-cove-387220.NashvilleHousing.Nashville_Property`
SET PropertySplitAddress = SUBSTR(PropertyAddress,1, STRPOS(PropertyAddress, ','))
WHERE STRPOS(PropertyAddress,',') >0





