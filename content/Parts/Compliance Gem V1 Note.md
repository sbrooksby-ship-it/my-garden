
2026-08-26 14:31

Status: #adult #AIquote

Tags: [[GemTool]]   [[D7]]   [[DP20]] [[Compliance]]


# Compliance Gem V1 Note

Feedback: Okay we have finished testing it, the Gem is great at spotting compliance issues, but its results are inconsistent across identical tests. We would like it to hit a 70% consistency before launching to other departments.

  

We think consistency could be improved if it had a two-pass approach built into it. Right now, the AI skims the content, gets distracted by different elements each run. A two-pass approach forces it to inventory all page text first, then grade that list against our compliance rules kinda like checking every aisle in the grocery store before matching it to your recipe list. If you have specific material you would like to test it on (we used screenshots of BON Ads/ our website, and transcripts) or want help with getting it more consistent we are happy to help.

Possible statement: ### AUDIT EXECUTION INSTRUCTIONS

Phase 1: Visual Inventory Extraction
Before auditing, scan the image completely and list all visible text, headers, pricing terms, and symbols (asterisks, daggers) into a preliminary inventory.

Phase 2: Mandatory Audit Checklist
Evaluate the extracted elements against ALL THREE core compliance categories. You MUST evaluate every category on EVERY run.

1. STATEMENT OF IDENTITY (FDA 21 CFR § 101.3 / FD&C § 403(s))
   - Check: Do product titles omit mandatory classification terms ("Supplement", "Dietary Supplement")?
   - Status: VIOLATION DETECTED / PASS

2. DECEPTIVE CLAIMS & ORPHANED SYMBOLS (FTC Act Sec 5 / FTC Dot Com Disclosures)
   - Check: Are there asterisks (*) without high-contrast, adjacent explanatory footnotes?
   - Status: VIOLATION DETECTED / PASS

3. AUTOPAY & SUBSCRIPTION DISCLOSURES (ROSCA 15 U.S.C. § 8403 / FTC 16 CFR Part 425)
   - Check: Are auto-renewal prices displayed without adjacent baseline prices, billing schedules, and clear cancellation instructions?
   - Status: VIOLATION DETECTED / PASS

Phase 3: Tabular Output Generation
Format ONLY the items marked VIOLATION DETECTED into the standard TSV/tabulated output. If a category passes, do not generate a violation entry for it, but ensure all categories were evaluated during Phase 2.




### References
[[complaince gem tool test 1 data csv]]

