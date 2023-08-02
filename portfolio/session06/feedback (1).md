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

# Session 6 Feedback and Answers

## 2. Advantages and Disadvantages of Direct and Indirect Interaction

Below is an aggregation of some of the arguments for/against different forms of interaction that you provided. You can compare your own answers with these tables below and decide whether or not you agree with the rationales.

You may find this paper by [Dimara and Perin, 2020](https://hal.archives-ouvertes.fr/hal-02197062/document) useful to consider a framework for using interaction choices in visualization. Quite a few of you were pushing the limits of a 'minimal contribution' only suggesting a couple of advantages/disadvantages with little else. Please do try to engage with the exercises as they are designed to help you become better data visualisers.

### Direct Interaction

| Advantages                                             | Disadvantages                                                                                                                                  |
| :----------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| Zooming and panning more intuitive                     | Difficult to select densely positioned marks                                                                                                   |
| Can be precise with spatial selection                  | Hard to select by non-spatially encoded data                                                                                                   |
| Keeps eyes focussed on the data-driven graphical marks | May not be obvious that interaction is possible (i.e. lack of [affordances](https://www.interaction-design.org/literature/topics/affordances)) |
| User in control of view, when zooming/panning          | User may get lost in view (e.g. extreme zoom in/out) when zooming/panning                                                                      |
| Can support Shneiderman's 'detail on demand'           | Can be too easy to make wrong selection (e.g. zoom too far, select wrong mark etc.)                                                            |
| Can highlight point of interest when spatially encoded | Narrative harder to control                                                                                                                    |
| Good for touchscreen users                             | Can be easy to accidentally remove selection                                                                                                   |
| Encourages exploration and connection with data        | May require instructions if interaction not expected or intuitive                                                                              |
| Tooltips provide uncluttered detail on demand          | Less able to direct user towards a specific question/task                                                                                      |
| Increases sense of 'ownership' of the visualization    | Can be tedious to set up view in a particular way                                                                                              |

### Indirect Interaction

| Advantages                                                                                                                                | Disadvantages                                                                |
| :---------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------- |
| Can select non position-encoded data ranges                                                                                               | Can take up large amount of space (e.g. radio buttons)                       |
| GUI widget can indicate the current selection                                                                                             | Difficult for spatial selection                                              |
| [Affordances](https://www.interaction-design.org/literature/topics/affordances) of the GUI widget follow conventions and so already known | Difficult for multiple spatial selections                                    |
| Can support people with disabilities who might struggle with direct interaction                                                           | Interaction limited by available widgets                                     |
| Can scale well for larger datasets                                                                                                        | Need to move eyes between two places while interacting                       |
| Can act as a legend and interaction device (interactive legends)                                                                          | Can create visual clutter and goes against some of Tufte's design principles |
| Can combine multiple filters and selections                                                                                               | Can make user feel more separate from the data                               |
| Cursor doesn't obscure data                                                                                                               | Can make it harder to accommodate some visual impairments                    |
| Good for categorical selection                                                                                                            | Can 'distance' viewer from data (opposite of data humanism)                  |

## 3. Data Challenge: Interacting With a Larger Crime Dataset

Remember when dealing with larger datasets, it is quite likely that simply using the same approach as you would for a smaller one will not work. Perhaps because it takes too long to display all the data, and/or the visualization is too cluttered to be useful. I saw a few example submissions where the same visualizations as used in the lecture materials were applied to the larger crime dataset, often producing quite cluttered results. Think about the ways you might produce an overview, such as [aggregation](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#3-2-aggregation) and [sampling](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#3-11-data-sampling). You can then use interaction to provide zoomed and filtered views or details on demand.

Keith produced a good summary overview by showing the z-scores (i.e. how different each measurement was from the average) over time as bars. This showed both the variation across the West Midlands for any one month as well as the trend over time:

```elm {l=hidden}
crimeData : Data
crimeData =
    dataFromUrl "https://gicentre.github.io/data/westMidlands/westMidsCrimesShort.tsv" []
```

```elm {v}
selZoom : Spec
selZoom =
    let
        enc =
            encoding
                << position X [ pName "month", pTemporal, pTitle "" ]
                << position Y [ pName "zScore", pQuant ]
    in
    toVegaLite [ width 540, crimeData, enc [], bar [ maPoint (pmMarker []) ] ]
```

Given this overview, how might you introduce some interaction to show details on demand?

I saw some good examples of interaction as means of filtering, for example by crime type or region. A few filtered by numbers of reported crimes. I am less convinced this would be a useful interaction in practice but perhaps you have a use-case in mind?

I saw a few attempts to generate maps from the dataset, which is one way of aggregating (by local region such as Neighbourhood Policing Unit, NPU). This was possible even without knowledge of specialised geographic mapping techniques. The extended crime dataset included `gridX` and `gridY` of each Neighbourhood Policing Unit, which provides a mapped position.

Below is an example with an interactive legend that allows maps of different crime types to be created. How might you adapt it so that a range slider is used to filter by time also?

```elm {v interactive}
circleMap : Spec
circleMap =
    let
        interactiveLegend field =
            let
                encLegend =
                    encoding
                        -- Just encode in the Y position direction to create column of marks
                        << position Y
                            [ pName field
                            , pAxis [ axTitle "", axDomain False, axTicks False ]
                            ]
                        << color
                            [ mCondition (prParam "legendSel")
                                [ mName field
                                , mScale [ scScheme "category10" [] ]
                                , mLegend []
                                ]
                                [ mStr "lightgrey" ]
                            ]

                ps =
                    params
                        -- Project selection to all matching values in the colour channel
                        << param "legendSel" [ paSelect sePoint [ seEncodings [ chColor ] ] ]
            in
            asSpec [ ps [], encLegend [], square [ maSize 120, maOpacity 1 ] ]

        trans =
            transform
                << filter (fiSelection "legendSel")
                << joinAggregate [ opAs opSum "reportedCrimes" "crimesInArea" ]
                    [ wiGroupBy [ "gridX", "gridY" ] ]

        enc =
            encoding
                << position X [ pName "gridX", pQuant, pAxis [] ]
                << position Y [ pName "gridY", pQuant, pAxis [] ]
                << size [ mName "reportedCrimes", mQuant, mLegend [] ]
                << color [ mName "crimeType", mScale [ scScheme "category10" [] ] ]
                << tooltips
                    [ [ tName "NPU" ]
                    , [ tName "crimesInArea" ]
                    ]

        chartSpec =
            asSpec [ width 540, trans [], enc [], circle [ maInterpolate miMonotone ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite
        [ crimeData
        , cfg []
        , hConcat [ interactiveLegend "crimeType", chartSpec ]
        ]
```

Some produced filtered views by time slice, which could be useful. It raises the challenge of dealing with large datasets: Selection allows smaller subsets to be visualized, but at the cost of the 'overview'. It's worth thinking about how to show both, possibly in sequence as suggested by Shneiderman's mantra. One way of doing this would have been to include Shneiderman's _TDIA narrative schema_ (and associated [example template](https://staff.city.ac.uk/~jwo/datavis2022b/session06/tdiaExample.md)) in your document. Narrative schemas can provide useful checklists that lead you though some of the design process choices and I'd encourage you to take advantage of them. They're a bit like having a helper with you prompting you to reflect on your design choices.

In a few cases I saw a sketch / written description of what you would like to have implemented (e.g. map or time slider for temporal selection or animation over time), but were unable to implement. This is still useful as it shows you are thinking about the role interaction can play in effective visualization even if you have some technical implementation constraints.
