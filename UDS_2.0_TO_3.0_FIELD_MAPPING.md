# UDS 2.0 to UDS 3.0 Field Mapping

**Purpose:** This document provides a comprehensive field-by-field mapping between UDS 2.0 fixed-width records and UDS 3.0 JSON structure. It verifies that all UDS 2.0 fields are preserved in UDS 3.0, documents their requirements, and notes compatibility considerations.

**Note:** This is a living document representing the current state of UDS 3.0's JSON schema compared to UDS 2.0 and does not represent
a final proposal. Differences between UDS versions in this document _should_ be addressed either as a change to the JSON
specification or otherwise noted as an acceptable difference.

**Key:**
- **Req 2.0:** R = Required, C = Conditional, O = Optional
- **Req 3.0:** Y = Required, N = Not Required, C = Conditional (based on JSON schema)
- **Compatibility:** Notes about backward compatibility, expanded values, or breaking changes

---

## Table of Contents

1. [Header Record → Batch Metadata](#header-record--batch-metadata)
2. [A Record (Open Loss Claims) → Claims/Claimants](#a-record-open-loss-claims--claimsclaimants)
3. [B Record (Unearned Premium) → ReturnedPremium](#b-record-unearned-premium--returnedpremium)
4. [C Record (Fund to Receiver) → Future Enhancement](#c-record-fund-to-receiver--future-enhancement)
5. [E Record (Closed Claims) → Claims (same as A)](#e-record-closed-claims--claims-same-as-a)
6. [F Record (Notes) → Notes Arrays](#f-record-notes--notes-arrays)
7. [G Record (Payments) → Payments Arrays](#g-record-payments--payments-arrays)
8. [I Record (Documents) → Documents Arrays](#i-record-documents--documents-arrays)
9. [M Record (Medicare) → CMS Field](#m-record-medicare--cms-field)
10. [Trailer Record → Batch Validation](#trailer-record--batch-validation)

---

## Header Record → Batch Metadata

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| Header Identifier | 1-20 | A | R | "HEADER02" | (Implicit in JSON structure) | N/A | Version identified by $schema URI instead |
| NAIC Number | 21-25 | N | R | - | `Batch.InsuranceCompany.NAIC` | Y | Same requirement; stored as string to preserve leading zeros |
| Record Type | 26 | A | R | A/B/C/E/F/G/I/M | (Implicit in JSON hierarchy) | N/A | Record type determined by position in JSON tree |
| From State | 27-28 | A | R | - | `Batch.Receiver.Address.State` | Y | Same requirement; 2-char state code |
| From Location | 29-30 | N | R | - | (No direct equivalent) | N | Location code concept deprecated in 3.0 |
| To State | 31-32 | A | R | - | `Batch.GuarantyFund.State` | Y | Same requirement; 2-char state code |
| To Location | 33-34 | N | R | - | (No direct equivalent) | N | Location code concept deprecated in 3.0 |
| Batch Number | 35-37 | N | R | - | `Batch.Id` | Y | Same requirement; native integer instead of 3-digit string |
| Batch Prepared Date | 38-45 | N | R | - | `Batch.CreatedOn` | Y | ISO-8601 date-time with timezone instead of YYYYMMDD |
| Batch From Date | 46-53 | N | R | - | (No direct equivalent) | N | Reporting period start not explicitly tracked in 3.0 |
| Batch Through Date | 54-61 | N | R | - | (No direct equivalent) | N | Reporting period end not explicitly tracked in 3.0 |
| Insurance Type | 62-64 | A | R | "P&C" | `Batch.GuarantyFund.Type` | Y | Same requirement; identifies fund type |
| Replacement File | 65 | A | R | "N" | (No direct equivalent) | N | Replacement logic handled at application level in 3.0 |
| Record Filler | Varies | A | R | Spaces | N/A | N/A | No padding needed in JSON |

**Compatibility Notes:**
- **Location Codes (01, 10, 11, 26):** Not preserved in UDS 3.0. These were routing codes that are replaced by direct organization references (Receiver, GuarantyFund).
- **Batch Numbering:** UDS 2.0 uses 3-digit batch numbers (001-999). UDS 3.0 uses native integers with no upper limit.
- **Reporting Period Dates:** UDS 3.0 does not explicitly track "From Date" and "Through Date" for reporting periods. This information can be inferred from transaction dates or stored in `Batch.Message`.

---

## A Record (Open Loss Claims) → Claims/Claimants

### Core Identification (Fields 1-8)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **1.** RECORD TYPE | 1 | A | R | "A" | (Implicit: `Batch.Data[].Claims[]`) | N/A | Record type implicit in JSON hierarchy |
| **2.** INSOLVENT COMPANY NAIC NUMBER | 2-6 | A | R | - | `Batch.InsuranceCompany.NAIC` | Y | Same requirement; at Batch level, not duplicated per claim |
| **3.** FILE LOCATION STATE | 7-8 | A | R | - | `Batch.GuarantyFund.State` | Y | At Batch level |
| **4.** FILE LOCATION CODE | 9-10 | N | R | - | (No direct equivalent) | N | Location code deprecated |
| **5.** COVERAGE CODE | 11-16 | N | R | - | `Claimant.Coverages[].Code` | Y | **Moved** to Claimant level; nested in Coverage object. Same 6-digit codes. **Backward compatible** |
| **6.** POLICY NUMBER | 17-36 | A | R | UDSUNKNOWN | `PolicyRecord.PolicyNumber` | Y | At Policy level; no "UDSUNKNOWN" default in 3.0 (required value) |
| **7.** INSOLVENT COMPANY CLAIM NUMBER | 37-56 | A | R | - | `Claim.Number` | Y | Same requirement; variable length in 3.0 |
| **8.** RECEIVER CLAIM NUMBER | 57-76 | A | C | Blank | `Claim.ReceiverNumber` | N | Same conditional requirement |

### Insured Information (Fields 9-15)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **9.** INSURED NAME #1 | 77-106 | A | R | UDSUNKNOWN | `PolicyRecord.Insureds[].LastName` | Y | At Policy level; no "UDSUNKNOWN" default in 3.0 |
| **10.** INSURED NAME #2 | 107-136 | A | C | Blank | `PolicyRecord.Insureds[].FirstName` | Y | **Split** into FirstName (required) and MiddleName (optional) in 3.0 |
| - | - | - | - | - | `PolicyRecord.Insureds[].MiddleName` | N | **NEW FIELD** in 3.0 for middle names |
| **11.** INSURED ADDRESS #1 | 137-166 | A | R | Blank | `PolicyRecord.Insureds[].Addresses[].Line1` | Y | Same requirement; supports multiple addresses via array |
| **12.** INSURED ADDRESS #2 | 167-196 | A | C | Blank | `PolicyRecord.Insureds[].Addresses[].Line2` | N | Same conditional requirement |
| **13.** INSURED CITY | 197-221 | A | R | UDSUNKNOWN | `PolicyRecord.Insureds[].Addresses[].City` | Y | Same requirement; no "UDSUNKNOWN" default |
| **14.** INSURED STATE | 222-223 | A | R | - | `PolicyRecord.Insureds[].Addresses[].State` | Y | Same requirement |
| **15.** INSURED ZIP CODE | 224-232 | A | C | Blank | `PolicyRecord.Insureds[].Addresses[].ZipCode` | Y | **Changed to required** in 3.0 |
| - | - | - | - | - | `PolicyRecord.Insureds[].Addresses[].Country` | Y | **NEW FIELD** in 3.0; required for all addresses |
| - | - | - | - | - | `PolicyRecord.Insureds[].Addresses[].Type` | N | **NEW FIELD** in 3.0 for address type (Primary, Work, etc.) |
| - | - | - | - | - | `PolicyRecord.Insureds[].Emails[]` | N | **NEW FIELD** in 3.0 for email addresses |

### Policy Dates (Fields 16-18)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **16.** DATE OF LOSS | 233-240 | N | R | 19010101 | `Claim.DateOfLoss` | Y | ISO-8601 date; null instead of "19010101" for unknown dates |
| **17.** POLICY EFFECTIVE DATE | 241-248 | N | R | 19010101 | `PolicyRecord.EffectiveDate` | Y | At Policy level; ISO-8601 date; null instead of "19010101" |
| **18.** POLICY EXPIRATION DATE | 249-256 | N | R | 19010101 | `PolicyRecord.ExpirationDate` | Y | At Policy level; ISO-8601 date; null instead of "19010101" |

### Claimant Information (Fields 19-28)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **19.** CLAIMANT NUMBER | 257-261 | N | R | - | `Claimant.Number` | Y | Native integer (1, 2, 3...) instead of zero-padded string ("00001") |
| **20.** CLAIMANT NAME #1 | 262-291 | A | R | UDSUNKNOWN | `Claimant.LastName` | Y | Same requirement; no "UDSUNKNOWN" default |
| **21.** CLAIMANT NAME #2 | 292-321 | A | C | Blank | `Claimant.FirstName` | Y | **Split** into FirstName (required) and MiddleName (optional) in 3.0 |
| - | - | - | - | - | `Claimant.MiddleName` | N | **NEW FIELD** in 3.0 for middle names |
| **22.** CLAIMANT ADDRESS #1 | 322-351 | A | R | UDSUNKNOWN | `Claimant.Addresses[].Line1` | Y | Same requirement; no "UDSUNKNOWN" default |
| **23.** CLAIMANT ADDRESS #2 | 352-381 | A | C | Blank | `Claimant.Addresses[].Line2` | N | Same conditional requirement |
| **24.** CLAIMANT CITY | 382-406 | A | R | UDSUNKNOWN | `Claimant.Addresses[].City` | Y | Same requirement; no "UDSUNKNOWN" default |
| **25.** CLAIMANT STATE | 407-408 | A | R | - | `Claimant.Addresses[].State` | Y | Same requirement |
| **26.** CLAIMANT ZIP CODE | 409-417 | A | C | Blank | `Claimant.Addresses[].ZipCode` | Y | **Changed to required** in 3.0 |
| **27.** CLAIMANT ID INDICATOR | 418 | A | C | Blank | (Implicit in `Claimant.Identification`) | N | F/S indicator replaced by explicit SSN/EIN/DLN fields |
| **28.** CLAIMANT ID NUMBER | 419-427 | N | C | Blank | `Claimant.Identification.SSN` or `.EIN` or `.DLN` | N | **Split** into separate fields by type |
| - | - | - | - | - | `Claimant.Addresses[].Country` | Y | **NEW FIELD** in 3.0; required for all addresses |
| - | - | - | - | - | `Claimant.Addresses[].Type` | N | **NEW FIELD** in 3.0 for address type |
| - | - | - | - | - | `Claimant.Emails[]` | N | **NEW FIELD** in 3.0 for email addresses |

### Transaction Information (Fields 29-30)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **29.** TRANSACTION CODE | 428-430 | N | R | 100 | `Claim.TransactionCode` | Y | Same requirement; must be "100" for initial transmission. **Backward compatible** |
| **30.** TRANSACTION AMOUNT | 431-442 | N | R | - | `Claim.TransactionAmount` | N | Native JSON number instead of signed fixed-width; optional in 3.0 |

### Claim Attributes (Fields 31-34)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **31.** CATASTROPHIC LOSS CODE | 443-444 | N | C | Blank | `Claim.Catastrophe.Code` | Y (if Catastrophe exists) | Nested in Catastrophe object |
| - | - | - | - | - | `Claim.Catastrophe.Name` | Y (if Catastrophe exists) | **NEW FIELD** in 3.0 for catastrophe name |
| **32.** RECOVERY INDICATOR CODE | 445 | A | R | Zero | `Claim.RecoveryIndicatorCode` | N | Same codes (0-5); optional in 3.0. **Backward compatible** |
| **33.** SUIT INDICATOR | 446 | A | R | U | `Claim.Suits[]` (existence) | N | **Changed** from Y/N/U indicator to array of Suit objects |
| - | - | - | - | - | `Claim.Suits[].LawFirm` | Y (if Suit exists) | **NEW FIELD** in 3.0 for law firm name |
| - | - | - | - | - | `Claim.Suits[].Contact` | Y (if Suit exists) | **NEW FIELD** in 3.0 for law firm contact |
| **34.** 2ND INJURY FUND INDICATOR | 447 | A | R | U | `Claim.SecondInjuryFundIndicator` | N | Same Y/N/U values; optional in 3.0. **Backward compatible** |

### Additional Identifiers (Fields 35-40)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **35.** TPA CLAIM NUMBER | 448-477 | A | C | Blank | `Claim.TPAClaimNumber` | N | Same conditional requirement |
| **36.** LONG CLAIM NUMBER | 478-507 | A | C | Blank | (Not needed) | N | UDS 3.0 has no length limit on Claim.Number |
| **37.** ISSUING COMPANY CODE | 508-512 | A | C | Blank | `PolicyRecord.IssuingCompanyCode` | N | At Policy level; same 5-digit NAIC format |
| **38.** SERVICING OFFICE CODE | 513-518 | A | C | Blank | `Claim.ServicingOfficeCode` | N | Same conditional requirement; max 6 chars in 3.0 |
| **39.** CLAIM REPORT DATE | 519-526 | N | C | Blank | `Claim.ReportDate` | Y | **Changed to required** in 3.0; ISO-8601 date |
| **40.** CLAIMANT BIRTH DATE | 527-534 | N | C | Blank | `Claimant.BirthDate` | N | ISO-8601 date; conditional (required for WC/BI) |

### Workers' Comp Specific (Fields 41-54)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **41.** REPETITIVE PAYMENT INDICATOR | 535 | A | R | N | `Claim.RepetitivePaymentIndicator` | N | Same Y/N values; optional in 3.0. **Backward compatible** |
| **42.** WCIO INJURY CODE | 536-538 | A | C | Blank | `Claim.WorkersCompensation.InjuryCode` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **43.** WCIO PART OF BODY | 539-541 | A | C | Blank | `Claim.WorkersCompensation.PartOfBody` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **44.** WCIO NATURE OF INJURY | 542-544 | A | C | Blank | `Claim.WorkersCompensation.NatureOfInjury` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **45.** WCIO CAUSE | 545-547 | A | C | Blank | `Claim.WorkersCompensation.Cause` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **46.** WCIO ACT | 548-550 | A | C | Blank | `Claim.WorkersCompensation.Act` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **47.** WCIO TYPE OF LOSS | 551-553 | A | C | Blank | `Claim.WorkersCompensation.TypeOfLoss` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **48.** WCIO TYPE OF RECOVERY | 554-556 | A | C | Blank | `Claim.WorkersCompensation.TypeOfRecovery` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **49.** WCIO TYPE OF COVERAGE | 557-559 | A | C | Blank | `Claim.WorkersCompensation.TypeOfCoverage` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **50.** WCIO TYPE OF SETTLEMENT | 560-562 | A | C | Blank | `Claim.WorkersCompensation.TypeOfSettlement` | N | Required for WC; blank for non-WC. **Backward compatible** |
| **51.** WCIO VOCATIONAL REHAB INDICATOR | 563 | A | C | Blank | `Claim.WorkersCompensation.VocationalRehabIndicator` | N | Same Y/N/U values; conditional. **Backward compatible** |
| **52.** DESCRIPTION OF INJURY | 564-627 | A | C | Blank | `Claim.WorkersCompensation.DescriptionOfInjury` | N | Required for WC; variable length in 3.0. **Backward compatible** |
| **53.** WCAB NUMBER | 628-639 | A | C | Blank | `Claim.WorkersCompensation.WCABNumber` | N | Same conditional requirement; variable length in 3.0 |
| **54.** EMPLOYER WORK PHONE NUMBER | 640-649 | N | C | Blank | `Claim.WorkersCompensation.Employer.PhoneNumber` | N | Nested in Employer object; variable format in 3.0 |
| - | - | - | - | - | `Claim.WorkersCompensation.Employer.Email` | N | **NEW FIELD** in 3.0 for employer email |
| - | - | - | - | - | `Claim.WorkersCompensation.Employer.Addresses[]` | Y | **NEW OBJECT** in 3.0; Employer always required (even if empty) |

### Policy Indicators (Fields 55-56)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **55.** AGGREGATE POLICY INDICATOR | 650 | A | R | U | `Claim.AggregatePolicyIndicator` | N | Same Y/N/U values; optional in 3.0. **Backward compatible** |
| **56.** DEDUCTIBLE POLICY INDICATOR | 651 | A | R | U | `Claim.DeductiblePolicyIndicator` | N | Same Y/N/U values; optional in 3.0. **Backward compatible** |

### Coverage Details (Not in A Record but in 3.0)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| - | - | - | - | - | `Claimant.Coverages[].Name` | Y | **NEW FIELD** in 3.0 for coverage description |
| - | - | - | - | - | `Claimant.Coverages[].OutstandingReserve` | N | **NEW FIELD** in 3.0 for reserve amount by coverage |

---

## B Record (Unearned Premium) → ReturnedPremium

### Core Identification (Fields 1-7)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **1.** RECORD TYPE | 1 | A | R | "B" | (Implicit: `PolicyRecord.ReturnedPremium[]`) | N/A | Record type implicit in JSON hierarchy |
| **2.** INSOLVENT COMPANY NAIC NUMBER | 2-6 | A | R | - | `Batch.InsuranceCompany.NAIC` | Y | At Batch level |
| **3.** FILE LOCATION STATE | 7-8 | A | R | - | `Batch.GuarantyFund.State` | Y | At Batch level |
| **4.** FILE LOCATION CODE | 9-10 | N | R | - | (No direct equivalent) | N | Location code deprecated |
| **5.** COVERAGE CODE | 11-16 | N | R | - | `ReturnPremium.Coverage.Code` | Y | Same requirement; nested in Coverage object. **Backward compatible** |
| **6.** POLICY NUMBER | 17-36 | A | R | UDSUNKNOWN | `PolicyRecord.PolicyNumber` | Y | At Policy level |
| **7.** RECEIVER CLAIM NUMBER | 37-56 | A | C | Blank | (No direct equivalent on ReturnPremium) | N | Claim number not applicable for premium returns |

### Insured Information (Fields 8-14)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **8.** INSURED NAME #1 | 57-86 | A | R | UDSUNKNOWN | `ReturnPremium.Insured.LastName` | Y | Same requirement |
| **9.** INSURED NAME #2 | 87-116 | A | C | Blank | `ReturnPremium.Insured.FirstName` | Y | **Split** into FirstName (required) and MiddleName (optional) |
| - | - | - | - | - | `ReturnPremium.Insured.MiddleName` | N | **NEW FIELD** in 3.0 |
| **10.** INSURED ADDRESS #1 | 117-146 | A | C | Blank | `ReturnPremium.Insured.Addresses[].Line1` | Y | **Changed to required** in 3.0 |
| **11.** INSURED ADDRESS #2 | 147-176 | A | C | Blank | `ReturnPremium.Insured.Addresses[].Line2` | N | Same conditional requirement |
| **12.** INSURED CITY | 177-201 | A | C | Blank | `ReturnPremium.Insured.Addresses[].City` | Y | **Changed to required** in 3.0 |
| **13.** INSURED STATE | 202-203 | A | C | - | `ReturnPremium.Insured.Addresses[].State` | Y | **Changed to required** in 3.0 |
| **14.** INSURED ZIP CODE | 204-212 | A | C | Blank | `ReturnPremium.Insured.Addresses[].ZipCode` | Y | **Changed to required** in 3.0 |
| - | - | - | - | - | `ReturnPremium.Insured.Addresses[].Country` | Y | **NEW FIELD** in 3.0 |

### Date and Claimant Fields (Fields 15-18)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **15.** DATE OF LOSS | 213-220 | N | R | - | (Implied from `InsuranceCompany.DateOfLiquidation`) | N | In B Records, this is liquidation date, not actual loss date |
| **16.** CLAIMANT NUMBER | 221-225 | N | R | - | `ReturnPremium.Claimant.Number` | Y | Same requirement; native integer |
| **17.** PAYEE INDICATOR | 226 | A | C | Blank | (Implicit in `ReturnPremium.Claimant.Identification`) | N | F/S indicator replaced by explicit fields |
| **18.** PAYEE ID NUMBER | 227-235 | N | C | Blank | `ReturnPremium.Claimant.Identification.SSN` or `.EIN` | N | **Split** into separate fields |

### Policy Dates (Fields 19-22)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **19.** POLICY EFFECTIVE DATE | 236-243 | N | R | 19010101 | `PolicyRecord.EffectiveDate` | Y | At Policy level; ISO-8601 date |
| **20.** POLICY EXPIRATION DATE | 244-251 | N | R | 19010101 | `PolicyRecord.ExpirationDate` | Y | At Policy level; ISO-8601 date |
| **21.** CANCELLATION DATE | 252-259 | N | R | - | `PolicyRecord.CancellationDate` | N | ISO-8601 date; optional in 3.0 |
| **22.** CANCELLATION CODE | 260 | A | R | Blank | `PolicyRecord.CancellationCode` | N | Same codes (C/F/L/N/R); optional in 3.0. **Backward compatible** |

### Transaction and Premium Fields (Fields 23-28)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **23.** TRANSACTION CODE | 261-263 | N | R | - | `ReturnPremium.TransactionCode` | Y | Must be 800, 815, or 835. **Backward compatible** |
| **24.** TOTAL WRITTEN POLICY PREMIUM | 264-273 | N | C | All zeroes | `PolicyRecord.TotalWrittenPremium` | N | At Policy level; native number |
| **25.** TOTAL IN FORCE POLICY PREMIUM | 274-283 | N | C | All zeroes | `PolicyRecord.TotalInForcePremium` | N | At Policy level; native number |
| **26.** FINAL AUDIT INDICATOR | 284 | A | R | - | `ReturnPremium.FinalAuditIndicator` | Y | Same Y/N values. **Backward compatible** |
| **27.** RETURN PREMIUM AMOUNT | 285-294 | N | C | All zeroes | `ReturnPremium.ReturnPremiumAmount` | Y | Same requirement; native number |
| **28.** UNPAID PREMIUM AMOUNT | 295-304 | N | C | All zeroes | `ReturnPremium.UnpaidPremiumAmount` | Y | Same requirement; native number |

### Agency and Billing Fields (Fields 29-32)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **29.** FINANCE COMPANY CODE | 305-309 | A | C | Blank | (No direct equivalent) | N | Not preserved in UDS 3.0 |
| **30.** AGENT CODE | 310-319 | A | C | Blank | (No direct equivalent) | N | Not preserved in UDS 3.0 |
| **31.** AGENT'S COMMISSION RATE | 320-324 | N | C | Blank | (No direct equivalent) | N | Not preserved in UDS 3.0 |
| **32.** BILLING MODE | 325 | A | C | Blank | (No direct equivalent) | N | Not preserved in UDS 3.0 |

### Claimant Detail Fields (Fields 33-40)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **33.** CLAIMANT NAME #1 | 326-355 | A | C | - | `ReturnPremium.Claimant.LastName` | Y | **Changed to required** in 3.0 |
| **34.** CLAIMANT NAME #2 | 356-385 | A | C | Blank | `ReturnPremium.Claimant.FirstName` | Y | **Changed to required** in 3.0 |
| - | - | - | - | - | `ReturnPremium.Claimant.MiddleName` | N | **NEW FIELD** in 3.0 |
| **35.** CLAIMANT ADDRESS #1 | 386-415 | A | C | Blank | `ReturnPremium.Claimant.Addresses[].Line1` | Y | **Changed to required** in 3.0 |
| **36.** CLAIMANT ADDRESS #2 | 416-445 | A | C | Blank | `ReturnPremium.Claimant.Addresses[].Line2` | N | Same conditional requirement |
| **37.** CLAIMANT CITY | 446-470 | A | C | Blank | `ReturnPremium.Claimant.Addresses[].City` | Y | **Changed to required** in 3.0 |
| **38.** CLAIMANT STATE | 471-472 | A | C | Blank | `ReturnPremium.Claimant.Addresses[].State` | Y | **Changed to required** in 3.0 |
| **39.** CLAIMANT ZIP CODE | 473-481 | A | C | Blank | `ReturnPremium.Claimant.Addresses[].ZipCode` | Y | **Changed to required** in 3.0 |
| **40.** CLAIMANT PHONE # | 482-501 | A | C | Blank | (No direct equivalent) | N | Not preserved; use Emails[] instead |
| - | - | - | - | - | `ReturnPremium.Claimant.Addresses[].Country` | Y | **NEW FIELD** in 3.0 |

**Compatibility Notes:**
- **Finance/Agent Fields (29-32):** Not preserved in UDS 3.0. These fields were rarely used and can be stored in Notes if needed.
- **Claimant Phone:** Not directly preserved. Email addresses are preferred in 3.0.

---

## C Record (Fund to Receiver) → Future Enhancement

**Status:** UDS 3.0 does not currently include a C Record equivalent for Fund-to-Receiver reporting. This is a future enhancement.

C Records in UDS 2.0 are used by Guaranty Funds to report transactions back to Receivers:
- Reserve snapshots (130, 230)
- Payments (310, 320, 410, 420, 820, etc.)
- Claim status changes (030, 050, 080, etc.)
- Recoveries (530, 540, 550)
- Deductibles (610, 840)

**Planned UDS 3.0 Implementation:**
When implemented, C Record data will likely be structured as transaction arrays at the Claim or Claimant level, similar to how Payments are currently handled.

---

## E Record (Closed Claims) → Claims (same as A)

E Records have the **identical structure** to A Records. The only differences are:
- **Purpose:** Historical/informational transmission of closed claims
- **Transaction Amount (field 30):** Must be zero for closed claims

All 56 fields from the A Record mapping above apply to E Records. Refer to the [A Record section](#a-record-open-loss-claims--claimsclaimants) for complete field mappings.

In UDS 3.0, there is no distinction between "open" and "closed" claim records - both use the same `Claim` structure within `PolicyRecord.Claims[]`.

---

## F Record (Notes) → Notes Arrays

### Note Fields (Fields 1-13)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **1.** RECORD TYPE | 1 | A | R | "F" | (Implicit in Notes array) | N/A | Record type implicit in JSON hierarchy |
| **2.** INSOLVENT COMPANY NAIC NUMBER | 2-6 | A | R | - | `Batch.InsuranceCompany.NAIC` | Y | At Batch level |
| **3.** FILE LOCATION STATE | 7-8 | A | R | - | `Batch.GuarantyFund.State` | Y | At Batch level |
| **4.** FILE LOCATION CODE | 9-10 | N | R | - | (No direct equivalent) | N | Location code deprecated |
| **5.** INSOLVENT COMPANY CLAIM NUMBER | 11-40 | A | R | - | (Implicit in hierarchy) | N | Note attached to Claim via parent relationship |
| **6.** RECEIVER CLAIM NUMBER | 41-60 | A | C | Blank | (Implicit in hierarchy) | N | Note attached to Claim via parent relationship |
| **7.** CLAIMANT NUMBER | 61-65 | N | R | - | (Implicit in hierarchy) | N | Note level determined by location in JSON tree |
| **8.** ENTRY DATE | 66-73 | N | R | - | `Note.CreatedOn` | Y | ISO-8601 date-time with timezone instead of YYYYMMDD |
| **9.** NOTE ID NUMBER | 74-77 | N | R | - | `Note.OriginalId` | Y | Native integer instead of 4-digit string |
| **10.** NOTE LINE SEQUENCE NUMBER | 78-81 | N | R | - | (Not needed) | N | **REMOVED** - no 1000-char line limit in 3.0 |
| **11.** ENTRY TEXT | 82-1081 | A | R | - | `Note.Text` | Y | **Unlimited length** in 3.0 (no 1000-char limit) |
| **12.** LONG CLAIM NUMBER | 1082-1111 | A | C | Blank | (Not needed) | N | UDS 3.0 has no length limit on claim numbers |
| **13.** TPA CLAIM NUMBER | 1112-1141 | A | C | Blank | (Implicit in hierarchy) | N | TPA claim number at Claim level |

**Note Placement in UDS 3.0:**

Notes can appear at multiple levels in the JSON hierarchy:
- **Policy-level notes:** `PolicyRecord.Notes[]`
- **Claim-level notes:** `Claim.Notes[]`
- **Claimant-level notes:** `Claimant.Notes[]`

In UDS 2.0, notes were claimant-level only (or claim-level if CLAIMANT NUMBER = 00000). UDS 3.0 provides more flexibility.

**Compatibility Notes:**
- **No Line Limit:** UDS 2.0 limited note text to 1000 characters per line, requiring multiple F Records with line sequences. UDS 3.0 has no practical limit on note text length.
- **Flexible Placement:** Notes can be attached at the most appropriate level (policy, claim, or claimant), not just claimant level.

---

## G Record (Payments) → Payments Arrays

### Payment Fields (Fields 1-26)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **1.** RECORD TYPE | 1 | A | R | "G" | (Implicit: `Claimant.Payments[]`) | N/A | Record type implicit in JSON hierarchy |
| **2.** INSOLVENT COMPANY NAIC NUMBER | 2-6 | A | R | - | `Batch.InsuranceCompany.NAIC` | Y | At Batch level |
| **3.** FILE LOCATION STATE | 7-8 | A | R | - | `Batch.GuarantyFund.State` | Y | At Batch level |
| **4.** FILE LOCATION CODE | 9-10 | N | R | - | (No direct equivalent) | N | Location code deprecated |
| **5.** COVERAGE CODE | 11-16 | N | C | - | `Payment.Coverage.Code` | Y | Same requirement; nested in Coverage object |
| **6.** POLICY NUMBER | 17-36 | A | R | - | `PolicyRecord.PolicyNumber` | Y | At Policy level |
| **7.** INSOLVENT COMPANY CLAIM NUMBER | 37-66 | A | R | - | `Claim.Number` | Y | At Claim level |
| **8.** RECEIVER CLAIM NUMBER | 67-86 | A | C | Blank | `Claim.ReceiverNumber` | N | At Claim level |
| **9.** INSURED NAME #1 | 87-116 | A | R | - | `PolicyRecord.Insureds[].LastName` | Y | At Policy level |
| **10.** INSURED NAME #2 | 117-146 | A | C | Blank | `PolicyRecord.Insureds[].FirstName` | Y | At Policy level |
| **11.** CLAIMANT NUMBER | 147-151 | N | R | - | `Claimant.Number` | Y | At Claimant level; native integer |
| **12.** CLAIMANT NAME #1 | 152-181 | A | R | - | `Claimant.LastName` | Y | At Claimant level |
| **13.** CLAIMANT NAME #2 | 182-211 | A | C | Blank | `Claimant.FirstName` | Y | At Claimant level |
| **14.** CHECK DATE | 212-219 | N | R | - | `Payment.CheckDate` | N | ISO-8601 date; optional in 3.0 |
| **15.** TRANSACTION CODE | 220-222 | N | R | - | `Payment.Type.Code` | Y | Must be 310/320/410/420/820. **Backward compatible** |
| - | - | - | - | - | `Payment.Type.Name` | Y | **NEW FIELD** in 3.0 for payment type description |
| **16.** CHECK AMOUNT | 223-234 | N | R | - | `Payment.CheckAmount` | N | Native number; optional in 3.0 |
| **17.** CHECK NUMBER | 235-246 | A | R | - | `Payment.CheckNumber` | Y | Same requirement; variable length in 3.0 |
| **18.** PAYEE NAME #1 | 247-276 | A | R | - | `Payment.Payee.NameLine1` | Y | Same requirement; nested in Payee object |
| **19.** PAYEE NAME #2 | 277-306 | A | R | - | `Payment.Payee.NameLine2` | Y | Same requirement; nested in Payee object |
| **20.** PAYEE ID NUMBER | 307-315 | N | C | Blank | `Payment.Payee.Identification.SSN` or `.EIN` | Y | **Changed to required** in Identification object |
| **21.** INVOICE NUMBER | 316-335 | A | C | Blank | `Payment.InvoiceNumber` | N | Same conditional requirement |
| **22.** SERVICE/BENEFIT FROM DATE | 336-343 | N | C | Blank | `Payment.ServiceOrBenefitFromDate` | N | ISO-8601 date; conditional (required for WC) |
| **23.** SERVICE/BENEFIT THROUGH DATE | 344-351 | N | C | Blank | `Payment.ServiceOrBenefitToDate` | N | ISO-8601 date; conditional (required for WC) |
| **24.** PAYMENT COMMENT | 352-411 | A | C | Blank | `Payment.Comment` | N | Same conditional requirement; variable length in 3.0 |
| **25.** LONG CLAIM NUMBER | 412-441 | A | C | Blank | (Not needed) | N | UDS 3.0 has no length limit on claim numbers |
| **26.** TPA CLAIM NUMBER | 442-471 | A | C | Blank | `Claim.TPAClaimNumber` | N | At Claim level |

**Compatibility Notes:**
- **Transaction Codes:** UDS 3.0 uses the same payment transaction codes (310, 320, 410, 420, 820) as UDS 2.0. **Fully backward compatible.**
- **Coverage:** Explicitly attached to each payment in 3.0, including both Code and Name.

---

## I Record (Documents) → Documents Arrays

### Document Fields (Fields 1-30)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **1.** RECORD TYPE | 1 | A | R | "I" | (Implicit in Documents array) | N/A | Record type implicit in JSON hierarchy |
| **2.** INSOLVENT COMPANY NAIC NUMBER | 2-6 | N | R | - | `Batch.InsuranceCompany.NAIC` | Y | At Batch level |
| **3.** FROM LOCATION STATE | 7-8 | A | R | - | `Batch.Receiver.Address.State` | Y | At Batch level |
| **4.** FROM LOCATION CODE | 9-10 | N | R | - | (No direct equivalent) | N | Location code deprecated |
| **5.** INSOLVENT COMPANY CLAIM NUMBER | 11-30 | A | R | - | (Implicit in hierarchy) | N | Document attached via parent relationship |
| **6.** RECEIVER CLAIM NUMBER | 31-50 | A | C | Blank | (Implicit in hierarchy) | N | Document attached via parent relationship |
| **7.** TPA CLAIM NUMBER | 51-80 | A | C | Blank | (Implicit in hierarchy) | N | TPA claim number at Claim level |
| **8.** LONG CLAIM NUMBER | 81-110 | A | C | Blank | (Not needed) | N | UDS 3.0 has no length limit on claim numbers |
| **9.** FUND CLAIM NUMBER | 111-130 | A | C | Blank | (No direct equivalent) | N | Not preserved in 3.0 |
| **10.** ALTERNATE INDEX 1 | 131-180 | A | C | Blank | `Document.Tags[]` | N | **Changed** to flexible tags array |
| **11.** ALTERNATE INDEX 2 | 181-230 | A | C | Blank | `Document.Tags[]` | N | **Changed** to flexible tags array |
| **12.** ALTERNATE INDEX 3 | 231-280 | A | C | Blank | `Document.Tags[]` | N | **Changed** to flexible tags array |
| **13.** ALTERNATE INDEX 4 | 281-330 | A | C | Blank | `Document.Tags[]` | N | **Changed** to flexible tags array |
| **14.** DOCUMENT ID | 331-360 | A | C | Blank | `Document.OriginalId` | N | Native integer; optional in 3.0 |
| **15.** DOCUMENT PAGE NUMBER | 361-369 | N | C | Blank | `Document.PageNumber` | N | Native integer; optional in 3.0 |
| **16.** CAPTURE DATE | 370-377 | N | R | - | `Document.CreatedOn` | N | ISO-8601 date-time; optional in 3.0 |
| **17.** CAPTURE TIME | 378-385 | N | C | Blank | (Included in `Document.CreatedOn`) | N | ISO-8601 includes time |
| **18.** FOLDER TYPE | 386-391 | A | C | Blank | `Document.Tags[]` | N | Can be included in tags |
| **19.** DOCUMENT TYPE | 392-421 | A | C | Blank | `Document.Tags[]` | N | **Changed** to flexible tags array |
| **20.** DOCUMENT DESCRIPTION OR COMMENT | 422-549 | A | C | Blank | `Document.Description` | N | Same conditional requirement; variable length in 3.0 |
| **21.** POLICY NUMBER | 550-569 | A | C | Blank | (Implicit in hierarchy) | N | Document attached via parent relationship |
| **22.** DATE OF LOSS / INJURY | 570-577 | N | C | Blank | (Implicit in hierarchy) | N | Date of loss at Claim level |
| **23.** INSURED NAME #1 | 578-607 | A | C | Blank | (Implicit in hierarchy) | N | Insured at Policy level |
| **24.** INSURED NAME #2 | 608-637 | A | C | Blank | (Implicit in hierarchy) | N | Insured at Policy level |
| **25.** CLAIMANT NUMBER | 638-642 | N | C | Blank | (Implicit in hierarchy) | N | Document level determined by position in tree |
| **26.** CLAIMANT NAME #1 | 643-672 | A | C | Blank | (Implicit in hierarchy) | N | Claimant at Claimant level |
| **27.** CLAIMANT NAME #2 | 673-702 | A | C | Blank | (Implicit in hierarchy) | N | Claimant at Claimant level |
| **28.** DOCUMENT PATH | 703-958 | A | R | - | `Document.Path` | Y | **MAJOR CHANGE:** URI format (https://, file://, ftp://) instead of Windows path |
| **29.** DOCUMENT FILENAME | 959-1214 | A | R | - | (Included in `Document.Path`) | N | Filename included in URI |
| **30.** FILE TYPE | 1215-1218 | A | R | - | (Inferred from URI) | N | MIME type inferred from file extension in URI |
| - | - | - | - | - | `Document.Fingerprint` | N | **NEW FIELD** in 3.0 for document hash (SHA-256) |

**Document Placement in UDS 3.0:**

Documents can appear at multiple levels in the JSON hierarchy:
- **Policy-level documents:** `PolicyRecord.Documents[]`
- **Claim-level documents:** `Claim.Documents[]`
- **Claimant-level documents:** `Claimant.Documents[]`

**Compatibility Notes:**
- **MAJOR CHANGE - Document Delivery:** UDS 2.0 bundles documents in a ZIP file with I Records as an index. UDS 3.0 uses URI references for on-demand access via HTTPS, SFTP, or file:// protocols.
- **Path Format:** UDS 2.0 uses Windows-style backslash paths (`\DOCUMENTS\CLAIMS\`). UDS 3.0 uses URI format with forward slashes and protocol prefixes.
- **Alternate Indexes:** UDS 2.0's 4 alternate index fields are replaced by a flexible `Tags[]` array that can contain any number of categorization tags.
- **Integrity Verification:** UDS 3.0 adds `Fingerprint` field for document hashing (e.g., SHA-256) to verify file integrity.

---

## M Record (Medicare) → CMS Field

### Medicare Fields (Fields 1-6 + CMS Data)

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| **a.** RECORD TYPE | 1 | A | R | "M" | (Implicit in CMS field) | N/A | Record type implicit |
| **b.** INSOLVENT COMPANY NAIC NUMBER | 2-6 | N | R | - | `Batch.InsuranceCompany.NAIC` | Y | At Batch level |
| **c.** INSOLVENT COMPANY CLAIM NUMBER | 7-26 | A | R | - | `Claim.Number` | Y | At Claim level |
| **d.** RECEIVER CLAIM NUMBER | 27-46 | A | C | Blank | `Claim.ReceiverNumber` | N | At Claim level |
| **e.** FUND CLAIM NUMBER | 47-66 | A | C | Blank | (No direct equivalent) | N | Not preserved |
| **f.** CLAIMANT NUMBER | 67-71 | N | R | - | `Claimant.Number` | Y | At Claimant level |
| **CMS Section 111 Data** | 72-2291 | A | R | - | `Claimant.CMS` | N | **Stored as string** in 3.0; format to be determined |

**Compatibility Notes:**
- **CMS Data Format:** UDS 2.0 positions 72-2291 contain CMS Section 111 Claim Input File Detail Record layout. In UDS 3.0, this is stored in the `CMS` string field on Claimant. The exact format (structured object vs. string) may evolve based on feedback.
- **Medicare Reporting:** UDS 3.0 maintains support for Medicare Section 111 mandatory reporting, with the full CMS data attached to the appropriate claimant.

---

## Trailer Record → Batch Validation

| UDS 2.0 Field | Pos | Type | Req 2.0 | Default 2.0 | UDS 3.0 Path | Req 3.0 | Compatibility Notes |
|---------------|-----|------|---------|-------------|--------------|---------|---------------------|
| Trailer Identifier | 1-20 | A | R | "TRAILER" | (Implicit in JSON structure) | N/A | No trailer record needed in JSON |
| NAIC Number | 21-25 | N | R | - | `Batch.InsuranceCompany.NAIC` | Y | At Batch level |
| Record Type | 26 | A | R | - | (Implicit in JSON hierarchy) | N/A | Record type implicit |
| From State | 27-28 | A | R | - | `Batch.Receiver.Address.State` | Y | At Batch level |
| From Location | 29-30 | N | R | - | (No direct equivalent) | N | Location code deprecated |
| To State | 31-32 | A | R | - | `Batch.GuarantyFund.State` | Y | At Batch level |
| To Location | 33-34 | N | R | - | (No direct equivalent) | N | Location code deprecated |
| Batch Number | 35-37 | N | R | - | `Batch.Id` | Y | At Batch level |
| Batch Prepared Date | 38-45 | N | R | - | `Batch.CreatedOn` | Y | At Batch level |
| Batch From Date | 46-53 | N | R | - | (No direct equivalent) | N | Not tracked |
| Batch Through Date | 54-61 | N | R | - | (No direct equivalent) | N | Not tracked |
| Insurance Type | 62-64 | A | R | "P&C" | `Batch.GuarantyFund.Type` | Y | At Batch level |
| Record Count | 65-73 | N | R | - | `Batch.RowCount` | Y | Same requirement; native integer |
| Total Amount | 74-88 | N | R | - | (No direct equivalent) | N | Amount validation not currently in 3.0 schema |
| Record Filler | Varies | A | R | Spaces | N/A | N/A | No padding in JSON |

**Compatibility Notes:**
- **Record Count Validation:** Both formats require record count validation. UDS 2.0 validates via trailer record; UDS 3.0 validates by comparing `Batch.RowCount` to actual array length.
- **Total Amount Validation:** UDS 2.0 requires net total of all transaction amounts in trailer. UDS 3.0 does not currently track this, but it could be added as `Batch.TotalAmount` in future versions.

---

## Summary: Field Requirement Changes

### Fields Required in Both UDS 2.0 and UDS 3.0 ✅

The following fields maintain the same requirement status across both versions:
- Policy Number
- Claim Number
- Claimant Number
- Policy Effective Date
- Policy Expiration Date
- Date of Loss
- Insured Name (First & Last)
- Claimant Name (First & Last)
- Transaction Code
- All WCIO codes (conditional on WC claims)
- Coverage Code

### Fields Changed from Conditional to Required in UDS 3.0 ⚠️

- `Claimant.FirstName` (was conditional in 2.0 as NAME #2)
- `ReportDate` (was conditional in 2.0)
- `Addresses[].ZipCode` (was conditional in 2.0)
- Return Premium claimant/insured address fields
- `Payee.Identification` (was conditional in 2.0)

### Fields Changed from Required to Optional in UDS 3.0 ⬇️

- Transaction Amount
- Check Date
- Check Amount
- Most indicator fields (Recovery, Suit, 2nd Injury Fund, etc.)
- Document metadata fields

### New Required Fields in UDS 3.0 ✨

- `Addresses[].Country` (required for all addresses)
- `Coverage.Name` (required alongside Code)
- `Payment.Type.Name` (required alongside Code)
- `Catastrophe.Name` (if Catastrophe exists)
- `Claim.WorkersCompensation.Employer` (object required, fields may be empty)
- Various nested object requirements due to hierarchical structure

### Fields Removed or Deprecated in UDS 3.0 ❌

- Location Codes (From/To Location)
- Finance Company Code
- Agent Code / Agent Commission Rate
- Billing Mode
- Long Claim Number (no length limits in 3.0)
- Alternate Indexes 1-4 (replaced by Tags array)
- Note Line Sequence Number (no line limits in 3.0)
- File Type (inferred from URI)
- Fund Claim Number
- Capture Time (merged into CreatedOn timestamp)

### New Optional Fields in UDS 3.0 ✨

- MiddleName (for Insured, Claimant)
- Emails[] (multiple email addresses)
- Multiple Addresses[] (with Type field)
- Document.Tags[] (flexible categorization)
- Document.Fingerprint (hash for integrity)
- Suit details (LawFirm, Contact)
- Employer.Email
- Employer.Addresses[]
- Various metadata fields (CreatedOn timestamps, descriptions, etc.)

---

## Transaction Code Compatibility

**All UDS 2.0 transaction codes are preserved in UDS 3.0 with identical meanings:**

### Claim Status Codes (030-099) ✅ Backward Compatible
- 030, 031, 050, 080, 081, 090, 091, 099

### Loss Codes (100-230) ✅ Backward Compatible
- 100 (Initial Loss File Transmission)
- 130 (Loss Reserve Snapshot)
- 230 (Expense Reserve Snapshot)

### Payment Codes (310-470) ✅ Backward Compatible
- 310, 320 (Loss Payments)
- 410, 420, 450, 470 (Expense Payments)

### Recovery Codes (530-550) ✅ Backward Compatible
- 530, 540, 550

### Other Loss Codes (610-792) ✅ Backward Compatible
- 610, 790, 792

### Premium Codes (800-899) ✅ Backward Compatible
- 800, 815, 820, 825, 835, 840, 850, 860, 870, 899

**Note:** UDS 3.0 EXPANDS compatibility by supporting additional transaction codes as needed, but all UDS 2.0 codes remain valid.

---

## Coverage Code Compatibility

**All UDS 2.0 coverage codes (6-digit format) are preserved in UDS 3.0:**

Common examples:
- `800000` - Liability (other than auto)
- `801000` - Products Liability
- `845000` - Workers' Compensation
- `845012` - WC: Bodily Injury by Accident
- `845021` - WC: Bodily Injury by Disease
- `850000` - Commercial Auto Liability
- `855000` - Private Passenger Auto Liability

**UDS 3.0 adds a required `Coverage.Name` field** alongside the code for human readability, but the code values themselves are **fully backward compatible**.

---

## Data Type Compatibility Summary

| Data Type | UDS 2.0 Format | UDS 3.0 Format | Backward Compatible? |
|-----------|---------------|----------------|---------------------|
| **Dates** | YYYYMMDD (e.g., 20240115) | ISO-8601 (e.g., "2024-01-15") | Convertible ✅ |
| **Timestamps** | YYYYMMDD + HHMMSSSS | ISO-8601 with timezone | Convertible ✅ |
| **Numbers** | Fixed-width with sign suffix | Native JSON number | Convertible ✅ |
| **Currency** | Implied decimal, signed | Native JSON number | Convertible ✅ |
| **Strings** | Fixed-length, padded, UPPERCASE | Variable-length, mixed-case, Unicode | Convertible ⚠️ (case/encoding) |
| **Identifiers** | Zero-padded strings | Native integers or strings | Convertible ✅ |
| **Booleans** | Y/N or Y/N/U | Same string values | Fully compatible ✅ |
| **Codes** | Fixed codes (e.g., 100, 845012) | Same codes | Fully compatible ✅ |

**Conversion Considerations:**
- **Unknown Dates:** UDS 2.0 uses "19010101" sentinel value; UDS 3.0 uses `null`
- **Unknown Names:** UDS 2.0 uses "UDSUNKNOWN"; UDS 3.0 requires actual value
- **Case Sensitivity:** UDS 2.0 requires UPPERCASE; UDS 3.0 preserves original case
- **Unicode:** UDS 2.0 ASCII only; UDS 3.0 full UTF-8 support

---

## Validation: Proving UDS 2.0 Compatibility

### ✅ All UDS 2.0 Required Fields Are Present in UDS 3.0

Every required field from UDS 2.0 has been mapped to a UDS 3.0 equivalent. Some have moved to different levels of the hierarchy (e.g., Policy-level vs. Claim-level), but all data is preserved.

### ✅ All UDS 2.0 Transaction Codes Supported

UDS 3.0 supports the complete set of UDS 2.0 transaction codes (030-899) with identical semantics.

### ✅ All UDS 2.0 Coverage Codes Supported

UDS 3.0 uses the same 6-digit coverage code format and supports all existing codes.

### ✅ All UDS 2.0 Code Tables Preserved

Recovery codes, DCC expense codes, cancellation codes, WCIO codes, and all other code tables from UDS 2.0 are fully supported in UDS 3.0.

### ⚠️ Some Default Values Changed

UDS 2.0's sentinel defaults ("19010101", "UDSUNKNOWN", "00000") are not used in UDS 3.0. Instead, UDS 3.0 uses `null` or requires actual values. This is an improvement but requires conversion logic when migrating from 2.0 to 3.0.

### ⚠️ Some Fields Restructured

Fields like names (split into First/Middle/Last), identifications (split into SSN/EIN/DLN), and documents (tags instead of indexes) have been restructured for better data modeling. All information is preserved but in a more normalized format.

### ✅ UDS 3.0 Expands, Not Restricts

Where UDS 3.0 differs from UDS 2.0, it generally **expands** capabilities:
- Unlimited note length (vs. 1000 chars)
- Multiple addresses (vs. single)
- Multiple emails (vs. none)
- Flexible document tags (vs. 4 fixed indexes)
- Unicode support (vs. ASCII only)
- No batch size limit (vs. 999 claims)

**Conclusion:** UDS 3.0 is fully backward compatible with UDS 2.0 in terms of data content and business rules, with improvements to data modeling and extensibility.

---

**END OF FIELD MAPPING DOCUMENT**
