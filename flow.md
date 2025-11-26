## 1. request received
- mdmId is -99 and not blanck -> set as ""

## 2. cds check
- if sourcesSystem is ATHENA && isEpmsRecord
    - get requestJson
    - get configJson
    - duplicateCheck
- cdsCheck true then continue
    - else normal flow
- check if batch is null
    - if not set batckId, processStartTime and channel to BatchmetaData

## 3. data validation : DV check
### a. manadatory check | isBlank fail
- BE
- BU
- SS
- epms && SSPID
- typeId
- name
- status
- submittedTimeStamp
- documentTpe
- preferenceValue !empty

### b. Optional | isBlank set ""
- mdmId
- location
- PN
- PG
- nature
- sm
- si
- grantor
- granteee
- startTS
- eTS
- ddTS
- rTS
- provideIn
- refTypeId

### c. static check
- invalid hxadecimal fail
    - typeId
    - reftypeId
- ENUM check | invalid string other than ENUM fail
    - status
    - nature
    - sourceMethod
    - signInitial
> if a,b,c success then DV success
> if dv is success then only go fo ds

## 4. data standardization : DS check
### a. db init
- using typeId fetch template from DB
- from template get below Ids and find from db and set to variables of class
    - category from categoryId
    - BE from BEId
    - BU from BUId
    - SS from SSId
    - locatons from locationIds

### b. stardardize
- validate these values against DB that we already set on variables
    - BE
    - BU
    - SS
    - locations
    - category/preferenceValue
        - channelName
        - OptionName
        - optionValue 
- if they are not same DS is failed 

## 5. save rawPreference
### a. prepare finalRawPreference
- if DS success
    - verifyAndMdmCheck()
    - evaluate isMdmSuccess 
        - check isMdmCheckSuccessForEPMS
        - check isMdmCheckSuccessForNormal
    - isMdmSuccess
        - true 
            - prepare finalRawPreference with fetched mdmId from mdm
       - false
           - return finalRawPreference as null
           - if isEpmsRecordPublish epms error to topic
- if DS failed
    - prepare finalRawPreference with ingeationRequest
    - mark MDMSKipped in validationStatus
      
### b. verifyAndMdmCheck
- check if *checkMdmCallRequired
- do *mdmCheck
- add mdmId to chache

### b.1 checkMdmCallRequired()
- if mdmId IS-BLANK
  - check if cacheIsApplicable
  - if yes using key find mdmId
  - if mdmId is not blank from cache then set in request
  - `return false`(mdm call not required)
- else (mdmId not blank)
  - callRequired = false
  - add mdmId to cache
  - add MDM_SKIPPED to validationStatus
  - `return false`
- mdmId is BLANK && `cache is not applicable` THEN MDM CALLL REQUIRED
  - `return true`

### b.2 mdmCheck()
- if call required, pass `mdmSourceSystemName & SourceSystemPersonId` to mdm cross walk api and get mdmId
- if response empty mark MDM_FAILED
- if response !empty
    -  if epms set SSPID and MDMID
    -  for normal set MDMID

### c. save raw preference
- finalRawPreferenceToBeSaved isNull && empsRecord
    - publish to errorTopic skip ingestion log saving & continue with other records
- finalRawPreferenceToBeSaved isNotNull && finalRawPreferenceToBeSaved.getMmdId isNotNull
    - save to rawPreference 
- save in ingestionLog if
    - finalRawPreferenceToBeSaved == null && not epms
    - finalRawPreferenceToBeSaved notNull && null mdmId
 

### extra info
```
mdm stores mdmId wrt to mdmSourceSystemName(one of the field from SourceSystem POJO class) & SourceSystemPersonId(from ingestion request)

if record is KAFKA_HISTORICAL || KAFKA_HISTORICAL_EPMS
then cacheIsApplicaple

if epmsRecord : key = ss + sdrPersonId
if normal : key = ss + sourceSystemPersonId

- for epms record 
    - sourceSystem should be EPOSTRX
    - sdrPersonId should be !empty
```
