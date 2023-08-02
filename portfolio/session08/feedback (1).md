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

# Session 8 Feedback and Answers

Completion this week was lower than usual. Perhaps it was the Easter break, perhaps you've made a choice to prioritise other assessment activity. In general most who submitted their weekly practical exercises completed the map projections table, but fewer completed the two coding questions. Do make sure you have a look at the examples below as it will be useful for many of you in your coursework to be able to construct visualizations like this (text labels, maps, interactive maps).

## 1. Labelling Spatial Features

The first question asked you to modify the car ownership proportional circle map given in the lecture so that it also displayed borough labels.

You need to ensure access to the data table containing the car ownership figures. If you had modified your lecture notes document directly, the table was already included in that page, but if you were showing the map in a separate document, you'd need to copy the table across to the document with the specification (which was `practicalExercises.md` for most of you). For clarity, I've placed the data in the Appendix of this document.

A few of you tried to add the text directly to the original london map (replacing the circle marks with [text marks](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#textMark) ). As well as no longer showing the circles, this resulted in very large text as it was being modified by the size channel. Instead you need to keep the circles and text as separate layers with the size encoding only applying to the circles.

The new layer has its own encoding as you only need to position each label (based on `cx` and `cy`) and to encode the label text (based on `FID` which is the name of each borough, not 'name' or 'shortName' which is in the geo file, not the centroid data file which you are using here. Note also that the position of each label is offset in a vertical direction by 20 pixels using [maDy](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#maDy) so it does not sit on top of the borough's proportional circle.

Once you have stored the new text layer with [asSpec](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#asSpec) you just need to add that spec to the other layers provided to [layer](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#layer).

```elm {l v highlight=[35-42,54]}
londonCarOwnership : Spec
londonCarOwnership =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        carData =
            dataFromColumns []
                << dataColumn "borough" (strColumn "borough" carOwnershipTable |> strs)
                << dataColumn "carOwnership" (numColumn "carOwnership" carOwnershipTable |> nums)

        centroidData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonCentroids.csv"

        backgroundSpec =
            asSpec [ geoData, geoshape [ maFill "#ddd", maStroke "white" ] ]

        trans =
            transform
                << lookup "FID" (carData []) "borough" (luFields [ "carOwnership" ])

        enc =
            encoding
                << position Longitude [ pName "cx" ]
                << position Latitude [ pName "cy" ]
                << size
                    [ mName "carOwnership"
                    , mScale [ scRange (raNums [ 0, 1000 ]), scType scPow, scExponent (1 / 0.86) ]
                    ]

        circleSpec =
            asSpec [ centroidData [], trans [], enc [], circle [ maOpacity 0.5 ] ]

        labelEnc =
            encoding
                << position Longitude [ pName "cx" ]
                << position Latitude [ pName "cy" ]
                << text [ tName "FID" ]

        labelSpec =
            asSpec [ centroidData [], trans [], labelEnc [], textMark [ maDy 20, maFontSize 8 ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset 0 ])
    in
    toVegaLite
        [ width 640
        , height 480
        , cfg []
        , carData []
        , layer [ backgroundSpec, circleSpec, labelSpec ]
        ]
```

### 2. Map Projections

There were plenty of projection choices available for this question and no objectively best answers (although there were clearly some projections less suitable than others). Make sure you consider which properties of the map projection are most important for the task at hand so you can provide a justification for your choice.

I saw some answers that just named the map projection type but said nothing about the rotation parameters and a few that just gave generic answers like "yes" or "3d". I was after a named map projection and in most cases some projection parameters to customise the style of projection. Remember you can change these to centre a projection on the area of interest. Good to see not too many of you were suggesting _Mercator_ as a choice for global views. I would argue that while the Mercator is a well-known projection, it is very rarely a suitable one for global data. This is largely because it greatly exaggerates the area of high latitudes (i.e. towards the poles). Look particularly at Antarctica and Greenland; are they important to the map and what proportion of the mapping area do they take up? Be careful not to select a projection [simply because it is well-known](https://youtu.be/vVX-PrBRtTY).

For the ice sheet task, several of you suggested a rotation to make the south pole the centre of the map. Note the question asked for **arctic** not **antarctic** ice. As in the live session, it pays to read the question carefully!

To focus your rationales, note that 'equal area' type projections attempt to make the graphic area of the projected countries approximately proportional to their relative geographic areas. 'Equidistant' projections attempt to keep some graphical distances (usually from a specific point or line) proportional to their relative geographic distance from a point or line. 'Conformal' projections attempt to preserve the shapes of features (although not all can be preserved).

Whether distances, areas or shape should be preserved will depend on the task at hand (and hence this exercise). The interactive map projection visualization I included in the lecture notes is helpful here in exploring those properties. I've added centre and zoom scale sliders for examples focussing on a particular region of the globe.

```elm {v interactive}
projections : Spec
projections =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/world-110m.json"
                [ topojsonFeature "countries1" ]

        ps =
            params
                << param "projection"
                    [ paValue (str "EqualEarth")
                    , paBind
                        (ipSelect
                            [ inName "prType "
                            , inOptions
                                [ "Albers"
                                , "AlbersUsa"
                                , "AzimuthalEqualArea"
                                , "AzimuthalEquidistant"
                                , "ConicConformal"
                                , "ConicEqualArea"
                                , "ConicEquidistant"
                                , "EqualEarth"
                                , "Equirectangular"
                                , "Gnomonic"
                                , "Mercator"
                                , "NaturalEarth1"
                                , "Orthographic"
                                , "Stereographic"
                                , "TransverseMercator"
                                ]
                            ]
                        )
                    ]
                << param "rotLambda" [ paValue (num 0), paBind (ipRange [ inName "prRotate λ", inMin -180, inMax 180, inStep 1 ]) ]
                << param "rotPhi" [ paValue (num 0), paBind (ipRange [ inName "prRotate 𝜙", inMin -90, inMax 90, inStep 1 ]) ]
                << param "scale" [ paValue (num 100), paBind (ipRange [ inName "prScale", inMin 100, inMax 1000, inStep 1 ]) ]
                << param "cenLambda" [ paValue (num 0), paBind (ipRange [ inName "prCenter λ", inMin -180, inMax 180, inStep 1 ]) ]
                << param "cenPhi" [ paValue (num 0), paBind (ipRange [ inName "prCenter 𝜙", inMin -90, inMax 90, inStep 5 ]) ]

        proj =
            projection
                [ prType (prExpr "projection")
                , prRotateExpr "rotLambda" "rotPhi" "0"
                , prCenterExpr "cenLambda" "cenPhi"
                , prNumExpr "scale" prScale
                ]

        sphereSpec =
            asSpec
                [ sphere
                , geoshape [ maFill "rgb(204,206,186)", maStroke "black", maStrokeOpacity 0.3 ]
                ]

        gratSpec =
            asSpec
                [ graticule [ grStep ( 10, 10 ) ]
                , geoshape [ maStroke "grey", maStrokeWidth 0.2 ]
                ]

        landSpec =
            asSpec
                [ geoData
                , geoshape
                    [ maFill "rgb(235,219,181)"
                    , maStroke "rgb(216,148,112)"
                    , maStrokeWidth 0.2
                    ]
                ]
    in
    toVegaLite
        [ width 640
        , height 320
        , ps []
        , proj
        , layer [ sphereSpec, landSpec, gratSpec ]
        ]
```

Here is one possible set of choices and justifications.

| Task                                                                    | Projection                                                                  | Justification                                                                                                                                                                                                                                                                                                                             |
| :---------------------------------------------------------------------- | :-------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Global distribution of bad teeth per capita                             | Equal Earth; default rotation and scaling.                                  | For global data focussed on continental landmasses, this is a good general projection that balances equal-area and shape                                                                                                                                                                                                                  |
| Arctic ice sheet fluctuation over the last 50 years                     | Conic equal area with prRotate 𝜙 set to -90                                 | Need an equal area projection as we are considering area of ice sheets. Rotation by -90 centres projection on North Pole                                                                                                                                                                                                                  |
| Inter-country migration flows                                           | Azimuthal Equidistant with rotate λ of -20, rotate 𝜙 of -25                 | Allows most intercontinental flows to be shown without crossing boundary of map (e..g. Japan to/from West coast USA). Shape not too distorted.                                                                                                                                                                                            |
| Bird migration patterns from the Netherlands to Europe, Asia and Africa | Azimuthal Equidistant with rotate λ of -5, rotate 𝜙 of -55                  | Projection centred approximately on the Netherlands with distances preserved from this location will make judging line length of bird migrations more reliable.                                                                                                                                                                           |
| Crime patterns in the West Midlands                                     | transverseMercator with rotate λ of 2 or Identity projection                | The Ordnance Survey national grid, common for UK data is very similar to a transverse Mercator projection with rotate λ of 2, so this would work well. Alternatively it is quite possible that such data data would already be in Ordnance Survey National Grid projection so using an identity projection would preserve that.           |
| Global incidences of Coronavirus                                        | Equal Earth; default rotation and scaling.                                  | For global data focussed on continental landmasses, this is a good general projection that balances equal-area and shape preservation. Given there have been some important successes in controlling the virus in the far East and New Zealand, there is an argument for rotating λ to -150 so that region becomes the centre of the map. |
| Major ocean current flows                                               | Azimuthal equal area with rotate 𝜙 of 90 and another with rotate phi of -90 | Most ocean currents are broadly along lines of latitude rather than pole-to-pole, so possible to split two polar projections without breaking flow lines. Projections enlarge oceanic space at cost of (less important) continental landmass.                                                                                             |

### 3. Global Data Geospatial Data Challenge

Not so many attempted this exercise, and for some who did, remember you need to transfer the data tables as well as the visualization to the document.

Apologies, but I made a mistake in providing you with the original VisionCarto atlas file for world countries. That dataset had countries identified by their [three letter code](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3), but the GapMinder datasets tended to use the [two letter code](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2). As such linking the geographic and thematic data was harder than I intended. This was, at least in some cases, the reason why some of you were getting blank maps when trying to join the two data sources.

I have provided a version of the atlas dataset with both the alpha-3 and alpha-2 country codes (`properties.a3` and `properties.a2`), which I use in the worked example below using the democracy data from Session 5, leading you through the stages to build a map and ultimately one with some interaction allowing a more useful comparison over space and time.

To keep everything in this in one place, the data tables have been moved to the appendix of this document. As a reminder, here is the original visualization that simply plotted circles at the longitude and latitude coordinates of the dataset:

```elm {l}
data2011 : List DataColumn -> Data
data2011 =
    dataFromColumns []
        << dataColumn "demIndex" (numColumn "demIndex" demGeoTable2011 |> nums)
        << dataColumn "lon" (numColumn "longitude" demGeoTable2011 |> nums)
        << dataColumn "lat" (numColumn "latitude" demGeoTable2011 |> nums)
        << dataColumn "CountryCode" (strColumn "CountryCode" demGeoTable2011 |> strs)
```

```elm {l v}
democracyMap1 : Spec
democracyMap1 =
    let
        enc =
            encoding
                << position Longitude [ pName "lon" ]
                << position Latitude [ pName "lat" ]
                << color [ mName "demIndex", mQuant, mScale [ scScheme "blueorange" [] ] ]
    in
    toVegaLite [ width 700, height 300, data2011 [], enc [], circle [ maSize 180 ] ]
```

In preparation for adding a background map, we can move the circles into their own layer. And we can ensure we are using a suitable map projection, such as EqualEarth:

```elm {l v highlight=[4-5,13-14,16]}
democracyMap2 : Spec
democracyMap2 =
    let
        proj =
            projection [ prType equalEarth ]

        enc =
            encoding
                << position Longitude [ pName "lon" ]
                << position Latitude [ pName "lat" ]
                << color [ mName "demIndex", mQuant, mScale [ scScheme "blueorange" [] ] ]

        circleSpec =
            asSpec [ enc [], circle [ maSize 180 ] ]
    in
    toVegaLite [ width 700, height 380, data2011 [], proj, layer [ circleSpec ] ]
```

We can add a global background map using the topojson file [visionscarto-world-atlas/world/110m.json](https://cdn.jsdelivr.net/npm/visionscarto-world-atlas/world/110m.json) as indicated in the lecture notes. It then becomes a simple task of displaying that map in its own layer:

```elm {l v highlight=[4-6,20-21,23]}
democracyMap3 : Spec
democracyMap3 =
    let
        geoData =
            dataFromUrl "https://cdn.jsdelivr.net/npm/visionscarto-world-atlas/world/110m.json"
                [ topojsonFeature "countries" ]

        proj =
            projection [ prType equalEarth ]

        enc =
            encoding
                << position Longitude [ pName "lon" ]
                << position Latitude [ pName "lat" ]
                << color [ mName "demIndex", mQuant, mScale [ scScheme "blueorange" [] ] ]

        circleSpec =
            asSpec [ enc [], circle [ maSize 100 ] ]

        mapSpec =
            asSpec [ geoData, geoshape [ maFill "#ddd" ] ]
    in
    toVegaLite [ width 700, height 350, data2011 [], proj, layer [ mapSpec, circleSpec ] ]
```

Does this work as a map? Unlike the London map in task 1, we have much greater variation in density of points, with Europe and central America looking rather cluttered. Perhaps a choropleth map would work more effectively, where we colour each country's polygon by the democracy index.

Remember that to create a choropleth we need to link the geographic features (countries) with the data values (democracy index), using some kind of lookup that relates the two data sources. In this case, the global map data have a property `Alpha-2` (see Table in **geo/topo JSON data sources** in the lecture notes) that corresponds to the `CountryCode` in the democracy table:

```elm {l v highlight=[13,27]}
demChoropleth : Spec
demChoropleth =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/datavis/world110m.json"
                [ topojsonFeature "countries" ]

        proj =
            projection [ prType equalEarth ]

        trans =
            transform
                << lookup "properties.a2" (data2011 []) "CountryCode" (luFields [ "demIndex" ])

        enc =
            encoding
                << color
                    [ mName "demIndex"
                    , mQuant
                    , mLegend [ leOrient loBottomLeft, leTitle "Democracy Index" ]
                    , mScale [ scScheme "blueorange" [] ]
                    ]
    in
    toVegaLite
        [ width 700
        , height 350
        , geoData
        , proj
        , trans []
        , enc []
        , geoshape [ maStroke "black", maStrokeOpacity 0.2 ]
        ]
```

You may recall from the lecture that there are two approaches to linking geo- and attribute data with [lookup](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookup). In the example above, the primary dataset was the map (`geoData` above) and the secondary data contained the attributes such as the democracy index. The secondary data are linked to the primary with the `lookup` function.

Alternatively we use the second approach and make the attribute data the primary data source and link the map data to it as a secondary source. Here is how the same choropleth map would be specified if you did this. Note the differences in the highlighted lines of code.

```elm {l v highlight=[13,17,28]}
demChoropleth2 : Spec
demChoropleth2 =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/datavis/world110m.json"
                [ topojsonFeature "countries" ]

        proj =
            projection [ prType equalEarth ]

        trans =
            transform
                << lookup "CountryCode" geoData "properties.a2" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "demIndex"
                    , mQuant
                    , mLegend [ leOrient loBottomLeft, leTitle "Democracy Index" ]
                    , mScale [ scScheme "blueorange" [] ]
                    ]
    in
    toVegaLite
        [ width 700
        , height 350
        , data2011 []
        , proj
        , trans []
        , enc []
        , geoshape [ maStroke "black", maStrokeOpacity 0.2 ]
        ]
```

Just like the small multiples of London in the main lecture notes, this allows us to apply filtering and selection on our table that we would not be able to apply if our primary dataset was the geo-boundaries. Here, for example, is an interactive version that allows a time slider to choose the year democracy data to show:

```elm {l}
dataAllYears : List DataColumn -> Data
dataAllYears =
    dataFromColumns []
        << dataColumn "demIndex" (numColumn "demIndex" demGeoTableByYear |> nums)
        << dataColumn "lon" (numColumn "longitude" demGeoTableByYear |> nums)
        << dataColumn "lat" (numColumn "latitude" demGeoTableByYear |> nums)
        << dataColumn "year" (numColumn "year" demGeoTableByYear |> nums)
        << dataColumn "CountryCode" (strColumn "CountryCode" demGeoTableByYear |> strs)
```

```elm {l v interactive }
demChoroplethTime : Spec
demChoroplethTime =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/datavis/world110m.json"
                [ topojsonFeature "countries" ]

        proj =
            projection [ prType equalEarth ]

        ps =
            params
                << param "timeSlider"
                    [ paValue (dataObject [ ( "year", num 1980 ) ])
                    , paSelect sePoint []
                    , paBind (ipRange [ inName "Year", inMin 1980, inMax 2011, inStep 1 ])
                    ]

        trans =
            transform
                << lookup "CountryCode" geoData "properties.a2" (luAs "geo")
                << filter (fiSelection "timeSlider")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "demIndex"
                    , mQuant
                    , mLegend [ leOrient loBottomLeft, leTitle "Democracy Index" ]
                    , mScale [ scScheme "blueorange" [] ]
                    ]
    in
    toVegaLite
        [ width 700
        , height 350
        , dataAllYears []
        , proj
        , ps []
        , trans []
        , enc []
        , geoshape [ maStroke "black", maStrokeOpacity 0.2 ]
        ]
```

For those looking at data that vary over both space and time ("spatio-temporal"), this might be a useful approach to experiment with in your designs.

---

### Appendices

#### Car Ownership Data

^^^elm{m=(tableSummary 6 carOwnershipTable)}^^^

```elm {l=hidden}
carOwnershipTable : Table
carOwnershipTable =
    """borough, carOwnership
       Barking and Dagenham,60.4
       Barnet,71.3
       Bexley,76.3
       Brent,57.0
       Bromley,76.5
       Camden,38.9
       City of London,30.6
       Croydon,66.5
       Ealing,64.7
       Enfield,67.5
       Greenwich,58.0
       Hackney, 35.4
       Hammersmith and Fulham,44.8
       Haringey,48.2
       Harrow,76.5
       Havering,77.0
       Hillingdon,77.3
       Hounslow,68.4
       Islington,35.3
       Kensington and Chelsea,44.0
       Kingston upon Thames,74.9
       Lambeth,42.2
       Lewisham,51.9
       Merton,67.4
       Newham,47.9
       Redbridge,72.1
       Richmond upon Thames,75.3
       Southwark,41.6
       Sutton,76.6
       Tower Hamlets,37.0
       Waltham Forest,58.1
       Wandsworth,54.7
       Westminster,37.1"""
        |> fromCSV
```

#### Democracy Data

```elm {l}
demGeoTable2011 : Table
demGeoTable2011 =
    let
        demTable =
            demTableFull
                |> filterColumns ((<=) "2011")
                |> filterRows "2011" ((/=) "")
                |> renameColumn "2011" "demIndex"
    in
    innerJoin "CountryName" ( demTable, "country" ) ( locationLookup, "CountryName" )
```

^^^elm{m=(tableSummary 6 demGeoTable2011)}^^^

```elm {l}
demGeoTableByYear : Table
demGeoTableByYear =
    let
        demTable =
            demTableFull
                |> filterColumns ((<=) "1980")
    in
    innerJoin "CountryName" ( demTable, "country" ) ( locationLookup, "CountryName" )
        |> gather "year"
            "demIndex"
            (List.map (\y -> ( String.fromInt y, String.fromInt y ))
                (List.range 1980 2011)
            )
```

^^^elm{m=(tableSummary 6 demGeoTableByYear)}^^^

```elm {l=hidden}
locationLookup : Table
locationLookup =
    """CountryName,CountryCode,latitude,longitude
Afghanistan,AF,33.93911,67.709953
Albania,AL,41.153332,20.168331
Algeria,DZ,28.033886,1.659626
American Samoa,AS,-14.270972,-170.132217
Andorra,AD,42.546245,1.601554
Angola,AO,-11.202692,17.873887
Anguilla,AI,18.220554,-63.068615
Antarctica,AQ,-75.250973,-0.071389
Antigua and Barbuda,AG,17.060816,-61.796428
Argentina,AR,-38.416097,-63.616672
Armenia,AM,40.069099,45.038189
Aruba,AW,12.52111,-69.968338
Australia,AU,-25.274398,133.775136
Austria,AT,47.516231,14.550072
Azerbaijan,AZ,40.143105,47.576927
Bahamas,BS,25.03428,-77.39628
Bahrain,BH,25.930414,50.637772
Bangladesh,BD,23.684994,90.356331
Barbados,BB,13.193887,-59.543198
Belarus,BY,53.709807,27.953389
Belgium,BE,50.503887,4.469936
Belize,BZ,17.189877,-88.49765
Benin,BJ,9.30769,2.315834
Bermuda,BM,32.321384,-64.75737
Bhutan,BT,27.514162,90.433601
Bolivia,BO,-16.290154,-63.588653
Bosnia and Herzegovina,BA,43.915886,17.679076
Botswana,BW,-22.328474,24.684866
Bouvet Island,BV,-54.423199,3.413194
Brazil,BR,-14.235004,-51.92528
British Indian Ocean Territory,IO,-6.343194,71.876519
Brunei,BN,4.535277,114.727669
Bulgaria,BG,42.733883,25.48583
Burkina Faso,BF,12.238333,-1.561593
Burundi,BI,-3.373056,29.918886
Cambodia,KH,12.565679,104.990963
Cameroon,CM,7.369722,12.354722
Canada,CA,56.130366,-106.346771
Cape Verde,CV,16.002082,-24.013197
Cayman Islands,KY,19.513469,-80.566956
Central African Republic,CF,6.611111,20.939444
Chad,TD,15.454166,18.732207
Chile,CL,-35.675147,-71.542969
China,CN,35.86166,104.195397
Christmas Island,CX,-10.447525,105.690449
Cocos (Keeling) Islands,CC,-12.14,96.88
Colombia,CO,4.570868,-74.297333
Comoros,KM,-11.875001,43.872219
"Congo, Republic",CG,-0.228021,15.827659
"Congo, Rep.",CG,-0.228021,15.827659
"Congo, Dem. Rep.",CD,-0.228021,15.827659
Cook Islands,CK,-21.236736,-159.777671
Costa Rica,CR,9.748917,-83.753428
Cote D'Ivoire,CI,7.539989,-5.54708
Cote d'Ivoire,CI,7.539989,-5.54708
Croatia,HR,45.1,15.2
Cuba,CU,21.521757,-77.781167
Cyprus,CY,35.126413,33.429859
Czech Republic,CZ,49.817492,15.472962
Denmark,DK,56.26392,9.501785
Djibouti,DJ,11.825138,42.590275
Dominica,DM,15.414999,-61.370976
Dominican Republic,DO,18.735693,-70.162651
Ecuador,EC,-1.831239,-78.183406
Egypt,EG,26.820553,30.802498
El Salvador,SV,13.794185,-88.89653
Equatorial Guinea,GQ,1.650801,10.267895
Eritrea,ER,15.179384,39.782334
Estonia,EE,58.595272,25.013607
Ethiopia,ET,9.145,40.489673
Falkland Islands (Malvinas),FK,-51.796253,-59.523613
Faroe Islands,FO,61.892635,-6.911806
Fiji,FJ,-16.578193,179.414413
Finland,FI,61.92411,25.748151
France,FR,46.227638,2.213749
French Guiana,GF,3.933889,-53.125782
French Polynesia,PF,-17.679742,-149.406843
French Southern Territories,TF,-43,67
Gabon,GA,-0.803689,11.609444
Gambia,GM,13.443182,-15.310139
Georgia,GE,32.960500,83.1132
Germany,DE,51.165691,10.451526
Ghana,GH,7.946527,-1.023194
Gibraltar,GI,36.137741,-5.345374
Greece,GR,39.074208,21.824312
Greenland,GL,71.706936,-42.604303
Grenada,GD,12.262776,-61.604171
Guadeloupe,GP,16.995971,-62.067641
Guam,GU,13.444304,144.793731
Guatemala,GT,15.783471,-90.230759
Guernsey,GG,49.465691,-2.585278
Guinea,GN,9.945587,-9.696645
Guinea-Bissau,GW,11.803749,-15.180413
Guyana,GY,4.860416,-58.93018
Haiti,HT,18.971187,-72.285215
Heard Island and McDonald Islands,HM,-53.08181,73.504158
Holy See (Vatican City State),VA,41.911,12.452
Honduras,HN,15.199999,-86.241905
"Hong Kong, China",HK,22.396428,114.109497
Hungary,HU,47.162494,19.503304
Iceland,IS,64.963051,-19.020835
India,IN,20.593684,78.96288
Indonesia,ID,-0.789275,113.921327
Iran,IR,32.427908,53.688046
Iraq,IQ,33.223191,43.679291
Ireland,IE,53.41291,-8.24389
Isle of Man,IM,54.236107,-4.548056
Israel,IL,31.046051,34.851612
Italy,IT,41.87194,12.56738
Jamaica,JM,18.109581,-77.297508
Japan,JP,36.204824,138.252924
Jersey,JE,49.214439,-2.13125
Jordan,JO,30.585164,36.238414
Kazakhstan,KZ,48.019573,66.923684
Kenya,KE,-0.023559,37.906193
Kiribati,KI,-3.370417,-168.734039
"Korea, Dem. Republic",KP,40.0,127.0
North Korea,KP,40.0,127.0
"Korea, Republic",KR,37.55,126.9667
South Korea,KR,37.55,126.9667
Kuwait,KW,29.31166,47.481766
Kyrgyzstan,KG,41.20438,74.766098
Kyrgyz Republic,KG,41.20438,74.766098
Laos,LA,19.85627,102.495496
Lao,LA,19.85627,102.495496
Latvia,LV,56.879635,24.603189
Lebanon,LB,33.854721,35.862285
Lesotho,LS,-29.609988,28.233608
Liberia,LR,6.428055,-9.429499
Libya,LY,25,17
Liechtenstein,LI,47.166,9.555373
Lithuania,LT,55.169438,23.881275
Luxembourg,LU,49.815273,6.129583
"Macao, China",MO,22.198745,113.543873
"Macedonia, FYR",MK,41.608635,21.745275
Madagascar,MG,-18.766947,46.869107
Malawi,MW,-13.254308,34.301525
Malaysia,MY,4.210484,101.975766
Maldives,MV,3.202778,73.22068
Mali,ML,17.570692,-3.996166
Malta,MT,35.937496,14.375416
Marshall Islands,MH,7.131474,171.184478
Martinique,MQ,14.641528,-61.024174
Mauritania,MR,21.00789,-10.940835
Mauritius,MU,-20.348404,57.552152
Mayotte,YT,-12.8275,45.166244
Mexico,MX,23.634501,-102.552784
"Micronesia, Fed. Sts.",FM,7.425554,150.550812
Moldova,MD,47.411631,28.369885
Monaco,MC,43.750298,7.412841
Mongolia,MN,46.862496,103.846656
Montenegro,ME,42.708678,19.37439
Montserrat,MS,16.742498,-62.187366
Morocco,MA,31.791702,-7.09262
Mozambique,MZ,-18.665695,35.529562
Myanmar,MM,21.913965,95.956223
Namibia,NA,-22.95764,18.49041
Nauru,NR,-0.522778,166.931503
Nepal,NP,28.394857,84.124008
Netherlands,NL,52.132633,5.291266
Netherlands Antilles,AN,12.226079,-69.060087
New Caledonia,NC,-20.904305,165.618042
New Zealand,NZ,-40.900557,174.885971
Nicaragua,NI,12.865416,-85.207229
Niger,NE,17.607789,8.081666
Nigeria,NG,9.081999,8.675277
Niue,NU,-19.054445,-169.867233
Norfolk Island,NF,-29.040835,167.954712
Northern Mariana Islands,MP,17.33083,145.38469
Norway,NO,60.472024,8.468946
Oman,OM,21.512583,55.923255
Pakistan,PK,30.375321,69.345116
Palau,PW,7.51498,134.58252
"Palestinian Territory, Occupied",PS,42.094445,17.266614
Panama,PA,8.537981,-80.782127
Papua New Guinea,PG,-6.314993,143.95555
Paraguay,PY,-23.442503,-58.443832
Peru,PE,-9.189967,-75.015152
Philippines,PH,12.879721,121.774017
Pitcairn,PN,-24.703615,-127.439308
Poland,PL,51.919438,19.145136
Portugal,PT,39.399872,-8.224454
Puerto Rico,PR,18.220833,-66.590149
Qatar,QA,25.354826,51.183884
Reunion,RE,-21.115141,55.536384
Romania,RO,45.943161,24.96676
Russia,RU,61.52401,105.318756
Rwanda,RW,-1.940278,29.873888
Saint BarthŽlemy,BL,17.9,-62.833
"Saint Helena, Ascension and Tristan da Cunha",SH,-24.143474,-10.030696
Saint Kitts and Nevis,KN,17.357822,-62.782998
Saint Lucia,LC,13.909444,-60.978893
Saint Martin (French part),MF,43.589046,5.885031
Saint Pierre and Miquelon,PM,46.941936,-56.27111
Saint Vincent and the Grenadines,VC,12.984305,-61.287228
Samoa,WS,-13.759029,-172.104629
San Marino,SM,43.94236,12.457777
Sao Tome and Principe,ST,0.18636,6.613081
Saudi Arabia,SA,23.885942,45.079162
Senegal,SN,14.497401,-14.452362
Serbia,RS,44.016521,21.005859
Serbia and Montenegro,RS,44.016521,21.005859
Seychelles,SC,-4.679574,55.491977
Sierra Leone,SL,8.460555,-11.779889
Singapore,SG,1.352083,103.819836
Slovak republic,SK,48.669026,19.699024
Slovak Republic,SK,48.669026,19.699024
Slovenia,SI,46.151241,14.995463
Solomon Islands,SB,-9.64571,160.156194
Somalia,SO,5.152149,46.199616
South Africa,ZA,-30.559482,22.937506
South Georgia and the South Sandwich Islands,GS,-54.429579,-36.587909
Spain,ES,40.463667,-3.74922
Sri Lanka,LK,7.873054,80.771797
Sudan,SD,12.862807,30.217636
Suriname,SR,3.919305,-56.027783
Svalbard and Jan Mayen,SJ,77.553604,23.670272
Swaziland,SZ,-26.522503,31.465866
Sweden,SE,60.128161,18.643501
Switzerland,CH,46.818188,8.227512
Syria,SY,34.802075,38.996815
"Taiwan, Province of China",TW,23.69781,120.960515
Tajikistan,TJ,38.861034,71.276093
Tanzania,TZ,-6.369028,34.888822
Thailand,TH,15.870032,100.992541
Timor-Leste,TL,-8.874217,125.727539
Togo,TG,8.619543,0.824782
Tokelau,TK,-8.967363,-171.855881
Tonga,TO,-21.178986,-175.198242
Trinidad and Tobago,TT,10.691803,-61.222503
Tunisia,TN,33.886917,9.537499
Turkey,TR,38.963745,35.243322
Turkmenistan,TM,38.969719,59.556278
Turks and Caicos Islands,TC,21.694025,-71.797928
Tuvalu,TV,-7.109535,177.64933
Uganda,UG,1.373333,32.290275
Ukraine,UA,48.379433,31.16558
United Arab Emirates,AE,23.424076,53.847818
United Kingdom,GB,52.378051,-1.435973
United States,US,37.09024,-95.712891
United States Minor Outlying Islands,UM,24.747346,-167.594906
Uruguay,UY,-32.522779,-55.765835
Uzbekistan,UZ,41.377491,64.585262
Vanuatu,VU,-15.376706,166.959158
Venezuela,VE,6.42375,-66.58973
Vietnam,VN,14.058324,108.277199
"Virgin Islands, British",VG,18.335765,-64.896335
Virgin Islands (U.S.),VI,18.335765,-64.896335
Wallis and Futuna,WF,-13.768752,-177.156097
West Bank and Gaza,PS,32.33,35
Western Sahara,EH,24.215527,-12.885834
Yemen,YE,15.552727,48.516388
Zambia,ZM,-13.133897,27.849332
Zimbabwe,ZW,-19.015438,29.154857"""
        |> fromCSV
```

```elm {l=hidden}
demTableFull : Table
demTableFull =
    """country,1800,1801,1802,1803,1804,1805,1806,1807,1808,1809,1810,1811,1812,1813,1814,1815,1816,1817,1818,1819,1820,1821,1822,1823,1824,1825,1826,1827,1828,1829,1830,1831,1832,1833,1834,1835,1836,1837,1838,1839,1840,1841,1842,1843,1844,1845,1846,1847,1848,1849,1850,1851,1852,1853,1854,1855,1856,1857,1858,1859,1860,1861,1862,1863,1864,1865,1866,1867,1868,1869,1870,1871,1872,1873,1874,1875,1876,1877,1878,1879,1880,1881,1882,1883,1884,1885,1886,1887,1888,1889,1890,1891,1892,1893,1894,1895,1896,1897,1898,1899,1900,1901,1902,1903,1904,1905,1906,1907,1908,1909,1910,1911,1912,1913,1914,1915,1916,1917,1918,1919,1920,1921,1922,1923,1924,1925,1926,1927,1928,1929,1930,1931,1932,1933,1934,1935,1936,1937,1938,1939,1940,1941,1942,1943,1944,1945,1946,1947,1948,1949,1950,1951,1952,1953,1954,1955,1956,1957,1958,1959,1960,1961,1962,1963,1964,1965,1966,1967,1968,1969,1970,1971,1972,1973,1974,1975,1976,1977,1978,1979,1980,1981,1982,1983,1984,1985,1986,1987,1988,1989,1990,1991,1992,1993,1994,1995,1996,1997,1998,1999,2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011
Afghanistan,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-8,-8,-8,0,0,0,0,-7,-7,-7,-7,-7,,,,,,,,,,,
Albania,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-2,0,0,0,0,0,0,0,0,0,0,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,0,0,0,0,0,0,-5,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,1,1,5,5,5,5,0,5,5,5,5,5,7,7,7,9,9,9,9,9,9,9
Algeria,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-2,-2,-2,-7,-7,-7,-3,-3,-3,-3,-3,-3,-3,-3,-3,2,2,2,2,2,2,2,2
Angola,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-3,0,-1,-1,-2,-2,-3,-3,-3,-3,-3,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2
Argentina,,,,,,,,,,,,,,,,,,,,,,,,,,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-4,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,-8,-8,-8,-8,-8,-8,-8,5,5,5,5,5,5,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-6,-3,-1,-1,-1,-1,-1,-1,-1,-1,-1,-9,-9,-9,-9,-9,-9,-9,6,6,6,-9,-9,-9,-9,-9,-8,-8,8,8,8,8,8,8,7,7,7,7,7,7,7,7,7,7,8,8,8,8,8,8,8,8,8,8,8,8,8
Armenia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,7,7,7,7,3,-6,-6,5,5,5,5,5,5,5,5,5,5,5,5,5,5
Australia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Austria,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,0,4,8,8,8,8,8,8,8,8,8,8,8,8,8,-1,-9,-9,-9,-9,0,0,0,0,0,0,0,5,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Azerbaijan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,-3,1,-3,-3,-6,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Bahrain,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-10,-10,-7,-7,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-9,-9,-9,-9,-9,-9,-9,-9,-8,-7,-7,-7,-7,-7,-7,-7,-7,-5,-8
Bangladesh,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,-2,-7,-7,-7,-4,-4,-4,-4,-7,-7,-7,-7,-5,-5,-5,-5,-5,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,-6,-6,5,5,5
Belarus,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,7,7,7,7,0,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Belgium,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,7,7,7,7,7,7,9,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,-10,-10,-10,-10,-10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,8,8,8,8,8
Benin,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,2,2,2,-1,-4,-7,-7,-7,-7,-7,-2,-2,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,7,7,7,7,7,7
Bhutan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-6,-6,-6,3,3,3,3
Bolivia,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-7,-7,-7,-7,-7,-7,-7,-6,-4,-3,-3,-3,-7,-7,-7,-7,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-4,-4,-4,-4,-3,-3,-3,-3,-3,-3,-3,-3,-4,-4,-4,-4,-4,-4,-5,-7,-7,-7,-7,-7,-7,-7,-4,-4,-7,-7,8,8,8,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,8,8,8,8,8,8,7,7,7
Bosnia and Herzegovina,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,0,0,0,0,0,-10,-10,-10,-10,-10,-10,-9,-9,-5,-2,2,2,0,0,0,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,0,0,0,,,,,,,,,,,,,,,,,
Botswana,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,7,7,7,7,7,7,7,7,7,7,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8
Brazil,,,,,,,,,,,,,,,,,,,,,,,,,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-4,-5,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,7,5,5,5,5,5,5,5,5,5,5,5,6,6,6,5,5,3,-3,-9,-9,-9,-9,-9,-9,-9,-9,-9,-4,-4,-4,-4,-4,-4,-4,-4,-3,-3,-3,7,7,7,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8
Bulgaria,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-4,-4,-6,-6,-1,-1,-1,-5,-5,-5,-5,-5,-5,-5,-5,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,2,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-7,-10,-10,-10,-10,-10,-10,-10,-10,0,-2,-4,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,8,8,8,8,8,8,8,8,8,8,8,9,9,9,9,9,9,9,9,9,9,9
Burkina Faso,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,-4,-4,-4,-4,-4,-4,1,5,5,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-4,-4,-4,-3,0,0,0,0,0,0,0,0,0,0,0
Burundi,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,-3,-3,-5,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-3,0,0,0,-5,-5,-1,-1,-1,0,2,3,5,6,6,6,6,6,6,6
Cambodia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-6,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-6,-5,-5,-5,0,-7,-7,-7,-6,-6,-5,-4,-4,-3,-2,-1,-1,0,0,1,1,1,1,1,1,1,-7,2,2,2,2,2,2,2,2,2,2,2,2,2,2
Cameroon,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-6,-6,-6,-6,-6,-6,-7,-7,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-7,-8,-8,-8,-8,-8,-8,-8,-8,-8,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4
Canada,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Cape Verde,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-3,-3,-3,-3,-3,-2,8,8,8,8,8,8,8,8,8,8,10,10,10,10,10,10,10,10,10,10,10
Central African Republic,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,5,5,5,5,5,5,5,5,5,5,-1,-1,-1,-1,-1,-1,-1,-1,-1
Chad,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-3,0,0,0,0,0,-4,-7,-7,-7,-7,-7,-7,-5,-4,-4,-4,-4,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2
Chile,,,,,,,,,,,,,,,,,,,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,3,3,3,3,3,3,3,3,3,3,3,3,3,3,5,5,5,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,0,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,5,5,5,5,5,5,5,5,5,6,6,6,6,6,6,6,6,6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-6,-1,8,8,8,8,8,8,8,8,8,8,8,9,9,9,9,9,9,10,10,10,10,10,10
China,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-2,2,-2,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-10,-10,-10,-10,-10,-10,-10,-10,-10,-5,-5,-5,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-9,-9,-9,-8,-8,-8,-8,-8,-8,-8,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Colombia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,0,-5,-5,-5,-5,-5,-5,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,0,0,0,0,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,-5,-5,-5,-5,-5,-5,-5,-5,-5,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,9,9,9,9,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7
Comoros,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,5,-4,-4,-5,-5,-5,-5,-6,-6,-6,-7,-7,-7,-7,-7,4,4,4,4,4,0,4,4,4,-2,-1,0,4,4,6,6,9,9,9,9,9,9
"Congo, Dem. Rep.",,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,-3,-6,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-8,0,0,0,0,0,0,0,0,0,0,0,1,3,4,5,5,5,5,5,5
"Congo, Rep.",,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-1,5,5,5,5,5,-6,-6,-6,-6,-5,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4
Costa Rica,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,1,1,1,1,1,1,1,1,1,1,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,5,5,5,5,5,5,5,5,6,6,6,6,6,6,6,6,7,7,7,7,7,7,7,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Cote d'Ivoire,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-6,-6,-6,-6,-6,-1,4,4,0,0,0,0,0,0,0,0,0,0
Croatia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,0,0,0,0,0,-10,-10,-10,-10,-10,-10,-9,-9,-5,-2,2,2,0,0,0,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-3,-3,-3,-3,-5,-5,-5,-5,1,8,8,8,8,8,9,9,9,9,9,9,9
Cuba,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,1,1,1,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,0,-3,-6,-9,-9,-9,-9,0,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Cyprus,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,8,0,0,0,0,0,7,7,7,7,7,7,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Czech Republic,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,0,0,0,0,0,0,10,10,2,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,8,8,8,10,10,10,10,10,10,10,10,10,10,10,10,10,8,8,8,8,8,8
Denmark,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,-1,-1,-1,-1,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-2,-1,0,1,2,3,3,4,4,5,6,7,8,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,-10,-10,-10,-10,-10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Djibouti,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-7,-7,-7,-7,-6,-6,-6,2,2,2,2,2,2,2,2,2,2,2,2,2
Dominican Republic,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,0,0,0,0,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-5,-7,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,0,8,0,0,0,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,5,5,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8
Ecuador,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,2,2,2,2,2,2,2,2,2,2,2,2,2,-1,-1,-1,-1,-1,-1,-1,5,5,0,0,-5,-5,-5,-5,-5,-5,-5,9,9,9,9,9,8,8,8,8,9,9,9,9,9,9,9,9,9,8,9,9,6,6,6,6,6,6,7,5,5,5,5,5
Egypt,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,4,4,4,1,-3,-6,-6,-6,-6,-2,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-3,-3,-3,-3,-3,-3,-2
El Salvador,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-8,-8,-8,-7,-7,-6,-6,-6,-6,-6,-6,-5,-5,-5,-5,-3,-3,-3,-3,0,0,0,0,0,0,0,0,-1,-1,-1,-1,-1,-6,-6,-4,-2,0,2,4,6,6,6,6,6,6,6,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,8,8,8
Equatorial Guinea,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,2,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5
Eritrea,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-6,-6,-6,-6,-6,-6,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Estonia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,10,10,10,10,10,10,10,10,10,10,10,10,10,10,6,2,-2,-6,-6,-6,-6,-6,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,6,6,6,6,6,6,6,6,7,9,9,9,9,9,9,9,9,9,9,9,9
Ethiopia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,-5,-5,-5,-5,-5,-5,-10,-10,-10,-10,-10,-10,-5,-5,-5,-5,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,0,-7,-7,-7,-7,-7,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-8,0,0,0.5,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
Fiji,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,-3,-3,-3,5,5,5,5,5,5,5,5,5,6,5,5,5,5,6,6,-3,-4,-4,-4,-4,-4
Finland,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,10,10,10,10,10,10,10,10,10,10,10,7,4,4,4,4,4,4,4,4,4,4,4,4,4,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
France,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,6,6,6,-1,-8,-8,-8,-8,-8,-8,-8,-8,-8,-7,-6,-6,-6,-6,-6,-6,-6,-3,-2,-1,1,2,3,5,6,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,9,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10,-9,-9,-9,-9,-3,4,10,10,10,10,10,10,10,10,10,10,10,10,5,5,5,5,5,5,5,5,5,5,5,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9
Gabon,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-6,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,3,3,3
Gambia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,7,7,7,7,7,7,7,7,7,8,8,8,8,-7,-7,-6,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5
Georgia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,4,4,4,4,5,5,5,5,5,5,5,5,5,7,7,7,6,6,6,6,6
Germany,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-6,-5,-5,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,2,2,2,2,2,2,2,2,2,4,6,6,6,6,6,6,6,6,6,6,6,6,6,6,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-5,-1,2,6,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Ghana,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-8,-8,-9,-9,-9,-9,-7,-7,-7,-2,3,3,-7,-7,-7,-7,-7,-7,0,6,6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-4,-1,-1,-1,-1,2,2,2,2,2,6,6,6,8,8,8,8,8,8,8,8
Greece,,,,,,,,,,,,,,,,,,,,,,,,,,,,-5,-5,-5,-4,-4,-4,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,0,3,7,7,7,7,7,7,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,1,1,1,1,1,1,1,0,-2,-3,-6,10,10,10,10,10,10,10,10,8,8,-8,-8,-8,-8,-8,0,0,0,8,8,8,8,8,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,-7,-7,-7,-7,-7,-7,-7,1,8,8,8,8,8,8,8,8,8,8,8,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Guatemala,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-1,-1,-1,-1,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-2,-2,-1,-1,-1,-5,-5,-5,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,-3,-3,2,2,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,2,2,2,2,2,2,2,2,2,2,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,5,5,5,5,5,5,2,2,2,2,-6,-6,-6,-6,-5,-5,-5,-5,-5,-5,-5,-5,3,3,3,3,1,1,1,1,-3,-3,-3,-3,-5,-5,-5,-5,-7,-7,-6,-1,3,3,3,3,3,3,3,3,3,3,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8
Guinea,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,5,5
Guinea-Bissau,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-8,-6,-6,-6,5,5,5,5,0,3,5,5,5,-1,-1,6,6,6,6,6,6,6
Guyana,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,2,2,1,1,1,1,1,1,1,1,1,1,0,0,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6
Haiti,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,0,0,0,0,0,0,0,0,0,0,0,-1,-2,-3,-4,-5,-5,-5,-5,-5,-5,-5,-5,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-10,-10,-10,-10,-10,-10,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-8,-7,-6,7,-7,-7,-7,7,7,7,7,7,2,-2,-2,-2,-2,0,3,5,5,5,5,0,0
Holy See,,,,,,,,,,,,,,,,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Honduras,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,-3,-3,-3,-3,-3,-3,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,5,5,5,5,5,5,5,5,5,5,0,0,0,0,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,0,-1,-1,-1,-1,-1,-1,-1,-1,1,4,6,6,6,5,5,5,5,6,6,6,6,6,6,6,6,6,6,7,7,7,7,7,7,7,7,7,7,7,7,7
Hungary,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-6,-7,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-10,-2,-4,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-2,4,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
India,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,7,7,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9
Indonesia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-5,2,2,3,3,0,0,0,0,0,0,0,-1,-1,-5,-5,-5,-5,-5,-5,-5,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,6,6,6,6,6,8,8,8,8,8,8,8,8
Iran,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-1,-3,-4,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-4,-7,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,0,-2,-4,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,3,3,3,3,3,3,3,-6,-6,-6,-6,-6,-7,-7,-7
Iraq,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-3,-3,-3,-3,-3,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-6,-5,-3,-1,0,1,3,3
Ireland,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,8,8,8,8,10,10,10,10,10,10,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Israel,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10
Italy,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-2,-3,-4,-6,-7,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,0,0,2,5,8,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Jamaica,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9
Japan,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-9,-8,-7,-6,-5,-4,-3,-2,-1,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,2,3,4,5,7,8,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Jordan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-10,-10,-10,-10,-10,-4,-1,-1,-1,-1,-1,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-9,-9,-9,-9,-9,-4,-4,-4,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-3,-3,-3,-3,-3
Kazakhstan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,-3,-3,-3,-3,-4,-4,-4,-4,-4,-4,-4,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6
Kenya,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,2,2,2,0,0,0,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-2,-2,-2,-2,-2,8,8,8,8,8,7,7,7,8,8
Kuwait,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-8,-8,-9,-9,-9,-9,-9,-9,-8,-8,-8,-8,-8,-10,-10,-10,-10,-10,-8,-8,-8,-8,-8,-10,-10,-10,-10,-10,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Kyrgyz Republic,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,3,4,3,3,1,4,7
Lao,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,2,4,6,8,8,-1,0,0,0,0,0,0,0,0,0,0,0,0,-2,-5,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Latvia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,7,7,7,7,7,7,7,7,7,7,7,7,7,7,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8
Lebanon,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,5,5,5,5,5,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,7,7,7,7,7,7,7
Lesotho,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,9,9,9,9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,8,8,8,8,8,0,2,4,6,8,8,8,8,8,8,8,8,8,8
Liberia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,-3,-3,-3,-3,-3,-3,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-7,-7,-7,-7,-6,-6,-6,-6,-6,-6,0,0,0,0,0,0,0,0,0,0,0,0,0,1,3,5,6,6,6,6,6,6
Libya,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0
Lithuania,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,4,4,4,4,4,0,-4,-8,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
"Macedonia, FYR",,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,0,0,0,0,0,-10,-10,-10,-10,-10,-10,-9,-9,-5,-2,2,2,0,0,0,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,6,6,6,6,6,6,6,6,6,6,6,9,9,9,9,9,9,9,9,9,9
Madagascar,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-3,-3,-3,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,2,9,9,9,9,9,8,7,7,7,7,7,7,7,7,7,7,7,0,0,0
Malawi,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,6,6,6,6,6,6,6,4,4,5,6,6,6,6,6,6,6,6
Malaysia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,10,10,10,10,10,10,10,10,10,10,10,10,1,1,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,3,3,3,3,3,3,3,3,3,3,3,3,3,6,6,6,6
Mali,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,7,7,7,7,7,6,6,6,6,6,7,7,7,7,7,7,7,7,7,7
Mauritania,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-4,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-5,-3,4,-5,-2,-2,-2
Mauritius,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,9,9,9,9,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Mexico,,,,,,,,,,,,,,,,,,,,,,,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-2,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-5,-5,-5,-5,-5,-5,-5,-5,-5,-6,-7,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,0,0,0,0,0,0,-1,-1,-1,-1,-1,-1,-1,-3,-3,-3,-3,-3,-3,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,0,0,0,0,0,0,4,4,4,6,6,6,8,8,8,8,8,8,8,8,8,8,8,8
Moldova,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,5,5,7,7,7,7,7,7,7,7,8,8,8,8,8,8,8,8,8,8,8
Mongolia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,2,2,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Montenegro,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,0,0,0,0,0,-10,-10,-10,-10,-10,-10,-9,-9,-5,-2,2,2,0,0,0,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-7,-7,-7,-7,-6,-6,-6,7,7,7,6,6,6,9,9,9,9,9,9
Morocco,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-5,-5,-5,-5,-5,-4,-4,-3,-3,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-4
Mozambique,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-7,-7,-7,-7,-7,-6,-6,-6,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5
Myanmar,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,8,8,8,8,8,8,8,8,8,8,8,8,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-6,-6,-8,-8,-8,-8,-8,-8,-8,-8,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-8,-8,-8,-8,-6,-6,-6,-3
Namibia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6
Nepal,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-5,-5,-5,-5,-7,-7,-7,-7,-7,-7,-4,-1,2,-10,-10,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-2,-2,-2,-2,-2,-2,-2,-2,-2,5,5,5,5,5,5,5,5,5,6,6,6,-6,-6,-6,-6,6,6,6,6,6,6
Netherlands,,,,,,,,,,,,,,,,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-4,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,-10,-10,-10,-10,-10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
New Zealand,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,4,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Nicaragua,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,0,0,-5,-5,-5,-1,-1,-1,-1,-1,-1,6,6,6,6,6,8,8,8,8,8,8,8,8,8,8,8,8,9,9,9,9,9
Niger,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,1,8,8,8,8,-6,-6,-6,5,5,5,5,5,6,6,6,6,6,-3,3,6
Nigeria,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,8,8,7,7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,7,7,7,7,7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-7,-7,-6,-6,-6,-1,4,4,4,4,4,4,4,4,4,4,4,4,4
North Korea,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10
Norway,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,-10,-10,-10,-10,-10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Oman,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8
Pakistan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-4,2,4,4,5,5,5,5,5,8,8,-7,-7,-7,-7,1,1,1,1,1,1,1,1,0,0,4,8,8,8,8,-7,-7,-7,-7,-7,-7,-7,-7,-4,-4,-4,8,8,8,8,8,8,8,8,8,7,7,-6,-6,-6,-5,-5,-5,-5,-5,2,5,5,6,6
Panama,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-1,-1,-1,-1,-1,-1,4,4,4,4,4,4,4,4,4,4,4,4,4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-5,-5,-6,-6,-6,-8,-8,8,8,8,8,8,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9
Papua New Guinea,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4
Paraguay,,,,,,,,,,,,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-6,-3,-3,-3,-3,-3,-3,-3,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-7,2,2,2,-9,-9,-9,-9,-9,-9,-9,-5,-5,-5,-5,-5,-5,-5,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,2,2,2,7,7,7,7,7,7,6,7,7,7,7,8,8,8,8,8,8,8,8,8
Peru,,,,,,,,,,,,,,,,,,,,,,-3,-5,-5,-3,-1,1,3,5,5,5,5,5,5,5,-3,-3,-3,-3,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-3,-3,-3,-3,-3,-3,-3,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,-4,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-6,-3,0,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,-6,-6,-2,-2,-2,-2,-2,-2,5,5,5,5,5,5,-6,5,5,5,5,5,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-2,3,7,7,7,7,7,7,7,7,7,7,8,8,-3,1,1,1,1,1,1,1,5,9,9,9,9,9,9,9,9,9,9,9
Philippines,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,5,5,5,5,5,5,-10,-10,-10,2,2,2,2,2,2,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,2,2,2,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-7,-6,-6,-6,1,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8
Poland,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,8,8,8,8,8,8,-3,-3,-3,-3,-3,-3,-3,-3,-3,-6,-6,-6,-6,0,0,0,0,0,0,-2,-5,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-8,-8,-7,-7,-7,-7,-6,-6,5,5,8,8,8,8,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10
Portugal,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-4,-4,-4,1,1,1,1,1,1,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-6,-6,-6,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-3,-3,-3,-3,-3,-3,-1,-9,2,2,5,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,4,1,-3,-6,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-3,3,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Qatar,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10
Romania,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,0,0,-2,-2,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-2,5,5,5,5,5,5,8,8,8,8,8,8,8,8,9,9,9,9,9,9,9,9
Russia,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-8,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-1,-1,-1,-1,-1,-1,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,0,5,3,3,3,3,3,3,3,6,6,6,6,6,6,6,4,4,4,4,4
Rwanda,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-6,-6,-4,-4,-4,-3,-3,-3,-3,-3,-3,-3,-4,-4
Saudi Arabia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10
Senegal,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-1,-1,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-6,-2,-2,-2,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,8,8,8,8,8,8,8,7,7,7,7,7
Serbia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,-9,-9,-4,0,0,0,0,0,0,0,-3,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,4,4,4,4,4,4,4,4,4,4,4,4,-10,-10,4,5,6,7,0,0,0,0,0,0,0,0,-10,-10,-10,-10,-10,-10,-9,-9,-5,-2,2,2,0,0,0,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-7,-7,-7,-7,-6,-6,-6,7,7,7,6,6,6,8,8,8,8,8,8
Sierra Leone,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,6,6,6,6,6,6,-7,1,1,1,-6,-6,-6,-6,-6,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-7,-7,-7,-7,4,0,0,0,0,2,5,5,5,5,5,7,7,7,7,7
Singapore,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,7,7,7,7,4,1,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2
Slovak Republic,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,0,0,0,0,0,0,10,10,2,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,8,8,8,7,7,7,7,7,9,9,9,9,9,9,9,9,10,10,10,10,10,10
Slovenia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,0,0,0,0,0,-10,-10,-10,-10,-10,-10,-9,-9,-5,-2,2,2,0,0,0,-4,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Solomon Islands,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,7,7,7,7,7,7,7,7,7,7,7,7,8,8,8,8,8,8,8,8,8,8,0,0,0,4,8,8,8,8,8,8,8,8
Somalia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,7,7,7,7,7,7,7,7,7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
South Africa,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,5,5,6,8,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9
South Korea,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,-3,-4,-4,-4,-4,-4,-4,-4,-4,8,-7,-7,3,3,3,3,3,3,3,3,3,-9,-8,-8,-8,-8,-8,-8,-8,-8,-5,-5,-5,-5,-5,-5,1,6,6,6,6,6,6,6,6,6,6,8,8,8,8,8,8,8,8,8,8,8,8,8,8
South Sudan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0
Spain,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-9,-9,-9,-9,-9,-9,-4,-4,-4,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-3,-1,-1,-1,-1,-1,-1,-1,-1,-2,-2,-2,-2,-2,-2,-2,-5,-5,-5,-5,-5,-5,-5,-6,-6,-6,-6,-6,-6,-6,-6,-6,-4,-2,-1,1,1,-5,-4,-2,-1,-1,-1,4,4,4,4,4,4,4,4,4,4,4,5,5,5,5,5,4,4,4,4,4,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,6,-7,-7,-6,-6,-6,-6,-6,1,7,7,7,7,7,7,7,7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-3,1,5,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Sri Lanka,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,8,8,8,8,8,8,8,8,6,6,6,6,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,6,6,5,5,5,6,6,6,6,4,4
Sudan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,-7,-7,-7,-7,-7,-7,0,7,7,7,7,2,-2,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,0,7,7,7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-6,-6,-4,-4,-4,-4,-4,-2,-2
Suriname,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,5,5,5,5,5,-1,-1,-6,-6,-6,-6,-6,-1,-1,-1,2,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5
Swaziland,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,0,0,0,0,0,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9
Sweden,-10,-10,-10,-10,-10,-10,-10,-10,-10,-9,-9,-9,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-3,-2,0,1,2,4,5,6,8,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Switzerland,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Syria,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,5,5,5,5,5,-7,2,-7,-7,-7,7,7,7,7,-7,-7,-7,-2,-2,-7,-7,-7,-7,-7,-7,-7,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Tajikistan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,-2,-6,-6,-6,-6,-6,-5,-1,-1,-1,-1,-1,-3,-3,-3,-3,-3,-3,-3,-3,-3
Tanzania,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-5,-5,-5,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
Thailand,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-8,-6,-4,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-6,-6,-6,-3,-3,-3,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-2,2,2,-7,-7,-2,3,3,-7,-2,2,2,2,2,2,2,2,2,2,2,3,3,3,-1,9,9,9,9,9,9,9,9,9,9,9,9,9,9,-5,-1,4,4,4,7
Timor-Leste,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,6,6,6,6,7,7,7,7,7,7
Togo,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-6,-6,-6,-6,-6,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-5,-3,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-4,-4,-4,-4,-4,-2,-2
Trinidad and Tobago,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,9,9,9,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Tunisia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-8,-8,-8,-8,-8,-8,-5,-5,-5,-5,-5,-5,-3,-3,-3,-3,-3,-3,-3,-3,-3,-4,-4,-4,-4,-4,-4,-4,-4,-4,
Turkey,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-4,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-10,-4,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,0,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,7,7,7,7,7,7,7,7,4,4,4,4,4,4,7,9,9,9,9,8,8,8,8,8,8,-2,-2,9,9,9,9,9,9,9,-5,-5,-5,7,7,7,7,7,7,9,9,9,9,8,8,8,8,7,7,7,7,7,7,7,7,7,7,7,7,7,7,9
Turkmenistan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9
Uganda,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,7,7,7,7,0,-6,-6,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,3,3,3,3,3,0,-7,-7,-7,-7,-7,-7,-7,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-1,-1,-1,-1,-1,-1,-1
Ukraine,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,6,6,5,7,7,7,7,7,7,6,6,6,6,6,6,7,7,7,7,6,6
United Arab Emirates,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8,-8
United Kingdom,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,7,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
United States,4,4,4,4,4,4,4,4,4,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,10,10,10,10,10,9,9,9,9,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Uruguay,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,0,0,0,0,0,0,2,2,2,2,2,2,2,2,2,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,3,-3,-8,-8,-8,-8,-8,-7,-7,-7,-7,-7,-7,-7,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10
Uzbekistan,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-8,-8,-8,-8,-8,-8,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-6,-4,0,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9
Venezuela,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-5,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-4,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-7,-6,-5,-5,-5,-5,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,-3,6,6,6,6,6,6,6,6,6,6,7,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,8,8,8,8,8,8,8,7,7,6,6,6,6,6,5,5,5,-3,-3,-3
Vietnam,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7,-7
Yemen,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,-5,-4,-3,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2,-2
Zambia,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,2,2,2,2,0,0,0,0,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,-9,6,6,6,6,6,1,1,1,1,1,5,5,5,5,5,5,5,7,7,7,7
Zimbabwe,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,4,4,4,4,4,4,4,4,4,4,4,4,4,1,1,1,1,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-6,-3,-3,-4,-4,-4,-4,-4,-4,-4,-4,1,1,1
"""
        |> fromCSV
```
