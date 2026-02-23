# Power BI projects

### 1. HR Analytics Report (Microsoft Power BI)

**Business Objective:**
* Analyze workforce trends and identify drivers of employee retention and turnover.
* Key Highlights:
* Cleaned and transformed HR data using Power Query
* Designed interactive dashboard with KPIs and drill-through analysis
* Identified headcount trends and turnover patterns
* Highlighted key retention drivers by department and role

**Measures used (DAX):**
```dax
All employees = DISTINCTCOUNT(People_Fact[employee_id])
AVG # of Empoyees = ([Starting Headcount] + [HeadCount]) / 2
HeadCount = 
CALCULATE ([All employees],
  FILTER (People_Fact,People_Fact[hire_date]<=LASTDATE ('Dates'[Date]) 
  && (People_Fact[term_date] > LASTDATE ('Dates'[Date]) 
  || People_Fact[term_date] = BLANK())))

* analysing retention:

Starting Headcount = 
CALCULATE ([All employees], 
  FILTER (People_Fact, People_Fact[hire_date]<FIRSTDATE('Dates'[Date]) 
  && (People_Fact[term_date] = BLANK() 
  || People_Fact[term_date] >= FIRSTDATE ('Dates'[Date]))))

Ending Headcount = 
CALCULATE ([All employees],
FILTER (People_Fact, People_Fact[hire_date]<FIRSTDATE('Dates'[Date])
&& (People_Fact[term_date] = BLANK()
|| People_Fact[term_date] >= LASTDATE ('Dates'[Date]))))

Retention = DIVIDE ([Ending Headcount],[Starting Headcount])

Min-Max Retention = 
SWITCH(TRUE(),
SELECTEDVALUE('Dates'[Year])=CALCULATE(MIN('Dates'[Year]),
ALLSELECTED('Dates')),[Retention],
SELECTEDVALUE('Dates'[Year])=CALCULATE(MAX('Dates'[Year]),
ALLSELECTED('Dates')),[Retention],
BLANK()
)

* calculating turnover rate:
Turnover % = 
DIVIDE ([Departing Employees],[AVG # of Empoyees])

**Key insights based on the analysis:**
* 1. Retention remained stable at 95% over the final three years of the review period, with Directors exhibiting the   lowest retention levels and by department Finance and Marketing averaged approximately 95%. No significant differences were observed between remote and on-site roles.

* 2. Over the final three years of the examined period (2019–2022), the cumulative turnover rate was 11.3%, with the highest annual turnover recorded in 2022 at 5%. Directors exhibit the highest turnover rate at 16%, while the highest departmental turnover rate reaches 15%. Of all employee separations, 85% are voluntary and 15% are involuntary. The most common reasons for voluntary departures include better career opportunities, more flexible benefits, and higher compensation. These findings suggest that the company should prioritize competitive career development, benefits flexibility, and compensation strategies to retain high-performing employees.

---

### 2. Wine Mag Report (Microsoft Power BI)

**Business Objective:**
* Evaluate price performance, review reliability, and identify high-value wines across countries.
* Key Analysis Areas:
* Price volatility and rating distribution
* Correlation between price and review scores
* “Value for Money” segmentation
* Identification of high-rating, low-price “hidden gems”

**Measures used (DAX):**

# of Reviews = COUNT(winemag_Fact[points])
Avg point = AVERAGE(winemag_Fact[points])
Avg price = AVERAGE(winemag_Fact[price])

Global Avg Score = 
CALCULATE(AVERAGE(winemag_Fact[points]), 
ALL(Taster_Dim[Taster name]))

Max Point = MAX(winemag_Fact[points])
MAX Price = MAX(winemag_Fact[price])
Median Price = MEDIAN(winemag_Fact[price])
Min Point = MIN(winemag_Fact[points])
MIN Price = MIN(winemag_Fact[price])
Number of products = COUNT (winemag_Fact[Wine_ID])
Number of Varities = COUNT(Variety_Dim[Variety_ID])
Nunber of Countries = COUNT(Country_Dim[Country ID])

Perfect Score Winery = 
CALCULATE(COUNTROWS('Winery_Dim'),winemag_Fact[points] = 100)

Price Variance = STDEV.P(winemag_Fact[price])
Ranking Top 10 VFM = IF ( [VFM Rank] <= 10, 1 )
Reviewer Bias = [Avg Point] - [Global Avg Score]

Reviewer Character = 
IF([Reviewer Bias]<0, "Strict", 
IF ([Reviewer Bias]>0 && [Reviewer Bias]<1, "Generous", 
IF([Reviewer Bias]>1,
"Very Generous")))

The most Expensive Product = 
CALCULATE(SELECTEDVALUE(Product_Dim[Product Name]),
TOPN(1, winemag_Fact, winemag_Fac[price], DESC))

Top 10 VFM Wines = 
VAR CurrentVFM = [VFM Score]
VAR Top10Rank = IF([# of Reviews] > 4,
RANKX(ALLSELECTED('Product_Dim'), [VFM Score], CurrentVFM, DESC, Dense))
RETURN IF(Top10Rank <= 10, [VFM Score], BLANK())

Top Quality Variety = 
VAR TopVarietyTable = 
    TOPN(1, ALLSELECTED(Variety_Dim[variety]), 
        [Max Point],DESC)
RETURN
    CONCATENATEX(TopVarietyTable, Variety_Dim[variety], ", ")

Top Variety Measure = 
CALCULATE(
    SELECTEDVALUE(Variety_Dim[variety]), 
    TOPN(1, winemag_Fact, winemag_Fact[price], DESC))

Top Winery Country = 
VAR TopWineryID = MAXX(TOPN(1, ALLSELECTED(Winery_Dim), [Avg Point], DESC), Winery_Dim[Winery_ID])
RETURN
CALCULATE(
    SELECTEDVALUE(Country_Dim[Country]),
    FILTER(ALL(Country_Dim),Country_Dim[Country ID] = LOOKUPVALUE(winemag_Fact[Country ID], winemag_Fact[Winery_ID], TopWineryID)))

Total Price = SUM(winemag_Fact[price])

VFM Rank = 
RANKX(
    ALLSELECTED(Product_Dim[Product Name]),
    [VFM Score],
    ,
    DESC,
    DENSE
)

VFM Score = [Avg Point] / [AVG price]

**Key insights based on the analysis:**
* 1. High price volatility. The bulk of our product price is concentrated in the low-to-mid-tier range (under 50). This means the prices of our sample don't "cluster" neatly around the average. 
* 2. Switzerland records the highest average wine prices in the dataset, while France leads the global luxury segment with the highest single wine price observed.
* 3. The dataset includes more than 140,000 total reviews. Among all countries, England achieves the highest average rating (points), indicating strong overall quality performance.
* 4. A positive relationship exists between price and rating (average points). As median price increases, the average product rating also rises, suggesting that premium positioning is generally associated with higher perceived quality.
* 5. The market remains heavily concentrated in the low-price, mid-rating segment, indicating that the majority of wines compete in accessible price tiers.
* 6. While higher prices generally increase the probability of receiving higher ratings, a substantial number of high-rated, lower-priced “value gems” are also present in the dataset.

---

### 3. Sales Performance Analysis (Microsoft Power BI)

**Business Objective:**
*This report analyzes the sales performance of "E-Commerce Corp," a retail company specializing in electronic devices. The analysis covers the period from January 1, 2022, to December 31, 2022, evaluating performance across four geographical regions and three primary product categories: Electronics, Accessories, and Office.
* Provide executive-level insights into revenue, profitability, and product performance.
* Highlight Insights:
* Revenue and profit trends over time
* Market share and sales stability analysis
* Profit quality assessment by product category
* Identification of high-margin vs high-volume products

**Measures used (DAX):**

Avg Daily Sales = AVERAGEX(VALUES(dim_date[Date]),[Daily Sales])
Avg Monthly Sales = 
AVERAGEX(VALUES('dim_date'[YearMonth]),
CALCULATE([Total Sales]))

Avg Product Lifespan = AVERAGEX(VALUES(dim_product[Product ID]),
DATEDIFF(CALCULATE(MIN(sales_fact[Order Date])),
CALCULATE(MAX(sales_fact[Order Date])),
MONTH))

Category Share % = DIVIDE([Total Sales], [Total Sales All Categories])
Daily Sales = CALCULATE([Total Sales])
Days Active = DISTINCTCOUNT(dim_date[Date])
First Sale Date = CALCULATE(MIN(sales_fact[Order Date]))
Last Sale Date = CALCULATE(MAX(sales_fact[Order Date]))

Product Lifespan Months = 
DATEDIFF([First Sale Date], 
[Last Sale Date], 
MONTH)

Product Rank by (%)Profit Margin = RANKX(ALLSELECTED(dim_product[Product Name]),
[Profit Margin %],,DESC,DENSE)

Product Rank by Profit = RANKX(ALLSELECTED(dim_product[Product Name]),
[Total Profit],,DESC,DENSE)

Product Rank by Sales = RANKX(ALLSELECTED(dim_product[Product Name]),
[Total Sales],,DESC,DENSE)

Product Stage = SWITCH(TRUE(),
    [Product Lifespan Months] < 3, "New",
    [Product Lifespan Months] < 12, "Growing",
    [Product Lifespan Months] < 24, "Mature",
    "Legacy")

Products sold = SUM(sales_fact[Quantity])
Profit Margin % = DIVIDE([Total Profit],[Total Sales])

Rank change (sales-profit margin) = [Product Rank by Sales]-[Product Rank by (%)Profit Margin]

Rank change (sales-profit) = [Product Rank by Sales]-[Product Rank by Profit]

Rank Indicator (sales-profit margin) = 
VAR Change = [Rank change (sales-profit margin)]
RETURN
SWITCH(
    TRUE(),
    Change > 0, "Possitive",
    Change < 0, "Negative",
    "No change")

Rank Indicator (sales-profit) = 
VAR Change = [Rank change (sales-profit)]
RETURN
SWITCH(
    TRUE(),
    Change > 0, "Possitive",
    Change < 0, "Negative",
    "No change")

Sales StdDev = STDEVX.P(VALUES(dim_date[Date]),[Daily Sales])

Stability Index = DIVIDE([Avg Daily Sales], [Sales StdDev])

Top Earner by Profit = CALCULATE(
    MIN(dim_product[Product Name]),
    FILTER(sales_fact, [Product Rank by Profit] = 1))

Top Earner by Sales = CALCULATE(
    MIN(dim_product[Product Name]),
    FILTER(sales_fact, [Product Rank by Sales] = 1))

Top Earners Total Profit = 
VAR TopProduct =
    CALCULATE(
        MIN(dim_product[Product Name]),
    FILTER(sales_fact, [Product Rank by Profit] = 1)
    )RETURN
CALCULATE(
    [Total Profit],
    dim_product[Product Name] = TopProduct)

Top Earners Total Sales = 
VAR TopProduct =
    CALCULATE(
        MIN(dim_product[Product Name]),
    FILTER(sales_fact, [Product Rank by Sales] = 1)
    )RETURN
CALCULATE(
    [Total Sales],
    dim_product[Product Name] = TopProduct
)

Total Orders = COUNT(sales_fact[Order Date])
Total Profit = sum (sales_fact[Profit])
Total Sales = SUM (sales_fact[Sales])
Total Sales All Categories = CALCULATE([Total Sales], ALL(dim_product))

YoY % sales = 
VAR PrevYearSales =
    CALCULATE(
        [Total Sales],
        SAMEPERIODLASTYEAR('dim_date'[Date])
    )
RETURN
DIVIDE([Total Sales] - PrevYearSales, PrevYearSales)```

**Key insights based on the analysis:**
**Category & Regional Insights**
* Market Dominance: The Electronics category is the primary revenue driver, accounting for nearly 50% of the total market share.
* Regional Distribution: Sales are exceptionally balanced across the company's four operating areas, with each region contributing an approximately equal share of total revenue.
* Margin Consistency: Profit margins remain stable across all three product categories and all four regions, indicating a consistent pricing and cost structure company-wide.
**Seasonal Trends & Forecasting**
* Growth Trajectory: Based on current data, sales show a consistent increasing trend projected from 2022 through 2024 across all categories.
* Monthly Performance Peaks: 
Accessories: Experienced slight volatility throughout the year, peaking in May.
Electronics: Maintained steady performance with a peak in April.
Office: Demonstrated stable month-over-month results, with March being the highest-performing month.
**Product-Level Analysis**
* Top Performers: The 'Camera' is the company's flagship product, leading in both total sales and profit, followed closely by the 'Monitor'.
* Profitability Leader: While Cameras lead in volume and gross profit, 'Laptops' boast the highest Profit Margin.
* Underperformers: 'Tablets', 'Headphones', and 'Keyboards' recorded the lowest sales figures. Specifically, 'Tablets' and 'Printers' rank lowest in terms of profit margin, suggesting a need for strategy reassessment in these segments.

---
