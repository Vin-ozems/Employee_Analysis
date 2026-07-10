# Employee_Analysis

## Project Overview
This project analyzes Comcast customer complaint data to identify complaint volume, common issue channels and statuses, geographic complaint patterns, and repeat complainers. It also automates part of the complaint resolution workflow using a database trigger.

## Dataset Description
**comcast** — a customer complaint log including:
- Complaint identity: Id, Ticket (unique), Customer Complaint description
- Timing: Date, Date_month_year, Time
- Channel & location: Received Via, City, State, Zip code
- Resolution: Status, Proxy (whether the complaint was handled via proxy)

Data cleaning confirmed no duplicate records and no missing values across all fields before analysis proceeded.

## Cleaning & Transformation Steps
- **Date standardization**: Converted the string-based `Date` column into a proper `DATE` type
- **Month extraction**: Derived a readable `month` name column from the `Date_month_year` field, then dropped the original raw column once the cleaner version was in place

## Key Findings
- **Total complaint volume** was calculated across the full dataset.
- **Status distribution** was ranked to identify the most common complaint status (e.g. Open, Solved, Pending).
- **Complaints by city** were counted, showing geographic concentration of complaint volume.
- **Most common intake channel** was identified (e.g. Customer Care Call, Internet, Email) by ranking `Received_Via`.
- **Repeat complainers**: identified customers who filed more than 5 complaints, useful for flagging chronic service issues or particularly dissatisfied customers.
- **Top 10 cities by complaint volume** were ranked using a window function (`ROW_NUMBER()`), giving a clear priority list for regional service investigation.
- **Peak complaint month identified**: using a two-step CTE, the month with the single highest complaint volume was isolated.

## Recommendation / Tool
- **Automated status verification**: A trigger (`trg_UpdateStatusOnProxy`) automatically updates a complaint's status to "Verified" whenever a complaint is marked "Solved" and its Proxy field is "No" — removing a manual verification step from the resolution workflow and ensuring consistent status tracking without extra staff intervention.
- **On-demand city ranking**: A stored procedure (`top_city()`) returns the top complaint-generating cities ranked by volume, ready to be called whenever an updated view is needed (e.g. for a weekly ops review) without rewriting the query each time.

## Tools & Technology
- MySQL
- Window functions (`ROW_NUMBER()`) for ranking
- CTEs for multi-step aggregation
- Triggers and stored procedures for workflow automation

## Notes
- The "repeat complainers" query groups by `CustomerID` (aliased from `ID`), but `Id` is an auto-increment primary key — meaning each row already has a unique ID, so grouping by it will never return more than 1 row per group. This query likely needs to group by an actual customer identifier (e.g. name or a dedicated customer ID field) rather than the row's own primary key, assuming one exists elsewhere in the source data.
