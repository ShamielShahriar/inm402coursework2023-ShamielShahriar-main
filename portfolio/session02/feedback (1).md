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


data =
    dataFromUrl "https://vega.github.io/vega-lite/data/movies.json" []
```

<!-- Everything above this line should probably be left untouched. -->

# Session 2 Feedback and Answers

## 1. Evaluating Design Approaches

It was great to see some thoughtful answers to the various evaluation tasks (Jonathan Corum's lecture, Chart Junk etc.). I also saw some evidence of wider reading, such as [Lee-Robins and Adar (2022)](https://arxiv.org/pdf/2208.04078.pdf), which was great to see. Please do read more widely than just my lecture notes as gaining multiple perspectives on design and data visualization research will enrich your own views. Thinking about what motivates design will help you improve your own designs as you create them in litvis and other packages.

There is an interesting tension between datavis as explicitly "storytelling" (largely the approach advocated by Corum as a journalist) and more abstract presentations of data (for example some of the Dear Data visualizations). There are still stories being told in the latter but a little like visual art, those stories can sometimes require greater thought and interpretation by the reader. As a designer it is useful to ask yourselves how much should you lead the reader through your design and how much should you expect them to find their own way. Striking the right balance will often depend on context and the aims of your design.

A few of you provided only a very small token answer to the practical exercises (e.g. copying over the Brexit Tufte histogram but not using it). Please in future weeks make a good effort to complete all the tasks to your best ability. They are designed to help you learn and assess your own progress.

I wanted to share this comment from Kate on the Sara Weber's train delay scarf that I thought captured some of the essecnce of data feminism and data humanism:

> Similarly, data feminism argues that data should tell untold stories about minoritized groups of people, people with less power. And the best way to do that is actually involve these groups of people in the creation of how data is used and visualized. Being a commuter at the mercy of delayed transportation can feel quite powerless -- filing a complaint to those who run the system through some arduous, opaque process isn't likely to make much of a change. But if you're able to tell your story through data in a way that's true to you and how you think about the world, you can actually communicate your experience. For the scarf creator, knitting was something they could do to communicate -- and it garnered a lot of attention.

## 2. Creating a Minimalist Tufte Visualization

- One or two people we getting into difficulties in setting up the headers for the two litvis documents. When one or more documents _follows_ another, you only need to specify the elm dependencies once, and usually in the 'top' document (i.e. the one that is followed by the others). So in this case you can create a root document called [imdbRoot.md](https://staff.city.ac.uk/~jwo/datavis2023/session02/imdbRoot.md) that sets up the elm dependencies as well as a link to the IMDB dataset.

  The root document therefore has the following header:

  ```txt
  ---
  elm:
    dependencies:
      gicentre/elm-vegalite: latest
  ---
  ```

  And the document that follows it just has a simple header

  ```txt
  ---
  follows: imdbRoot
  ---
  ```

  (My examples also have an `id: litvis` line, which isn't strictly necessary, but is used as an identifier for the css styling in `datavis.less`.)

- It would help if you could assemble your answers to the weekly exercises in the `exercises.md` document provided for you in the portfolio folder. That saves me having to search for files containing your work. I'd also like to encourage you to think of litvis documents as _narratives_ that contain an explanation of your thinking, not just a collection of code files.

- Quite a few of you retained the long colour legend in your Tufte makeovers. I am not convinced the legend is necessary and certainly not one with so many entries.

- Many people added a midpoint line to their IMDB histogram just as I did for the Brexit example in the lecture notes. I would argue this is less relevant for IMDB where the midpoint on the y-axis is less important than it was for a simple majority vote like the EU referendum where 50% has a significant meaning. And remember, with percentages we can be reasonably confident that the full domain of values is between 0 and 100%, but for movie frequency we don't know the upper limit, so a single midpoint label may carry less information.

My [tufteHistogram.md](https://staff.city.ac.uk/~jwo/datavis2023/session02/tufteHistogram.md) is one way we can create (and explain) a more minimalist design. Below is the content of that file:

---

### A Tufte Makeover of a default Frequency Histogram

For reference, here is the original histogram:

```elm {l v}
histogram : Spec
histogram =
    let
        enc =
            encoding
                << position X [ pName "IMDB Rating", pBin [] ]
                << position Y [ pAggregate opCount ]
                << color [ mName "IMDB Rating", mOrdinal ]
    in
    toVegaLite [ data, enc [], bar [] ]
```

My first stage was to remove the colour encoding, which simply repeats the coding of bars by horizontal position. This also has the effect of removing the distracting and unnecessary legend.

```elm {l v}
histogram1 : Spec
histogram1 =
    let
        enc =
            encoding
                << position X [ pName "IMDB Rating", pBin [] ]
                << position Y [ pAggregate opCount ]
    in
    toVegaLite [ data, enc [], bar [] ]
```

I then removed all other non-data ink before choosing what to build back in:

```elm {l v}
histogram2 : Spec
histogram2 =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        enc =
            encoding
                << position X [ pName "IMDB Rating", pBin [], pAxis [] ]
                << position Y [ pAggregate opCount, pAxis [] ]
    in
    toVegaLite [ cfg [], data, enc [], bar [] ]
```

As with the example in the lectures, widening the chart (and aiming for a golden aspect ratio), provides more space to label the bars.

```elm {l}
goldenRatio : Float
goldenRatio =
    1.618
```

Specifying that the IMDB ratings are `Quant` rather than `Ordinal` labels the gaps between bars and removes the `null` values, providing a clearer indication of the overall distribution of allocated scores. Leaving labels to 1 decimal place helps to indicate these are aggregate average scores not whole number scores.

I dispensed with the y-axis entirely as the relative distribution of scores is probably more useful than the absolute numbers. And I set a more subdued colour to replace the default dark blue. Note how this colour specification, which is independent of any data value is expressed via [maFill](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#maFill) as a member of the list following [bar](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#bar) rather than as a data driven encoding.

```elm {v l}
histogram3 : Spec
histogram3 =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])

        enc =
            encoding
                << position X [ pName "IMDB Rating", pBin [], pTitle "" ]
                << position Y [ pAggregate opCount, pAxis [] ]
    in
    toVegaLite
        [ cfg []
        , width 300
        , height (300 / goldenRatio)
        , data
        , enc []
        , bar [ maFill "#aab" ]
        ]
```

---

I made several judgements in the design, such as not requiring the number of unclassified movies (`null`) and not showing the absolute counts in each category. Whether this is appropriate would normally depend on the context of the visualization and would be something you as a designer would have to justify.

### 3. _Creativity Challenge:_ Hand Drawn Personal Visualization Sketch

A few of you were still having trouble getting images to appear in your litvis documents. To do this, place the image file (.jpg, .png etc.) in the same folder as the litvis document, and then within the document you include the following:

```txt
![Text to appear if image is missing](myImage.png)
```

(changing `myImage.png` to whatever your actual image file is called). Don't forget that the image also needs to be pushed to your GitHub repo for me to be able to see it.

I saw some creative personal data collections, many of which were quite moving to see. Thanks to those of you who posted on Slack with your sketches. If you haven't done so already, please do post your examples on Slack when you've sketched your personal vis design.

While there were plenty of bar charts, it was good to see some people liberated from the constraints of technology in their hand-drawn examples. For example, Divya's waterbottle charts that used variously filled water bottle icons to represent quantities and proportions. I was moved by the impact of the (finger) nail visualizations as an act of reflection that was simultaneously public and private. I liked Elizabeth's "books on my bedroom floor" that included some nice classifications and annotations that went beyond simply recoding data. I like that the act of creating this visualization prompted some personal reflection and future action. Similiarly Ying Ying's daily steps visualization acted as a prompt for reflection and action that illustrated nicely how effective (and affective) personal visualization can be. Kate's "food texture wheel" was an intersting example of thinking creatively about how to capture (eating) experience and show it visually.

If you are comfortable doign so, please share on Slack for others to see.

The act of creating or designing something about your own activity can be surprisingly insightful, and often emotional, and I'd encourage you to continue to create such personal visualizations both as an act of personal reflection and an exercise in the power of visualization.
