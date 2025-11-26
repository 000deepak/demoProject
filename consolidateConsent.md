## request after raw pref saved

## a. get rule
- if DS success
  - *validateBusniess rule
- else(DS failed)
  - BR_SKIPPED in validation status
- return empty rule

## b. verifyAndConsolidate(rawPref, validationStatus, rule)
- if BR sucess
  - *consolidatePreference 
- else mark CONSOLIDATION_SKIPPED in validationStatus
  
### consolidatePreference
- if mdmId == -99
  - mark CONSOLIDATION_FAILED
  - return
- *performCosolidaton
- save preference
- publish consolidatedPreference
- process audit
- mark mark CONSOLIDATION_SUCESS in validationStatus

## performCosolidaton
```
preferences collection have unique records based on
- mdmId
- templateName
```
- get criteriaAndLevelMap from rule
- fetch old pref from db
- if pref !empty
  - set auditAction update
  - set modifiedTimeStamp
  - setModifiedBySystem
- pref empty
  - create new pref with given mdmId
  - modifiedTimeStamp
  - auditAction create


> helper methods
### validateBusinessRule()
- fetch rule from db using templateType,templateName,ruleType
- rule empty
  - set BR_validaton_sucess
  - create new rule with given templateType,templateName with default construct
- rule !empty
  - set BR_validaton_sucess
  - return the rule

 ## extra info
```
receivedDate = submittedTimeStamp
effectiveDate = startTimeStamp
```
