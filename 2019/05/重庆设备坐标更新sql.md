UPDATE BAYONET_INFO a,
temp_kakou b 
SET a.ADDRESS = b.device_name,
a.BRAND = b.changshang,
a.DIRECTION = b.fangxiang,
a.LATITUDE = b.weidu,
a.LONGITUDE = b.jingdu,
a.`NAME` = b.device_name 
WHERE
	( a.NUMBER = b.imsi OR a.NUMBER = CONCAT( b.imsi, '-1' ) ) 
	AND b.device_name IS NOT NULL 
	AND b.device_name <> '' 
	AND b.changshang IS NOT NULL 
	AND b.device_name <> '' 
	AND b.fangxiang IS NOT NULL 
	AND b.device_name <> '' 
	AND b.weidu IS NOT NULL 
	AND b.device_name <> '' 
	AND b.jingdu IS NOT NULL 
	AND b.device_name <> '' 
	AND b.device_name IS NOT NULL 
	AND b.device_name <> ''



SELECT
	a.ADDRESS,
	b.device_name,
	a.BRAND,
	b.changshang,
	a.DIRECTION,
	b.fangxiang,
	a.LATITUDE,
	b.weidu,
	a.LONGITUDE,
	b.jingdu,
	a.`NAME`,
	b.device_name 
FROM
	BAYONET_INFO a,
	temp_kakou b 
WHERE
	( a.NUMBER = b.imsi OR a.NUMBER = CONCAT( b.imsi, '-1' ) ) 
	AND b.device_name IS NOT NULL 
	AND b.device_name <> '' 
	AND b.changshang IS NOT NULL 
	AND b.device_name <> '' 
	AND b.fangxiang IS NOT NULL 
	AND b.device_name <> '' 
	AND b.weidu IS NOT NULL 
	AND b.device_name <> '' 
	AND b.jingdu IS NOT NULL 
	AND b.device_name <> '' 
	AND b.device_name IS NOT NULL 
	AND b.device_name <> ''