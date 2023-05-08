# CleaningSQL
Data Cleaning with SQL Server of Housing Nashville Data


--DATA CLEANING 
SELECT *
FROM [Portfolio Project]..NashvilleHousing

--***STANDARIZE DATE FORMAT***

--Changing the data type
SELECT SaleDate, CAST(SaleDate as date) AS Date
FROM [Portfolio Project]..NashvilleHousing

--Uptdating the table

UPDATE NashvilleHousing
SET SaleDate = CAST(SaleDate as date)

--***POPULATE PROPERTY ADRESS DATA***
--Checking when parcelId is the same, propertyaddress is the same too 

SELECT *
FROM [Portfolio Project]..NashvilleHousing
ORDER BY ParcelID

-- Joining information in the same table 

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM [Portfolio Project]..NashvilleHousing AS a --Only using letters because it is the same table we are joining to itself
JOIN [Portfolio Project]..NashvilleHousing AS b
	ON a.ParcelID = b.ParcelID --We want them to joing if the Parcel Id is equal 
	AND a.[UniqueID ] <> b.[UniqueID ] -- only if the unique Id is different
WHERE a.PropertyAddress is NULL

--Inserting new information in the nulls place 
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress) --Function ISNULL lets you populate null values with the value you choose in the second place
FROM [Portfolio Project]..NashvilleHousing AS a --Only using letters because it is the same table we are joining to itself
JOIN [Portfolio Project]..NashvilleHousing AS b
	ON a.ParcelID = b.ParcelID --We want them to joing if the Parcel Id is equal 
	AND a.[UniqueID ] <> b.[UniqueID ] -- only if the unique Id is different
WHERE a.PropertyAddress is NULL

--*** BREAKING OUT PROPERTY AND OWNER ADDRESS INTO INDIVIDUAL COLUMNS (Address, City, State)***

-- Breaking out Property Address
SELECT 
	SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) AS Address, --CHARINDEX indicates the position of a specific value within a string (-1 was used to not include the comma)
	SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress))  AS City
FROM [Portfolio Project]..NashvilleHousing

--Creating new columns for the separated values
ALTER TABLE NashvilleHousing
ADD PropertySplitAddress nvarchar(255); --Creating the new column to store the address only

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) --Filling the column with address values only

ALTER TABLE NashvilleHousing
ADD PropertySplitCity nvarchar(255); --Creating the new column to store the city information only

UPDATE NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) --Filling the column with city information only

-- Breaking out Owner Address
SELECT
	PARSENAME(REPLACE(OwnerAddress,',','.'),3), --PARSENAME function separates a column delimitated by '.', we replaced the commas first, and then used PARSENAME, the numeric value in the fuctions equals like the part we want to display in the column, it works backwards.
	PARSENAME(REPLACE(OwnerAddress,',','.'),2),
	PARSENAME(REPLACE(OwnerAddress,',','.'),1)
FROM [Portfolio Project]..NashvilleHousing

--Creating columns for the new values from the Owner Address

ALTER TABLE [Portfolio Project]..NashvilleHousing
ADD OwnerSplitAddress nvarchar(255); --Creating the new column to store the city information only

UPDATE [Portfolio Project]..NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress,',','.'),3)

ALTER TABLE [Portfolio Project]..NashvilleHousing
ADD OwnerSplitCity nvarchar(255); --Creating the new column to store the city information only

UPDATE [Portfolio Project]..NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress,',','.'),2)

ALTER TABLE [Portfolio Project]..NashvilleHousing
ADD OwnerSplitState nvarchar(255); --Creating the new column to store the city information only

UPDATE [Portfolio Project]..NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress,',','.'),1)

--*** CHANGE Y AND N TO YES AND NO, IN SoldAsVacant***

--Checking which options have the largest count 
SELECT DISTINCT(SoldAsVacant), COUNT(SoldASVacant)
FROM [Portfolio Project]..NashvilleHousing
GROUP BY SoldAsVacant
ORDER BY 2

--Changing Y and N to Yes and No
SELECT SoldAsVacant,
	CASE 
	WHEN SoldAsVacant = 'Y' THEN 'Yes'
	WHEN SoldAsVacant = 'N' THEN 'No'
	ELSE SoldAsVacant
	END 
FROM [Portfolio Project]..NashvilleHousing

--Uptading the table with the new values
UPDATE [Portfolio Project]..NashvilleHousing
SET SoldAsVacant = 
	CASE 
		WHEN SoldAsVacant = 'Y' THEN 'Yes'
		WHEN SoldAsVacant = 'N' THEN 'No'
		ELSE SoldAsVacant
	END 
