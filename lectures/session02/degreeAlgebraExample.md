---
id: "litvis"
narrative-schemas:
  - ../narrative-schemas/algebra
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import VegaLite exposing (..)
```

# Kindlmann and Scheidegger Visualization Algebra Schema Example

Uses a schema for considering the Kindlmann and Scheidegger (2014) algebra in visualization design.

As a reminder, we consider the transformations between data (**D**) and visualization (**V**) through an intermediate stage of data representation on computer (**R**). _**α**_ represents some transform in data and _**ω**_ some transform in visualization.

In a good visualization design we would expect:

- a _correspondence_ between **α** and **ω**: as data varies in some way we would expect to see a corresponding variation in the visualization sufficient to infer how the data varies.
- that if **α** is the identity transform (i.e. no change in data), two different data representations should yield identical visualizations (therefore **ω** is also an identity transform).
- that if **ω** is the identity transform (i.e. two similar looking visualizations), this should only arise when **α** is also an identity transform (i.e. two similar data sources).

This can be summarised by saying that the paths from the **D** in the top left corner to the **V** in the bottom right should be equivalent either by traversing right then down _(**r1** >> **v** >> **ω**)_ or down then right _(**α** >> **r2** >> **v** )_. That is, the diagram is _commutative_.

![Commutative diagram](https://staff.city.ac.uk/~jwo/datavis2023/session02/images/algebra1.png)

- Failures in the invariance principle result in _hallucinators_ (unimportant changes in **R** result in salient changes in **V**).

- Failures in the unambiguity principle result in _confusers_ (meaningful changes in **D** cannot be seen in **V**).

- Failures in the correspondence principle result in _jumblers_ (change in **D** does not result in an equivalent change in **V**) or _misleaders_ (change in **V** don't reflect equivalent change in **D**).

## Education Level Map

Suppose we wished to understand the extent to which educational attainment varies across England and Wales. We can use the 2021 census data [TSO67 - qualification](https://www.ons.gov.uk/datasets/TS067/editions/2021/versions/1) to calculate the proportion of the population with a degree or equivalent ("level 5") and then compare that value for different local authorities with respect to the national average. So a local authority with a value of 11 would mean the proportion of people with a degree is 11% points higher than the national average of 34%, and a value of -5 means 5% points lower than the national average.

^^^elm {interactive v=(educationMap Large NoChange DefaultOrder Desc)}^^^

Does the visual representation (**V**) above satisfy the principles of _invariance_, _unambiguity_ and _correspondence_?

## Principle of Representation Invariance

{( invariance |}

^^^elm {v=[(educationMap Small NoChange DefaultOrder Asc),(educationMap Small NoChange ByEasting Asc),(educationMap Small NoChange ByNorthing Asc),(educationMap Small NoChange ByMagnitude Asc),(educationMap Small NoChange ByMagnitude Desc)]}^^^

{| invariance)}

{( invarianceAssessment |}

Varying the order in which data rows are plotted shows noticeable differences, especially in the London region. Therefore **failing** this principle because it introduces _hallucinators_. This is due to considerable overlap of the circle symbols.

- [ ] passed?

{| invarianceAssessment)}

## Principle of Unambiguous Data Depiction

{( unambiguity |}

^^^elm v=[(educationMap Medium (Change 2 0) ByMagnitude Desc),(educationMap Medium (Change 0 0) ByMagnitude Desc),(educationMap Medium (Change -2 0) ByMagnitude Desc)]^^^

{| unambiguity )}

{( unambiguityAssessment |}

Systematic shifts of 2% of in the proportion of people with a degree are visually detectable as a shift to and from orange and purple. Therefore, no evidence for _confusers_ in design but we should be aware that changes of less than 2% may not be detectable visually.

- [x] passed?

{| unambiguityAssessment )}

## Principle of Visual-Data Correspondence

{( correspondence |}

^^^elm v=[(educationMap Medium (Change 0 0) ByMagnitude Desc),(educationMap Medium (Change 0 5) ByMagnitude Desc)]^^^

Adding a random 5% uniform random perturbation to the results above right, which is not a meaningful change in data, gives rise to a largely similar visualization as we would wish.

^^^elm {v=[(educationMap Medium (Change -10 5) ByMagnitude Desc),(educationMap Medium (Change -5 5) ByMagnitude Desc),(educationMap Medium (Change 0 5) ByMagnitude Desc)]}^^^

^^^elm v=[(educationMap Medium (Change 5 5) ByMagnitude Desc),(educationMap Medium (Change 10 5) ByMagnitude Desc),(educationMap Medium (Change 15 5) ByMagnitude Desc)]^^^

{| correspondence )}

{( correspondenceAssessment |}

As noted under _unambiguous data depiction_, systematic changes in value result in expected and detectable changes in colour. Most easily detectable are shifts between below (purple) and above (orange) average values. Less discernable are shifts further away from 0 (e.g. dark purple to very dark purple).

Overall there is no evidence of any _jumblers_. While not tested directly, there is no evidence to point towards _misleaders_.

- [x] passed?

{| correspondenceAssessment )}

---

```elm {l=hidden}
type OrderType
    = ByEasting
    | ByNorthing
    | ByMagnitude
    | DefaultOrder


type SortOrder
    = Asc
    | Desc


type DataChange
    = NoChange
    | Change Float Float


type MapSize
    = Small
    | Medium
    | Large


educationMap : MapSize -> DataChange -> OrderType -> SortOrder -> Spec
educationMap mapSize dChange orderType oDirection =
    let
        orderParams =
            let
                sortOrder =
                    case oDirection of
                        Asc ->
                            oSort [ soAscending ]

                        Desc ->
                            oSort [ soDescending ]
            in
            case orderType of
                DefaultOrder ->
                    [ oName "code", oQuant, sortOrder ]

                ByEasting ->
                    [ oName "easting", oQuant, sortOrder ]

                ByNorthing ->
                    [ oName "northing", oQuant, sortOrder ]

                ByMagnitude ->
                    [ oName "percDegree", oQuant, sortOrder ]

        dChangeExpr =
            case dChange of
                Change av var ->
                    "(" ++ String.fromFloat av ++ "+(random()*2-1)*" ++ String.fromFloat var ++ ")"

                NoChange ->
                    "0"

        titleText =
            let
                oText =
                    case oDirection of
                        Asc ->
                            "increasing"

                        Desc ->
                            "decreasing"

                cText pc =
                    if pc < 0 then
                        "shift of " ++ String.fromFloat (abs pc) ++ "% points below original"

                    else if pc == 0 then
                        "Original values"

                    else
                        "shift of " ++ String.fromFloat pc ++ "% points above original"
            in
            case ( mapSize, dChange, orderType ) of
                ( Large, _, _ ) ->
                    ""

                ( _, NoChange, DefaultOrder ) ->
                    oText ++ " local authority code"

                ( _, NoChange, ByEasting ) ->
                    oText ++ " easting"

                ( _, NoChange, ByNorthing ) ->
                    oText ++ " northing"

                ( _, NoChange, ByMagnitude ) ->
                    oText ++ " magnitude"

                ( _, Change av var, _ ) ->
                    if var == 0 then
                        cText av

                    else if av == 0 then
                        "Random " ++ String.fromFloat var ++ "% change"

                    else
                        "Random " ++ String.fromFloat var ++ "% change and\n" ++ cText av

        ( w, h, legend ) =
            case mapSize of
                Small ->
                    ( 160, 180, [ mLegend [] ] )

                Medium ->
                    ( 270, 300, [ mLegend [] ] )

                Large ->
                    ( 450, 500, [] )

        backgroundSpec =
            asSpec
                [ dataFromUrl "https://gicentre.github.io/data/census21/englandAndWales/countries.json"
                    [ topojsonFeature "countries" ]
                , geoshape [ maStroke "#fff", maStrokeWidth 0.1, maFill "#eee" ]
                ]

        censusData =
            dataFromUrl "https://gicentre.github.io/data/datavis/degreeLA.csv" [ parse [ ( "percDegree", foNum ) ] ]

        trans =
            transform
                << calculateAs ("(datum.percDegree + " ++ dChangeExpr ++ ") - 33.8") "diff"

        degreeEnc =
            encoding
                << position Longitude [ pName "easting" ]
                << position Latitude [ pName "northing" ]
                << color
                    ([ mName "diff"
                     , mQuant
                     , mScale [ scScheme "purpleorange" [], scDomain (doMid 0) ]
                     , mLegend [ leOrient loTopLeft, leGradientLength 150, leGradientThickness 10, leDirection moVertical, leTitle "Diff from average (% points)", leTitleLimit 300 ]
                     ]
                        ++ legend
                    )
                << order orderParams
                << tooltip [ tName "areaName" ]

        degreeSpec =
            asSpec [ censusData, trans [], degreeEnc [], circle [ maOpacity 1, maSize (w / 3), maStroke "black", maStrokeWidth 0.2 ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coScale [ sacoMaxSize w ])
                << configuration (coTitle [ ticoFontSize 10, ticoBaseline vaBottom ])
    in
    toVegaLite
        [ cfg []
        , width w
        , height h
        , title titleText []
        , projection [ prType identityProjection, prReflectY True ]
        , layer [ backgroundSpec, degreeSpec ]
        ]
```
