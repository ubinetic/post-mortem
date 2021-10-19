# Ubinetic Oracle Postmortem (incident #1)

**Date:** 2021-10-14

**Authors:** dcale

**Status:** Complete, action items in progress

**Summary:** For 4 hours 20 minutes (10:30 UTC - 14:49 UTC) the ubinetic oracle used in the youves engine blocked because not enough "valid" prices were provided by the data transmitters - even though the data transmitters were providing 2 valid prices.

**Impact:** Mint, Withdraw, Liquidate, Convert and Bailout functionality on the youves engine have been blocked. No funds were at risk at any time during the incident.

**Root Causes:** The ubinetic oracles perform certificate pinning on the provided prices. This means that only data points from specific servers are accepted (removing the MITM attack vector). Binance and Coinbase had previously changed their server certificate; therefore the answers from those servers were no longer "valid" for the oracle. To restore the availability of these prices, some changes to the oracle would have been necessary. At the time, this had been missed by the ubinetic team. Bitfinex was providing still valid answers but went into scheduled maintenance from 9:00 UTC to 18:30 UTC.

**Trigger:** Bitfinex maintenance stopped Bitfinex from providing prices, putting the oracle in the "stale price" scenario, where instead of providing old prices it blocks all price-relevant functionality.

**Resolution:** The Bitfinex prices started coming back after 18:30 UTC, but in the meantime, we re-deployed the oracle with the updated Binance and Coinbase certificates, allowing those prices to be "valid" again. The youves engine was then configured to use the "new" oracle (same code as before with different storage).

**Detection:** A user in discord complained about not being able to convert.

**Action Items:**

| Action Item | Type | Owner | State |
| -------- | -------- | -------- | -------- |
| Deploy oracle with correct certificates | mitigate     | dcale     | DONE |
| Configure engine to point to new oracle in multisig ceremony | mitigate     | youves signers   | DONE |
| Monitor exchange certificate changes  | prevent     | lionel   | DONE |

## Lessons Learned

### What went well

- The oracle behaved correctly, instead of providing stale prices it blocked functionality, keeping everyone's funds safe.
- The certificate pinning is a key security feature of the ubinetic oracle, marking the prices with different certificates as invalid is exactly what it is designed to do.
- The multisig processes and the used code was documented properly allowing for a fast response.
- As soon as the ubinetic team was made aware of the issue, the mitigation was implemented and deployed within less than 3 hours (including the multisig ceremony).
- The youves multisig parties understood the urgency and reacted on this unplanned issue within 30 minutes. (thank you!)

### What went wrong

- The ubinetic team did not monitor the earlier certificate changes on the exchange servers properly.

### Where we got lucky

- Bitfinex planned maintenance during our working hours.

### Conclusion

- The state of the server certificates needs to be monitored by ubinetic on an ongoing basis. 
- In case changes happen, the  amendments to remedy the situation have to be implemented in the oracles promptly.

## Timeline

2021-10-14 (all times UTC)

- 09:00 Bitfinex goes into scheduled maintenance
- 09:30 ubinetic oracle receives the last Bitfinex price
- 10:30 ubinetic oracle blocks with stale price status
- 11:07 user notifies strange behaviour with youves engine
- 13:17 new oracle is deployed on mainnet (KT1HjoLU8KAgQYszocVigHW8TxUb8ZsdGTog)
- 13:53 multisig ceremony is kicked-off
- 14:44 required signatures are collected
- 14:49 youves engines is configured to use new oracle (op2s2sWAdvVG13ff7Y6Gy1WPB1F6PW2pyJZ4TFm34yt3aFmVSA1)
- 14:54 users are informed in the youves discord that the engine resumed
- 16:09 frontend now also points on new oracle 
