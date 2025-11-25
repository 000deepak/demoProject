request received
mdmId is -99 and not blanck -> set as ""

for epms record 
sourceSystem should be EPOSTRX
sdrPersonId should be !empty

## 2.cds check
if sourcesSystem is ATHENA && is epms record
get requestJson
get configJson
duplicateCheck

cdsCheck true then continue
else normal flow

check if batch is null
if not set batckId, processStartTime and channel to BatchmetaData

## data validation 
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
## data standardization
### db init
- using typeId find temoplate
- from template get and find from db
- category from categoryId
- BE from BEId
- BU from BUId
- SS from SSId
- locatons from locationIds

### stardardize

