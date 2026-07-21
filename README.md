# Making-the-Cut-Marathon-Finish-Time-Distribution-Analysis

# 🏃 Making the Cut: Marathon Finish Time Distribution Analysis

This project is based on the Maven Analytics **Making the Cut** Data Drill. The dataset contains finish times for **37,250 marathon runners**, along with demographic information such as age, gender, and half-marathon split times. The objective is to analyze how runners are distributed across predefined marathon finish-time bands. 

Understanding finish-time distributions is useful for race organizers, coaches, and sports analysts because it helps identify performance trends, benchmark runner achievements, and better understand the composition of the field.

## 🎯 Objective

Calculate the percentage of runners who finished within each of the following marathon time bands: 

- Sub 3:00
- 3:00 - 3:30
- 3:30 - 4:00
- 4:00 - 4:30
- 4:30 - 5:00
- 5:00 - 5:30
- 5:30 - 6:00
- 6:00+ 【1-bd8fae】

### Important Rule

The bands are **lower-inclusive**, meaning a finish time of exactly **3:30:00** belongs to the **3:30 - 4:00** band, not the **3:00 - 3:30** band. 
---

# Tools Used

- Power Query
- Power BI

---

# Solution Approach

## Step 1: Create Finish Time Bands

The first step was to create a **Conditional Column** in Power Query to classify each runner into the appropriate finish-time band.

Examples:

| Finish Time | Label |
|------------|---------|
| 2:58:34 | Sub 3:00 |
| 3:14:12 | 3:00 - 3:30 |
| 4:47:03 | 4:30 - 5:00 |
| 6:11:45 | 6:00+ |

This created a categorical field (`Label`) that could be used for aggregation and reporting.

---

## Step 2: Group the Data

I then created a reference query from the original dataset and grouped the runners by their finish-time label.

The purpose was to count how many runners belonged to each band.

### Power Query M Code

```powerquery
let
    Source = #"marathon-data",

    #"Added Index" =
        Table.AddIndexColumn(
            Source,
            "Index",
            1,
            1,
            Int64.Type
        ),

    #"Renamed Columns" =
        Table.RenameColumns(
            #"Added Index",
            {{"Index", "Runner ID"}}
        ),

    #"Grouped Rows" =
        Table.Group(
            #"Renamed Columns",
            {"Label"},
            {{"Count", each Table.RowCount(_), Int64.Type}}
        ),

    #"Added Index1" =
        Table.AddIndexColumn(
            #"Grouped Rows",
            "Index",
            0,
            1,
            Int64.Type
        )

in
    #"Added Index1"
```

### Why Add an Index?

A custom index was added after grouping to control the order of the finish-time bands in Power BI visuals.

Without it, the labels would be sorted alphabetically rather than chronologically.

---

## Step 3: Build the Visualization in Power BI

Using the grouped table, I created a report showing:

- Number of runners per finish-time band
- Percentage of total runners per band
- Ordered finish-time distribution

This provided a clear overview of how the marathon field was distributed across performance levels.

---

#  Alternative Approach Using DAX

After completing the challenge, I realized the aggregation could also be handled entirely through DAX.

Instead of creating a separate grouped table, the `Label` column could remain in the original dataset and the percentage calculation could be created as a measure:

```DAX
Percentage of Runners =
VAR Runners =
    COUNTROWS('marathon-data')

VAR All_Runners =
    CALCULATE(
        COUNTROWS('marathon-data'),
        ALL('marathon-data')
    )

RETURN
    DIVIDE(Runners, All_Runners)
```


The DAX approach has one major advantage:

- The original table remains intact.
- The model stays fully connected.
- Percentages automatically respond to report filters.

This makes it easy to analyze finish-time distributions by:

- Gender
- Age Group
- Half-Marathon Split
- Any other slicer in the report

For exploratory analysis, this flexibility is extremely valuable.

