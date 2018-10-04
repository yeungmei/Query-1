# Query-1
# Debt Collection
# CABLE New Biz Query

# - Heading -
SELECT branch, ra0134.account, confirmdate AS discodate, curbal, (progbal::numeric), 
CASE WHEN (((equipchrg::numeric)) + ((equipcredit::numeric))) > 0::numeric AND equipcount > '0' THEN ( (equipchrg::numeric) + (equipcredit::numeric) ) 
	ELSE 0::numeric END AS Equip_balance,
-- remove serial# when only free rental and uncharge equip
CASE WHEN char_length( unreturnedequip_serial)<6 THEN NULL
		WHEN ( (equipchrg::numeric)+ (equipcredit::numeric) ) < 20 THEN NULL ELSE unreturnedequip_serial END AS EquipSerial, 
CASE WHEN ( (equipchrg::numeric)+ (equipcredit::numeric) ) < 20 THEN '0' ELSE equipcount END AS equipcount,
NULL AS status, 
CASE WHEN woamt = '0' THEN NULL ELSE wodate END AS wodate, 
CASE WHEN woamt = '0' THEN NULL ELSE woamt END AS woamt, 
CASE WHEN easyown = 'N' THEN '' ELSE trim('EOP') END AS EOP,
CASE WHEN cfpp = 'N' THEN '' ELSE trim('CFPP') END AS CFPP, 
CASE WHEN ( ra0134.at = '10' OR ra0134.at = '2' OR ra0134.at = '28' ) AND (((equipchrg::text::money)) + ((equipcredit::text::money))) > 0::money 
	AND ( firstname IS NULL OR firstname LIKE any(array['%BULK%', '%SUITES%', '%UNIT%'])) THEN trim('BULK') END AS bulk,
CASE WHEN disco_reason LIKE any(array[ '%145 %']) THEN trim('Deceased') END AS deceased, 
CASE WHEN ( lpamt = '0' OR lpamt::numeric = 0 ) THEN NULL ELSE lpdate END,
CASE WHEN ( lpamt = '0' OR lpamt::numeric = 0 ) THEN NULL ELSE lpamt::money END AS lpamt, firstname, lastname, 
CASE WHEN char_length( phone)<10 THEN '' ELSE phone END, 
CASE WHEN char_length( alterphone)<10 THEN '' ELSE alterphone END, address, postal, city, prov, 
CASE WHEN disco_reason IS NULL THEN trim('123 Moving to Address With Service Included') ELSE disco_reason END, -- if is blank default reason
CASE WHEN maname = '-' THEN NULL ELSE maname END,
CASE WHEN madelivery = '-' THEN NULL ELSE madelivery END,
CASE WHEN maline1 = '-' THEN NULL ELSE maline1 END,
CASE WHEN maline2 = '-' THEN NULL ELSE maline2 END,
CASE WHEN macity = '-' THEN NULL ELSE macity END,
CASE WHEN maprov = '-' THEN NULL ELSE maprov END,
CASE WHEN mapostal = '-' THEN NULL ELSE mapostal END,
at.class AS AccountType,
CASE WHEN emailtype = 'none' THEN NULL ELSE emailaddress END AS email,
CASE WHEN ( ra0134.at = '11' OR ra0134.at = '14' OR ra0134.at = '15' OR ra0134.at = '16' OR ra0134.at = '17' OR ra0134.at = '18' OR ra0134.at = '19' OR 
	ra0134.at = '20' OR ra0134.at = '21' OR ra0134.at = '22' OR ra0134.at = '23' OR ra0134.at = '24' OR ra0134.at = '25' OR ra0134.at = '26' OR 
	ra0134.at = '27' OR ra0134.at = '31' OR ra0134.at = '33' OR ra0134.at = '36' OR ra0134.at = '51' OR ra0134.at = '52' OR ra0134.at = '53' OR 
	ra0134.at = '55' OR ra0134.at = '56' OR ra0134.at = '57' OR ra0134.at = '58' OR ra0134.at = '59' OR ra0134.at = '60' ) 
		AND ( now()::date - confirmdate ) > 59 THEN trim('CABLE')
		WHEN ( ra0134.at = '11' OR ra0134.at = '14' OR ra0134.at = '15' OR ra0134.at = '16' OR ra0134.at = '17' OR ra0134.at = '18' OR ra0134.at = '19' OR 
	ra0134.at = '20' OR ra0134.at = '21' OR ra0134.at = '22' OR ra0134.at = '23' OR ra0134.at = '24' OR ra0134.at = '25' OR ra0134.at = '26' OR 
	ra0134.at = '27' OR ra0134.at = '31' OR ra0134.at = '33' OR ra0134.at = '36' OR ra0134.at = '51' OR ra0134.at = '52' OR ra0134.at = '53' OR 
	ra0134.at = '55' OR ra0134.at = '56' OR ra0134.at = '57' OR ra0134.at = '58' OR ra0134.at = '59' OR ra0134.at = '60' ) 
		THEN trim('Biz Remove') END AS biz,
ra0134.at AS acct_type, ( now()::date - disco_date ) AS count_discodate,
CASE WHEN ( entrydate - confirmdate ) > 30 AND ( entrydate + 30 > now()::date ) THEN trim('backdate') ELSE NULL END AS count_backdate,
CASE WHEN disco_reason LIKE any(array[ '%137 %']) THEN trim('NONPAY') ELSE trim('VOL') END AS disco,
CASE WHEN ( lastname LIKE any(array[ '%TEST%']) OR firstname LIKE any(array[ '%TEST%'])) THEN trim('REMOVE') END AS remove,
CASE WHEN ( maname LIKE any(array[ '%NATIONAL CLIENT %'])) THEN trim('HOUSE ACCT') END AS houseremove
# - REMOVE - CASE WHEN ( entrydate - confirmdate ) < 31 THEN NULL ELSE ( entrydate + 30 ) END AS backdate_audit,

FROM bi.ra0134

LEFT JOIN cbs.r38 ON ( ra0134.account = r38.account )

LEFT JOIN cbs.at ON ( ra0134.at = at.at )

WHERE branch <> 'STA'  --exclude Shaw Direct
AND confirmdate < now()::date - '39 day'::interval 
AND status = 'Off' - ONLY USE WHEN R38 is not updated
AND curbal::numeric > 20
AND branch <> 'ABC' 
# 2016-05-09 remove until further notice, resume collection on Sept 1st
ORDER BY count_backdate ASC, biz ASC
;
# -- END of FILE --



#  -- Notes -- 
#  Create by Helen Yeung 2014-09-24
#  Input Bi File and output combine with R38 and AT
#  change bi account format first 
#  updated no charged rental equip flagging//2015-01-13 by Helen
#  updated Rental Check Review flagging // 2015-01-20 by Helen
#  updated highlight by model # flag instead of "Y" for Rental Check & No Chg Rental// 2015-03-13 by Helen
#  updated highlight by eqiup count compare with BI report via HDW 10// 2015-03-13 by Helen
#  moved send date to the last columns by HELEN
#  add JEReverse & Uncharged calcuation // 2015-03-17 by Helen
#  add sum hdw balance from HDW before tax // 2015-03-18 by Helen
#  add flagging for business account to Email BUS SPS // 2015-06-03 by Helen
#  remove flag for new price as HDW diff already calculated
#  2015-07-15 remark line 51 -56_Victor 
#  2015-09-16 add net j28 TR22 when net eqp balance equal to zero by Helen
#  2015-10-13 add email fileld by HELEN
#  2015-11-10 update biz remove and count backdate by HELEN
#  2015-11-30 update bulk equip audit, add flag for BULK AT 2, 10, 28 with blink 1st name or BULK where there is equip charge balance
#  2015-12-16 update Line 40 and 31, "CASE WHEN lpdate = '2000-01-01' to '1900-01-01' by Victor
#  2016-01-14 update equipaudit and hdw12 and jeamount to remove when it's zero will be come NULL and only show HDW12 when equip balance  is equip to HDW12 balance by HELEN
#  2016-01-20 remove hdwbalance & HDW12 column as no need for audit by HELEN
#  2016-02-23 remove serial when only free rental and uncharged, update eqiup count only when there is serial by Helen
#  2016-02-29 update residential and business flag by Helen
#  2016-04-26 add column for new price adj audit for model 342,326,366 by Helen
#  2016-05-09 tempory remove "ABC" due to fire incident by Helen
#  2016-07-19 remove price adjust by Helen
#  2016-08-31 update flaging for ST12 by Helen
#  2017-02-21 change T3 h10.lastwodate to changedate by HELEN
#  2017-04-03 add flag for SC request by Barry Nguyen
#  2017-05-19 add flag for equip charged after disco date by HELEN
#  2017-10-16 add flag "TEST BIZ HOUSE ACCT" by remove
#  2017-12-01 modify hdw balance difference by HELEN
#  2018-04-19 remove all extra audit from server, remind BI report audit by HELEN
