# ai-case-triage-agent
# Case Triage Agent — Technical Documentation

## Overview

The **Case Triage Agent** is an AI-powered solution that automatically assigns Salesforce Cases to the appropriate owner and identifies the case reason based on customer intent. It compares customer-submitted text against a set of configurable **Case Triage Rules**, leveraging the Salesforce core platform (Data Cloud, Apex, Flow, Prompt Builder, Agent Builder).

This solution uses natural language understanding for routing and is designed to be both **scalable** and **dynamic**.

---

## 1. Custom Object: Case Triage Rule

Stores the routing logic used by the AI agent.

**Fields:**

- `Keyword__c` (Text): Trigger words/phrases from case subject or description.
- `Case_Owner__c` (Lookup to User or Queue): Target owner of the case.
- `Case_Reason__c` (Picklist or Text): Business reason code.
- *(Optional)*: Additional fields like `Intent`, `Priority`, `Department`, and `Active`.

---

## 2. Data Cloud Stream

Feeds Case Triage Rules into Data Cloud.

- **Name:** `Case_Routing_Rules__Stream`
- **Source:** `Case_Routing_Rules__c` (Custom Object)
- **Purpose:** Provides triage data to AI via streaming ingestion.

---

## 3. Data Cloud Data Lake Object (DLO)

- **Name:** `Case_Routing_Rules__DLO`
- **Mapped From:** `Case_Routing_Rules__Stream`
- **Purpose:** Normalized storage for Case Triage Rule data.

---

## 4. Data Cloud Data Model Object (DMO)

- **Name:** `Case_Routing_Rules__DMO`
- **Maps Fields From DLO:**
  - `Keyword`
  - `Case Owner`
  - `Case Reason`

---

## 5. Data Cloud Search Index

- **Name:** `Case_Routing_Rules__SI`
- **Purpose:** Searchable index for retriever queries in Einstein Studio.
- **Configuration:** Indexes fields such as `Keyword`, `Case Owner`, and `Case Reason`.

---

## 6. Einstein Studio Retriever

- **Name:** `Case_Routing_Rules__SI Retriever`
- **Search Index Used:** `Case_Routing_Rules__SI`
- **Returned Fields:**
  - `Case Owner`
  - `Case Reason`
  - `Keywords`
- **Purpose:** Supports retrieval-augmented generation (RAG) in Prompt Builder.

---

## 7. Prompt Builder Templates

**Templates Used:**

- `Triage Agent (flex)`: Orchestration template for routing decisions.
- `Case Owner by Intent (flex)`: Suggests case owner.
- `Case Reason by Intent (flex)`: Suggests reason for the case.

**Prompt Requirements:**

- Uses Retriever context.
- Returns standardized, parsable output.
- Includes confidence scores for validation.

---

## 8. Salesforce Flows

### Record-Triggered Flow

- **Name:** `Triage Agent Flow`
- **Triggered On:** New Case creation.
- **Flow Logic:**
  - Calls `Triage Agent` Prompt Template.
  - Uses Apex class (`TextParser`) to parse results.
  - Routes to "Manual Review Queue" if confidence is low.
  - Updates `Case Owner` and `Case Reason`.

### Autolaunched Flows

- **Update Case Owner**
  - Triggered by Agent Action.
  - Calls `Case Owner by Intent` template.
  - Updates Case Owner field.

- **Update Case Reason**
  - Triggered by Agent Action.
  - Calls `Case Reason by Intent` template.
  - Updates Case Reason field.

---

## 9. Agentforce Agent (Agent Builder)

**Topics Configured:**

- `Case Intent`: Extracts and summarizes case description.
- `Case Owner Determination`: Uses retriever + prompt to suggest and update owner.
- `Case Reason Determination`: Uses retriever + prompt to suggest and update reason.

**Deployment Options:**

- Internal Service Console
- External Customer Portal (Real-time classification)

---

## 10. Agent Actions

| Name                  | Type   | Purpose                                  |
|-----------------------|--------|------------------------------------------|
| Triage Agent          | Prompt | Main entry for full case routing         |
| Case Owner by Intent  | Prompt | Suggests case owner                      |
| Case Reason by Intent | Prompt | Suggests case reason                     |
| Update Case Owner     | Flow   | Applies owner to Case                    |
| Update Case Reason    | Flow   | Applies reason to Case                   |

---

## 11. Additional Configurations

- **Email-to-Case:** Primary entry channel for incoming customer inquiries.
- **Queues and Public Groups:** Owner values in `Case Triage Rule` must match active queues or users.
- **Apex Class (`TextParser`)**: Parses Prompt Builder responses and assigns values to flow variables.

---
