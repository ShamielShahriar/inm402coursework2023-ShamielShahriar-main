---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../lectures/css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 5 Feedback and Answers

## 1. Tidying Data Tables

The answers from quite a lot people suggest to me that not all of you have yet fully grasped what it is for a dataset to be 'tidy'. One common problem was the incorrect assumption that tidying a table means filtering or aggregating data or that a table has more than a certain number of columns. While this might be valid parts of the data shaping process, tidying specifically is just about rearranging existing data, not changing them.

Please do compare your own answers with those below and if necessary, revisit the lecture notes so that you feel confident in spotting tidy tables and gathering messy ones.

**(a)** This table is already in Tidy format so does not need shaping.

| Party                     | percentVote | numMPs |
| :------------------------ | ----------- | ------ |
| Conservative              | 43.6        | 365    |
| Labour                    | 32.2        | 202    |
| Scottish National Party   | 3.9         | 48     |
| Liberal Democrats         | 11.6        | 11     |
| Democratic Unionist Party | 0.8         | 8      |

Even though there are numbers in two columns, they represent different variables (percentages and counts of MPs respectively). Each column is a separate variable and each row a separate observation.

**(b)** This table is not yet in tidy format as the counts of medals are spread across three columns. Note that while gold, silver and bronze medals may be different, the numbers in the table are all 'medal counts' so belong in the same column. This is just like the lecture notes TB example where we had counts of TB in different columns relating to different cohorts, but were combined into a single column of TB counts with a separate column indicating which cohort the count was associated with.

| Rank | Country | NumGold | NumSilver | NumBronze |
| ---- | :------ | ------- | --------- | --------- |
| 1    | China   | 96      | 60        | 51        |
| 2    | GB      | 41      | 38        | 45        |
| 3    | US      | 37      | 36        | 31        |
| 4    | RPC     | 36      | 33        | 49        |

_(RPC = Russia Paralympic Committee)._

To tidy it, we need to be gather those tallies into a single column. We would also need to add a new column indicating whether a medal tally is for gold, silver or bronze medals.

| Rank | Country | MedalType | NumMedals |
| ---- | ------- | --------- | --------- |
| 1    | China   | gold      | 96        |
| 1    | China   | silver    | 60        |
| 1    | China   | bronze    | 51        |
| 2    | GB      | gold      | 41        |
| 2    | GB      | silver    | 38        |
| 2    | GB      | bronze    | 45        |
| 3    | US      | gold      | 37        |
| 3    | US      | silver    | 36        |
| 3    | US      | bronze    | 31        |
| 4    | RPC     | gold      | 36        |
| 4    | RPC     | silver    | 33        |
| 4    | RPC     | bronze    | 49        |

If we were to visualize this table, we could imagine encoding the number of medals with the height of a bar and the medal type with the colour channel - something that we would have struggled to do with the original messy table.

Here's simple example:

```elm {l v}
medalVis : Spec
medalVis =
    let
        data =
            dataFromColumns []
                << dataColumn "country" (strColumn "Country" tidyTableB |> strs)
                << dataColumn "medalType" (strColumn "MedalType" tidyTableB |> strs)
                << dataColumn "numMedals" (numColumn "NumMedals" tidyTableB |> nums)
                << dataColumn "rank" (numColumn "Rank" tidyTableB |> nums)

        medalColours =
            categoricalDomainMap
                [ ( "bronze", "rgb(157,110,73)" )
                , ( "silver", "rgb(163,164,169)" )
                , ( "gold", "rgb(199,170,76)" )
                ]

        enc =
            encoding
                << position X
                    [ pName "country"
                    , pSort [ soByField "rank" opMin ]
                    , pAxis [ axTitle "", axLabelAngle 0 ]
                    ]
                << position Y [ pName "numMedals", pQuant ]
                << color [ mName "medalType", mScale medalColours, mTitle "" ]
    in
    toVegaLite [ width 150, data [], enc [], bar [] ]
```

Note that by default, medal types are stacked in alphabetical order (b -> g -> s) rather than the more intuitive g -> s -> b order. To change the stacking order we would need to create a new variable that represents the desired stacking order, which we could do with a table join:

```elm {l}
orderedMedals : Table
orderedMedals =
    let
        typeOrderTable =
            """MedalType,MedalOrder
gold,1
silver,2
bronze,3"""
                |> fromCSV
    in
    leftJoin ( tidyTableB, "MedalType" ) ( typeOrderTable, "MedalType" )
```

^^^elm m=(tableSummary -1 orderedMedals)^^^

This would allow us to use this new column to provide the stack order:

```elm {l v highlight=[10,28]}
medalVis2 : Spec
medalVis2 =
    let
        data =
            dataFromColumns []
                << dataColumn "country" (strColumn "Country" orderedMedals |> strs)
                << dataColumn "medalType" (strColumn "MedalType" orderedMedals |> strs)
                << dataColumn "numMedals" (numColumn "NumMedals" orderedMedals |> nums)
                << dataColumn "rank" (numColumn "Rank" orderedMedals |> nums)
                << dataColumn "medalOrder" (numColumn "MedalOrder" orderedMedals |> nums)

        medalColours =
            categoricalDomainMap
                [ ( "bronze", "rgb(157,110,73)" )
                , ( "silver", "rgb(163,164,169)" )
                , ( "gold", "rgb(199,170,76)" )
                ]

        enc =
            encoding
                << position X
                    [ pName "country"
                    , pSort [ soByField "rank" opMin ]
                    , pAxis [ axTitle "", axLabelAngle 0 ]
                    ]
                << position Y [ pName "numMedals", pQuant ]
                << color [ mName "medalType", mScale medalColours, mTitle "" ]
                << order [ oName "medalOrder" ]
    in
    toVegaLite [ width 150, data [], enc [], bar [] ]
```

**(c)** This table is already in tidy format so does not need shaping:

| id      | date       | maxTemperature | minTemperature |
| ------- | ---------- | -------------- | -------------- |
| MX17004 | 2010-01-30 | 27.8           | 14.5           |
| MX17004 | 2010-02-02 | 27.3           | 14.4           |
| MX17004 | 2010-02-03 | 24.1           | 14.4           |
| MX17004 | 2010-02-11 | 29.7           | 13.4           |
| MX17004 | 2010-02-23 | 29.9           | 10.7           |
| MX17004 | 2010-03-05 | 32.1           | 14.2           |
| MX17004 | 2010-03-10 | 34.5           | 16.8           |
| MX17004 | 2010-03-16 | 31.1           | 17.6           |
| MX17004 | 2010-04-27 | 36.3           | 16.7           |
| MX17004 | 2010-05-27 | 33.2           | 18.2           |

You may have disagreed and combined `maxTemperature` and `minTemperature` in a single column. That would be a valid approach if, in the context of possible interpretation and visualization, you regard 'minimum temperature' and 'maximum temperature' as being the same type and carrying the same meaning.

Although they are both temperatures, I would argue they measure different things so should not be combined. For those who read [Wickham, 2014](https://www.jstatsoft.org/index.php/jss/article/view/v059i10/v59i10.pdf), you hopefully recognised these data as those appearing in Table 12b.

**(d)** This table is not in Tidy format because percentage of the vote is spread across two columns (one for 2017 and one for 2019), as is the number of MPs.

| Party                     | percentVote2017 | numMPs2017 | percentVote2019 | numMPs2019 |
| ------------------------- | --------------- | ---------- | --------------- | ---------- |
| Conservative              | 42.3            | 317        | 43.6            | 365        |
| Labour                    | 40.0            | 262        | 32.2            | 202        |
| Scottish National Party   | 3.0             | 35         | 3.9             | 48         |
| Liberal Democrats         | 7.4             | 12         | 11.6            | 11         |
| Democratic Unionist Party | 0.9             | 10         | 0.8             | 8          |

To tidy it, we would need to place all the percentage votes values in a single column and all the MP counts in a single column, then add another column indicating whether an observation was in 2017 or 2019. Note that unlike the previous examples we have tidied the data into three new columns, not two.

| Party                     | year | percentVote | numMPs |
| ------------------------- | ---- | ----------- | ------ |
| Conservative              | 2017 | 42.3        | 317    |
| Conservative              | 2019 | 43.6        | 365    |
| Labour                    | 2017 | 40.0        | 262    |
| Labour                    | 2019 | 32.2        | 202    |
| Scottish National Party   | 2017 | 3.0         | 35     |
| Scottish National Party   | 2019 | 3.9         | 48     |
| Liberal Democrats         | 2017 | 7.4         | 12     |
| Liberal Democrats         | 2019 | 11.6        | 11     |
| Democratic Unionist Party | 2017 | 0.9         | 10     |
| Democratic Unionist Party | 2019 | 0.8         | 8      |

After tidying, each column contains data of a different type and meaning (party names, years, percentages and counts).

## 2. Tidying tables programmatically

### Question (b) Medal Table

This needs tidying, which can be achieved with a gathering operation to combine the three medal columns:

```elm {l}
tidyTableB : Table
tidyTableB =
    """Rank,Country,NumGold,NumSilver,NumBronze
       1,China,96,60,51
       2,GB,41,38,45
       3,US,37,36,31
       4,RPC,36,33,49"""
        |> fromCSV
        |> gather "MedalType"
            "NumMedals"
            [ ( "NumGold", "gold" )
            , ( "NumSilver", "silver" )
            , ( "NumBronze", "bronze" )
            ]
```

A few of you were having trouble displaying the newly created tables. Remember the functions that gather and join tables create new `Table` specifications. This is what we use when importing the data table into a vis function (with [dataFromColumns](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataFromColumns)). To display the table summary directly in a litvis document we create a separate function calling [tableSummary](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#tableSummary) that references the table. Because this display function is quite short, it is often convenient to place it inside the more compact 'triple hat' syntax, like this one:

```txt
^^^elm m=(tableSummary -1 tidyTableB)^^^
```

which produces the following formatted table:

^^^elm m=(tableSummary -1 tidyTableB)^^^

### Question (c) Min and Max Temperatures

If you had regarded table (c) as requiring tidying, this could have been shaped with just a single gathering operation:

```elm {l}
tidyTableC : Table
tidyTableC =
    """id,date,maxTemperature,minTemperature
       MX17004, 2010-01-30, 27.8, 14.5
       MX17004, 2010-02-02, 27.3, 14.4
       MX17004, 2010-02-03, 24.1, 14.4
       MX17004, 2010-02-11, 29.7, 13.4
       MX17004, 2010-02-23, 29.9, 10.7
       MX17004, 2010-03-05, 32.1, 14.2
       MX17004, 2010-03-10, 34.5, 16.8
       MX17004, 2010-03-16, 31.1, 17.6
       MX17004, 2010-04-27, 36.3, 16.7
       MX17004, 2010-05-27, 33.2, 18.2"""
        |> fromCSV
        |> gather "temperatureType"
            "temperature"
            [ ( "minTemperature", "min" )
            , ( "maxTemperature", "max" )
            ]
```

^^^elm{m=(tableSummary -1 tidyTableC)}^^^

Note also that had you been given the table in this format and had wished to split the `min` and `max` temperature types into two separate columns (i.e. return it back to the form originally provided in the question), you can use the inverse of the 'gather' operation: [spread](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#spread):

```elm {l}
tidyTableCRestored : Table
tidyTableCRestored =
    spread "temperatureType" "temperature" tidyTableC
```

^^^elm{m=(tableSummary -1 tidyTableCRestored)}^^^

### (d) Tidying the MPs Table

This was the toughest of the tidying questions to implement programmatically. This is because we have two pairs of columns that need to be gathered separately (the MPs/votes for 2017 and the MPs/votes for 2019). Note this is different to gathering just one pair of columns from multiple original columns (as I showed for the TB example in the lecture notes). For this MPs table we need to perform two separate gathering operations.

As before, the first step will be to store the (messy) input data as a table:

```elm {l}
messyD : Table
messyD =
    """Party,percentVote2017,numMPs2017, percentVote2019, numMPs2019
Conservative,42.3,317,43.6,365
Labour,40.0,262,32.2,202
Scottish National Party,3.0,35,3.9,48
Liberal Democrats,7.4,12,11.6,11
Democratic Unionist Party,0.9,10,0.8,8"""
        |> fromCSV
```

Some of you got as far as moving the numeric data into a single column, but still had a mix of numbers of votes and percentage of vote in that column. This would need further tidying so that the two different numeric variables are in two separate columns. Make sure you 'sense check' the transformed table to see if the numbers in their new positions make sense.

When faced with more than one pair of columns to gather in the same table there are a number of alternative approaches we could take.

### Approach 1: Separate tables; gather each and extract columns

```elm {l}
tidyTableD1 : Table
tidyTableD1 =
    let
        t1 =
            messyD
                |> gather "year"
                    "percentVote"
                    [ ( "percentVote2017", "2017" )
                    , ( "percentVote2019", "2019" )
                    ]

        t2 =
            messyD
                |> gather "year"
                    "numMPs"
                    [ ( "numMPs2017", "2017" )
                    , ( "numMPs2019", "2019" )
                    ]
    in
    empty
        |> insertColumn "Party" (strColumn "Party" t1)
        |> insertColumn "year" (strColumn "year" t1)
        |> insertColumn "percentVote" (strColumn "percentVote" t1)
        |> insertColumn "numMPs" (strColumn "numMPs" t2)
```

^^^elm{m=(tableSummary -1 tidyTableD1)}^^^

### Approach 2: Separate tables; gather each, filter columns and left-join

A few people followed this approach, which is similar to the previous one except we join the intermediate tables `t1` and `t2` rather than extract the columns we need. This may be the approach you are most comfortable with if you are familiar with relational database table joins.

```elm {l}
tidyTableD2 : Table
tidyTableD2 =
    let
        t1 =
            messyD
                |> gather "year"
                    "percentVote"
                    [ ( "percentVote2017", "2017" ), ( "percentVote2019", "2019" ) ]
                |> filterColumns (\c -> c == "Party" || c == "year" || c == "percentVote")
                |> insertIndexColumn "key" ""

        t2 =
            messyD
                |> gather "year"
                    "numMPs"
                    [ ( "numMPs2017", "2017" ), ( "numMPs2019", "2019" ) ]
                |> filterColumns (\c -> c == "Party" || c == "year" || c == "numMPs")
                |> insertIndexColumn "key" ""
    in
    leftJoin ( t1, "key" ) ( t2, "key" )
        -- We don't need the key after joining.
        |> filterColumns (\c -> c /= "key")
```

^^^elm{m=(tableSummary -1 tidyTableD2)}^^^

### Approach 3: Reference original columns in gathered key, bisect and spread

This is the approach shown in the [Tidy documentation](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#gather) and the Tidy cheat sheet, that scales to any number of pairs of columns. It relies on naming the 'category columns' (here `p2017` and `p2019`) with a single first letter that distinguishes the percent from the number labels which we extract with [headTail](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#headTail).

```elm {l}
tidyTableD3 : Table
tidyTableD3 =
    messyD
        |> gather "yearCode"
            "votesAndMPs"
            [ ( "percentVote2017", "p2017" )
            , ( "percentVote2019", "p2019" )
            , ( "numMPs2017", "n2017" )
            , ( "numMPs2019", "n2019" )
            ]
        |> bisect "yearCode" headTail ( "type", "year" )
        |> spread "type" "votesAndMPs"
        |> renameColumn "p" "percentVote"
        |> renameColumn "n" "numMPs"
```

^^^elm{m=(tableSummary -1 tidyTableD3)}^^^

As some of you did, if you had wanted to use a different way of indicating whether a value was a percent or number (`p` and `n` above), for example with the prefixes `pct` and `num`, you could provide [splitAt](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#splitAt), for example `(splitAt 3)`, instead of `headTail`. In other words we can chose where to split the string rather than just split at the first character.

## 3. Data Challenge 1: Global Development Literate Data and Visualization

_I saw fewer people attempt this and the Titanic question. While there is no requirement to complete all questions, I would recommend attempting to do so as this provides good practice relevant to your final coursework submission and is designed to help you improve your datavis design skills._

It was good to see some of you separating your data documents ('literate data') from your visualization documents ('literate visualization'). A few had problems linking the two files with `follows`. Remember the name you provide after `follows` should match the name of the file it links to, except without the `.md` extension.

A few of you were finding your data were not being displayed, despite a correct specification in your litvis document. One possible cause is that if you saved a CSV file from Excel, it can have a [Byte Order Mark (BOM)](https://en.wikipedia.org/wiki/Byte_order_mark) which prevents the first line from being read. To remedy this, see point 3.5 in _Dealing with errors in Litvis documents_ on Moodle.

When browsing [Gapminder](https://www.gapminder.org/data/) for datasets to visualize, you may well have selected a dataset that had values not just for a single year, but for several. We will see in later sessions how we could visualize time series like these that are embedded in geographic locations, but to keep things simpler at this stage, just extracting a single year is possible by filtering columns. We can also combine the shaping with the removal of blank rows:

```elm
myFilteredTable : Table
myFilteredTable =
    myDatatable
        |> filterColumns (\colName -> colName == "2010")
        |> filterRows "2010" (\val -> val /= "")
```

Kate's example showed a useful design approach. She looked at net changes in mortality ratios for different countries and symbolised the change with a diverging colour scheme with white at the neutral point. The effect was that counties with little change were close to invisible, but those with either a larger positive or negative change stood out as red or blue. Sometimes making the less important/interesting data less salient can be a good way to highlight what you think is most interesting.

Melody's approach was to focus on those passengers for whom we had an incomplete record (age). The visualization did a good job of highlighting a potential systemic problem with that missing data that appeared to preferentially be associated with people holding third class tickets.

You can see a fully worked example using global democracy index data in [democracyData.md](https://staff.city.ac.uk/~jwo/datavis2023/session05/democracyData.md) and [democracyVisualization.md](https://staff.city.ac.uk/~jwo/datavis2023/session05/democracyVisualization.md) (download these files and put them in your `session05` folder).

## 4. Data Challenge 2: Titanic Survival

I didn't see that many Titanic examples in the submissions this week. I would encourage you to revisit this task and see if you can show anything interesting. It would be good practice for your coursework project.

I liked Emily's data humanistic approach of symbolising people's tickets as a way of connecting with individual passengers, moving from an initial view of a single ticket and then a cluster of tickets that help to emphasise the scale of the disaster.

Teresa created bar charts composed of dots (one per passenger). This was a simple but effective design choice that also helped to convey the scale and human cost of the disaster.
