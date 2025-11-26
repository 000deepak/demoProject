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

## asd
- loop criteriaAndLevelMap
- get cirteria
- get ingestionRuleLevel
- getLatest rawConsent using criteria from all raw consents that were fetched earlier
- get consent using mdmId and templateName(from rule, rule.getTemplateName)
- if `consent is present pass it to consolidation for updation(UPDATE)`
- `if not create new one and pass to consolidation, will create new record(CREATE)`
- for each ruleLevel perform consolidation, pass latest rawConsent according to criteria, consent, ruleLevel, criteria
- *performConsolidation
- *setLastUpdatedAndAllSourceSystems
- save consent

  ## *performConsolidation(latest rawConsent according to criteria, consent, ruleLevel, criteria)
  - get parameter from ingestionRuleLevel
  - get values
  - findConsentData*
  - if ConsentData present(if consent is not present in DB for a given whil fetching using mdmId and templateName(from rule, rule.getTemplateName))
    - *processExistingConsentData 
  - else *processNewConsentData(new consent record will get created)

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
