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
- typeId
```
## consent consolidation
- loop criteriaAndLevelMap
- get cirteria
- get ingestionRuleLevel
- getLatest rawConsent using criteria from all raw consents that were fetched earlier
- get consent using mdmId and templateName(from rule, rule.getTemplateName)
- if `consent is present pass it to consolidation for updation(UPDATE)` c-case1
- else `if not create new one and pass to consolidation, will create new record(CREATE)` c-case2
- for each ruleLevel perform consolidation, pass latest rawConsent according to criteria, consent, ruleLevel, criteria
- *performConsolidation
- *setLastUpdatedAndAllSourceSystems
- save consent

## *performConsolidation(latest rawConsent according to criteria, consent, ruleLevel, criteria)
- get parameter from ingestionRuleLevel
- get values
- findConsentData*
- if ConsentData !empty(c-case1, consents[] will be !empty)
  - *processExistingConsentData (update existing consents[])
- else ConsentData empty(c-case2, consents[] will be empty)
  - *processNewConsentData(c-case2) (create new consents[])

## processExistingConsentData
- getConsentDateTIme()
- getRawConsentDateTime()
- if rawConsentDateTime > consentDateTIme(raw consent data has latest date than already consolidated consent)
  - updateConsent()

## getConsentDateTIme()
- if criteria effective date return startTimeStamp
- else return submittedTimeStamp

## getRawConsentDateTime()
- if criteria effective date return startTimeStamp
- else return submittedTimeStamp

## processNewConsentData()
- if isMatchingParameter() then
  - buildNewConsents and add to consent to save

## isMatchingParameter()
- there are only 3 parameters while creating rule(BE, BU, SS)
- if parameter is equal to BE, check rawConsent BE is matching value(from rule) then return true
- check same for BU && SS
- else false

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

## rule info
```
each template has single or multiple rules, if yes how consolidation will happen??
what is level?

suppose rule is
"BusinessEntity" Equals to "Centerwell" criteria "Effective Date"

*in code
BusinessEntity - parameter
Equals to - operator
Centerwell - value
Effective Date - criterial
```
