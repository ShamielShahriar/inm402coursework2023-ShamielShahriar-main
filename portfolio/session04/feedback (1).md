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

# Session 4 Feedback

## 1. Matching visual variables to data type and task property

You may have matched slightly different sets of measurement types to the visual variables, but below is my list. What I think is the best choice indicated in bold, with possible, but less well suited measurement types indicated in parentheses. What's most important is that you are considering the measurement type when choosing which visual variable to use, and that you make a justifiable choice (something I will be looking for in your CW submissions).

If your own choices are very different to mine (e.g. you think location is only suited to nominal data) please do review the lecture materials or discuss on Slack as it is important you do understand these different measurement types.

|     Visual variable | Suitable measurement types                                                                                        |
| ------------------: | :---------------------------------------------------------------------------------------------------------------- |
|        _colour hue_ | **Nominal**                                                                                                       |
| _colour saturation_ | **Ordinal**, (Quantitative, but hard to estimate precisely)                                                       |
|  _colour lightness_ | **Ordinal**, (Quantitative, slightly easier to estimate than saturation)                                          |
|       _orientation_ | **Nominal**, (Ordinal and Quantitative if data are circular)                                                      |
|             _shape_ | **Nominal**                                                                                                       |
|           _texture_ | **Nominal**, (Ordinal if texture relates to colour lightness)                                                     |
|       _arrangement_ | **Nominal**, (Ordinal if arrangement relates to colour lightness or a clear progression from spaced to clustered) |
|              _size_ | **Quantitative**, (Ordinal if size categories do not imply quantitative scaling)                                  |
|             _focus_ | **Ordinal**, (Quantitative, but hard to estimate precisely)                                                       |
|          _location_ | **Quantitative**, **Ordinal**, **Nominal**                                                                        |

My classification (1=weak, 3-strong) of the properties of each visual variable is less well defined, but my suggestions are below. While you might suggest this is a subjective judgement, and to an extent it is, I would argue there are undoubtedly at least some property-variable combinations that work better than others. The important thing is to make considered choices that you can justify in your selection of visual variables rather than just guess.

|     Visual variable | quantitative | orderable | selective | associative | dissociative |
| ------------------: | :----------: | :-------: | :-------: | :---------: | :----------: |
|        _colour hue_ |      1       |     1     |     3     |      3      |      3       |
| _colour saturation_ |      2       |     2     |     1     |      2      |      3       |
|  _colour lightness_ |      2       |     3     |     2     |      3      |      3       |
|       _orientation_ |      2       |     3     |     3     |      2      |      2       |
|             _shape_ |      1       |     1     |     3     |      3      |      2       |
|           _texture_ |      1       |     2     |     3     |      2      |      3       |
|       _arrangement_ |      1       |     2     |     2     |      1      |      3       |
|              _size_ |      3       |     2     |     3     |      2      |      2       |
|             _focus_ |      2       |     3     |     3     |      2      |      2       |
|          _location_ |      3       |     3     |     3     |      3      |      2       |

One interesting question that a couple of people asked was in what circumstances might the associative and dissociative effect of a visual variable be different? In many cases if a visual variable has a strong associative effect (e.g colour hue) it also has the potential to be strongly dissociative. But I would argue that something like arrangement has a very weak associative effect (would be hard to group similar arrangements), but it could have a stronger interfering effect in its dissociative properties.

You may or may not agree with this, but as with the other criteria, the important lesson is that you consider in your designs whether a particular choice of visual variable will help or hinder the tasks (e.g. grouping) you expect your visualization should support.

I note that some of you just completed this task but not tasks 2 or 3. While this technically counts as a contribution that receives a mark, I strongly encourage you to complete the remaining tasks, even after the deadline, as this gives you valuable datavis design and implementation experience.

## 2. Visualizing the Visual Variables Property Table

Most of you who completed the table were able to produce useful graphical summaries of the results. One issue I noted with a few of you was that you encoded the score (1 to 3) with size, which is sensible, but also specified this to be quantitative. While this sized your symbols correctly, it produces a legend with intermediate points (0.5, 1, 1.5 ... 3.0) rather than just 1, 2 and 3. A more faithful symbolisation of the scores would be to specify them as ordinal, reflecting the fact they are categories not measurements.

I am also continuing to see a few questionable design choices when it comes to colour and shape encoding. For example, how effective do you think it is to encode scores of 1, 2 and 3 only with colour and using red, orange and green? Perhaps refer to your own assessment of the orderable properties of colour hue as well as discussions on the most common forms of [colour blindness](https://en.wikipedia.org/wiki/Color_blindness). Similarly I saw some people encode scores (1-3) with shape. Scores clearly have an order, but shape is arguably unordered so only really suitable for unordered nominal data.

Updating the `vvTable` function should have been easy if you had completed the table as above. Below, the table values are stored in `table` and _tidied_ (see Session 5) in a form suitable for encoding.

```elm {l}
vvTable =
    let
        table =
            """visualVariable,    q, o, s, a, d
               colour hue,        1, 1, 3, 3, 3
               colour saturation, 2, 2, 1, 2, 3
               colour lightness,  2, 3, 2, 3, 3
               orientation,       2, 3, 3, 2, 2
               shape,             1, 1, 3, 3, 2
               texture,           1, 2, 3, 2, 3
               arrangement,       1, 2, 2, 1, 3
               size,              3, 2, 3, 2, 2
               focus,             2, 3, 3, 2, 2
               location,          3, 3, 3, 3, 2"""
                |> fromCSV
                |> gather "property"
                    "rating"
                    [ ( "q", "quantitative" )
                    , ( "o", "orderable" )
                    , ( "s", "selective" )
                    , ( "a", "associative" )
                    , ( "d", "dissociative" )
                    ]
    in
    dataFromColumns []
        << dataColumn "visualVariable" (table |> strColumn "visualVariable" |> strs)
        << dataColumn "property" (table |> strColumn "property" |> strs)
        << dataColumn "rating" (table |> numColumn "rating" |> nums)
```

In terms of visual design, this was my solution: Adapting the example provided in the lecture materials, I double encoded rating with size and colour lightness (reinforced by encoding with opacity against a white background). Note the use of size here helps to distinguish ratings of 1 (unsuitable) from the other two ratings. Additionally, I configured the chart to be more 'Tufte-like' removing unnecessary chart embellishments (see Session 2).

```elm {l v}
summary : Spec
summary =
    let
        enc =
            encoding
                << position Y [ pName "visualVariable", pTitle "" ]
                << position X [ pName "property", pTitle "", pAxis [ axOrient siTop ] ]
                << color [ mName "rating", mOrdinal ]
                << opacity [ mName "rating", mQuant ]
                << size
                    [ mName "rating"
                    , mScale [ scRange (raNums [ 50, 900 ]) ]
                    ]

        cfg =
            configure
                << configuration (coAxis [ axcoDomain False, axcoTicks False, axcoLabelAngle 0 ])
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite
        [ autosize [ asNone ]
        , padding (paEdges 80 20 0 0)
        , width 400
        , height 400
        , cfg []
        , vvTable []
        , enc []
        , circle []
        ]
```

It looked like some people were getting a little confused over the difference between a _domain_ and _range_ in their specifications. A **domain** describes the extent of something in 'data space'. For example the domain of the rating in this exercise is 1 to 3. A **range** describes the extent of something in 'visual space', for example the range of the rating in this exercise is 50 to 900 square pixels describing the size of the smallest and largest symbols used. When encoding with colour, a range will describe the colour space and might, for example, range from red to blue. You can think of visual encoding as the scaling of a domain to a range.

I note that a few of you double-encoded `property` (associative, orderable etc.) with both position and shape. You may wish to compare that design (below) with one that just uses position (above). I would argue that shape has a dissociative effect here making it just a bit harder to compare values along a row, adding visual complexity but without much gain in clarity or information.

```elm {v}
summary2 : Spec
summary2 =
    let
        enc =
            encoding
                << position Y [ pName "visualVariable", pTitle "" ]
                << position X [ pName "property", pTitle "", pAxis [ axOrient siTop ] ]
                << color [ mName "rating", mOrdinal, mLegend [] ]
                << shape [ mName "property" ]
                << opacity [ mName "rating", mQuant, mLegend [] ]
                << size
                    [ mName "rating"
                    , mScale [ scRange (raNums [ 50, 900 ]) ]
                    , mLegend []
                    ]

        cfg =
            configure
                << configuration (coAxis [ axcoDomain False, axcoTicks False, axcoLabelAngle 0 ])
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite [ width 400, height 400, cfg [], vvTable [], enc [], point [ maFilled True ] ]
```

### 3. Design Challenge: Exploring data-channel design options for Gapminder data

It's good to see a number of you didn't just create charts for this task, but provided a brief evaluation of your designs, including ones you rejected and why. This is a good habit to get into and I'd encourage all of you to do the same when you are exploring possible visualization designs. To illustrate, I've done this in my example below.

For those interested in the source of the data (which I think is a good habit to get into when creating data visualizations) 'health' values represent life expectancy at birth – see this [description from GapMinder](https://www.gapminder.org/data/documentation/gd004/).

A few of your designs showed one of the variables (e.g. population or income) encoded with Y position and the country encoded on the X position. While this can work, by default this orders countries alphabetically along the x-axis. Such ordering may be useful for _lookup tasks_ where you have a particular country in mind, need to find it and then read its population value. But it is poorer for showing trends in the quantitative variable. To show trends in a single variable, it can be more useful to order category position by the value of the variable with [pSort](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pSort), or to create a frequency histogram showing the distribution of the variable.

```elm {l}
gmData =
    dataFromUrl "https://vega.github.io/vega-lite/data/gapminder-health-income.csv" []
```

```elm {l v}
sortedCategories : Spec
sortedCategories =
    let
        enc =
            encoding
                << position Y
                    [ pName "country"
                    , pSort [ soByChannel chX ] -- Try commenting out this line to remove sorting
                    , pAxis [ axTitle "", axLabelFontSize 6 ]
                    ]
                << position X [ pName "health", pQuant, pScale [ scZero False ] ]
    in
    toVegaLite [ width 600, height 1200, gmData, enc [], bar [] ]
```

```elm {l v}
histogram : Spec
histogram =
    let
        enc =
            encoding
                << position X [ pName "health", pBin [] ]
                << position Y [ pAggregate opCount ]
    in
    toVegaLite [ gmData, enc [], bar [] ]
```

In my example, for clarity in this feedback report I have placed some of the different design choices in the same document, but exploring different options in parallel branching documents may be a better way of organising your design explorations.

As a first design for the Gapminder data, we might choose to prioritise location as the most _effective_ visual variable for estimating quantitative values (See 'Expressiveness and Effectiveness' in the Session 4 lecture notes):

```elm {v}
gapminder1 : Spec
gapminder1 =
    let
        enc =
            encoding
                << position X [ pName "income", pQuant ]
                << position Y [ pName "health", pQuant ]
    in
    toVegaLite [ gmData, enc [], circle [] ]
```

This suggests a linear scaling of income for the X position is probably not appropriate as too many values are clustered below 40,000. Equally, there is no need to scale the Y values starting at 0. So let's use a log scaling and dispense with forcing the data to use a zero origin and those grid lines.

```elm {l v}
gapminder2 : Spec
gapminder2 =
    let
        enc =
            encoding
                << position X
                    [ pName "income"
                    , pScale [ scType scLog, scNice niFalse ]
                    , pAxis [ axGrid False ]
                    ]
                << position Y
                    [ pName "health"
                    , pQuant
                    , pScale [ scZero False ]
                    , pAxis [ axGrid False ]
                    ]
    in
    toVegaLite [ width 600, gmData, enc [], circle [] ]
```

To include the population of each country in our design we should select a visual variable capable of representing quantitative values. Size would be appropriate here, scoring well for its _quantitative_ property:

```elm {l v}
gapminder3 : Spec
gapminder3 =
    let
        enc =
            encoding
                << position X
                    [ pName "income"
                    , pScale [ scType scLog, scNice niFalse ]
                    , pAxis [ axGrid False ]
                    ]
                << position Y
                    [ pName "health"
                    , pQuant
                    , pScale [ scZero False ]
                    , pAxis [ axGrid False ]
                    ]
                << size [ mName "population", mQuant ]
    in
    toVegaLite [ width 540, gmData, enc [], circle [] ]
```

We know that our area estimation of circles is flawed, so we could apply some perceptual Flannery scaling, but before we do that, referring to Munzner's 'channel effectiveness ranking' from the lecture notes, we see that 'length' (1d size) is more effective than 2d area estimation, especially from circles. So let's consider whether bars provide a better encoding:

```elm {l v}
gapminder4 : Spec
gapminder4 =
    let
        enc =
            encoding
                << position X
                    [ pName "income"
                    , pScale [ scType scLog, scNice niFalse ]
                    , pAxis [ axGrid False ]
                    ]
                << position Y
                    [ pName "health"
                    , pQuant
                    , pScale [ scZero False ]
                    , pAxis [ axGrid False ]
                    ]
                << size [ mName "population", mQuant ]
    in
    toVegaLite [ width 540, gmData, enc [], tick [ maThickness 10 ] ]
```

While these bars may allow us to make an estimation of population size more reliably than with the circles, they also have a dissociative character that interferes with our ability to place them in the income-health graphical space. So perhaps the circles work better for these data, especially as accurate population estimation is not likely to be as important here as income and health measures (we just want to get a more general sense of where the bulk of the world's population sits in this space).

Perhaps we can use colour to provide more context, using colour hue to represent nominal country names?

```elm {l v}
gapminder5 : Spec
gapminder5 =
    let
        enc =
            encoding
                << position X
                    [ pName "income"
                    , pScale [ scType scLog, scNice niFalse ]
                    , pAxis [ axGrid False ]
                    ]
                << position Y
                    [ pName "health"
                    , pQuant
                    , pScale [ scZero False ]
                    , pAxis [ axGrid False ]
                    ]
                << size [ mName "population", mQuant ]
                << color [ mName "country" ]
    in
    toVegaLite [ width 540, gmData, enc [], circle [] ]
```

Clearly there are way too many countries to encode uniquely so the legend becomes superfluous. Circles are also quite small by default, so we can enlarge them by specifying an explicit [range](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scRange) in squared pixel units, here ranging from 0 to 4,000 (diameter of 0 to 63 pixels):

```elm {l v}
gapminder6 : Spec
gapminder6 =
    let
        enc =
            encoding
                << position X
                    [ pName "income"
                    , pScale [ scType scLog, scNice niFalse ]
                    , pAxis [ axGrid False ]
                    ]
                << position Y
                    [ pName "health"
                    , pQuant
                    , pScale [ scZero False ]
                    , pAxis [ axGrid False ]
                    ]
                << size
                    [ mName "population"
                    , mQuant
                    , mScale [ scRange (raNums [ 0, 4000 ]) ]
                    , mLegend []
                    ]
                << color [ mName "country", mLegend [] ]
    in
    toVegaLite [ width 900, height 450, gmData, enc [], circle [] ]
```

Does Flannery perceptual scaling make a difference? Here is the same with Flannery scaling in place:

```elm {v l}
gapminder7 : Spec
gapminder7 =
    let
        enc =
            encoding
                << position X
                    [ pName "income"
                    , pScale [ scType scLog, scNice niFalse ]
                    , pAxis [ axGrid False ]
                    ]
                << position Y
                    [ pName "health"
                    , pQuant
                    , pScale [ scZero False ]
                    , pAxis [ axGrid False ]
                    ]
                << size
                    [ mName "population"
                    , mScale
                        [ scRange (raNums [ 0, 4000 ]), scType scPow, scExponent (1 / 0.86) ]
                    , mLegend []
                    ]
                << color [ mName "country", mLegend [] ]
    in
    toVegaLite [ width 900, height 450, gmData, enc [], circle [] ]
```

While there is some minor difference (the smaller circles are smaller still with the Flannery scaling), it is not clear in this case it helps us to assess the relationship between income and health with greater ease or insight. This is perhaps in part because the primary message is in the correlation and the precise population of each country (circle size) is of less importance.

Finally, it would be more useful to encode continent with colour hue rather than individual countries. Unfortunately we don't have a data column representing the continent to which each country belongs, but if we did, we might be able to design something similar to [GapMinder's own visualization](https://www.gapminder.org/tools/#$state$marker$axis_y$domainMin:null&domainMax:null&zoomedMin:null&zoomedMax:null&spaceRef:null;;;&chart-type=bubbles):

![Gapminder bubble chart](https://staff.city.ac.uk/~jwo/datavis2023/session04/images/gapminderExample.png)

We will see how we can incorporate geographic location in our visualizations when we consider geographic data visualization in Session 8.
