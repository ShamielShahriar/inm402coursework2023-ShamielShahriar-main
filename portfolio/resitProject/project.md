---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/project.yml

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

# Data Visualization Project Summary

{(whoami|} Shamiel Shahriar Shameem Akter shamiel.shameem-akter@city.ac.uk{|whoami)}

{(task|}

You should complete this datavis project summary document and submit it, along with any necessary supplementary files to **Moodle** by **Sunday 30th April, 5pm UK time**. Submissions will be awarded up to **80 marks** towards your coursework assessment total.

You are also encouraged to regularly commit and push changes to your datavis project throughout the term as you develop your project.

{|task)}

{(questions|}

- How does a series' length affect its viewership and score overtime?
- Do longer Arcs and Sagas do better for a series or worse?
- What do the most watched/scored episodes have in common in order to do so well?
- Did the age group watching this series change overtime or stay within the same demographic?
- Do roughly the same chapters/episodes receive similar scores or viewers?
- Can the introduction of new characters affect a series' viewership in any way?

{|questions)}

One Piece is one of the most popular long running series in the world, beating many stories and series of similar length it has now reached the Number 2 spot behind Superman as the highest selling comic book of all time even beating out Batman, which is made a more impressive feat considering the fact that it was written only by one person.

In an industry where most series tend to fail within their first Season or overall Arc (a story Arc or commonly known as a Narrative arc), understanding what made this particular series so popular in the first place can help emerging series to possibly do just as well. This carries over to its animated Series by the same name which has been running since 1999 telling the everlasting story of Luffy and his crew as they sail the seas and journey across the world of One Piece of which I’m conducting this research in order to find out more concretely how the series has been doing over the years and any of its nuances that helped it reach these peaks.
This research will be conducted using various Datasets in Kaggle as they contain a large amount of varying information about the Series One Piece and its many factors that lead to its success.


```elm {l=hidden}
jsonRatings : Data
jsonRatings =
          dataFromUrl "https://shamielshahriar.github.io/data/One%20Piece%20json.json" []
```

```elm {l=hidden}
csvRatings : Data
csvRatings =
          dataFromUrl "https://shamielshahriar.github.io/data/ONE%20PIECE.csv" [ csv ]
          
```

```elm {l=hidden}
arcs : Data
arcs =
          dataFromUrl "https://shamielshahriar.github.io/data/OnePieceArcs.csv" [ csv ]
          
```

```elm {l=hidden}
imdbRatings : Data
imdbRatings =
          dataFromUrl "https://shamielshahriar.github.io/data/One%20Piece%20IMDB%20ratings%20Ep%201-1002%20(1).csv" []
```

```elm {l=hidden}
chapterInfo : Data
chapterInfo =
            dataFromUrl "https://shamielshahriar.github.io/data/Chapters.csv" [ csv ]
```

```elm {l=hidden}
characterInfo : Data
characterInfo =
            dataFromUrl "https://shamielshahriar.github.io/data/onepiece.csv" [csv]
```

```elm {l=hidden}
arcInfo : Data
arcInfo =
            dataFromUrl "https://shamielshahriar.github.io/data/episode-list.csv" [csv]
```

```elm {l=hidden}
ratingAndArcs : Data
ratingAndArcs =
          dataFromUrl "https://shamielshahriar.github.io/data/One%20Piece%20IMDB%20ratings%20Ep%201-1002%20with%20Arcs.csv" [ csv ]
```
```elm {l=hidden}
narutoAndOnePiece : Data
narutoAndOnePiece =
          dataFromUrl "https://shamielshahriar.github.io/data/Naruto%26OnePieceRatings.csv" [ csv ]
```


{(visualization|}

**VIS 1** *(All Episodes Ranked)*
```elm {v interactive}
episodeRating : Spec
episodeRating =
  let
      trans=
        transform
          << filter (fiExpr "datum.Episode < '9'")
      enc =
          encoding
              << position X [ pName "Episode", pQuant ,pAxis [ axTitle "", axLabelFontSize 10 , axTitle "Episode Number"]]
              << position Y [ pName "IMDB Rating", pQuant ]
              << color [ mName "Year of Release", mOrdinal, mScale [ scScheme "rainbow" [], scReverse True ]]
              << tooltips
                    [ [ tName "Episode" ], [ tName "IMDB Rating" ], [ tName "Year of Release" ]]
    in
    toVegaLite [height 500, width 1500, ratingAndArcs, enc [], point [] ]

```
**VIS 2** *(Length of Arcs)*
```elm {l=hidden v interactive}
arcEpisodeCount : Spec
arcEpisodeCount =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])
        ps =
            params
                << param "mySel"
                    [ paValue (str "Romance Dawn"),
                    paBind
                      (ipSelect
                            [ inName "Arc or Saga ", inOptions
                                [ "Amazon Lily", "Anime 20th Anniversary Special", "Arabasta", "Arlong Park",

                                "Baratie", "Boss Luffy Historical Episode","Buggy's Crew Adventure Chronicles",

                                "Caesar Retrieval", "Chopper Man Special","Cidre Guild",

                                "Diary of Koby-Meppo","Dressrosa",
                                "Drum Island",

                                "Enies Lobby",

                                "Fish-Man Island", "Foxy's Return",

                                "G-8", "Goat Island",

                                "Ice Hunter", "Impel Down",

                                "Jaya",

                                "Little East Blue", "Little Garden","Loguetown", "Long Ring Long Land",
                                
                                "Marine Rookie", "Marineford",
                                
                                "Ocean's Dream", "Orange Town",

                                "Post-Arabasta", "Post-Enies Lobby","Post-War", "Punk Hazard",
                                
                                "Return to Sabaody", "Reverie",
                                "Reverse Mountain", "Romance Dawn",
                                "Ruluka Island",
                                
                                "Sabaody Archipelago", "Silver Mine", "Skypiea", "Spa Island", "Straw Hat's Separation Serial", "Syrup Village", 
                                
                                "Thriller Bark", "Toriko & Dragon Ball Crossover", "Toriko Crossover",
                                
                                "Wano Country", "Warship Island", "Water 7", "Whisky Peak", "Whole Cake Island",
                                
                                "Zou","Z's Ambition"]
                            ]),
                            paSelect sePoint [ seFields [ "arc" ] ]]
        enc =
            encoding
                << position X [ pName "Episode", pQuant, pAggregate opCount, pAxis [ axTitle "", axLabelFontSize 10 , axTitle "Number of Episodes"]]
                << position Y [ pName "arc", pAxis [ axTitle "", axLabelFontSize 10 , axTitle "Arcs"]]
                << color [ mName "arc", mScale [ scScheme "accent" []], mLegend []]
                << opacity
                    [ mCondition (prParamEmpty "mySel")
                    [ mNum 1][ mNum 0.1]]
    in
    toVegaLite [ height 750 , width 500, ratingAndArcs, ps[], enc[],cfg[], bar [maTooltip ttEncoding ] ]
```

**VIS 3** *(IMDB Ratings over the Arcs)*
```elm {v interactive}
episodeRating2 : Spec
episodeRating2 =
  let
      enc =
          encoding
              << position Y [ pName "IMDB Rating", pQuant, pAggregate opMean, pSort [soByChannel chX, soAscending ]]
              << position X [ pName "Episode",pQuant, pAxis [ axTitle "", axLabelFontSize 10 , axTitle "Episode Number"]]
              << color [ mName "arc", mScale [ scScheme "accent" [], scReverse False], mLegend[]]
              << tooltips [ [ tName "Episode" ],[ tName "IMDB Rating" ],[ tName "Year of Release" ],[ tName "arc" ]]
    in
    toVegaLite [height 750, width 3000, ratingAndArcs, enc [], area [maOpacity 0.5, maTooltip ttEncoding,maPoint (pmMarker [])] ]

```
### What do the most watched/scored episodes have in common in order to do so well?

**VIS 4** *(All Naruto Episodes Ranked)*
```elm {l=hidden v interactive}
narutoVSOnePiece : Spec
narutoVSOnePiece =
    let
        enc =
            encoding
                << position X [ pName "episode_number_overall", pQuant,pAxis [ axTitle "", axLabelFontSize 10 , axTitle "Episode Number"]]
                << position Y [ pName "rating",pQuant ,pAxis [ axTitle "", axLabelFontSize 10 , axTitle "IMDB Rating"]]
                << color [ mName "original_air_date", mScale [ scScheme "Category10" []], mLegend []]
    in
    toVegaLite [ height 300, width 1500, narutoAndOnePiece,
         enc[], point [maTooltip ttEncoding ]]
```

**VIS 5** *(IMDB Ratings over the Arcs)*
```elm {v interactive}
narutoSeasonRating : Spec
narutoSeasonRating =
  let
      enc =
          encoding
              << position Y [ pName "rating", pQuant, pAggregate opMean, pSort [soByChannel chX, soAscending ]]
              << position X [ pName "episode_number_overall",pQuant, pAxis [ axTitle "", axLabelFontSize 10 , axTitle "Episode Number"]]
              << color [ mName "season", mScale [ scScheme "accent" [], scReverse False], mLegend[]]
              << tooltips [ [ tName "episode_number_overall" ],[ tName "rating" ],[ tName "original_air_date" ],[ tName "season" ]]
    in
    toVegaLite [height 500, width 1500, narutoAndOnePiece, enc [], area [maOpacity 0.5, maTooltip ttEncoding,maPoint (pmMarker [])] ]

```
**VIS 6** *(Number of Characters Introduced each Year)*

```elm {l=hidden v interactive}
characterIntro : Spec
characterIntro =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])
        ps =
            params
                << param "mySel"
                    [ paValue (str "1997"),
                    paBind
                        (ipSelect
                            [ inName "Year "
                            , inOptions
                                [ "1997","1998","1999", "2000", "2001", "2002", "2003", "2004", "2005", "2006", "2007", "2008", "2009", "2010", "2011", "2012", "2013", "2014", "2015", "2016", "2017", "2018", "2019", "2020", "2021","2022"]
                            ]
                        )
                    , paSelect sePoint [ seFields [ "Intro_Year" ] ]
                    ]
        enc =
            encoding
                << position X [ pName "Intro_Year",pQuant, pAxis [ axTitle "Year Introduced", axGrid False ]]
                << position Y [ pName "Name", pAggregate opCount, pAxis [ axTitle "Number of Characters Introduced", axGrid True ] ]          
                << color [ mName "Intro_Year", mScale [ scScheme "reds" []], mLegend []]
                << opacity
                     [ mCondition (prParamEmpty "mySel")
                    [ mNum 1] [ mNum 0.1]]
    in
    toVegaLite [ height 300, width 500, characterInfo,
        ps[], enc[],cfg[], bar [maTooltip ttEncoding ]]
```

**VIS 7** *(Mean of Arcs)*
```elm {v interactive}
episodeRating3 : Spec
episodeRating3 =
  let
      enc =
          encoding
              << position X [ pName "arc"]
              << position Y [ pName "IMDB Rating", pQuant, pAggregate opMean]
              << size [ mName "IMDB Rating"]
              << color [ mName "arc", mScale [ scScheme "accent" [], scDomain (doMid 10), scReverse False ], mLegend []]
              << tooltips [ [ tName "Episode" ],[ tName "Year of Release" ]]

    in
    toVegaLite [height 750, width 1000, ratingAndArcs, enc [], bar [] ]

```

{|visualization)}

### How does a series' length affect its viewership and score overtime?
As VIS 1 shows, One Piece has had Critical Acclaim pretty much from the very beginning with very few dips, implying that the viewers have consistently agreed on its quality and enjoyment they received from the series. Each point in the Visualization contains the Episode, its Rating and the Year it was released which can be differentiated through the colours of the points.
With its lowest IMDB Rating being a 5.7 and 5.9 score in 2007 & 2013 respectively, and a highest score being a 9.8 score in 2021 over a 20+ year period, it can be understood that whilst One Piece might not always score highly, it has a had a fairly steady level of Quality from the beginning to today with both its viewership and score.

This means that the although the popularity of the series has not massively increased due to its length, it also did not drop possibly indicating that the overall perception to the series has been generally positive and most likely most of its viewers decided to stick with as the years went along to see how the story would unfold.


### Do longer Arcs and Sagas do better for a series or worse?
VIS 2 provides a visual representation of the length of each Arc in One Piece by the number of episodes it contained. This ranges from just 1 episode an Arc to 118 episodes which is currently the longest Arc of the story.

This alongside VIS 3 which contains an area graph sectioning each Arc along with a point along the lines to show each individual episode and their score show us that Longer Arcs do tend to do better for a series. With both VIS 2 & 3 we can see arcs such as “Dressrosa” or “Wano Country”, the longest Arcs of One Piece have some of the highest rated episodes, with many episodes reaching above a 9.0 in IMDB and some even above 9.5. Contrast to this some of the lowest performing episodes tend to be from shorter arcs like “Buggy’s Crew Adventure Chronicles” (with a 6.3 & 6.4) or single one-off episodes such as “Chopper Man Special” (with 5.7) which are some of the worst scored episodes.

This can be for a number of reasons, but more likely than not; the longer the arc is the more it has to do keep its viewers engaged in the story and the characters, especially if the Story Arc is good then the watchers will be enjoying it and scoring it more highly.


### What do the most watched/scored episodes have in common in order to do so well?
Looking through VIS 3 we can see a trend of the highest ranked episodes being in the later parts of a story arc or the Climax of the arc itself as well as episodes focusing on the main Cast. For example, ep. 24 is the first one to go above a 9.0 and it’s an important episode relating to two fan favourite characters (Zoro & Mihawk) clashing for the first time in the “Baratie Arc”, ep 37 is the next above a 9.0 and it’s beginning of the “Arlong Park” climax where the main cast faces off against the antagonist for first time.

One piece is a very character heavy series and as much as the story revolves around these characters, the characters themselves can push episodes above others. This can be seen in Episode 312 in the “Post-Enies Lobby” arc which has to do with the departure of this character and has become a fan favourite episode even if it doesn’t have to do with the climax of the story or similarly ep. 483 in “Marineford” where another important character is removed from the story has a 9.6 IMDB Score as well.

So for a long Stories such as One Piece the most common factors that have lead to its success are good character writing for viewers to get invested in as well as a balanced Story Arc that will lead to a satisfactory pay off or climax, so that the viewer doesn’t feel deceived and appreciate the journey that lead to this point. 


### Do roughly the same chapters/episodes receive similar scores or viewers?
Comparing two popular series’ Ranking such as One Piece (VIS 1) and Naruto (VIS 4) that came out roughly around the same time you can see a fairly similar overall ratings among all the episodes. However, once we take a deeper look at their Seasons/Arcs (VIS 3 & 5) we can see a bigger discrepancy between their respective highs and lows.

One Piece had a bigger focus on longer arcs and kept the viewers’ engagement through the series a lot better than Naruto, but similarly to One Piece character-based episodes did well regardless of how the arc or season did.
For example, a season revolving a fan favourite character “Jiraiya” in Season 6 (Ep 133, 134) did incredibly well even though the surrounding arcs did poorly purely due to this character being adored by fans.

Inversely Season 20 of Naruto which is by far the longest season had a fairly strong opening with high 9.0’s but quickly dipped in the in the Rankings due to what are called “fillers”, essentially side stories in the world of Naruto that do not revolve with the main story or it’s characters deeply harming the quality and score of the arc in the midpoints. However, it picked steam back up during the climax of season once the story got back on track once again hitting high 9.0’s with episode 477 even reaching the highest point of the story with a 9.7 which is an emotionally charged episode about the two main characters of the story Naruto and Sasuke.

All of this once again proves how a good narrative with a consistent pace and well written and adapted characters can ensure a good long running and successful series. It’s possible to do well with either however as it can be seen in Naruto the story or narrative can greatly suffer.

### Can the introduction of new characters affect a series' viewership in any way?
In VIS 6 we are able to see the number of Characters added to the series of One Piece each Year and if we Compare it to VIS 3, we can see that on years where a lot of characters were introduced like 2009 with 77 new characters or 2018 with 95 new characters the story did seem to get a push to do better. 

This is especially true during some of the longer arcs as the creator might have done it in order to keep up engagement during the beginning stages of an arc as it might be a little harder to the viewers to begin a long new arc. And doing this absolutely paid off since the score at the beginning and end of the arcs are pretty similar even though you’d normally expect for an Arc to ramps up as it keeps going.

## Conclusion:
In conclusion, the research that I have conducted shows that a series’ longevity can affect greatly the success of said series. In the case of One Piece the fact that it ran over 20 plus years and still doing great even now proves how a strong narrative and well-built characters ensure that people will keep watching and enjoying the series for a long time. One Piece in particular has various 100+ episode arcs and they all have done wonderful for the series where viewers keep coming back for more.

This is in contrast to other similar series that also did well such a Naruto, but never seemed to hit the consistency that One Piece even with episodes doing as well as it. It is also valid to keep in mind that One Piece did start earlier and had more time to build momentum in comparison, but this should not take away from its accomplishments.

If there’s one thing that should be taken away from this study is that keeping a long running series such as this is very difficult and takes a lot of effort to understand what your story is trying to tell, and it needs to be successful. Furthermore, knowing your audience helps tremendously to ensure that you’re able to keep them engaged with the story. 

Unfortunately i was not able to answer this question as i couldn't find a dataset that would allow me to conduct this research which is something that I will definetely take a look at if i were to do it again.

  - Did the age group watching this series change overtime or stay within the same demographic?

### Colour:
I decide not to use strong or vibrant colours in my graphs and instead use softer more pastel like colours so that they can be easily distinguished as there was a lot of data to be presented. By picking these colours I believe that it should clarify distinctions between certain points and different elements in the graphs. Also, this should make it easier for readers to comprehend the data as it is more memorable and not hurt readability as it shouldn’t strain viewers’ eyes whilst reading through it.

### Layout:
For this project I made use of various graphs to represent the different data sets in a more interesting and diverse manner. I used a scatter plot points to showcase the episodes and their ratings as I believe this makes the most sense and makes it easier to pin-point each individual episode as well as its ranking. Furthermore, this helps with the visibility of the graph and the correlation between the years and the rankings of the episodes. 
Bar charts were also used and made to be interactive so that the user could individually pick out which Year or Arc they wanted to see and help it stand out from the other bars. Moreover, Area and line graphs were used to showcase the Arcs in a more distinctive way so that when quickly looking over the data one can clearly tell where they are.
Lastly, for further readability and interactivity I added descriptions to appear whenever the user hovers above a certain point to ensure that even if the colours where hard to visualise there would be a clear indication of what one was looking at.








{(references|}

Sisodiya, A. (2021) One piece anime, Kaggle. Available at: https://www.kaggle.com/datasets/aditya2803/one-piece-anime (Accessed: April 27, 2023). 
Bogacz, M. (2022) One piece characters and chapters, Kaggle. Available at: https://www.kaggle.com/datasets/michau96/one-piece-characters-and-chapters?select=Characters.csv (Accessed: April 27, 2023). 
Putri, O. (2021) One piece, Kaggle. Available at: https://www.kaggle.com/datasets/odhiayustika/one-piece-tracklist?select=episode-list.csv (Accessed: April 27, 2023). 
Boyapally, S. (2023) One piece characters, Kaggle. Available at: https://www.kaggle.com/datasets/sonikaboyapally/one-piece-characters (Accessed: April 27, 2023). 
Natarajan, R.M. (2021) One piece anime dataset (EP 1 - EP 1002), Kaggle. Available at: https://www.kaggle.com/datasets/ranimeiyammai/one-piece-anime-dataset-ep-1-ep-1002 (Accessed: April 27, 2023). 
Sisodiya, A. (2021) One piece anime, Kaggle. Available at: https://www.kaggle.com/datasets/aditya2803/one-piece-anime (Accessed: May 7, 2023). 

{|references)}
