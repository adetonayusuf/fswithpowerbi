## FINANCIAL STATEMENT WITH POWER BI

### Resources provided
- Complete complete transactional GL from the company's central database - accounting software
- SQL & Power Query to create data model
- Create DAX & power bi to create financial statement
- Cube formular in excel to create alternative way to present financial statement
  

  ### The power of creating the financial statement in power bi are
  - Automate and visualise current reporting
  - Improve efficiency and reporting integrity
  - Develop new insights to provide a competitive advantage
  
 ### CFO points
 - Choice of power bi is because it's more cost effective
 - Time pressure to deliver results
 - Many reports to automate
 - Current excel users need convincing to use Power BI
 
 ### Reason why buisness intelligence projects fail;
 - Poor data quality
 - Lagos of focus on a specific objective
 - Stakeholders don't buy into the project because of there lack of trust in the data or the report been created.
 
 To ensure success we must consider the following;
 - Deliver results quickly
 - Demonstrate real value
 - Narrow the scope of a single objective
 - Focus on reliable data first
 - Replicate current functionality & more
 - Give excel users access to PBI data
 
### Preparing the FS with power bi. The data will be extracted from the company's database
 - SQL to extract data from database using Azure data studio(Use database documentation to create SQL quaries to access the information needed.)
 - Use power query to import & transform the data before creating the data model(At this point we have a power bi model that can refresh data from the database)
 -Build Inome statement in power bi - using dax formulas
 - Build balance sheet in power bi - using dax formulas
 - Give then access to analyse the data in excel
 Financial Statements overview
 These three statements helps us measure the financial performace and health of a company
 - Income statement - Statement of Operation, Profit and Loss Statement P&L
	- measure financial statement over a period of time(Revenues, Expenses, Profit or Loss)

- Balance Sheet - Statement of Financial Position
	- Gives the snapshot of assets, liabilities and equity at a specific point in time
	
- Statement of Cash flow - Cash flow statement
  - Measures the movement of cash in and out of the buisness over a period of time( Operating, Investing & Financing)
  
 
The information in the 3 above financial reports can be use to create financial ratios which provide a further measures of financial performance about
	- Liquidity
	- Leverage
	- Efficiency
	- Profitabillity
	
	
### Using SQL Data Studio to extract that from the organization database

-CREATE A VIEW to capture & combine the columns needed from all the tables in the database provided for financial statement, so that all the data will be refreshed at once.

"CREATE VIEW vwGlTrans

-- This view contains all the information required to create financial statements

AS

SELECT 

    --FactGLTran
    gl.FactGLTranID,
    gl.GLTranAmount,
    gl.JournalID,
    gl.GLTranDescription,
    gl.GLTranDate,

    --GL Accounts
    acc.Alternatekey 'GLAccNum',
    acc.GLAcctName,
    acc.Statement,
    acc.Category,
    acc.Subcategory,

    --Stores
    sto.AlternateKey 'StoreNum',
    sto.StoreName,
    sto.ManagerID,
    sto.PreviousManagerID,
    sto.ContactTel,
    sto.AddressLine1,
    sto.AddressLine2,
    sto.ZipCode,

    --Region
    reg.AlternateKey,
    reg.RegionName,
    reg.SalesRegionName,

    --Last Refrsh Date
        CONVERT(datetime2, GETDATE() at time zone 'UTC' at time zone 'Central Standard Time') AS LastRefreshDate

FROM FactGLTran as gl
    INNER JOIN dimGLAcct AS acc ON gl.GLAcctID = acc.GLAcctID
    INNER JOIN dimStore AS sto ON gl.StoreID = sto.StoreID
    INNER JOIN dimRegion AS reg ON sto.RegionID = reg.RegionID

    GO"
	
The view statement is usually created by an admin, if you do not have admin access, share with an admin to create it.

The clean dataset created with the above view is what is needed to build a reliable data model.

## Power BI

### Build the Data Model
Following the steps below
- Connect to the database using the Power Query Editor in power bi
- Navigate to the SQL view and load as a staging Query
- Create the fact and dimension queries by referencing the staging query
- Challenge - load and relate the queries to create the data model
- Create a DAX measure to aggregate the GLTranAmount column
 
 
 - Connected to database to Power bi by using the credentials
		- server name
		- Database name
		- user name 
		-password
		
	 after conencting to the database you picked the view created and click on transform data.
	 
	 Then we used the view created(staging query) to create the necessary Fact & dimtables
	 
	 Create data model in powerbi, baase on the tables imported into power bi. see below.
	 
### Building Income statement
We started by populating the matrix table with year from DimDate and categoty from DimGLAccts
then create a SumAmount measures using this DAX
SumAmount = SUM(FactGL_Tran[GLTranAmount])
	
Though using FactGL_Tran[GLTranAmount] column directly in the matrix will give us the same value but
best matrix is to use measures and it also help reduces the sizer of the report to avoid lagging.
	
To ensure consistency in the display of the items in the income statement, we need to build a custom headers tables.
We can use the table to control the content and appearance of the financial statement. This custom headers table can control things like;
	- Adding additional line items to the financial statemenmt
	- Defining the sort order of the line items
	- Defining which measures we want each line items to contains
	- Defining the formatting of each line items
	
A well formatted custom header table is mostly created in excel, to align with the financial statement image we are trying to build.

Import the customer header table into Power query, then create a sort column to better arrance the rows in the financial statement, 
the create a relationship with DimGL in data model.

After using the SumAmount measures created to he matrix table, there was some blank rows with value, which relates to balance sheet values.

Now we need to create measures for Income Statement value - I/S Amount

I/S Amount = CALCULATE(ABS([SumAmount]), DimHeaders[Statement] = "Income Statement")

After adding the I/S Amount into the table, there are still some rows that were still blank, I created I/S subtotal measures to substract value of each row from each other
I/S Subtotal = CALCULATE([I/S Amount], FILTER(ALL(DimHeaders), DimHeaders[Sort] < MAX(DimHeaders[Sort])))

In a bid to combine both values in measures I/S amount and I/S subtotal amount, we created Income Statement measures making use of Switch(true) and selected value.

Income Statement = SWITCH(TRUE(),
SELECTEDVALUE(DimHeaders[MeasureName]) = "Subtotal", [I/S Subtotal],
[I/S Amount])

After adding the Income Statement measures into the matrix tables, the % rows were still blank. To fill in the blank rows, since each of the blank rows are % of revenue measures below

% of Revenue = 
var Revenue = CALCULATE([I/S Amount], FILTER(ALL(DimHeaders), DimHeaders[Category] = "Revenue"))
RETURN 
DIVIDE([I/S Subtotal], Revenue,0)

Once confirmed that it's working correctly, we can then incorporate it into Income Statement measure

Income Statement = SWITCH(TRUE(),
SELECTEDVALUE(DimHeaders[MeasureName]) = "Subtotal", [I/S Subtotal],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Per_Of_Revenue", [% of Revenue],
[I/S Amount])

After updating the Income statement measures, the % rows appeared as numeric, i formatted the rows with below

% of Revenue = FORMAT([Staging - % of Revenue], "0.00%").

Then add the year to the column from DimDate table

after then add the sub-category from the GLAcct table to the matrix table.


Uodated income statement measure to address the blank/rows not needed after adding the subcategory from DimGL table

Income Statement = 
var Display_Filtered = NOT ISFILTERED(DimGL_Accts[Subcategory])
RETURN
SWITCH(TRUE(),
SELECTEDVALUE(DimHeaders[MeasureName]) = "Subtotal" && Display_Filtered, [I/S Subtotal],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Per_Of_Revenue" && Display_Filtered, [% of Revenue],
[I/S Amount])

We have created the Income statement in matris, we need to create a LastRefreshDate table from the staging data from SQL
mainly to show the date when the data was collected.

Then populate other visuals on the dashbaord

create Gros Margin Ratio measure to populate the card visual

Gross Margin Ratio = 
var Gross_Profit = CALCULATE([I/S Subtotal], DimHeaders[Category] = "Gross Profit")
var Revenue = CALCULATE([I/S Amount], DimHeaders[Category] = "Revenue")
RETURN
DIVIDE(Gross_Profit, Revenue, 0)

Also created Operaring Margin Ratio measure to populate the last card

Operating Margin Ratio = 
var Operating_Margin = CALCULATE([I/S Subtotal], DimHeaders[Category] = "EBIT")
var Revenue = CALCULATE([I/S Amount], DimHeaders[Category] = "Revenue")
RETURN
DIVIDE(Operating_Margin, Revenue, 0)

After then, populate the combo chart with Revenue and Gross Profit %.

Below is the Income statememt we just created;




### Balance Sheet
We start building the balance sheet by following the following steps;
- Update the custom headers to include balance sheet items
- Create the balance sheet in matrixtable
- Create the following DAX measures: Cumulative AMounts, B/S Subtotal, Retained Earnings, Total Equity, Total Liabilities & Equity
- Create a DAX measure that logically combines the staging measures in the balance sheet template
- Complete other visuals and format the report

Update the DimHeaders table with the balance sheet items
Look through the steps on the Dimheaher table and click on the gear icon infront of navigation to imoport the updated headers table.

Create B/S Amount measures, so that the table display value for only balance sheet items
B/S Amount = CALCULATE([SumAmount], DimHeaders[Statement] = "Balance Sheet")

One of the major differences between Income statement and balance sheet is that
The income statement displays everything that happens over a particular accountingting period whereas the balance sheet is a single snapshot in time.

The amount reported in balance sheet should include cumulative balance over the years during the company's existence. Hence the reason for the DAX measures below

Cumulative Amount = CALCULATE(ABS([B/S Amount]), FILTER(ALL(DimDate), DimDate[Date] <= MAX(DimDate[Date])))

To dispay the cumulative balance of the financial position over the years.

To populate the subtotal section, we start by creating a measures called B/S subtotal

B/S Subtotal = CALCULATE([Cumulative Amount], ALL(DimHeaders), DimHeaders[Balance Sheet Section] in VALUES(DimHeaders[Balance Sheet Section]))

Then create a measire that populate the whole balance sheet at once

Balance Sheet = SWITCH(TRUE(),
SELECTEDVALUE(DimHeaders[MeasureName]) = "Section_Subtotal", [B/S Subtotal],
[Cumulative Amount])

After adding the above DAX into the matrix table, 3 rows were still blank, they are Retained Earnings, Total Equity & Total Liabilities & Equity
On the Retained Earnings we start by creating Opening Retained Earnings

Opening Retianed Earnings = CALCULATE(ABS([SumAmount]),
FactGL_Tran[GLAcctNum] = 4100,
ALL(DimDate),
ALL(DimHeaders))

The current/yearly Retained earning is derived from the Income statement, we are bringing the Net Income from Income Statement to Balance sheet.
The Retained Earnings is the cummulative of yearly net income. Below is the Retained Earnings measures

Retained Earnings = [Opening Retianed Earnings] +
CALCULATE( ABS([SumAmount]),
FILTER(ALL(DimDate), DimDate[Date] <= MAX(DimDate[Date])),
FILTER(ALL(DimHeaders), DimHeaders[Statement] = "Income Statement"))

Then update the Balance SHeet measure with the Retained Earnings

Balance Sheet = SWITCH(TRUE(),
SELECTEDVALUE(DimHeaders[MeasureName]) = "Section_Subtotal", [B/S Subtotal],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Retained_Earnings", [Retained Earnings],
[Cumulative Amount])

Next item is Total Equity; Total Equity = [B/S Subtotal] + [Retained Earnings]

Updated Balance sheet measures 

Balance Sheet = SWITCH(TRUE(),
SELECTEDVALUE(DimHeaders[MeasureName]) = "Section_Subtotal", [B/S Subtotal],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Retained_Earnings", [Retained Earnings],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Total_Equity", [Total Equity],
[Cumulative Amount])

Final item is the TotalLiabilities & Equity

Total Liabilities & Equity = CALCULATE([Cumulative Amount], 
ALL(DimHeaders), DimHeaders[Balance Sheet Section] = "Total Liabilities" || 
DimHeaders[Balance Sheet Section] = "Total Equity") + [Retained Earnings]

updated balance sheet measures

Balance Sheet = SWITCH(TRUE(),
SELECTEDVALUE(DimHeaders[MeasureName]) = "Section_Subtotal", [B/S Subtotal],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Retained_Earnings", [Retained Earnings],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Total_Equity", [Total Equity],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Total_LE", [Total Liabilities & Equity],
[Cumulative Amount])

Add subtotal to the matrix table, to populate the subitem of each balance sheet items.
After after the subcategory from DimGL_Accts table, some of the items were displaying 
incorrectly, we need to wrtie the measure beloow to correct the display issue

Balance Sheet = 
var Display_Filter = NOT ISFILTERED(DimGL_Accts[Subcategory])
RETURN
SWITCH(TRUE(),
SELECTEDVALUE(DimHeaders[MeasureName]) = "Section_Subtotal" && Display_Filter, [B/S Subtotal],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Retained_Earnings" && Display_Filter, [Retained Earnings],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Total_Equity" && Display_Filter, [Total Equity],
SELECTEDVALUE(DimHeaders[MeasureName]) = "Total_LE" && Display_Filter, [Total Liabilities & Equity],
[Cumulative Amount])

Then we create measures for other visuals in the dashboard
Current Ratio = 
var CurrentAssets = CALCULATE([Cumulative Amount], DimHeaders[Category] = "Current Assets")
var CurrentLiabilities = CALCULATE([Cumulative Amount], DimHeaders[Category] = "Current Liabilities")
return
DIVIDE(CurrentAssets, CurrentLiabilities)

Debt Ratio = 
var TotalDebt = CALCULATE([Cumulative Amount], DimGL_Accts[Subcategory]= "Long-term debt")
var TotalAssets = CALCULATE([B/S Subtotal], DimHeaders[Category] = "Total Assets")
Return
DIVIDE(TotalDebt, TotalAssets,0)


#BIDA #CFI #PowerBI #FinancialStatementwithPowerBI
