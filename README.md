# Combined Insured Event Status Logic

**JIRA:** SCR-7966 / SCR-7949
**External interactive test page:** https://insured-event-status-v2.pages.dev/

---

## 1. Purpose

This page describes the logic for determining the **Combined Insured Event Status**.

The Combined Insured Event Status is a central calculated status based on:

* Insured Event location
* Insured Event status
* Detailed status / substatus on the Insured Event
* Related claim statuses
* Claim incoming channel for KPOR Postfach tab determination

The goal is to provide one clear status value for service logic, display logic and testing.

---

## 2. Interactive test page

The interactive status combination tester is available here:

**https://insured-event-status-v2.pages.dev/**

The tester can be used to simulate different combinations of:

* Insured Event location
* Claim incoming channel
* Insured Event status
* Detailed status
* Claim statuses

The result shows:

* Combined Insured Event Status
* Portal visibility information
* KPOR Postfach tab
* Reason for the calculated status

---

## 3. Logical summary / solution description

The proposed solution is to introduce one central **Combined Insured Event Status**, which is calculated based on the Insured Event status, the detailed status on the Insured Event and the related claim statuses.

The detailed status **3 - Requirement** has absolute priority.

If the detailed status on the Insured Event is **3 - Requirement**, the Combined Insured Event Status is always:

**REQUIREMENT**

This applies regardless of:

* Insured Event status
* Related claim statuses
* Claim incoming channel

ADUM documents are visible as soon as the document is created.

The target KPOR Postfach tab is determined by the **Claim incoming channel**:

| Claim incoming channel | KPOR Postfach tab  |
| ---------------------- | ------------------ |
| 1 - Post               | Posteingang        |
| 2 - E-Mail             | Posteingang        |
| 3 - Website            | Posteingang        |
| 4 - Merkur App         | Posteingang        |
| 5 - Customer Portal    | Posteingang        |
| 6 - IPS                | Von Makler Betreut |

The individual technical values, such as Insured Event status, Insured Event substatus / detailed status and claim status, may still be exposed for transparency, testing, debugging or other internal purposes.

The **Combined Insured Event Status** remains the central status value for display and service logic.

---

## 4. ADUM document visibility logic

Within **SCR-7966 MI: CP Inbox extension**, the following logic is defined for the documents:

* ADUM GV
* ADUM ANFORDERUNGEN

The visibility is triggered when the document is created.

| Parameter                 | Value / Rule                                                      |
| ------------------------- | ----------------------------------------------------------------- |
| Visibility trigger        | Document created                                                  |
| Visible for customer      | As soon as the ADUM document is created                           |
| KPOR display tab          | Determined by Claim incoming channel                              |
| Former visibility trigger | SETTLED / REJECTED is no longer used for this visibility decision |

### Important rule

The ADUM document is visible as soon as the document is created.

The former visibility trigger **SETTLED / REJECTED** is no longer used for this document visibility decision.

The KPOR Postfach tab is determined by the Claim incoming channel:

| Channel code | Claim incoming channel | KPOR Postfach tab  |
| -----------: | ---------------------- | ------------------ |
|            1 | Post                   | Posteingang        |
|            2 | E-Mail                 | Posteingang        |
|            3 | Website                | Posteingang        |
|            4 | Merkur App             | Posteingang        |
|            5 | Customer Portal        | Posteingang        |
|            6 | IPS                    | Von Makler Betreut |

---

## 5. Combined Insured Event Status values

### SAVED

The Insured Event was created in an external system, for example IPS, and has not yet been transferred to Symass.

### IN PROCESS

The Insured Event is still open, or at least one related claim is in process or postponed.

### REQUIREMENT

Detailed status on Insured Event = **3 - Requirement**.

This status has absolute priority and is independent of the Insured Event status and claim statuses.

### SETTLED

The Insured Event or the related claims are finally completed.

A mix of settled and rejected claims is also treated as **SETTLED** if at least one claim is settled and all remaining claims are rejected.

### REJECTED

The Insured Event is disposed of, or all related claims are rejected / mistakenly entered.

---

## 6. Status calculation priority

The Combined Insured Event Status is calculated according to the following priority:

| Priority | Condition                                                           | Result                        |
| -------: | ------------------------------------------------------------------- | ----------------------------- |
|        1 | Insured Event exists only in external system, for example IPS       | SAVED                         |
|        2 | Detailed status on Insured Event = 3 - Requirement                  | REQUIREMENT                   |
|        3 | At least one related claim is in process or postponed               | IN PROCESS                    |
|        4 | All related claims are settled / reverted / migrated                | SETTLED                       |
|        5 | Mix of settled and rejected claims, with at least one settled claim | SETTLED                       |
|        6 | All related claims are rejected / mistakenly entered                | REJECTED                      |
|        7 | No claims exist, Insured Event status is used                       | Based on Insured Event status |
|        8 | No clear final result can be determined                             | IN PROCESS                    |

---

## 7. Claim object mapping

| Combined group | Claim status           |
| -------------- | ---------------------- |
| PROCESS        | 1 - In Process         |
| PROCESS        | 5 - Postponed          |
| SETTLED        | 3 - Settled            |
| SETTLED        | 7 - Reverted           |
| SETTLED        | 4 - Migrated Claim     |
| REJECTED       | 2 - Rejected           |
| REJECTED       | 6 - Mistakenly entered |

---

## 8. Insured Event object mapping

| Combined group | Insured Event status     |
| -------------- | ------------------------ |
| PROCESS        | 1 - In Process           |
| PROCESS        | 5 - Re-opened            |
| PROCESS        | 6 - Postponed            |
| SETTLED        | 2 - Finished             |
| SETTLED        | 4 - Preliminary finished |
| SETTLED        | 7 - Time-barred          |
| REJECTED       | 3 - Disposed of          |

---

## 9. Claim incoming channel mapping to KPOR Postfach tab

The Claim incoming channel does **not** change the Combined Insured Event Status.

It only determines the KPOR Postfach tab where the created ADUM document is displayed.

| Claim incoming channel code | Claim incoming channel | KPOR Postfach tab  |
| --------------------------: | ---------------------- | ------------------ |
|                           1 | Post                   | Posteingang        |
|                           2 | E-Mail                 | Posteingang        |
|                           3 | Website                | Posteingang        |
|                           4 | Merkur App             | Posteingang        |
|                           5 | Customer Portal        | Posteingang        |
|                           6 | IPS                    | Von Makler Betreut |

---

## 10. Notes for implementation / testing

The Combined Insured Event Status should be used as the central status for display and service logic.

The following values may still be exposed for transparency and debugging:

* Insured Event status
* Insured Event detailed status / substatus
* Claim status
* Claim incoming channel
* Calculated KPOR Postfach tab

The external tester can be used to verify status combinations before implementation or during acceptance testing:

**https://insured-event-status-v2.pages.dev/**
