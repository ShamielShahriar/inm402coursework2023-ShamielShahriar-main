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

# Session 9 Feedback and Answers

## 1. Comparison of Network Visualization Approaches

Here is a compilation of some of your and my answers to the first task, which hopefully forms a useful summary of the kinds of rationale to consider when choosing a network representation.

I noted in several people's rationale, arguments such as "can be hard to differentiate lines if similar width" or "thinner lines harder to see". While these may be true observations, it is worth remembering that usually we want unimportant data to be hard to see, or similar data to look similar. This only becomes a disadvantage if we have not consistently mapped salience (how visually obvious something is) to data importance in relation to the task it addresses.

### Node-link diagram

| Advantages                                                                                                                                                                    | Disadvantages                                                         |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Intuitive representation of connections (gestalt theory of connectedness)                                                                                                     | Less linked nodes less visible (may also be an advantage)             |
| Can be visually appealing                                                                                                                                                     | Visual attractiveness can distract from poor network representation   |
| Can trace connection paths beyond immediate neighbours                                                                                                                        | Hairball effect – lines in well connected graph soon become cluttered |
| Can produce different layouts for same network so can see robustness of conclusions drawn from the visual by changing layout - do the conclusions still hold with new layout? | Positioning of nodes can be arbitrary / difficult to control          |
| Can be good for identifying clusters comprising multiple interconnected nodes (if not a large 'hairball')                                                                     | Short lines less visible                                              |
|                                                                                                                                                                               | Does not show links between a node and itself                         |
|                                                                                                                                                                               | Direction of flow can be hard to see in complex networks              |
|                                                                                                                                                                               | Not very well implemented in Vega-Lite                                |

### Matrix view

| Advantages                                                          | Disadvantages                                                        |
| ------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Scales well with a large number of edges                            | Scales poorly with a large number of nodes                           |
| No overlap or visual clutter, showing entire dataset                | Harder to trace non-adjacent connections                             |
| Can show flows from one node to itself                              | Order of rows/columns/cells can be arbitrary                         |
| Can show locally dense areas if rows/columns arranged appropriately | Half the matrix space wasted as it is symmetrical about its diagonal |
| Labelling nodes easier (row and column labels)                      | Not very space efficient for representing sparse networks            |
|                                                                     | Sparse networks with many nodes but few edges hard to make use of    |
|                                                                     | Poorly suited to geographically embedded flows                       |
|                                                                     | Less visually pleasing                                               |

### Sankey diagram

| Advantages                                            | Disadvantages                                                     |
| ----------------------------------------------------- | ----------------------------------------------------------------- |
| Intuitive representation of multiple pathways         | Less familiar to many, can require time to learn to read it       |
| Layout makes assessing group size (line width) easier | Can be harder to create (in code or with visualization package)   |
| Good for emphasising flow transitions between states  | Many crossing lines can make interpretation harder                |
| Level of abstraction can simplify complex patterns    | Categorical colour scheme limited for large numbers of categories |
|                                                       | Can be harder to trace non-adjacent connections                   |

### OD Map

| Advantages                                                   | Disadvantages                                                                     |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| Gives each place equal visual prominence                     | Unfamiliar to many, can require time to learn to read it                          |
| Long flows and short flows given equal prominence            | Harder to trace non-adjacent connections                                          |
| Easier to spot geographical structure (e.g. regional flows)  | Colour a less discriminating visual variable                                      |
| Can show flows from one place to itself                      | Representing location as square makes it harder to identify than conventional map |
| Simultaneously provides an overview and zoomed detailed view | Bearing, distance and shape of regions distorted                                  |
| Two OD maps allow asymmetry in flows to be quickly assessed  |                                                                                   |

## 2. Co-authorship Insights

This question asked you to identify the three most important insights about the co-authorship network citing the visualization that led to those insights. The idea was to give you practice for your coursework where you are asked a similar question. A few of you appeared to misunderstand what was required of you here so do please look at the insights below to make sure you know how to answer the equivalent CW question.

A couple of people said they weren't sure what Question 2 was asking of you. And some of you answered in more generic terms about what kinds of visualization worked best, some others reflected on the range of network visualization approaches they might take in the future. While these were interesting, it is important for the coursework (as well as often when using any data visualization) to think about _data_ insights. In other words what do you now know about the data that you didn't know before?

Quite a number of you suggested a disadvantage of the OD map / OD matrix is that direction is not symbolised. Direction is represented, but by position (a->b will appear in a different location to b->a. Perhaps arguably it is not "symbolised" but it is visually encoded.

A full interpretation of the co-authorship benefits from an understanding of how and why authors might be listed in a particular order on a paper. It is common for an alphabetical order, or sometimes a lead author who made the main contribution followed by an alphabetical ordering. It is also the case that the Sankey diagram implementation will order nodes alphabetically, so this can introduce some arbitrary patterns that might not mean very much in reality (e.g. compare authors with an A surname and those with a surname near the end of the alphabet).

Below is a collection of some of the data insights you mentioned in your submissions where I have attempted to order them from, arguably most important to least important. While you may legitimately dispute the ordering, what I have attempted to do here is promote the kinds of insights that were not initially obvious until seeing the visualizations. That is not to say the lower ranked ones are not true, but rather they are possibly more obvious insights that we might already have known before seeing the visualization; or perhaps they are too general to assess their validity; or they are so specific that perhaps they are less important in the wider picture.

- **Insight 1:** There are some exceptions to the trend that more papers -> more collaborators, reflecting individual differences in style of collaboration with some authors sticking with the same group of co-authors, while others are more diverse in their collaborators. This could, for example, allow us to distinguish PhD students who publish mostly or exclusively with their supervisors, from those working with a diversity of authors, even in a single paper. _(interactive scatterplot of papers vs authors)_.

- **Insight 2:** More prolific paper authors have 'favourite' co-authors, but this may also reflect the fact longer established authors dominate this particular database _(node-link diagram, Sankey diagram and matrix view)_.

- **Insight 3:** Appears to show a [Pareto distribution](https://en.wikipedia.org/wiki/Pareto_distribution) (small number of high volume, high volume of small numbers), prompting questions about _when_ papers are published in a career _(bar charts)_.

- **Insight 4:** Authors likely collaborate with both Natalia and Gennady Andrienko, or with neither of them, but very rarely with just one of them) _(node-link diagram, Sankey diagram, matrix view)_.

- **Insight 5:** There are a few sub-networks of strong collaboration (e.g. Dykes - Wood - Slingsby; and Andrienko - Andrienko - Fuchs) _(node-link diagram, Sankey diagram)_.

- **Insight 6:** There is relatively little connection between the three main sub-networks (Dykes - Slingsby - Wood), (Reyes Aldasoro) and (Andrienko - Andrienko) _(node-link diagram with Gephi layout)_.

- **Insight 7:** Some others more likely to lead research (e.g. Andrienkos) than others (e.g. Wood) even if large numbers of collaborations _(Sankey diagram)_\*1.

- **Insight 8:** The authors associated with only 1 paper all appear to have names at the end of the alphabet. Why might this be? Can you spot a problem with the way data have been compiled? _(scatterplot)_.

- **Insight 9:** There is a 'top 6' authors who seem to have a clearer separation from the others in terms of paper volumes _(Scatterplot, bar charts and Sankey diagram)_.

- **Insight 10:** Strong connections between G and N Andrienko and their common name suggests additional family/marital connection supporting collaboration _(node-link diagram; Sankey diagram)_.

- **Insight 11:** Top 6 authors (by publication number) have nearly 60% of overall papers _(Sankey diagram)_.

- **Insight 12:** No author has published prolifically by themselves _(scatterplot of papers vs authors)_.

- **Insight 13:** (giCentre) research is a more collaborative process than I realised _(all diagrams)_.

- **Insight 14:** Revealing sub-networks suggests common areas of shared interest/expertise _(node-link diagram)_.

- **Insight 15:** Shows there is a hierarchy in co-authorship _(node-link diagram)_.

- **Insight 16:** The more papers someone has written the larger the number of co-authors they are likely to have _(scatterplot of papers vs authors)_.

- **Insight 17:** Carlos Reyes Aldasoro has written the most papers and has most co-authors _(bar charts)\*2_.

\*1 _This was a valid observation to make from the Sankey diagram, but is actually just a function that paper authors are commonly listed in alphabetical order._
\*2 _This could be found by examining the top row of a sorted table. Note also this is only within the context of the giCentre sample, and does not reflect full publication record (the Andrienkos published many papers before joining the giCentre, which are not included in the dataset)._
