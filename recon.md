## process request
- received mdmRequest(patientGoldenProfile)
- get list of sources(mdmSourceBatchName) from mdmRequest
- for each mdmSourceBatchName fetch all ss
- get sspid from mdmSourceBatchName
- find all rawPreferences from db BY `sspid,sourceSystems && -99`
- for each ss get assigned mdmId from mdmRequest
- *updateRawPreferences
- *consolidatePreference
  
## updateRawPreferences
- for all rawPreferences set the new mdmId and save
- publish rawPreference
- process audit
