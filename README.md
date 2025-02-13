# CMA Trade Interface Enhancements Specification

This document details the required enhancements for the CMA trade interface, covering business rules, UI changes, and integration requirements.

---

## 1. ATS Flag Behavior (field017)

**Description**  
- The ATS flag (field017) must appear **only on the trader side** when explicitly provided in the uploaded file.

**Requirement**  
- The flag must **not propagate** to the salesperson side under any circumstances.

**Implementation Notes**  
- During file processing, detect field017 and, if present, store and display it only on the trader side.  
- Ensure that any data transmitted to the salesperson module explicitly excludes field017.

---

## 2. Removal of Intention Code (field041) on Trader Side

**Description**  
- Traders may currently enter an intention code in field041.

**Requirement**  
- Any intention code (field041) entered by a trader should be silently removed.  
- There will never be an intention code on the trader side.

**Implementation Notes**  
- Process trader input to detect field041 and remove it before any further processing or storage.

---

## 3. Date Range Report Grouping

**Description**  
- The date range report should allow filtering trades by date (no time component).

**Requirement**  
- Group trades based on the **lot ID**.  
- Within each lot, list **purchase and buyback trades above sales trades**.

**Implementation Notes**  
- Query trades using only the date portion.  
- Group results by lot ID and apply sorting within each group according to trade type (purchase/buyback first, then sales).

---

## 4. Price Formatting for Trader Input

**Description**  
- The manual input field for pricing must accept fractional formats similarly to the wizard prompt.

**Requirement**  
- Inputs such as “98-25 3/8” must be correctly interpreted.

**Conversion Logic**  
- **Price = 98 + ((25 + (3/8)) / 32)**

**Implementation Notes**  
- Reuse the existing conversion routine from the wizard prompt for manual inputs.  
- Test with various fraction formats (e.g., “98-25”, “98-25 3/8”, “98-25 1/2”).

---

## 5. Sales Credit Calculation

**Description**  
- Sales credit is determined based on principal values.

**Requirement**  
- Calculate sales credit as the difference between principal values (e.g., 990,000 vs. 980,000 = 10,000).  
- If a trade is split across multiple entries (e.g., 990,000 divided into two trades of 490,000 each), allocate the overall sales credit (10,000) proportionally (e.g., 5,000 each).

**Implementation Notes**  
- Ensure the calculation algorithm considers total principal values and allocates credit proportionally.

---

## 6. Expanded Decimal Precision for Calculations

**Description**  
- Internal calculations must support high precision.

**Requirement**  
- Use up to **10 decimal places** internally for all calculations.  
- Before transmission to FIS or PDF generation for myportal/traders, round to **2 decimal places**.

**Implementation Notes**  
- Apply high-precision arithmetic for all internal computations.  
- Use conventional rounding rules when reducing precision to 2 decimals.

---

## 7. Combined "Approve & Process" Button

**Description**  
- The current system uses separate buttons for "Approve" and "Process."

**Requirement**  
- Merge these two functions into a **single button action**.

**Implementation Notes**  
- The combined button must sequentially perform approval and processing.  
- If any error occurs during processing, display the error to the trader immediately.

---

## 8. Table Refresh Button for Wizard Answer Choices

**Description**  
- Users need the ability to refresh the answer choices used in the wizard.

**Requirement**  
- Provide a manual refresh button that updates the underlying database from which the answer choices are drawn.

**Implementation Notes**  
- The refresh button must trigger a reload of data so that any changes in the database are immediately reflected.

---

## 9. Bloomberg Ticket Functionality for Trader-Entered Trades

**Description**  
- Traders must be enabled to create Bloomberg tickets with the same functionality available to salespeople.

**Requirement**  
- Ensure that all necessary information for ticket creation is available.  
- The workflow must mirror that of the salesperson side.

**Implementation Notes**  
- Replicate the Bloomberg ticket creation process from the salesperson module to the trader module, ensuring identical validations and API calls.

---

## 10. Enhanced Date Range Report for Municipal Trades

**Description**  
- The date range report must incorporate additional fields for municipal trades.

**Requirement**  
- Include **Sales Officer, Trans Fee, and Trade Date** (already present in CMA).  
- Query the **XREF** from Intrader, where XREF is the reference number assigned by MSRB and equals the receipt number from Intrader after ticketing.

**Implementation Notes**  
- Integrate an additional query to Intrader to fetch the XREF for each relevant trade.

---

## 11. Cancel Reason in Blotter

**Description**  
- A field for “Cancel Reason” must be displayed on the trade blotter.

**Requirement**  
- Provide a dropdown with the following options:
  - BF Update
  - Dummy CUSIP Update
  - Wrong Price Provided by Broker
  - Update Intention Code
  - Wrong Information Entered
  - Manual Override
- Allow users to enter custom text if needed.

**Implementation Notes**  
- This field is for display only on the blotter; no downstream processing or audit logging is required beyond its display.

---

## 12. Calculation Type Description Field on File Upload

**Description**  
- During the file upload process, include a field for “Calculation Type Description.”

**Requirement**  
- If the value is “DISCOUNT,” require a yield input.  
- For any other value, require a dollar price input.

**Implementation Notes**  
- Implement validation rules that enforce the appropriate input type based on the calculation type.

---

## 13. Trade PDF Transmission Process

**Description**  
- After processing a trade, a PDF is generated and sent via email.

**Requirement**  
- Generate an internal reference number during trade processing.  
- The reference number should be available in Intrader once the trade is keyed.  
- The email must contain **only the trade PDF**.  
- If the internal reference number is not found in Intrader (indicating the trade has not yet been keyed), the system must retry until a match is found.  
- Once the match is confirmed, the trader clicks **“Reconcile”** to mark the lot as reconciled.

**Implementation Notes**  
- Implement a background or polling mechanism to check for the internal reference number in Intrader.  
- Ensure that the reconciliation process updates the trade state within CMA.

---

## 14. Hold Trader-Side Transmission for Non-Municipal Buyback/Sale Lots

**Description**  
- Transmission of trader-side trades must be held for non-municipal lots until the corresponding salesperson trade is submitted.

**Requirement**  
- A lot is considered municipal if its portfolio ID is **“TR MUNI” or “TRA minis”**.  
- For non-municipal buyback/sale lots, hold transmission until a salesperson logs in and creates the corresponding trade.

**Implementation Notes**  
- Implement logic to determine the lot type based on the portfolio ID.  
- Introduce a state that holds the trader-side transmission until the salesperson trade is confirmed.

---

## 15. Revised Trade Creation Process for Trader-Entered Trades

**Description**  
- Salespeople currently use a wizard-based question prompt for trade creation, while traders currently enter each field manually.

**Requirement**  
- Replace the manual entry process on the trader side with a wizard-based process that mirrors the salesperson’s workflow.

**Implementation Notes**  
- Utilize the pre-defined list of questions available for salespeople to design the trader’s wizard.  
- Ensure validations and business logic remain consistent between the two processes.

---

## 16. Cancel Trades Through CMA and Relay to FIS

**Description**  
- CMA must provide functionality to cancel trades and relay the cancellation to FIS.

**Requirement**  
- Query Intrader for the internal reference number to obtain the corresponding receipt number for the cancellation message.  
- The internal reference number is expected to always be present; if missing, display an error and prevent automatic cancellation, requiring manual intervention.

**Implementation Notes**  
- Integrate the cancellation workflow with a cross-check against Intrader.  
- Ensure clear error messaging and a fallback to manual cancellation if necessary.

---

## 17. Bad Factor Update Process

**Description**  
- A “bad factor” flag on the uploaded file indicates that a security trade settled on a prior month’s factor.

**Requirement**  
- Allow the factor to be updated, triggering:  
  - A recalculation of trade totals  
  - Creation of a new Bloomberg ticket  
  - Transmission of updated messages to Intrader  
- A new factor file (uploaded from Bloomberg) will be used for recalculation; the final file format/structure is yet to be finalized.

**Implementation Notes**  
- Develop a backend process triggered when a bad factor flag is detected.  
- Ensure recalculations use the expanded decimal precision rules.  
- Provide detailed logging for the update process.

---

## 18. Municipal Bond Buyback Transactions

**Description**  
- Municipal bond buyback transactions require modifications to the NSCC transmission flag.

**Requirement**  
- For these transactions, set the NSCC xmit flag to **“N”**.  
- This applies when a sale is sent before a buyback; both cancellation messages and new trade messages must have NSCC xmit set to “N.”

**Implementation Notes**  
- Update the messaging logic to force NSCC xmit to “N” for municipal bond buyback transactions.

---

## 19. Reduction of Salesperson Fields for Review

**Description**  
- Salespeople should view a streamlined interface showing only fields relevant to their inputs.

**Requirement**  
- Compile a specific list of non-essential fields and exclude them from the salesperson view.  
- The view should be role-based, showing the reduced set only for users assigned the salesperson role.

**Implementation Notes**  
- Coordinate with business stakeholders to finalize the list of fields to exclude.  
- Adjust the UI to conditionally render fields based on user role.

---

## 20. Reorganize Lot Display Page

**Description**  
- The lot display page must be reorganized to separate reconciled lots from outstanding lots.

**Requirement**  
- The default view should display only unreconciled (outstanding) lots.  
- Create a separate page or tab to show reconciled lots.

**Implementation Notes**  
- Await finalized UI/UX designs or wireframes.  
- Implement filtering, pagination, and search functionality as needed.

---

## 21. Reduction in Transmission Time for Trade PDFs

**Description**  
- The system must optimize the transmission of trade PDFs.

**Requirement**  
- Ensure that all trade PDFs are sent within **one minute** of processing.

**Implementation Notes**  
- Review and optimize the PDF generation and email transmission processes.  
- Benchmark performance to ensure the one-minute target is met consistently.

---

## 22. Page for Admins/Traders to Upload PDFs for Salespeople

**Description**  
- Create a new page in CMA for admins and traders to upload PDFs that bond salespeople can review.

**Requirement**  
- Support uploading PDF files as large as possible.  
- Files should be stored temporarily and auto-deleted at the end of each day (EOD).  
- An email notification must be sent to all salespeople with a link to the page.

**Implementation Notes**  
- Ensure secure file storage with proper access controls.  
- Implement a scheduled job to purge files daily.  
- Design a clear email notification template with the necessary link.

---

## 23. Reporting Functionality for Trade Activity in Intrader

**Description**  
- Integrate reporting functionality that pulls trade activity data from Intrader.

**Requirement**  
- Allow traders and admins to generate reports for all accounts, while restricting salespeople to only their assigned accounts.  
- Data downloads must include only **reconciled lots**.

**Implementation Notes**  
- Design the report interface with filters for all available CMA fields.  
- Implement role-based access controls to limit data visibility.  
- Integrate with Intrader via API or direct query to fetch current data.

---
