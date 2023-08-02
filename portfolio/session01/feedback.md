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

# Session 1 Feedback and Answers

## 1. Registering with the datavis slack and setting up your GitHub repo

It looks like most of you have successfully registered on Slack, but not everyone. If you haven't done so already, please do so now as we use Slack for discussion and questions on datavis.

Similarly, as of 6th February, I see 36 repos on GitGub, which means there are 12 of you who have not yet completed this step. I also see some repos without your name and email included in the two README files (one the root of your repo, the other in the `portfolio` folder). Please add these and push them to your GitHub repo if you haven't done so already. Without your repo or name, I cannot reliably allocate you marks for completing the exercises!

A few of you submitted very minimal answers (e.g a few sentences for Q.2 only). As this is the first week I am being a lttle more generous in the allocation of 'completion marks', but please try to submit complete answers in future weeks. If some questions are too difficult, even partial answers or statements about what you find difficult can be helpful.

Judging by the some of the code blocks in your submissions I think some of your setups are not auto-formatting code output when you save. This may be because you have opened the markdown file directly into VS-Code rather than opening the folder in VS-Code then selecting the file.

## 2. Datavis evaluation

Thanks very much to those who added your [Information is Beautiful](https://www.informationisbeautifulawards.com/showcase?action=index&award=2019&controller=showcase&page=1&pcategory=short-list&type=awards) critiques to your repos. One of the reasons for adding your evaluation now as it will be interesting to compare your critiques now and those you provide at the end of this module to see if your skill at evaluating good datavis design has improved over the period.

Some common themes that have emerged in the evaluations I have read:

- A tension between making a design visually engaging and making it functionally useful. Sometimes, these coincide and that is something certainly to aim for, but as you have noted, this is not always the case.

- Related to the first point: the role of 3d and perspective in design. Sometimes this can help create a sense of immersion and scale, but can also hinder our ability to make estimates of the data properties being depicted. It can be useful to consider the advantages come with using 3d and what are the costs.

- Many people made comments along the line "the visualization helps you to understand...". This is a useful start point for evaluation, but ask yourself what exactly do you now _understand_ that you didn't before. Sometimes (poor) visualizations just show some data values, but don't actually improve understanding. One experiment you can try is to try to recall what you now now about the subject of the visualization, a few hours after you looked at it. This is perhaps one way we might assess whether the vis has support the construction of _knowledge_. And you may recall my mention of Bloom's taxonomies of learning. This is a useful framework for thinking about the different levels of learning that can be supported by a visualization (there's a little more on this in Session 2).

- How does one tell a story with a static visualization? Is it possible to build a sequenced narrative without interaction? And if interaction is available, how does design help to guide people in the use of interaction?

- Good to see several of you considering 'aesthetics' in its widest sense: the ability to evoke an emotional response (e.g. Kate identified 'surprise' as one response). This recognises that the craft of storytelling and selecting what aspects of data to visualize can be part of the aesthetic design process in addtion to 'appearance'.

## 3. Setting up Litvis

I hope I have dealt with any outstanding litvis installation issues on Slack. If you still have problems with the installation, please post on Slack describing the problem as soon as possible or drop in to my Tuesday office hours (A401d, top floor of College Building, 11:00-13:00) so you can proceed with the module. It is vital that you have a working system in order to participate in this module.

### Errors in your litvis documents

I saw a few submissions with red text in place of working visualizations or outputs. These will have been generated because one or more code blocks have not been successfully compiled. Have a look at the document on Moodle _Dealing with errors in litvis documents_ that identifies common sources of errors, including indentation problems, and how to avoid them. Do have a look at that document as it will likely save you many frustrated hours over the course of this module. It also provides some good advice on how best to ask a question about a programming problem, such as on Slack.

## 4. Creating a litvis sketch

This task asked you to add some text to the litvis document containing the bike hires bar chart. Remember to place the code inside triple backticks – some of you forgot this and wondered why the code was not generating any graphics. Equally, there is no need to use triple backticks for text – they are only used when you need to get the computer to execute a block of code.

You can format markdown text if you need to as well as place text inside _narrative schema labels_. For example you could make the statement about the aim of the graphic inside `aim` labels.

````markdown
---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

```elm {l=hidden}
import VegaLite exposing (..)
```

{(aim|}

To show the pattern of hires over time in order to identify any seasonal cycles and
longer term trends.

{|aim)}

```elm {v}
myBarchart : Spec
myBarchart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv" []

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
    in
    toVegaLite [ width 640, data, enc [], bar [] ]
```

The winter-summer variation is very clear from the time series with peaks in July/August
of each year and fewest hires every December. There is a gradual longer term increase
in hires but this is minor in comparison to the seasonal variation. Since the lockdown
was relaxed in 2020, there has been a large increase in numbers of hires.
````

Here's what the document above would look like when previewed in VSCode:

---

{(aim|}

To show the pattern of hires over time in order to identify any seasonal cycles and longer term trends.

{|aim)}

```elm {v}
myBarchart : Spec
myBarchart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv" []

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
    in
    toVegaLite [ width 540, data, enc [], bar [] ]
```

The winter-summer variation is very clear from the time series with peaks in July/August of each year and fewest hires every December. There is a gradual longer term increase in hires in winder months but this is minor in comparison to the seasonal variation. During the first lockdown period of 2020 and its subsequent relaxation, there was a large increase in numbers of hires, but in the following year, the pattern has been a little more similar to earlier years.

---

## 5. Creating a scatterplot

Creating a scatterplot comparing number of hires with hire length involves two modifications to the original bar chart specification. Firstly, to change the variables mapped to the `position X` and `position Y` channels, both of which are now quantitative variables (`pQuant`), and secondly, to change the `bar` mark to a `circle` mark. It also makes sense to change the name of the function from `myBarChart` to something that fits this modified version (`hiresScatterplot` in this example).

You can also change the dimensions of the visualization to make the scatterplot a squarer shape by adding a `height` value to the specification:

```elm {l v}
hiresScatterplot : Spec
hiresScatterplot =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv" []

        enc =
            encoding
                << position X [ pName "NumberOfHires", pQuant ]
                << position Y [ pName "AvHireTime", pQuant ]
    in
    toVegaLite [ width 400, height 400, data, enc [], circle [] ]
```

You many also notice that _outliers_ of longer than usual hire times (27 to 36 minutes) as isolated dots towards the top of the chart.

The correlation between the two variables invites an explanation of the cause. Perhaps both are influenced by weather, with summer months both encouraging more people to cycle and to cycle for longer. But perhaps you can think of other explanations (e.g. do you think the reason people use the bikes might change during the year?).

## 6. Data Challenge

The structure of the specification is very similar to that we have already seen with the bike hire spec. Note that elm-vegalite can read JSON as well as CSV files so we simply refer to the URL of the JSON file just as we did for the CSV file.

Some people were asking how you download the data to display. You don't need to download the data files separately - you simply name the location of the file to use in the vega-lite specification. We leave it to litvis to do the downloading.

If you attempted to plot just the percentage response against time, you would have seen a confusing saw-tooth pattern as follows:

```elm {l v siding}
euChart : Spec
euChart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/euPolls2.json" []

        enc =
            encoding
                << position X [ pName "Date", pTemporal ]
                << position Y [ pName "Percent", pQuant ]
    in
    toVegaLite [ width 600, data, enc [], line [] ]
```

This is because the `Percent` column contains data for all response types (`Rejoin`, `Stay out` and `no answer / don't know`) and the specification is attempting to plot all of them in temporal sequence.

What we need to do is specify that the three response types (`Answer`) should each be encoded separately, and conveniently this can be done by encoding each with a different colour.

A number of people wished to change the colours used to show the three answer categories. Those who have programmed before (in non-declarative languages like Java and Python) may have been tempted to look for code that says 'make "leave" this colour, "remain" this colour and "don't know" that colour'. In Session 3 you will look at how we could do that, but for now it is better simply to describe the data accurately and let Vega-Lite come up with suitable colours. We can do this by specifying that response type isn't `Quant` or `Temporal` but used the default (`Nominal`) measurement type - i.e. unordered discrete categories. In doing so, Vega-Lite provides a default blue-orange-red colour scheme to reflect those properties.

```elm {l v siding highlight=[11]}
euChart : Spec
euChart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/euPolls2.json" []

        enc =
            encoding
                << position X [ pName "Date", pTemporal ]
                << position Y [ pName "Percent", pQuant ]
                << color [ mName "Answer" ]
    in
    toVegaLite [ width 600, data, enc [], line [] ]
```

I saw a couple of people wishing to show only one of the response type (e.g. "Rejoin") but were not sure how to do that. While you will cover the idea of filtering data to show only certain parts of it in later weeks, here's a preview for those who are interested:

We can [transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#3-transforming-data) any dataset in our specification. In this case we might chose to [filter](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#filter) just selected parts of the data with an expression that determines what is to be included after filtering:

```elm {l v siding highlight=[8-9,17]}
euChartFilter : Spec
euChartFilter =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/euPolls2.json" []

        trans =
            transform
                << filter (fiExpr "datum.Answer == 'Rejoin'")

        enc =
            encoding
                << position X [ pName "Date", pTemporal ]
                << position Y [ pName "Percent", pQuant ]
                << color [ mName "Answer" ]
    in
    toVegaLite [ width 600, data, trans [], enc [], line [] ]
```

It was good to see that some of you tried specifying different mark types. Please continue to experiment with different design elements as you develop your datavis. I encourage you to bookmark the [elm-vegalite documentation page](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite) to help you explore what is possible.

The `bar` mark causes Vega-Lite to create stacked bars, which can give a better impression of the 'part-to-whole' relationships of each answer:

```elm {l v siding highlight=[13]}
euChart : Spec
euChart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/euPolls2.json" []

        enc =
            encoding
                << position X [ pName "Date", pTemporal ]
                << position Y [ pName "Percent", pQuant ]
                << color [ mName "Answer" ]
    in
    toVegaLite [ width 600, data, enc [], bar [] ]
```

But because the polls were taken at irregular points in time, the gaps between them can be distracting, so the `area` mark does a better job at conveying the proportions of responses in each answer category:

```elm {l v siding highlight=[13]}
euChart : Spec
euChart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/euPolls2.json" []

        enc =
            encoding
                << position X [ pName "Date", pTemporal ]
                << position Y [ pName "Percent", pQuant ]
                << color [ mName "Answer" ]
    in
    toVegaLite [ width 600, data, enc [], area [] ]
```

A few of you suggested pie charts to show polling results. This can certainly be achieved with Vega-Lite and litvis (see the radial examples in the gallery of your repo) and they are good for showing 'part-to-whole' relationships (i.e. quick visual judgement about the proportion of 100% each voting preference occupies). But consider how easy (or difficult) it is to make comparisons across time. What is it about the pie charts that make this a more difficult task than the line, bar or area charts?

```elm {v}
euPies : Spec
euPies =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/euPolls2.json" []

        enc =
            encoding
                << position Theta [ pName "Percent", pQuant ]
                << color [ mName "Answer", mLegend [] ]
    in
    toVegaLite
        [ data
        , columns 7
        , facetFlow [ fName "Date", fHeader [ hdTitle "", hdFormatAsTemporal, hdFormat "%e/%m/%Y" ] ]
        , specification (asSpec [ width 50, height 50, enc [], arc [] ])
        ]
```

The lesson here is to go beyond 'show comparisons and connections' to consider what precisely do we wish to compare, and to adjust our designs to make those comparisons as easy as possible.

## 7. Design Challenge

Good to see plenty of alternative designs in your repos for the design challenge. Special mention to Paula for a thorough and well thought-through answer to the challenge. with a detailed mockup. If you are comfortable doing so, sharing your design and evaluation on Slack would be a great way to get feedback from each other.

Emily suggested an interactive chart that showed three circles in proportion to the numbers polling 'rejoin', 'leave' and 'don't know'. Those circles would change in size as an interactive slider moved over time. An interesting question to ask is why is it that circles seem to hold an attraction in visualization design? What is it about them that makes them so appealing? In future weeks we will consider ideas of design 'expressiveness' and 'effectiveness' that might allow us to address such questions.

Some of you recognised that the results from different polling companies appeared consistently different (especially the 'don't knows') and reflected that in your designs. It is a good reminder that there can be many different perspectives you can take on the same dataset.

A few of you were having problems getting your image to display in the litvis document. You need to make sure the image file itself sits in the same folder as your markdown document **and is pushed to your repo**, and then you display it in markdown as follows:

```txt
![Alternative text if image is not found](myImage.png)
```

Some of your sketches were created with existing packages (e.g. Tableau), which is fine, but does run the risk of constraining your design thoughts to what the particular package can implement.

I liked that many of you emphasised a 'baseline' (e.g. the 50% mark) to make comparisons.

I include some notes below on my attempt.

I wanted to create a design that (i) Used a better vertical ordering of the three categories than the default (alphabetical) one; (ii) showed the uncertainty in polling results by providing an intuitive representation of the 'don't know' responses; (iii) emphasised the importance of the 50% mark in determining the results of a referendum. My initial sketch design looked like this:

![sketched design](https://staff.city.ac.uk/~jwo/datavis2023/session01/images/euSketch.jpg)

This included scaling the rejoin/outside percentages from the top and bottom respectively; making the don't knows a neutral or background colour to contrast it with the more decided opinions and labelling the 50% mark; and labelling the rejoin and remain outside areas directly, dispensing with the need for a legend.

The actual implementation in the previous step goes part way there, but still some extra work required to change colour schemes and add labels. The gap between what you would like to design and what is achievable is quite common, especially when starting, so don't become disheartened. With practice, and the skills you develop in the coming sessions, you will be able to narrow that gap.
