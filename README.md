# Data-Exploration-in-SQL
This project showcases SQL data cleaning techniques using the Nashville Housing dataset. It covers converting data types, handling missing values, splitting address fields, standardizing categorical variables, removing duplicates, and dropping unnecessary columns to prepare the dataset for analysis.


select * from nashville


-------------------------------------------------------

--Convert Date Format

select saledate, convert (date,saledate)
from nashville


update nashville
set saledate = convert(date,saledate)

alter table nashville
add SaleDateConverted datetime

update nashville
set SaledateConverted = convert(datetime,saledate)


-----------------------------------------------------

--Populate Property Address Data


select * from Nashville
--where PropertyAddress is null
order by ParcelID


select a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress,b.PropertyAddress)
from Nashville a
JOIN Nashville b
on a.ParcelID = b.ParcelID
and a.UniqueID <> b.UniqueID
where a.PropertyAddress is null

update a
set PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
from Nashville a
JOIN Nashville b
on a.ParcelID = b.ParcelID
and a.UniqueID <> b.UniqueID

---------------------------------------------------------------------

--Breaking out Address into Individual Columns (Address, City, State)

select PropertyAddress
from Nashville
--where PropertyAddress is null
--order by ParcelID

select 
substring(PropertyAddress, 0, CHARINDEX(',',PropertyAddress)) as Address,
substring(PropertyAddress, CHARINDEX(',',PropertyAddress) +1, len(PropertyAddress)) as Address
from Nashville


alter table nashville
add PropertySplitAddress nvarchar(255)

update nashville
set PropertySplitAddress = substring(PropertyAddress, 0, CHARINDEX(',',PropertyAddress))

alter table nashville
add PropertySplitCity nvarchar(255)

update nashville
set PropertySplitCity = substring(PropertyAddress, CHARINDEX(',',PropertyAddress) +1, len(PropertyAddress))


select * from Nashville


select OwnerAddress from Nashville

select 
PARSENAME(replace(OwnerAddress, ',','.'), 3),
PARSENAME(replace(OwnerAddress, ',','.'), 2),
PARSENAME(replace(OwnerAddress, ',','.'), 1)
from Nashville



alter table nashville
add OwnerSplitAddress nvarchar(255)

update nashville
set OwnerSplitAddress = PARSENAME(replace(OwnerAddress, ',','.'), 3)


alter table nashville
add OwnerSplitCity nvarchar(255)

update nashville
set OwnerSplitCity = PARSENAME(replace(OwnerAddress, ',','.'), 2)

alter table nashville
add OwnerSplitState nvarchar(255)

update nashville
set OwnerSplitState = PARSENAME(replace(OwnerAddress, ',','.'), 1)

select * from nashville



---------------------------------------------------------------

--Change Y and N to Yes and No in 'Sold as Vacant' Field


select distinct(SoldasVacant), count(SoldasVacant)
from Nashville
group by SoldasVacant 
order by 2

select soldasvacant
,case when soldasvacant = 'Y' then 'Yes'
	 when soldasvacant = 'N' then 'No'
	 else soldasvacant
	 end
	 from Nashville


update nashville
set soldasvacant = case when soldasvacant = 'Y' then 'Yes'
	 when soldasvacant = 'N' then 'No'
	 else soldasvacant
	 end



---------------------------------------------------

--Remove Duplicates

with RowNumCTE as(
select *,
 row_number() over (
 partition by ParcelID,
			 Propertyaddress,
			 SalePrice,
			 Saledate,
			 LegalReference
			 order by
			 uniqueId
			 ) row_num
from nashville
--order by Parcelid
)
select *
from RowNumCTE
where row_num > 1
--order by PropertyAddress




----------------------------------------------------------------

--Delete Unused Columns

select * from Nashville

alter table nashville
drop column owneraddress, TaxDistrict, PropertyAddress


alter table nashville
drop column saledate
