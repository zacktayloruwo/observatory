
<!-- README.md is generated from README.Rmd. Please edit that file -->

# observatory

<!-- badges: start -->

[![R-CMD-check](https://github.com/zacktayloruwo/observatory/workflows/R-CMD-check/badge.svg)](https://github.com/zacktayloruwo/observatory/actions)
<!-- badges: end -->

# Canadian Communities Policy Observatory

Author: Zack Taylor Last Updated: Feb. 25, 2022

## Interface Structure

### Menu bar

**Tab/buttons:**

-   Observatory: Landing page; clicking returns to landing page.
-   Topic View
-   Place View

**Future tab/buttons:**

-   Analysis View
-   Data Stories
-   Data Extract Builder

**Anchored at the right-hand side:**

-   *Language Selector (future)*
-   *Data Extract Builder (future)*
-   Docs: Goes to documentation
-   Reset button: reinitializes session

### Implementing language modality

We will assume that this app is the English app. In the future, should
text be translated into French, the user will be able to use an EN / FR
selector. For programming purposes, the question is whether to construct
the English and French versions of the app separately, or build language
selection into the lookup tables (leaving the French columns empty)
within a single app.

-   e.g., If lang = “EN” then pull English text; if lang = “FR” then
    pull French text.

### Modular architecture

Results objects are implemented as modules. This is especially important
in Place View, where multiple objects of the same type will be created,
each requiring their own namespace.

-   Communication between modules demonstration:
    <https://thatdatatho.com/communicating-shiny-modules-simple-example/>
-   Modular programming blog post:
    <https://drdoane.com/the-shiny-module-design-pattern/>

### API

An API querying module will be created later.

### General issues

-   How shall we handle “help” or “how-to”? In a central page that opens
    in a new window, or contextually within the app?
-   Use colour-blind-safe palettes.

## Topic View

Sequential input selection: Topic \> Geography \> Time –> Results

### Inputs

**Step 1: Select topic and geography**

| Item                       | Source                                                                               |
|----------------------------|--------------------------------------------------------------------------------------|
| Topic picker               | populated by Topic Lookup Table                                                      |
| Focus variable picker      | within selected topic, populated by Topic Lookup Table                               |
| Focus geography picker     | populated by Geography Lookup Table                                                  |
| Reference geography picker | lists enclosing geographies for focus geography, populated by Geography Lookup Table |

*Click* `Action Button` *to continue*

-   *Availability of data by year depends on topic and geography
    selected.*
-   *Clicking the action button queries the database to identify which
    years are available.*
-   *Step 2 selector only appears after the action button is clicked.
    Implement using Dynamic Visibility as described in Mastering Shiny.
    The first action button then disappears.*

**Step 2: Select time**

| Item              | Source                                                                                                           |
|-------------------|------------------------------------------------------------------------------------------------------------------|
| Time range picker | slider with start and end year selectors (min/max), only shows years available for topic + geography combination |
| Focus year picker | focus year must be within selected time range, else defaults to most recent end year                             |

*Click* `Action Button` *to query data and plot widgets*

### Outputs

-   `ob_map` Choropleth map of focus variable indicating focus and
    reference geography.
-   `ob_distgeo` paired with map
-   Widgets associated with selected focus variable: `ob_distcat`,
    `ob_distord`, `ob_longabs`, `ob_longpct`
-   `ob_vardesc` Description of focus variable.

### Notes and Issues

**Reactivity**

-   For the first selection in the session, action buttons are used to
    avoid unnecessary computation and reinforce intentionality on the
    part of the user.
-   After the initial results are rendered, the user can adjust any of
    the selectors and click on the `Action Button`.
-   Use `freezeReactiveValue()` to minimize computation (Mastering
    Shiny, ch. 10).
-   Put a `Reset` button in the menu bar to purge and start a new
    session.

**Error handling**

-   What happens if the user selects a topic-geography-year combination
    that does not exist?

**Focus geography selection**

Option 1: A single selectize entry lists all named places *and* postal
codes. This is likely impracticable.

Option 2: Radio button to choose *Select by Name* or *Select by Postal
Code*

-   *Select by Name* would return a list of all matching place names
    with unit type and year range attached. e.g., typing “Toronto” would
    return “Toronto Township (CSD) 1881-1966”, “City of Toronto (CSD)
    1851-2021”, “City of Toronto (CD) 2001-2021”, “Metropolitan
    Toronto (CD) 1961-1996”, “Toronto (CMA) 1951-2021”.
-   *Select by Postal Code* would return a list of all overlapping
    geographic units on a best-fit basis (whichever unit overlaps the
    most). Both selection options would require creating new lookup
    tables:
-   *Select by Name*: Rows are unique named places. Columns are
    enclosing units, referenced in the Reference Geography Picker. Names
    have unit type and year range appended.
-   *Select by Postal Code*: Rows are 6-digit postal codes (FSA-LDU).
    Columns are geocodes of best-fit overlapping units.
-   Question: Do the cells in these tables contain place names or their
    unique geocodes? (Can include both if in long format.) If they
    contain geocodes only, there will have to be a further lookup of
    geocode-geoname relationships.

## Place View

Sequential input selection: Geography \> Time \> Topic –> Results

For now, place view is synchronic. It only shows data for the focus
year. Longitudinal plots can be added later. One option would be to
render `ob_distcat` or `ob_distord` (as appropriate) paired with
`ob_longpct` for the same focus variable and geography.

Questions:

-   Should there be a maximum number of widgets?
-   See note under Place View regarding focus geography selection.

### Inputs

**Step 1: Select geography and focus year**

| Item                       | Source                                                                               |
|----------------------------|--------------------------------------------------------------------------------------|
| Focus geography picker     | populated by Geography Lookup Table                                                  |
| Reference geography picker | lists enclosing geographies for focus geography, populated by Geography Lookup Table |
| Focus year picker          | focus year must be within selected time range, else defaults to most recent end year |

*Click* `Action Button` *to continue*

**Step 2: Select topics**

| Item         | Source                          |
|--------------|---------------------------------|
| Topic picker | populated by Topic Lookup Table |

*Click* `Action Button` *to add additional topic widgets*

### Outputs

-   `ob_map` Map with focus geography indicated with outline or
    transparent fill
-   Selected topic widgets: `ob_distcat` or `ob_distord`, depending on
    the variable type.

### Notes and Issues

-   Unlike in topic view, no focus variable is selected. For simplicity,
    the widgets render at the topic level only. If people are interested
    in specific variables, they can use topic view.

**Reactivity**

-   After selecting the focus and reference geographies and the focus
    year, the user selects a topic to render as a widget.
-   After selecting a topic, the user is given the option to add an
    additional topic using an `Action Button`.
-   The results render after each topic is selected.
-   We need a mechanism to remove a topic widget after it is created
    without disturbing the others. See
    <https://appsilon.com/how-to-safely-remove-a-dynamic-shiny-module/>
-   Use `freezeReactiveValue()` to minimize computation (Mastering
    Shiny, ch. 10).

**Error handling**

-   What happens if the user selects a topic-geography-year combination
    that does not exist?

**Displaying descriptive text**

-   How shall we display `ob_vardesc` descriptions of selected
    variables?

## Results Objects

### Notes

-   Results objects are “widgets”, implemented as functions within
    modules, each having their own namespace.
-   Widgets are selected based on the type specified in the Topic Lookup
    Table.
-   The Topic and Place Views call the same modules.

### List of widget objects

| Object       | Name                                                               | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Examples                                                                         |
|--------------|--------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| `ob_map`     | Map                                                                | Choropleth map of all geographic units at same level as focus geography within reference geographi cunit. Fill = focus variable percentage if count variable, absolute value if non-count variable. Focus geography is outlined. Tooltips show values of moused-over units.                                                                                                                                                                                                 |                                                                                  |
| `ob_distgeo` | Locate variable value within distribution of like geographic units | Area graph of focus variable percentage or, for non-count variables, absolute value, for all units at same geographic level as the focus geography within the reference geography, ordered from lowest to highest. Indicate median, average, and focus geographic unit values with vertical lines. Functions as legend for `ob_map`.                                                                                                                                        | % English Mother Tongue; Population Density (pop/sqkm); Median individual income |
| `ob_longabs` | Longitudinal data, absolute values                                 | Line graph of values across time range for focus geography. Use for count and non-count variables.                                                                                                                                                                                                                                                                                                                                                                          | \- Number of Tamils; Median household income                                     |
| `ob_longpct` | Longitudinal data, percentages                                     | Line graph of percentages across time range for focus and reference geographies. Count variables that can eb percentaged only.                                                                                                                                                                                                                                                                                                                                              | % Tamil                                                                          |
| `ob_distcat` | Distribution of topic counts by category (categorical variable)    | Horizontal bar graphs of % of total in each category of selected topic. Parallel bar graphs of focus and reference geography for comparison. If there are six or fewer categories, show them all. If there are more than 6 categories, show top 5 with “other” residual category. If focus variable is not in top 5 (ranked within focus geograophy), displace 5th-place variable in favour of focus variable. Reference geography shows same variables as focus geography. | Housing tenure (% vote / % rent)                                                 |
| `ob_distord` | Distribution of topic counts by cohort (ordinal variable)          | Vertical bar graph of % of total in each cohort or ordinal group within focus geography. Also shows median and/or average if available. Future extensions: (1) split population by sex/gender; (2) add comparison to reference geography.                                                                                                                                                                                                                                   | Age by cohort; income by dollar range                                            |
| `ob_vardesc` | Textual description of topic / variable                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |                                                                                  |

### Conditionality

| Focus variable data type             | Topic View                                                       | Place View                           |
|--------------------------------------|------------------------------------------------------------------|--------------------------------------|
| Count – categorical                  | `ob_map`, `ob_distgeo`, `ob_longabs`, `ob_longpct`, `ob_distcat` | `ob_map`, `ob_distgeo`, `ob_distcat` |
| Count – ordinal                      | `ob_map`, `ob_distgeo`, `ob_longabs`, `ob_longpct`, `ob_distord` | `ob_map`, `ob_distgeo`, `ob_distord` |
| Non-count (median, average, density) | `ob_map`, `ob_distgeo`, `ob_longabs`                             | `ob_map`, `ob_distgeo`               |

## Lookup tables

### Topic Lookup Table

Originally the plan was to use the three-character “theme” prefix
embedded in the varcode to select topics. This will not work in many
cases due to the design of the vacode naming convention. In some cases,
a “theme” is divided into multiple sub-concepts that sum separately. If
each topic is the basis of the percentaging algorithm, we need to
separate these out using a lookup table that links varcodes to topics.

Example: Individual income (theme = “ini”)

In some years, individual income is divided into components:

-   initotl = Total Income
-   iniempl = Employment Income
-   inigovt = Government Income
-   inicomp = % composition of total income by category

Individual income also further subdivides by type of person:

-   ini\*ninc = All individuals without income
-   ini\*winc = All individuals with income
-   ini\*all_nfam = All individuals, non-family persons

Example: Occupation (theme = “lfo”)

-   In some years, occupation counts are providid using different coding
    schemas:
-   lfon01\_ = 2001 National Occupational Classification
-   lfon80\_ = 1980 National Occupational Classification
-   Similar with Industry (theme = “lfi”)

Example: Educational attainment (theme = “eda”)

-   In some years, education attainment category counts are subdivided
    by age group:
-   eda1524 = counts for people between age 15 and 24
-   eda015p = counts for people age 15 and over.

Example: Ethnicity (theme = “eth”)

-   In some years, counts by ethnicity are separately provided in
    “single response” and “multiple response” versions.

Given sunk costs, recoding the variables so that each concept has its
own “theme” prefix is impracticable at this point, however we can create
a lookup table that links varcodes to topics without altering variable
coding.

### Geography Lookup Tables

We need to solve at least four problems:

1.  We need to populate the **named unit-based focus geography
    selector** with a list of all named geographic units. As places have
    similar or identifical names at different geographic levels and
    specific place concepts have been created or dissolved at different
    times, we need to attach the unit level (CSD, CD, CMA, PR) and the
    year range to which it pertains. The list would have to contain
    items like this: “Metropolitan Toronto (CMA) 1951-2021”. Attaching
    the unit level is straightforward. To create the date range, we will
    have to identify the earliest and latest years in which the
    geographic unit appears.

2.  We also need to populate the **postal code-based focus geography
    selector** with a table that lists all postal codes and their
    best-matched corresponding geographic units at all scales (CT, CSD,
    CD, CMA, PR). We can construct this by spatial-joining LDU polygons
    to our census geography. The DMTI Postal-to-Census crosswalk table
    or Canada Post Postal Code Conversion File are insufficient because
    they only include current census units, not historical ones.

3.  We need to populate the **reference geography selector** with a list
    of the enclosing geographic units for the selected focus geography.
    Do we maintain this as a separate lookup table, where columns
    include larger corresponding units for each smaller unit? Or do we
    somehow integrate it with the list of named geographic units? It
    seems logical to work with geographic identifiers (geouids) rather
    than text names.

4.  Finally, to make the **longitudinal graphs** we need to link
    nominally equivalent geographic units across years. We had
    previously called this the *geolinkage* table. How should it be
    optimally formatted? One row for each geographical concept; columns
    are years; cells contain geouids?

Note: DMTI Postal-to-Census Tables on ScholarsGeoPortal (Sept. 2020
vintage links geouids for 2006, 2011, and 2016 census years):
<http://geo.scholarsportal.info/#r/details/_uri@=3161935090$DMTI_2020_CMPCS_PostalToCensusTable>

<!-- ## Installation -->
<!-- You can install the development version of observatory like so: -->
<!-- ``` r -->
<!-- # FILL THIS IN! HOW CAN PEOPLE INSTALL YOUR DEV PACKAGE? -->
<!-- ``` -->
<!-- ## Example -->
<!-- This is a basic example which shows you how to solve a common problem: -->
<!-- ```{r example} -->
<!-- #library(observatory) -->
<!-- ## basic example code -->
<!-- ``` -->
<!-- You'll still need to render `README.Rmd` regularly, to keep `README.md` up-to-date. `devtools::build_readme()` is handy for this. You could also use GitHub Actions to re-render `README.Rmd` every time you push. An example workflow can be found here: <https://github.com/r-lib/actions/tree/v1/examples>. -->
