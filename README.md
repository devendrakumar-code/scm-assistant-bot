# SCM Assistant Bot

A Flowise-based supply chain chatbot built for the Trinamix Inc. Junior AI Engineer hiring task. The bot answers questions about supplier performance and procurement policy by combining structured supplier data from the CSV file with policy guidance from the governance PDF. 

## Public Chatbot Link

**Public URL:** https://cloud.flowiseai.com/chatbot/69275e54-9b4e-40bc-84bd-d01f2d639c43
## Project Overview

This project was built in Flowise Cloud as a supply chain assistant that can answer analytical and policy-based questions over two provided files: `supplier_performance_data.csv` and `SupplyChain_Governance_Policy_v3.2-1.pdf`. The assignment requires a publicly accessible chatbot URL, a Flowise chatflow export, screenshots, and a README containing setup details, chunking experiments, QA results, and improvement ideas.

## Files Used

- `supplier_performance_data.csv` — supplier network data with 2,000 purchase orders, 116 suppliers, and 27 columns including OTD rate, defect rate, compliance score, risk level, disruption flags, and PO value. 
- `SupplyChain_Governance_Policy_v3.2-1.pdf` — supplier governance policy covering tier thresholds, SLAs, penalties, risk escalation, sustainability requirements, audit rules, disruption response procedures, and alternate supplier activation rules. 

## Flow Design

The chatbot was designed to answer both quantitative and policy questions by separating responsibilities across tools:

- A structured data path for supplier metrics, filtering, counts, rankings, and aggregations from the CSV-backed supplier data. 
- A policy retrieval path for rules such as SWL restrictions, disruption response levels, rebate criteria, concentration limits, and defect thresholds. 
- A final grounded answer layer that combines data results with the relevant policy clause when a question requires both.

This design is important because several evaluation questions require a combination of supplier facts from the dataset and rule interpretation from the policy document.

## Model and Embeddings

- **Platform:** Flowise Cloud.
- **LLM:** Groq-hosted model used in the Flowise agent flow.  
- **Embeddings:** Embeddings model configured in the Flowise document store for PDF retrieval. 

> Replace the two lines above with the exact model names you used, for example:
> - **LLM:** `llama-3.3-70b-versatile`
> - **Embeddings:** `text-embedding-3-large`

## Chunking Experiments

Two chunking configurations were tested in the document store as required by the assignment. 

### Configuration 1
- **Chunk Size:** 1000
- **Chunk Overlap:** 200

**Observation:** Larger chunks were hard to embed. It kept throwing error.

### Configuration 2
- **Chunk Size:** 250
- **Chunk Overlap:** 50

**Observation:** Smaller chunks gave more focused retrieval for short policy clauses such as SWL restriction language and defect-rate thresholds, but sometimes required multiple retrieved chunks to fully answer a multi-part question.

**Chosen Approach:** The better setup was the one that gave the most reliable policy grounding while keeping responses concise. The final bot uses the configuration that performed better during testing in the Flowise chat panel.

> Replace the chunk values above with the exact ones you actually used, and add the chunk counts Flowise showed after upsert.

## How the Bot Answers Questions

The bot is intended to follow a grounded workflow:

1. Use supplier data for counts, names, filtering, averages, sums, and rankings.
2. Use the policy PDF for thresholds, eligibility criteria, restrictions, escalation levels, and compliance rules. [file:4]
3. Combine both only when the user asks a mixed question such as:
   - “Which region has the highest total PO value, and does it breach the concentration limit?” 
   - “Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?”

This separation is necessary because the CSV is the authoritative source for quantitative supplier metrics, while the policy PDF is the authoritative source for governance rules. [file:4]

## Sample QA Results


### Q1.Which Tier-3 suppliers have an active disruption flag?

**Answer:**  
Count: 24
Suppliers: Archipelago PCB Corp, Bogota Pack Ltda, Bohai Electronics, Buenos Aires Pack, Cerromax Mineria, DaNang Metal Works, Deccan Polymer Mfg, Deltaforge Vietnam, Dravex Components India, Helios Pack Greece, Lima Polymer SA, Longhua Polymer Works, Maghreb Castworks, MeKong Pack Co
Policy Rule: High Risk + disruption flag = Level 3 Activate: CPO escalation + alternate supplier activation for 40% or more of volume within 10 business days.
Conclusion: All 24 Tier-3 suppliers have an active disruption flag, which triggers a Level 3 Activate response according to the policy rule.


### Q2. Which suppliers qualify for the annual Volume Rebate Program and how many are there?



**Answer:**  
Count: 5
Suppliers: Hokkaido Alloy Tech, Kyushu Textile Co, Ridgecroft Textiles, Tasman Circuit Boards, Hanguk Circuit Works
Policy Rule: Tier-1 suppliers with OTD at or above 93.0%, Defect Rate below 0.5%, and Sustainability Score at or above 85 for a full calendar year qualify for an annual volume rebate of 2.5% of total annual invoice value.
Conclusion: There are 5 Tier-1 suppliers that qualify for the Volume Rebate Program based on the policy rule.



### Q3. Which region has the highest total PO value, and does it breach the concentration limit?



**Answer:**  

The region with the highest total PO value is APAC, with a total value of $131,620,356.14. This breaches the concentration limit of 45% of total annual procurement spend, as APAC accounts for 45.1% of total spend.

### Q4. Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?

**Answer:**  

The suppliers on Supplier Watch List (SWL) status are Dongfeng Castings Co (Supplier ID: SUP-006) and Maghreb Castworks (Supplier ID: SUP-080). SWL status limits new POs for these suppliers. 


### Q5. Which product category has the highest average defect rate and does it exceed the Tier-2 limit?

**Answer:**  

The product category with the highest average defect rate is Packaging Materials, with an average defect rate of 1.91%. This exceeds the Tier-2 limit of 2.50%, but does not exceed the Tier-3 limit of 4.00%.

## Key Policy Rules Used by the Bot

Some of the most important policy rules used during question answering are:

- Any supplier with a Compliance Score below 60 is placed on Supplier Watch List status. [file:4]
- SWL status limits new PO issuance to 20% of prior-quarter volume. [file:4]
- Tier-1 suppliers qualify for the Volume Rebate Program only if OTD is at or above 93.0, Defect Rate is below 0.5, and Sustainability Score is at or above 85 for a full calendar year. [file:4]
- No single region may account for more than 45% of total annual procurement spend. [file:4]
- High Risk suppliers with an active disruption flag require Level 3 activation with alternate supplier activation at a minimum of 40% volume within 10 business days. [file:4]
- Tier-2 maximum permissible defect rate is 2.50. [file:4]

## Issues Found During Testing

During testing, one failure mode was that the LLM sometimes tried to fabricate supplier names instead of returning grounded results from the data path. This led to placeholder outputs such as fake supplier names, which is unacceptable for the assignment because the expected answers are explicitly grounded in the provided dataset and policy. [file:3]  
The fix was to tighten the agent prompt so it never simulates results, never invents supplier names, and always uses real database results for supplier lists and counts while using the policy retriever only for rules and thresholds. [file:4]

## What I Would Improve Next

If given more time, the following improvements would be added:

- Enforce stricter tool routing so mixed questions always trigger both the structured data path and the policy retrieval path. 
- Add stronger output validation to block fabricated names, placeholder entities, or unsupported numeric claims.
- Add citation-style answer formatting in the chat output, showing which part comes from supplier data and which part comes from policy.
- Add a fallback guardrail so that if the bot cannot confirm a database result, it responds with a refusal instead of guessing.
- Add regression tests for all five case-study questions before submission.

## Repository Contents

The repository submission should include the following items as required by the task: 

- `scm-assistant.json` — exported Flowise chatflow JSON.
- Screenshots from each major build/test step.
- `README.md` — this file.
- `.gitignore` — excluding `.env` and any API key files. 

