---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

@import "../../lectures/css/datavis.less"

```elm {l=hidden}
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 3 Feedback and Answers

## 1. Measurement Types and Colour Schemes

I hope those of you who completed the table for this question found the exercise useful in clarifying the difference between nominal, ordinal, temporal and quantitative measurement types. Some of the answers I saw suggested some of you are still finding this a little confusing. Student cohorts (students grouped by the degree they are taking), for example are categorical and nominal (one degree type is no more or less than another). Even though it is possible to count the number of students in a cohort, the categories themselves are nominal. A few suggested the cohort data are ordinal - but I would argue there is no inherent order to the categories (is 'data science' and more or less than 'cyber security'?). Don't be confused by the fact category names could be placed in alphabetical order - this is not a property of the data themselves, but simply an artifact of the names we've given categories.

A number of people were having trouble when a description of data would appear to have more than one measurement type. For example, recognising that a time series such as the FTSE 100 index over 12 months is both temporal and quantitative. This is because the description captures two variables, not one, and each can have its own type. The FTSE index of shares itself is a quantitative measure (the average share price of 100 companies). But additionally we have another variable – the dates over the last 12 months – which is temporal. In a visualization specification we would typically encode each of those two variables in its own channel (e.g. time series with `position X` and the FTSE index with `position Y`. We might additionally encode just one of them – the FTSE index – with a colour channel.

You are likely to have had some different colour scheme allocations for the five data examples below, but the most important point is that whatever scheme you selected, it should reflect the measurement type and you should be able to justify your selection.

| Data                                                                                           | Measurement type                 | Colour Scheme                                                                                                           | Justification                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------------------------------------------------------------------------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1. Student cohorts in an attendance streamgraph                                                | `Nominal`                        | [category10](https://vega.github.io/vega/docs/schemes/#category10)                                                      | Need to show cohort categories as distinct and unordered. The first 7 colours of `category10` are easily separable and have similar saturation and lightness                                                                                                                                                                                                                                                                                                                                     |
| 2. FTSE 100 indices over the last 12 months                                                    | `Quant` and `Temporal`           | [blues](https://vega.github.io/vega/docs/schemes/#blues)                                                                | We are most likely to encode just the FTSE 100 value with colour so a sequential scheme allows changes in that index to be seen. If we were to encode the temporal aspect, a similar sequential scheme would also be appropriate.                                                                                                                                                                                                                                                                |
| 3. Your assessment marks for all completed modules                                             | `Quant` (and possibly diverging) | [greys](https://vega.github.io/vega/docs/schemes/#greys) / [redgrey](https://vega.github.io/vega/docs/schemes/#redgrey) | A sequential colour scheme shows the relative magnitude of each assessment mark. Given the importance of a passmark (40%), showing a diverging colour scheme centred on the pass mark might be appropriate to highlight marks above/below that critical boundary.                                                                                                                                                                                                                                |
| 4. Political party of elected MPs                                                              | `Nominal`                        | [custom categorical](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#categoricalDomainMap)  | There are conventional colours associated with political parties (e.g. in the UK: Labour Party red, Conservative Party blue etc.), so it is probably best to create a custom [categoricalDomainMap](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#categoricalDomainMap) scheme to use this convention. Those custom colours should vary the hue for each party but attempt to keep saturation and lightness approximately similar so as not to imply any ordering. |
| 5. Responses to a 5-point [Likert](https://en.wikipedia.org/wiki/Likert_scale) survey question | `Ordinal` and probably diverging | [redyellowblue](https://vega.github.io/vega/docs/schemes/#redyellowblue)                                                | Most Likert scale survey responses pivot around a neutral midpoint, so a diverging scheme represents this well. Midpoint answers ("neither agree or disagree" are still important so the yellow of the `redyellowblue` scheme helps to distinguish it from a white or dark background.)                                                                                                                                                                                                          |

## 2. Modifying the attendance streamgraph

Here's an example of the streamgraph using the [category10](https://vega.github.io/vega/docs/schemes/#category10) colours as justified above:

```elm { l v highlight=24}
attendance : Spec
attendance =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])

        data =
            dataFromUrl "https://gicentre.github.io/data/attendance.csv"
                -- Ensure 'session' column treated as number
                [ parse [ ( "session", foNum ) ] ]

        enc =
            encoding
                << position X [ pName "session" ]
                << position Y
                    [ pName "attendance"
                    , pQuant
                    , pStack stCenter -- Stacked from the centre not bottom.
                    , pAxis []
                    ]
                << detail [ dName "id" ]
                << color [ mName "cohort", mScale [ scScheme "category10" [] ] ]
    in
    toVegaLite
        [ width 600
        , height 300
        , cfg []
        , data
        , enc []
        , area
            [ maLine (lmMarker []) -- Add lines around each area 'stream'
            , maInterpolate miMonotone -- Monotone interpolation gives curved lines
            ]
        ]
```

The only addition to the example in the lecture notes is the line that sets the [mScale](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mScale) to the `category10` [scScheme](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scScheme).

A custom scheme might improve things further as the two purple shades (PG Info Sci and UG) are quite similar and the red for PG IST dominates in comparison to its neighbours. This also results in red-green being adjacent, which may cause problems for those with the most common form of colour blindness (deuteranopia). Here is how we might use a custom scheme, using [categoricalDomainMap](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#categoricalDomainMap):

```elm { l v highlight=[9-18,35]}
attendance2 : Spec
attendance2 =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])

        customColours =
            categoricalDomainMap
                [ ( "PG Cyber Sec", "hsl(26,80%,40%)" )
                , ( "PG Data Sci", "hsl(240,50%,60%)" )
                , ( "PG HCID", "hsl(90,70%,38%)" )
                , ( "PG IST", "hsl(340,50%,58%)" )
                , ( "PG Info Sci", "hsl(0,0%,50%)" )
                , ( "PG SE", "hsl(190,80%,40%)" )
                , ( "UG", "hsl(45,90%,45%)" )
                ]

        data =
            dataFromUrl "https://gicentre.github.io/data/attendance.csv"
                -- Ensure 'session' column treated as number
                [ parse [ ( "session", foNum ) ] ]

        enc =
            encoding
                << position X [ pName "session" ]
                << position Y
                    [ pName "attendance"
                    , pQuant
                    , pStack stCenter -- Stacked from the centre not bottom.
                    , pAxis []
                    ]
                << detail [ dName "id" ]
                << color [ mName "cohort", mScale customColours ]
    in
    toVegaLite
        [ width 600
        , height 300
        , cfg []
        , data
        , enc []
        , area
            [ maLine (lmMarker []) -- Add lines around each area 'stream'
            , maInterpolate miMonotone -- Monotone interpolation gives curved lines
            ]
        ]
```

I saw a couple of examples of the [redyellowblue](https://vega.github.io/vega/docs/schemes/#redyellowblue) and other diverging schemes, that although designed for diverging data, works well in this case because contrast between adjacent cohort colours is sufficient to distinguish them. It is reasonably colour-blind friendly too in this context. This is an example of data-led design, where we might dispense with universal rules if the specific context justifies an alternative design.

```elm { v}
attendance3 : Spec
attendance3 =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])

        data =
            dataFromUrl "https://gicentre.github.io/data/attendance.csv"
                -- Ensure 'session' column treated as number
                [ parse [ ( "session", foNum ) ] ]

        enc =
            encoding
                << position X [ pName "session" ]
                << position Y
                    [ pName "attendance"
                    , pQuant
                    , pStack stCenter -- Stacked from the centre not bottom.
                    , pAxis []
                    ]
                << detail [ dName "id" ]
                << color [ mName "cohort", mScale [ scScheme "redyellowblue" [] ] ]
    in
    toVegaLite
        [ width 600
        , height 300
        , cfg []
        , data
        , enc []
        , area
            [ maLine (lmMarker []) -- Add lines around each area 'stream'
            , maInterpolate miMonotone -- Monotone interpolation gives curved lines
            ]
        ]
```

## 3. Global Temperature Data Challenge

I saw some nice examples in your repos that addressed this task, where you had shown you had thought about what kind of colour mapping to apply. Also, several of you implemented a particular colour scheme and then realised once you had done so that it wasn't as effective as you had hoped. This is fine – it shows you are learning!

Here is a description of my thought process in providing some candidate visualizations.

### Colouring bar charts

A first attempt at showing these data might be to represent the temperature anomalies as bars and colour them with a diverging scheme. Don't forget to add `mQuant` to the `Anomaly` colour encoding specification so that the legend is a continuous bar rather than discrete nominal anomaly categories.

```elm {l v}
bars1 : Spec
bars1 =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/temperatureAnomalies.json" []

        enc =
            encoding
                << position X [ pName "Date", pTemporal, pAxis [ axTitle "" ] ]
                << position Y [ pName "Anomaly", pQuant ]
                << color
                    [ mName "Anomaly"
                    , mQuant
                    , mScale [ scScheme "redblue" [] ]
                    ]
    in
    toVegaLite [ width 600, data, enc [], bar [] ]
```

There are at least two problems with the colours of this initial design. Firstly, the red-blue diverging scheme isn't centred around zero. And as many of you pointed out, this default scheme associates red with cooler than average temperatures and blue associated with warmer than average. This is somewhat counterintuitive given the common association of red with warm, blue with cold.

We can adjust the colour scaling by setting [scDomain doMid](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#doMid) to 0. This sets the midpoint of the diverging colour scheme to be at 0. And as discussed in the lecture, to flip red/blue used in the scheme we can apply [scReverse](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scReverse) to reverse the scaling:

```elm {l v highlight=[15,24]}
bars2 : Spec
bars2 =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/temperatureAnomalies.json" []

        enc =
            encoding
                << position X [ pName "Date", pTemporal, pAxis [ axTitle "" ] ]
                << position Y [ pName "Anomaly", pQuant ]
                << color
                    [ mName "Anomaly"
                    , mQuant
                    , mScale [ scScheme "redblue" [], scDomain (doMid 0), scReverse True ]
                    ]
    in
    toVegaLite [ width 600, data, enc [], bar [] ]
```

A few of you used the red-blue scheme with blue indicating hotter than baseline and red indicating cooler. Even if you were not able to flip the colour mapping, I'd encourage you to get into the habit of at least commenting in your litvis documents what your preferred design is, even if you are not sure how to implement it.

A few of you chose to use the [turbo](https://vega.github.io/vega/docs/schemes/#turbo) colour scheme to show temperature anomalies. One of the interesting properties of this scheme is that arguably, it is actually better if you have deuteranopia (common red-green colour-blindness), as it transforms it into a two-hue diverging scheme (its disadvantage is that large parts of the scheme are indistinguishable, such as between 0 and 0.7 below):

![Turbo colour scheme](https://staff.city.ac.uk/~jwo/datavis2023/session03/images/turboScheme.jpg)

A remaining issue is that the bars overlap creating a 'crowded' multi-colour appearance. This might be considered a desirable outcome in that it shows that the annual data are made up of seasonal variations. But we may wish to explore ways of removing the overplotting.

If we were to use bars, how crowded are they? There are 116 years of data with each year having 12 months of readings giving 1392 bars. Given a pixel width of 600, it would not be possible to display all bars with this layout, so we should consider alternative/additional presentations of the data.

### Coloured scatterplot

One alternative to overcome the bar crowding might be simply to change the bar marks to circle marks:

```elm {l v}
scatter : Spec
scatter =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/temperatureAnomalies.json" []

        enc =
            encoding
                << position X [ pName "Date", pTemporal, pAxis [ axTitle "" ] ]
                << position Y [ pName "Anomaly", pQuant ]
                << color
                    [ mName "Anomaly"
                    , mQuant
                    , mScale [ scScheme "redblue" [], scDomain (doMid 0), scReverse True ]
                    ]
    in
    toVegaLite [ width 600, data, enc [], circle [] ]
```

This shows the variation more clearly but de-emphasises divergence from the 0 baseline.

The spread of temperature values in each year revealed by this representation more clearly shows that there are monthly variations each year, and hints that the nature of that variation might itself change over time.

### A monthly line encoding

A few people wanted to divide the time period into groups, such as quarters or months. Here is one way of doing that:

We can show the month of the year along the x-axis and plot a line for each year's temperature. To extract the month from the date we can use the function [pTimeUnit](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pTimeUnit) supplying [month](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#month) as a parameter. We apply a similar approach for colour encoding by extracting just the year from the date using [mTimeUnit](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mTimeUnit) and [year](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#year).

Other refinements include making the overlapping lines semi-transparent with [maOpacity](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#maOpacity) and making the lines curved with [miMonotone](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#miMonotone). Both these options are specified as mark properties of `line` rather than a channel encoding as they are independent of the data used to draw each line.

```elm {v l}
linechart2 : Spec
linechart2 =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/temperatureAnomalies.json" []

        enc =
            encoding
                << position X [ pName "Date", pTemporal, pTimeUnit month ]
                << position Y [ pName "Anomaly", pQuant ]
                << color
                    [ mName "Date"
                    , mTimeUnit year
                    , mScale [ scScheme "greens" [ 0, 2 ] ]
                    ]
    in
    toVegaLite
        [ width 400
        , height 400
        , data
        , enc []
        , line [ maOpacity 0.6, maInterpolate miMonotone ]
        ]
```

The colour scheme of `greens` is justified on the grounds that we don't want to choose a colour associated with either hot or cold temperatures (remembering that the colour is now encoding the year of the measurement, not temperature). The default range of greens does not distinguish the years very well, so we can increase the range by extending the higher years (i.e. more recent) into dark green. We do this by setting the colour range to use `[0, 2]` rather than the default scaling between [0,1]. You can try experimenting with different numeric values to see the effect.

Using vertical location (of the line) to show temperature gives us much more discriminating power than colour (as the scatterplot above) and allows us to be more confident that there are not consistent monthly variations in temperature anomaly.

### A calendar view

Melody and Kate suggested we might show the data as a 'calendar view'. That is, we position measurements by month along the x-axis and by year along the y-axis, colouring marks according to their temperature anomaly. Again we extract the month and the year separately from the date with [pTimeUnit](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pTimeUnit). We can additionally remove gaps between the bar marks using [scPaddingInner](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scPaddingInner) to create a continuous so-called "heatmap":

```elm {l v}
calendarView : Spec
calendarView =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/temperatureAnomalies.json" []

        enc =
            encoding
                << position X [ pName "Date", pTimeUnit month, pTitle "" ]
                << position Y [ pName "Date", pTimeUnit year, pTitle "" ]
                << color
                    [ mName "Anomaly"
                    , mQuant
                    , mScale [ scScheme "redblue" [], scDomain (doMid 0), scReverse True ]
                    ]
    in
    toVegaLite [ width 300, height 800, data, enc [], bar [] ]
```

This shows all the data compactly and emphasises new patterns such as the period in the 1940s of above-average temperatures (why do you think this might be the case?). It also offers the possibility of considering whether there are any seasonal variations (i.e. structures from left to right).

You may have different views as to which is the better design, but the ability to explore different design possibilities will increase the likelihood of coming up with an effective design that helps you understand patterns in the data.

## 4. Black and White Design Challenge

Thanks for sharing your monochrome name sketches. I saw some creative ideas, such as mapping the results across the US (data permitting), a pictogram representation, a word cloud of names sized by frequency and a 'brick wall' of names. Please do continue to think creatively about your designs, even if you feel you would struggle to implement them in LitVis/Vega-Lite.

A few are still having problems sharing their sketch images. Remember to put a copy of the image in the same portfolio folder as your markdown document, then you can refer to it with `![my image](myImage.jpg)` (replacing `myImage.jpg` with the name of your image file). Make sure you push the image to github as well as the markdown files.

Paula and Teresa both suggested that names could be displayed radially over time. It's a theme we are likely to return to in the next few weeks, but what does a radial layout bring that a Cartesian (two axes at right angles) layout does not? And what are the challenges of using a radial layout?

I liked Emily's observation that the diversity of names appears to have increased over time, and that suggested some kind of dynamic visualization that used a time-slider to change a word cloud of names. And that further this suggested (if the data were to support it) some kind of geographical mapping of names.

Iñaki's "name train" was a creative idea, having 'carriages' in which each name is represented by person symbols in proportion to popularity. It's worth considering in designs like that, what should determine the order of the names in the train (e.g. alphabetical, frequency, or something else). Melody suggested an interesting idea of using a "school register" that nicely captures what a typical class of student names might look like for a given year. It is interesting to consider the role of metaphor in design – what does it bring and what is lost?

Many of the designs I saw continued to emphasise gender in the design. I wonder how important that actually is when contemplating names. And how do we deal with names that are gender neutral (like mine for example)? There is also an issue of scale - there are many hundreds of names so thinking about how they can all be shown is a challenge (although reasonably, we might decide it is not important to show them all).

As ever, if you are willing to share your sketched designs on Slack, you may get useful feedback from others as well as inspiration for your own designs.
