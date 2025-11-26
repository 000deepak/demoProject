- request received
- mdmId is -99 and not blanck -> set as ""

## cds check
- if sourcesSystem is ATHENA && isEpmsRecord
    - get requestJson
    - get configJson
    - duplicateCheck
- cdsCheck true then continue
    - else normal flow
- check if batch is null
    - if not set batckId, processStartTime and channel to BatchmetaData

## data validation : DV check
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

### b. Option | isBlank set ""
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
- enum check | invalid string other than enum fail
    - status
    - nature
    - sourceMethod
    - signInitial
if a,b,c success then DV success

> if dv is success then only go fo ds
## data standardization : DS check
### a. db init
- using typeId find temoplate
- from template getIds and find from db and set to variables of class
- category from categoryId
- BE from BEId
- BU from BUId
- SS from SSId
- locatons from locationIds

### b. stardardize
- only for these values from request check
    - BE
    - BU
    - SS
    - locations
    - category/preferenceValue
        - channelName
        - OptionName
        - optionValue 
  - is same as that from db, that we already set in db init
  - if not DS is failed 

## save rawPreference
## a. prepare finalRawPreference
- if DS success
    - do MDM check
    - if mdm success set mdmId for epms or normal and prepare finalRawPreference with fetched mdmId from mdm
    - if failed publish epms error to topic
- if DS failed
    - prepare finalRawPreference with incoming MdmId
      
### b. mdmCheck
- check if mdmCallRequired
- if mdmId IS-BLANK
    - check if cacheIsApplicable
    - if yes using key find mdmId
    - if mdmId is not blank from cache then set in request
    - return false
- else (mdmId not blank)
    - callRequired = false
    - add mdmId to cache
    - add MDM_SKIPPED to validationStatus
- if call required, pass mdmSourceSystemName & SourceSystemPersonId to mdm cross walk api and get mdmId
- if response empty mark MDM_FAILED
- if response !empty
    -  if epms set SSPID and MDMID
    -  for normal set MDMID
- add mdmId to cache

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
