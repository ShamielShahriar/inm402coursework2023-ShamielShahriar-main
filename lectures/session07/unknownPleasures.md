---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    Herteby/simplex-noise: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Simplex
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Unknown Pleasures: Pulsar Detection and Shifted Line Charts

![Unknown Pleasures Album Cover](https://staff.city.ac.uk/~jwo/datavis2023/session07/images/albumCover.jpg)

The cover of Joy Division’s first album Unknown Pleasures, designed by Peter Saville, remains one of the all-time classic album images and one of many to feature data visualization.

For some background on the data behind the cover, see the _Scientific American_ blog post [Pop Culture Pulsar: Origin Story of Joy Division’s Unknown Pleasures Album Cover](https://blogs.scientificamerican.com/sa-visual/pop-culture-pulsar-origin-story-of-joy-division-s-unknown-pleasures-album-cover-video/).

In essence the cover is just a collection of line charts, with the vertical position of each representing the magnitude of an electromagnetic signal from a [pulsar](https://en.wikipedia.org/wiki/Pulsar) and the horizontal position representing time.

Pulsars rotate rapidly with a regular period, emitting a beam of energy. A telescope from earth can pick up that energy signal as the beam sweeps past every few seconds, much like a lighthouse. So if we were to visualize that signal over time we might see something like this:

![Pulsar Signal](https://staff.city.ac.uk/~jwo/datavis2023/session07/images/pulsarSignal.png)

The image on _Unknown Pleasures_ shows each pulse as a separate line so that time moves both left to right and top to bottom. By shifting the vertical position of each line, it is possible not only to see many pulses at once, but also how regular they are in time. This is a good example of how careful choice of _arrangement_ can add information to your visualization.

If you don’t have a radio telescope to hand, you can get Litvis to simulate the album cover using a form of random number generation called [simplex noise](https://package.elm-lang.org/packages/Herteby/simplex-noise/latest/) (see appendix for details). The example below creates 80 layers superposed from top to bottom so that the area under each line overlays any lines 'behind' it.

```elm {l v}
joyPlot : Spec
joyPlot =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        trans =
            transform
                << calculateAs "datum.energy + 10*datum.sequence" "y"
                << calculateAs "10*datum.sequence - 2" "baseline"

        enc =
            encoding
                << position X [ pName "x", pQuant, pAxis [] ]
                << position Y [ pName "y", pQuant, pAxis [] ]
                << detail [ dName "sequence" ]

        areaSpec seq =
            let
                transLayer =
                    transform
                        << filter (fiExpr ("datum.sequence == " ++ String.fromInt seq))
            in
            asSpec
                [ transLayer []
                , area
                    [ maLine (lmMarker [ maStroke "white", maStrokeWidth 0.8 ])
                    , maFill "black"
                    , maOpacity 1
                    , maInterpolate miMonotone
                    ]
                ]
    in
    toVegaLite
        [ cfg []
        , background "black"
        , padding (paEdges 150 50 110 100)
        , width 400
        , height 600
        , data []
        , trans []
        , enc []
        , layer (List.range 1 numRows |> List.reverse |> List.map areaSpec)
        ]
```

Of course you can use the same technique to show different data. For example, we could visualize the sound signal produced by one of the tracks from the Joy Division album:

![Control Joy Plot](https://staff.city.ac.uk/~jwo/datavis2023/session07/images/unknownPleasures2.png)
[Joy Division, Control](https://player.vimeo.com/video/120090681)

Perhaps more usefully, we can use this so-called _joy-plot_ to show other data where we wish to compare the timing of events by aligning lines to use a common temporal scale. Here, for example, are the top google search trends in 2020 ordered by the week of the year in which each search term peaks.

```elm {v interactive, l}
googleTrends : Spec
googleTrends =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coFacet [ facoSpacing -100 ])

        trendData =
            dataFromUrl "https://gicentre.github.io/data/googleTrends2020.csv" []

        enc =
            encoding
                << position X
                    [ pName "week"
                    , pTemporal
                    , pAxis
                        [ axGrid False
                        , axTitle ""
                        , axDomain False
                        , axLabelColor "white"
                        , axLabelExpr "monthAbbrevFormat(month(datum.value))"
                        , axLabelFont "Iosevka"
                        ]
                    ]
                << position Y [ pName "relFreq", pQuant, pAxis [], pScale [ scType scSqrt ] ]

        labelEnc =
            encoding
                << position X [ pDatum (dt [ dtYear 2020, dtMonth Dec, dtDate 23 ]) ]
                << position Y [ pDatum (num 0) ]
                << text [ tName "question" ]

        lineSpec =
            asSpec [ enc [], line [ maColor "white", maInterpolate miMonotone, maSize 1.5 ] ]

        trans =
            transform
                << filter (fiExpr "datum.qType != 'search'")

        transLabel =
            transform
                << filter (fiExpr "week(datum.week) == 1")

        labelSpec =
            asSpec
                [ transLabel []
                , labelEnc []
                , textMark [ maColor "white", maAlign haLeft, maDx 15, maFontSize 14, maFont "Iosevka" ]
                ]
    in
    toVegaLite
        [ cfg []
        , title "2020: A questionable year"
            [ tiColor "white"
            , tiFont "Roboto Slab"
            , tiFontSize 40
            , tiFontWeight fwNormal
            , tiSubtitle "Top Google search questions for every week in 2020"
            , tiSubtitleColor "white"
            , tiSubtitleFont "Iosevka"
            , tiSubtitleFontSize 16
            , tiSubtitlePadding 8
            , tiOffset 32
            ]
        , trendData
        , background "black"
        , padding (paSize 250)
        , trans []
        , columns 1
        , facetFlow
            [ fName "question"
            , fSort [ soByField "peakDate" opMin ]
            , fHeader [ hdTitle "", hdLabelFontSize 0 ]
            ]
        , specification (asSpec [ width 400, height 20, layer [ lineSpec, labelSpec ] ])
        ]
```

---

## Appendix: Data Generation Functions

Generate a list of numbers between a min and max with a given step between them:

```elm {l}
range : Float -> Float -> Float -> List Float
range mn mx step =
    let
        numItems =
            floor ((mx - mn) / step)
    in
    List.range 0 numItems |> List.map (\x -> mn + toFloat x * step)
```

Sets up the random number generation for simplex noise. Change the value given to the `permutationTableFromInt` to generate a different set of random numbers.

```elm {l}
pTable : Simplex.PermutationTable
pTable =
    Simplex.permutationTableFromInt 12345
```

Generate a random simplex noise value for any given pair of numbers x and y. Scaled to be between 0.25 and 0.75. When a pair of (x,y) values are close to another pair of (x,y) values, the two random numbers will also be quite similar.

```elm {l}
noise : Float -> Float -> Float
noise x y =
    (Simplex.noise2d pTable x y + 2) / 4
```

Now we can create the data comprising repeated sequences of noise peaking at semi-regular intervals.

```elm {l}
numRows : Int
numRows =
    80


stepSize : Float
stepSize =
    6


energyAtXY : Float -> Float -> Float
energyAtXY y t =
    -- Simulates a pulsar signal by combining simplex noise
    -- with a cubed cosine curve to peak at centre
    let
        baseline x =
            max 0 ((-4 * cos (degrees x)) ^ 3)
    in
    noise (t * 0.04) y * baseline t - (2 * abs (noise (t * 0.3) y))


noisyLine : Int -> List Float
noisyLine y =
    range 0 360 stepSize
        |> List.map (energyAtXY (toFloat y))


noisyLines : List Float
noisyLines =
    List.range 1 numRows |> List.concatMap noisyLine


data : List DataColumn -> Data
data =
    dataFromColumns []
        << dataColumn "x"
            (range 0 360 stepSize
                |> List.repeat numRows
                |> List.concat
                |> nums
            )
        << dataColumn "energy" (nums noisyLines)
        << dataColumn "sequence"
            (range 1 (toFloat numRows) 1
                |> (noisyLine 1 |> List.length |> List.repeat |> List.concatMap)
                |> List.reverse
                |> nums
            )
```
