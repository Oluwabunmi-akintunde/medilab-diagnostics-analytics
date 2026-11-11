# MediLab Diagnostics: Diagnostic Network Performance Report

**Tools:** Power BI, Power Query (M), DAX, star-schema data modelling

**Data:** A single flat file of 12,025 test-request records (26 columns) covering patients, tests, branches, referring clinicians, outcomes and finances across eight Nigerian branches. After cleaning, 12,000 records remain, one per test request.

## Business overview
MediLab Diagnostics is a growing laboratory network providing testing services to individuals, hospitals, clinics, HMOs and corporate healthcare partners across Nigeria. It operates 8 branches and performs 20 different tests across 7 categories (haematology, clinical chemistry, microbiology, serology, hormonal assays, molecular diagnostics and histopathology), with 10 referring clinicians.

## The business problem
MediLab keeps information on patients, tests, branches, referring clinicians, outcomes and finances in one flat file. This causes three problems:
- **One flat file:** patient, branch, clinician and test details repeat across many records, so the same information is stored again and again.
- **Hard to trust and maintain:** duplicates, inconsistent spellings and mixed formats make calculations unreliable and reports hard to update when new data arrives.
- **The insight bottleneck:** leadership needs faster, clearer reporting on volume, branch performance, services and finances.

## Project aim
Turn the flat file into a clean, structured data model and a two-page Power BI dashboard that answers management's questions on demand, service mix, branch performance, operations and profitability.

## My approach

### 1. Cleaned the data in Power Query
| Fault found | Fix applied |
|---|---|
| 25 duplicate Test Request IDs | Removed duplicates (12,025 rows became 12,000) |
| Gender in 6 spellings | Trim, Clean, Capitalize Each Word |
| Age Group in 10 spellings | Standardised to 0-17, 18-30, 31-45, 46-60, 60+ |
| Branch Name (20 variants), Customer Type (11), Payment Method (11), Test Status (8), Specialty (10) | Trim, Clean and Replace Values, leaving 8 branches, 5 customer types, 5 payment methods, 3 statuses and 5 specialties |
| ID columns with leading spaces | Trim and Clean (patient IDs went from 3,808 to 3,789) |
| "Lagos" used as a region, South West spelt 3 ways | Region lookup table merged on State, giving one region per state |
| Numbers stored as text (commas, " days") | Replace Values, then Change Type with Locale to decimal |
| 30 dates stored as text in mixed formats | Converted to Date type, with no blanks or errors left |

### 2. Documented data-quality decisions
I kept questionable records as recorded and flagged them, so management can choose to report or exclude them:
- **17 cancelled requests still carry revenue** (₦150.2K revenue, ₦62.4K cost), mostly at Kano and Abeokuta. I added a "Cancelled With Revenue" column.
- **4 completed requests show ₦0 revenue and cost** (about ₦33.5K at list price, 0.04% of revenue). This looks like an entry fault, so I reported it without changing the data.
- **2 pending requests show ₦0 revenue**, while the other 408 show revenue (₦754.8K in total). I kept it as recorded and noted it as a rule to confirm.

### 3. Built a star schema
A central fact table (`FactTestRequest`, 12,000 rows, one row per request) connects to five dimension tables: `DimPatient` (3,789 patients), `DimBranch` (8), `DimTest` (20), `DimReferringClinician` (10) and `DimCalendar`. All relationships are one-to-many and single direction, so revenue and cost are counted once and never repeated.


### 4. Created the measures
| Measure | What it means |
|---|---|
| Total Test | Distinct count of Test Request ID: how many separate test requests were made |
| Total Patient | Distinct count of Patient ID: how many different people used the lab |
| Revenue / COGS | Sum of Revenue / sum of Cost Incurred: money earned / direct cost of carrying out the tests |
| Profit | Revenue minus COGS (gross profit) |
| Profit Margin | Profit divided by Revenue |
| Avg. Turnaround (Days) | Average days from request to result |
| Completed Test Rate | Completed requests divided by all requests |
| Revenue per Patient | Revenue divided by distinct patients |
| Tests per Patient | Total Test divided by Total Patient |



### 5. Designed a two-page Report
- **Page 1, Executive and Diagnostic Overview:** KPIs, test status, completion rate, volume and turnaround by branch, revenue by test category, customer type and monthly trend, with Year, Branch and Customer Type slicers.
- **Page 2, Patient, Service and Financial Analysis:** revenue, profit and margin by category, revenue per patient by customer type, patient profile, referring clinicians, test results and a branch scorecard, with Year, Branch and Test Status slicers.


## Key findings

**Overall:** 12,000 test requests from 3,789 patients brought in ₦91.16M. After ₦38.08M in direct costs, ₦53.08M remained as gross profit, a 58.22% margin. The average turnaround is 1.04 days, 90.13% of requests are completed, and the average patient is worth ₦24.06K and takes 3.17 tests.

**Demand:** Volume is steady. Each full month has 620 to 717 tests and ₦4.7M to ₦5.6M in revenue, with no busy or quiet season. March 2026 earned the most (₦5.6M) even though December 2025 had more tests (717 against 704), because higher-priced tests were done.

**Branches:** The eight branches are well balanced, handling 1,451 to 1,568 tests with margins between 57.99% and 58.49%. Kano has the lowest completion rate (88.59%) and the most revenue on cancelled requests (₦44.63K), followed by Abeokuta (89.29%, ₦30.54K). Turnaround is almost identical everywhere (1.01 to 1.06 days), so the branch is not the problem.

**Services:** Three categories earn 73.6% of revenue: Clinical Chemistry (₦24.15M), Molecular Diagnostics (₦21.68M) and Hormonal Assay (₦21.23M). Serology (65.1%) and Haematology (63.3%) have the best margins but earn little overall, while Molecular Diagnostics keeps the least (about 54.5%). Turnaround depends on the test type: Histopathology takes about 3 days and Molecular Diagnostics 2 days, against 0.4 to 0.5 days for Haematology and Clinical Chemistry.

**Operations:** Of 12,000 requests, 10,815 (90.13%) were completed, 775 (6.46%) were cancelled and 410 (3.42%) are pending. Results look alike across categories (41% to 43% normal, 18.5% to 20.6% abnormal), with 722 inconclusive results (about 6%).

**Customers and patients:** Walk-ins are the biggest source of demand (4,219 tests, 35.16%), followed by HMOs (21.63%), corporate clients (18.18%) and hospital and clinic referrals (25% together). Walk-ins are also worth the most per patient (about ₦12.8K, against ₦8.7K for clinic referrals). Patients are 52.2% female, 31 to 45 is the largest age group, and about 77% are working-age adults (18 to 60). Across the ten referring clinicians, test counts differ by under 10% but revenue by 15%, so the type of tests ordered matters as much as the number.

## Recommendations
1. **Close the completion gap.** 1,185 requests (9.9%) are cancelled or pending. Start with Kano and Abeokuta, record a cancellation reason and follow up pending requests within 24 hours. Lifting Kano to the network average would complete about 23 more tests.
2. **Audit cancelled revenue.** 17 cancelled requests carry ₦150.2K, so add a refund status and check for unrecorded refunds.
3. **Fix the margin mix.** Review Molecular Diagnostics pricing and costs (23.8% of revenue at about a 54.5% margin, where reaching 58% would add about ₦0.8M profit) and grow Serology and Haematology.
4. **Speed up slow tests.** Focus on Histopathology and Molecular Diagnostics, for example by batching samples or adding staff, rather than on any single branch.
5. **Grow high-value channels.** Invest in walk-in retention (such as repeat-visit reminders) and sign more HMO and corporate contracts for predictable volume.
6. **Share the high-value test menu** (Molecular, Hormonal) with lower-revenue referrers to lift revenue without more volume.
7. **Target the core patient groups.** Design screening packages for the 31 to 45 and 46 to 60 age groups (2,038 patients) and consider women's health bundles.
8. **Tighten data capture.** Fix the ₦0 completed and pending requests and the patient details that differ across the same Patient ID.
9. **Use a monthly scorecard** with targets of a margin of 58% or better, turnaround of about one day and a completion rate above 90%.

## Limitations
- **Profit is gross profit.** COGS covers direct test costs only, so margins exclude staff, rent and other overheads and overstate net profitability.
- **Revenue includes cancelled and pending amounts** kept as recorded (about 1% of revenue combined), so some of it may never be realised.
- **Branch differences are small.** Volumes and margins vary by only a few percent, so rankings should be treated with caution.
- **Patient-level figures depend on Patient ID quality.** Some patient details differ across the same ID, and revenue per patient by customer type counts a patient under every type they used.


## What I learned
This project taught me to turn a messy flat file into a clean star schema, document data-quality decisions instead of silently changing the data, and present every chart with a business question, an insight and a recommendation.

## Repository contents
- README.md: this write-up
- dashboard/: screenshots of both report pages
- docs/: the full project deck (PDF) and data model image
- powerbi/: the Power BI (.pbix) file
