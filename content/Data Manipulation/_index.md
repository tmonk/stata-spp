---
title: Data Manipulation 
author: T. Hall, T. Monk, J. Dittmar, LSE
weight: 300
---
{{% pageinfo color="info" %}}
This section introduces basic commands for manipulating data in Stata. We will work with one dataset at a time. In Section 6 we will cover how to work with multiple datasets using “frames”.
{{% /pageinfo %}}

## 3.1 Importing Data {#s31}

Stata holds data as .dta files. Importing .dta files is simple: the command is <code><u>u</u>se filename.dta</code> if the file is in your working directory. You will have to include the folder path wrapped in quotation marks if you have not set the working directory <code><u>u</u>se "pathtofolder\\filename.dta"</code>. Alternatively, go through “File -> Open”.

Stata can also read data files saved in other formats, including .xls and .csv files. The easiest way to do this is going through “File -> Import” then select the type of file you want to import.

Before loading a new dataset, you must clear the data Stata is currently holding in its memory using the `clear` command (or add the `, clear` option after the `use` command).

## 3.2 Viewing Data in Stata {#s32}

Stata comes with a few sample data files. One of these has information on life expectancy, gross national product (GNP) per capita, population growth, and the percentage of the population with access to safe water for 68 countries in 1998. 

**Practical Exercise: Load life expectancy data.**

To load the life expectancy data file, type `sysuse lifeexp.dta`.

```
. sysuse lifeexp, clear 
(Life expectancy, 1998) 
```

Stata holds the data in memory like an excel file. But unlike excel, the main Stata interface does not show us our rows and columns of data. To see the actual data, select “Data - > Data Editor -> Data Browse” from the menus or click the data browse icon in the toolbar. Note there is also a “Data Edit” option which allows you to modify datapoints. We never use this option in data analysis, as we want to ensure that all changes made to our data are reproducible.

When you browse the life expectancy data file, you will see 6 columns and 68 rows. The columns in our data are called *variables* – values that measure the same underlying attribute (like life expectancy) across observed units. The rows are *observations* – an observation contains all values measured on the same unit (in our case countries) across attributes.

Another way to view data in Stata is using the <code><u>l</u>ist</code> command. This will print the entire dataset. You can also list only specific variables (e.g., `list country`) or list specific observations (e.g., for the first 5 observations `list in 1/5`). We can also list observations based on their value of a variable. For example, to list all observations with a GNP per capita of less than 500 use the command `list if gnppc < 500`. This example illustrates a powerful feature of Stata: the action of any command can be restricted to a subset of the data using the conditional `if`.

**Practical Exercise: Describe life expectancy data.**

To see how many variables and observations are in a data file we do not need to look directly at the data. We can type <code><u>d</u>escribe</code> in the command window.

```
. describe 
 
Contains data from \\rlab-app\StataSE$\ado\base/l/lifeexp.dta 
Observations:            68                  Life expectancy, 1998 
    Variables:             6                  26 Mar 2020 09:40 
                                              (_dta has notes) 
────────────────────────────────────────────────────────────────────────────────────────
Variable      Storage   Display    Value 
    name         type    format    label      Variable label 
────────────────────────────────────────────────────────────────────────────────────────
region          byte    %16.0g     region     Region 
country         str28   %28s                  Country 
popgrowth       float   %9.0g               * Avg. annual % growth 
lexp            byte    %9.0g               * Life expectancy at birth 
gnppc           float   %9.0g               * GNP per capita 
safewater       byte    %9.0g               * Safe water 
                                            * indicated variables have notes 
────────────────────────────────────────────────────────────────────────────────────────
```

This produces a description of the dataset in memory, listing the variable names and their labels. The dataset has notes that you can see by typing `notes`. Four of the variables have annotations that you can see by typing `notes varname`. You can also see some of this information at a glance in the *Variables* window.

Variable names can have up to 32 characters, are case sensitive, and cannot include spaces or start with a number. To change the name of a variable, use the `rename` command followed by the old variable name and new variable name. <code><u>l</u>abel <u>var</u>iable</code> allows you to change or include a brief description of the variable.

**Practical Exercise: Renaming variables and changing the variable label.**

Change the name and label of the `safewater` variable, adding more information which is found in the `notes`.

```
. rename safewater access_to_safe_water
. label var access_to_safe_water "Access to safe water, % of population"
```

## 3.3 Types of Variables {#s33}

There are essentially two types of variables in Stata, words (referred to as strings) and numbers. Each is handled slightly differently when you are manipulating your data. Within numerical variables, there are two further types: continuous data, such as income or height, and categorical data, such as level of education or gender. In the second case a value will take on a particular meaning. For example, 1=male, 2=female, and 3 = non-binary.

In the life expectancy data, variables `popgrowth`, `lexp`, `gnppc`, and `safewater` are clearly numerical variables (these appear in black in the data browser). However, region and country are different. `country` is a string variable, with the entries appearing as words in the dataset (strings appear in red).

On the other hand, for `region` the entry for observation 1 appears as “Europe & C. Asia” in the data browser, but “1” when you click on it. This means the data is in numerical form (in this case 1-3) but has been labelled so that each number refers to a different region. 

You can create or edit variable labels in Stata using <code><u>l</u>abel define</code>. Stata has a two-step approach to defining labels. First you define a named label set which associates integer codes with labels of up to 80 characters, using the <code><u>l</u>abel define</code> command. Then you associate the set of labels with a variable, using the <code><u>l</u>abel</code> values command.

**Practical Exercise: Changing value labels.**

Replace the current labels in our data with shorthand labels “EU & C.A”, “NA”, “SA”.

```
. label list region
region:
           1 Europe & C. Asia
           2 North America
           3 South America

. label define region 1 "EU & C.A" 2 "NA" 3 "SA", replace

. label values region region

. label list region
region:
           1 EU & C.A
           2 NA
           3 SA

```

If a variable has been read as a string but really contains only numerical values, you will want to use the command `destring`, specifying either the `, replace` option to replace the original string with a numeric variable, or `, gen(varname)` to keep both variables and save the numeric variable with `varname`. 

If instead you want to convert string data into a numeric variable, use the command <code><u>en</u>code</code>, specifying the name of the new variable you want to generate `, gen(varname)`. The new numeric variable will be automatically labelled using the original string names.

To convert a labelled numeric variable to a string, we similarly use the <code><u>dec</u>ode</code> command. Note that this will only work with numeric variables which are labelled. Otherwise, we can use `tostring` to create strings of numbers from unlabelled numeric variables. 

**Practical Exercise: Converting between string and numeric variables.**

Let us convert the string variable `country` to a labelled numeric variable and back to a string again.

```
. encode country, gen(country_code)

. decode country_code, gen(country_string)
```

If you have done this correctly, `country_string` and `country` should be identical (check in data browser). 

## 3.4 Summarising Data {#s34}

One of the first things you will want to do with your data is to summarise its main features. The <code><u>su</u>mmarize</code> command shows us the mean, standard deviation, minimum, maximum, and number of observations for each variable in our data. Alternatively, specifying a variable after the command (e.g. <code><u>su</u>mmarize gnppc</code>) Stata will summarize each variable on its own. If you would like more precise information (e.g., percentiles) then you can add the <code><u>d</u>etail</code> option to the end of that command, i.e., `summarize gnppc, detail`.

**Practical Exercise: Summary statistics of a variable.**

Run simple descriptive statistics for GNP per capita.

```
. summarize gnppc 
 
    Variable │        Obs        Mean    Std. dev.       Min        Max 
─────────────┼───────────────────────────────────────────────────────── 
       gnppc │         63    8674.857    10634.68        370      39980 

```

GNP per capita ranges from $370 to $39,980 with an average of $8,675. We also see that Stata reports only 63 observations on GNP per capita, so we must have some missing values. We discuss how to deal with missing values in section 3.6.6. below.

Sometimes you may wish to break down summaries by a particular variable. The <code><u>bys</u>ort</code> command is very useful here. It is best explained with an example. Suppose you want to see how GNP per capita varies by region. The command here would be `bysort region: summarize gnppc`. This means, in English, “summarize GNP per capita for every distinct value of region.” 

## 3.5 Tables {#s35}

Frequency tables are useful for summarising categorical data. They can be created in Stata using the `table` command. The simplest frequency table is a one-way tabulation of a variable (i.e., tells you how many times each answer is given in response to a particular variable). 

**Practical Exercise: Frequency table of a categorical variable.**

Create a frequency table of regions in the life expectancy data, showing how many countries there are in each region and what percentage of all countries each region makes up.

```
. table region, stat(frequency) stat(percent)
───────────────────┬─────────────────────
                   │  Frequency   Percent
───────────────────┼─────────────────────
Region             |                     
  Europe & C. Asia |         44     64.71
  North America    |         14     20.59
  South America    |         10     14.71
  Total            |         68    100.00
───────────────────┼─────────────────────

```

This is only suitable for variables with relatively few entries, such as categorical data. As with all commands, this can also be accessed through the menus via “Statistics -> Summaries -> Frequency tables -> One-way tables”.

We can also obtain two-way frequency tables which give a breakdown of a variable by another. For example, `table region lexp, stat(frequency) stat(percent)` will show how many countries with each life expectancy there are in each region. The same results can be achieved using the <code><u>tab</u>ulate</code> command. However, the `table` command is a more flexible and powerful tool. For example, it allows us to create three-way frequency tables `(table var1 var2 var3, stat(frequency) stat(percent))`.

The `table` command also allows us to explore summary statistics of continuous variables, like averages. To do this, simply add the continuous variable we want to summarise and summary statistic we want to report to the `stat()` option. The summary statistics we can report using `table` are in the command Helpfile. Common statistics include `mean`, `variance`, `sd`, `count`, `median`, `min`, `max`, `range`.

To specify the number of decimal places to show in our tables use the `nformat()` option. See `help format` for all of the formatting sub-options. Using `%12.2fc` specifies two decimal places and adds a comma to make large numbers more readable.

**Practical Exercise: Frequency table with mean of other variables.**

Add the mean life expectancy and GNP per capita for each region to our frequency table.

```
. table region, stat(freq) stat(mean gnppc) stat(mean lexp) nformat(%12.2fc)
───────────────────────────────────────────────────────────────────────────
                   |  Frequency                      Mean                  
                   |              GNP per capita   Life expectancy at birth
───────────────────────────────────────────────────────────────────────────
Region             |                                                       
  Europe & C. Asia |      44.00        10,738.05                      73.07
  North America    |      14.00         5,817.17                      71.21
  South America    |      10.00         3,645.00                      70.30
  Total            |      68.00         8,674.86                      72.28
───────────────────────────────────────────────────────────────────────────

```

We can also use the `table` command in a similar way to `summarize` by specifying all the summary statistics of a single variable. 

**Practical Exercise: Summary statistics using table.**

For GNP per capita, create a table with the average, standard deviation, number of observations, and min/max values. This gives us exactly the same results as using the `summarize` command we learned above.

```
. table  , statistic(count gnppc) statistic(mean gnppc) statistic(sd gnppc) statistic(max gnppc) statistic(min gnppc) nformat(%12.2fc)

────────────────────────────────────────
Number of nonmissing values |      63.00
Mean                        |   8,674.86
Standard deviation          |  10,634.68
Maximum value               |  39,980.00
Minimum value               |     370.00
────────────────────────────────────────

```

## 3.6 Generating New Variables {#s36}

We have briefly seen how new variables are generated when we switch between strings and labelled numeric data using `encode`/`decode`. This is just one example of the many instances in which we might want to create new variables from our existing data. The most important Stata commands for creating new variables are <code><u>gen</u>erate</code>, `replace`, and `recode`.

### 3.6.1 Generate, Replace

The <code><u>gen</u>erate</code> command creates a new variable using an expression of other variables or constants. For example, `gen gnppc_sq = gnppc^2`. If we have made a mistake in generating a variable, we can drop it using `drop varname`. 
 
Arithmetic operators allowed are:

| Code | Meaning |
| --- | --- |
| _+_ | Add |
| _-_ | Subtract |
| _\*_ | Multiply |
| _/_ | Divide |
| _^_ | Raise to the power of |
| _+_ | String concatenation |

`generate` is often used in conjunction with a logical expression using `if`. For example, if we wanted a variable equal to 1 for all countries with a high life expectancy (say above 75), we would type `gen high = 1 if lexp >75`. To make this variable equal to zero for all countries with life expectancy not above 75 (i.e., to create a dummy variable), we use the replace function:  `replace high = 0 if lexp <=75`. More succinctly, `gen high = lexp > 75` gives us exactly the same dummy variable.

Logical operators allowed are:

| Code | Meaning |
| --- | --- |
| _==_ | Equal |
| _!=_ | Not equal |
| _\>_ | Greater than |
| _\<_ | Less than |
| _\>=_ | Greater than or equal |
| _\<=_ | Less than or equal |
| l | Or |
| _&_ | And |
| _!_ | Not |

We can also generate variables using Stata’s inbuild functions. It is very common for economists to take the natural logarithm of large variables, such as GNP per capita. This is especially useful when a variable displays a non-linear relationship with another variable (more on this in section 5). To take the natural logarithm of GNP per capita, type `gen log_gnp = log(gnppc)`. Notice that this will create missing values for any zero value, since the natural logarithm of zero is undefined.

Here are a few frequently used functions (type `help mathfun` for the full list):

| Code | Meaning |
| --- | --- |
| _abs(x)_ | the absolute value of x |
| _exp(x)_ | the exponential function of x |
| _ln(x) or log(x)_ | the natural logarithm of x if x\>0 |
| _log10(x)_ | the log base 10 of x (for x\>0) |
| _max(x1,x2,…,xn)_ | the maximum of x1, x2, …, xn, ignoring missing values |
| _min(x1,x2,…,xn)_ | the minimum of x1, x2, …, xn, ignoring missing values |
| _round(x)_ | x rounded to the nearest whole number |
| _sqrt(x)_ | the square root of x if x \>= 0 |

### 3.6.2 Extended Generate

The `egen` (“extended gen”) command works just like `gen` but with extra options, which you can explore in the `egen` Helpfile. For example, for a variable of mean life expectancy `egen avg_lexp = mean(lexp)`. With `egen` we can also break commands down into percentiles very easily. For example, to create a variable equal to the 99th percentile of GNP per capita, enter `egen high_gnp = pctile(gnp), p(99)`. Changing the 99 to 50 in that command would produce a variable equal to the median price. 

`egen` is often combined with the `bysort` command to obtain a breakdown of a particular statistic by another variable. For example, we could obtain a variable containing the minimum life expectancy by region by typing: `bysort region: egen min_lexp = min(lexp)`. This essentially groups our data into regions and then applies the `egen` command separately to each region.

### 3.6.3 Summarize, Generate

Results of the `summarize` command are stored by Stata in `r()`. These can also be used to generate new variables. 

**Practical Exercise: Demeaning a variable**

In quantitative analysis, demeaning or centering is when we calculate the deviation of a variable from its mean value. The `summarize` command in conjunction with `generate` can be used to achieve this.  For example, if we wanted a new variable with the deviation of life expectancy from its mean value, we would run the following:

```
. sum lexp

    Variable |        Obs        Mean    Std. dev.       Min        Max
─────────────┼─────────────────────────────────────────────────────────
        lexp |         68    72.27941    4.715315         54         79

. gen dm_lexp = lexp - r(mean)

```

### 3.6.4 Recode

The `recode` command is used to group a numeric variable into categories.

Suppose for example, we wanted to group countries into three life expectancy age categories: “high” for above 75, “medium” for between 65 and 75 inclusive, and “low” for below 65. This could be done by:

```
recode lexp (min/64 = 1) (65/75 = 2) (76/max = 3), gen(lexp_category)
```

A range is specified using a slash and includes the two boundaries. So 65/75 is 65 to 75 inclusive. You can use `min` to refer to the smallest value and `max` to refer to the largest value of the variable you are recoding. 

Then apply labels using commands described above:

```
label define lexp_category 1 "low" 2 "medium" 3 "high", replace
label values lexp_category lexp_category
```

Or you can specify value labels in each recoding rule:

```
recode lexp (min/64 = 1 low) (65/75 = 2 medium) (76/max = 3 high), gen(lexp_cat)
```
	

### 3.6.5 Sorting and Counting

You can sort data by any chosen variable value using the `gsort` command. For example, to sort our countries by life expectancy in ascending order (lowest to highest), type `gsort lexp`. For descending order add a minus symbol before the variable you want to sort, `gsort -lexp`. You can also sort by multiple variables. To sort countries by life expectancy and then GNP per capita in ascending order, `gsort lexp gnppc`.

The subscripts `_n` and `_N` can be used to create variables that count and rank your data. `_N` is used to count the total number of observations. For example, `gen total_country = _N` gives us a new variable which is the total number of observations in the dataset. `_n` tells us the ranking of each observation in the dataset. If we wanted to create a variable which ranks GNP per capita from highest to lowest we would first `gsort -gnppc` then `gen gnp_rank = _n`. This is useful in quantitative analysis, where we are sometimes more interested in the rank than value of a variable, especially if the variable changes non-linearly across observations or over time.

The `_N` and `_n` subscripts are useful when used with the `bysort` command. For example, to generate a variable with the total number of countries in each region, `bysort region: gen reg_tot = _N`. Or for the regional ranking of GNP per capita, from highest to lowest, first `gsort region -gnppc` then `bysort region: gen reg_rank = _n`.  

### 3.6.6 Missing Values

Missing values are common, particularly in survey and census data where individuals might not respond to all questions.

Often missing values will just be blank entries, represented in Stata using a dot (.). For most commands, like the regression commands we will explore in section 5 below, observations which have missing values (for any of the variables which are involved in running that command) are excluded.

However, Stata actually equates missing values with infinity. So, if you try something like `gen gnppc_lb = 500 if gnppc>500` then the missing values will be (unintentionally) included. This would have to be reversed by replace `gnppc_lb = . if missing(gnppc)`.

In many established surveys missing values will be coded as numbers and appropriately labelled: -9 , 99, 9999 are often used. Reading the documentation for datasets you are working with will tell you how to identify missing values. To recode these values as missing in Stata, `recode var (99=.)` or `replace var=. if var==99`.

More conceptually, the challenge posed by missing values are a very important topic in policy analysis. A key question to consider is whether values are missing in a systematic way. For example, if in our life expectancy data only countries with good access to safe water report the % of the population with access, it can lead us to believe that this value is higher than it actually is. Our conclusions about the relationship between clean water and life expectancy would reflect a selected (or non-random) subset of the underlying evidence.  

**Practical Exercise: Generating new variables using generate.**

Create 3 new variables from the life expectancy data:

- A dummy variable called rich equal to 1 if GNP per capita for a country is greater than the sample mean GNP per capita, and 0 otherwise.
- A continuous variable called wellbeing equal to the natural logarithm of GNP per capita added a quarter of life expectancy.
- A dummy variable called missing_any equal to one if any variable is missing for the observation.

```
. sysuse lifeexp, clear 
(Life expectancy, 1998) 

. sum gnppc

    Variable |        Obs        Mean    Std. dev.       Min        Max
-------------+---------------------------------------------------------
       gnppc |         63    8674.857    10634.68        370      39980

. gen rich = gnppc > r(mean)

. replace rich = . if missing(gnppc)
(5 real changes made, 5 to missing)

. gen wellbeing = ln(gnppc) + lexp/4
(5 missing values generated)

. gen missing_any = 0

. replace missing_any = 1 if missing(region) | missing(country) | missing(popgrowth) | missing(lexp) | missing(gnppc) | missing(safewater) | missing(rich)
(31 real changes made)

```

## 3.7 Reshaping Datasets {#s37}

Data can either be in “wide” form or “long” form and we can switch between these. Reshaping is most often needed when we work with panel datasets that contain multiple units (e.g., countries) being observed over time. “Long” data will have a time variable (e.g., year) and units (e.g., countries) being repeated for each time period. “Wide” data will have each unit observed only once, and different variables for each time period.

This is difficult to explain without an example. Load the system blood pressure dataset in its long format: `sysuse bplong, clear`. Looking in our data browser we will see that we have 120 patients, identified by their `patient` id, with their blood pressure `bp`, observed in two time periods (`when`) “before” and “after”, coded as 1 and 2 respectively. Loading the wide version of the same data, `sysuse bpwide, clear`, you will see that we have the same 120 patients, this time with a separate variable for their blood pressure in the “before” and “after” period.  

To go between “long” and “wide” data, we use the `reshape` command. The syntax is a little tricky, but the most important part is to identify our outcome variables (in our case blood pressure `bp`), our time variable (in our case `when`), and our unit identifier (in our case `patient` id). The command then takes the form `reshape “destination shape (wide or long)” “outcome variables”, i(“identifier”) j(“time variable”)`. 

**Practical Exercise: Reshaping data.**

Reshape the blood pressure data from long to wide and back again.

```
. sysuse bplong, clear
(Fictional blood-pressure data)

. reshape wide bp, i(patient) j(when)
(j = 1 2)

Data                               Long   ->   Wide
-----------------------------------------------------------------------------
Number of observations              240   ->   120         
Number of variables                   5   ->   5           
j variable (2 values)              when   ->   (dropped)
xij variables:
                                     bp   ->   bp1 bp2
-----------------------------------------------------------------------------

. reshape long bp, i(patient) j(when)
(j = 1 2)

Data                               Wide   ->   Long
-----------------------------------------------------------------------------
Number of observations              120   ->   240         
Number of variables                   5   ->   5           
j variable (2 values)                     ->   when
xij variables:
                                bp1 bp2   ->   bp
-----------------------------------------------------------------------------
. label define when 1 "before" 2 "after", replace

. label values when when

```

## 3.8 Joining Datasets {#s38}

There are two different situations when you will need to join datasets, each requires a different command. Broadly these involve adding more observations or adding more variables.

Adding new observations is easier. This is done using the <code><u>ap</u>pend</code> command. Load a dataset form the Stata website with information on the population of three counties in California (Los Angeles, Orange, and Ventura). We can append another dataset with the population of three counties in Illinois (Cook, DeKalb, and Will) using the `append` command and specifying the option `generate()` to create a variable identifying from which dataset each observation originally came. We can then label this generated variable to identify which state each county is in.

Notice that the variable names need to be the same for Stata to append the datasets successfully. 

**Practical Exercise: Appending two datasets.**

Append data on the population of counties in California and Illinois. 

```
. use https://www.stata-press.com/data/r18/capop, clear

. list
     +-----------------------+
     |      county       pop |
     |-----------------------|
  1. | Los Angeles   9878554 |
  2. |      Orange   2997033 |
  3. |     Ventura    798364 |
     +-----------------------+

.  append using https://www.stata-press.com/data/r18/ilpop, generate(state)
. label define state 0 "CA" 1 "IL"

. label values state state

. list
     +-------------------------------+
     |      county       pop   state |
     |-------------------------------|
  1. | Los Angeles   9878554      CA |
  2. |      Orange   2997033      CA |
  3. |     Ventura    798364      CA |
  4. |        Cook   5285107      IL |
  5. |      DeKalb    103729      IL |
  6. |        Will    673586      IL |
     +-------------------------------+

```

Adding new variables is slightly more difficult. This is done using the <code><u>mer</u>ge</code> command. We can’t just add the information at random, we need to make sure that each observation is matched in each dataset. So, you need a variable in each dataset that uniquely identifies each observation (e.g., country, county, individual, firm). For example, consider two datasets from the Stata website on the weight and length / price and mileage of old car makes. To merge this data, we first need to identify the observation variable, which in this case is the car make. We then tell Stata that one unique car make in the first dataset corresponds to one unique car make in the second by using the command `merge 1:1 make`. Many-to-one (`m:1`) and one-to-many (`1:m`) merges are also possible, see `merge help` for more details. 

When a merge is performed in Stata, Stata generates a variable called `_merge` which tells us whether a match was found for each observation. If the observation is present in both the original and merged data, `_merge == 3`. If the observation is only in the original data, `_merge == 1`. If its only in the data being merged in, `_merge == 2`. If we wanted to keep only those car makes for which all information is present, we would use the command `keep if _merge == 3`.

**Practical Exercise: Merging two datasets.**

Merge data on the weight and length of car makes to information on price and mileage.  

```
. use https://www.stata-press.com/data/r18/autosize, clear
(1978 automobile data)

. list
     +------------------------------------+
     | make               weight   length |
     |------------------------------------|
  1. | Toyota Celica       2,410      174 |
  2. | BMW 320i            2,650      177 |
  3. | Cad. Seville        4,290      204 |
  4. | Pont. Grand Prix    3,210      201 |
  5. | Datsun 210          2,020      165 |
  6. | Plym. Arrow         3,260      170 |
     +------------------------------------+

. use https://www.stata-press.com/data/r18/autoexpense, clear
(1978 automobile data)

. list
     +---------------------------------+
     | make                price   mpg |
     |---------------------------------|
  1. | Toyota Celica       5,899    18 |
  2. | BMW 320i            9,735    25 |
  3. | Cad. Seville       15,906    21 |
  4. | Pont. Grand Prix    5,222    19 |
  5. | Datsun 210          4,589    35 |
     +---------------------------------+

. merge 1:1 make using https://www.stata-press.com/data/r18/autosize 
    Result                      Number of obs
    -----------------------------------------
    Not matched                             1
        from master                         0  (_merge==1)
        from using                          1  (_merge==2)

    Matched                                 5  (_merge==3)
    -----------------------------------------

. list
     +--------------------------------------------------------------------+
     | make                price   mpg   weight   length           _merge |
     |--------------------------------------------------------------------|
  1. | BMW 320i            9,735    25    2,650      177      Matched (3) |
  2. | Cad. Seville       15,906    21    4,290      204      Matched (3) |
  3. | Datsun 210          4,589    35    2,020      165      Matched (3) |
  4. | Pont. Grand Prix    5,222    19    3,210      201      Matched (3) |
  5. | Toyota Celica       5,899    18    2,410      174      Matched (3) |
  6. | Plym. Arrow             .     .    3,260      170   Using only (2) |
     +--------------------------------------------------------------------+
```














