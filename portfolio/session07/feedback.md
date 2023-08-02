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

# Session 7 Feedback and Answers

## 1. Juxtaposition vs Superposition vs Direct Encoding

Good to see not only sensible choices from those who answered this question, but sound rationales for your choices.

Don't forget, you can also use literature to support your choices (something for which you will get credit in your CW submission). In this case, I'd strongly encourage you to read the [Gleisher et al 2011](https://go.exlibris.link/GRVzHyPn) paper if you haven't done so already.

There a range of possible legitimate answers depending on priority and task (I think I saw in total J,S and D for all scenarios), but below are some suggestions based on the more frequent arguments presented collectively in your submissions.

| Research Question                                                            | J, S or D? | Rationale                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------------------------------------------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| How do monthly rainfall patterns compare in Sheffield and Manchester?        | S          | With only two sets of figures to compare, superposition would not be too cluttered. Both have common scales (time and rainfall), so could share common positional encoding. As such, crossing lines would be meaningful.                                                                                                                                                                                                                                                                         |
| In which age cohorts have covid cases changed the most over time?            | J / S      | Those who argued for juxtaposition made the point that the numbers in different age cohorts might vary and so separate scales (and visualizations) would help if there was a need to see the slope of the each curve (i.e. proportional change _within_ cohort). Those who argued for superposition emphasised the easier comparison _between_ cohorts afforded by sharing a common space.                                                                                                       |
| How do numbers of goals scored over a 5 decades vary between football teams? | S / D      | Those arguing for superposition made the point that with many teams over a long period, juxtaposition might result in too many charts making comparison difficult. Some form of symbolisation with thin lines would allow superposition (like the Paris-Brest-Paris bike race chart) even with large numbers. Others argued for some kind of direct measurement to simplify the visual encoding.                                                                                                 |
| To what extent does penguin morphology vary by island?                       | J          | With many samples and several measurements of morphology, superposition would be hard to interpret. Juxtaposition, like the SploMs considered in the lecture may make interpretation and island-to-island comparison easier. Some suggested D for this (e.g. via z-score). One of the challenges here though is that we are not just comparing a pair of islands, but three. A single measure of difference might not capture the variation that differs between different pairwise comparisons. |

## 2. Parallel Coordinates Plots

I did see quite a number of explanations for PCP properties that suggested some misunderstanding. Hopefully this feedback will help clarify. But it does also suggest a genuine problem with PCPs is the ease with which they can be misinterpreted (or simply not understood).

A couple of you suggested the midpoint of the vertical axes represents the mean of the data being shown. While there is a possibility it could, it is generally unlikely to be so. Usually, values are scaled between the minimum and maximum values of a variable, so the midpoint would represent just the midpoint value between extreme values which is not necessarily the same as then mean of all values.

Many suggested the direction of slope of the lines indicates either a positive or negative correlation. While this is true for a Cartesian scatterplot with regression line, it is not the case for a PCP. Remember a single line is just a 'point' in a conventional scatterplot, so its direction cannot indicate anything about correlation (just as a single point cannot on a scatterplot). Correlation is about the relationship between multiple values, so we need to look for patterns that relate multiple lines in order to infer anything about correlation.

A number of you suggested that horizontal lines on a PCP indicate positive correlation. While that is true, it is only one specific category of positive correlation, where the values in one variable are the same as the values in another. It is quite possible (and probably more likely) that positively correlated pairs of variables may have different values in each (e.g. 1 -> 11; 2 -> 12; 3 -> 13 etc.). Many of you recognised this in that you said for positively correlated values, both would increase (or decrease) together, resulting in parallel lines. But remember also that they need not increase / decrease at the same rate (e.g 1 -> 3; 2 -> 6; 3 -> 9), so lines need not be parallel.

I saw some well-reasoned arguments for the relative merits of SploMs vs PCPs, with some key arguments being about scale (SploMs don't scale well for large numbers of variables), familiarity/interpretability (PCPs less familiar) and the dependence of a PCP on the order of axes chosen.

Here are some further thoughts on some of the specific characteristics of PCPs with illustrative plots (scatterplot on left, PCP on the right).

### What would positively and negatively correlated pairs variables look like on a PCP?

Assuming we have a pair of variables scaled to the same range, a perfectly negative correlation would generate a set of intersecting lines at the same point (in the example below, somewhere to the left of the Y-axis). Note that a single pair of lines that intersect is not sufficient to indicate correlation, just as two points on a scatterplot are too few to show any correlation.

![pcp negative correlation](https://staff.city.ac.uk/~jwo/datavis2023/session07/images/pcp1.png)

Similarly, positively correlated points generate intersecting lines in PCP space, but that intersection will be 'outside' the space of the two parallel coordinate axes, to the left of the X-axis:

![pcp positive correlation](https://staff.city.ac.uk/~jwo/datavis2023/session07/images/pcp2.png)

When points are not perfectly correlated, the PCP lines won't all intersect at the same place, but within a small area, the poorer the correlation, the larger the area of intersecting points:

![pcp weaker negative correlation](https://staff.city.ac.uk/~jwo/datavis2023/session07/images/pcp3.png)

Visually therefore it could be argued that negative correlations (crossing lines in the same region) are easier to spot visually than positive correlations (we have to extrapolate lines beyond the region of the axes). As such, for tasks where spotting negative correlation is more important than spotting positives, PCPs may offer an advantage.

### Do crossing lines have any meaning? What is your reasoning?

Firstly note that a single pair of crossing lines does not have much meaning as only two 'points' are being represented. Just as two points on a scatterplot forming a line doesn't tell us much. Therefore what we are interested in is whether _multiple_ lines cross.

It's worth asking if the space between adjacent axes has any specific meaning. In most cases it will not (what would a space between penguin beak length and body mass mean?). But occasionally it does, such as time/distance on the Paris-Brest bicycle race chart. Asking whether there is any intrinsic meaning to the space for a given PCP is useful in knowing whether to spend effort in understanding the position of intersections.

Consideration of correlation should hopefully have demonstrated that the point of intersection of crossing lines in PCP space represents co-linear points in Cartesian space. However, just because several lines do not cross in a PCP does not mean the points they represent are not co-linear. For example see the positive correlation above.

We saw an example of crossing lines having meaning in a PCP plot in the Paris-Brest-Paris cycle chart from the previous session:

![Paris-Brest-Paris 2015](https://staff.city.ac.uk/~jwo/datavis2023/session06/images/pbp2015.jpg)

Those darker patches of multiple crossing lines represent adjacent pairs of variables (race positions at consecutive checkpoints) where there has been much changing of position. You can think of those stages as negatively correlated - riders who had slept prior to the stage start lower down in race order and end the stage higher up. Those who had not slept start ahead, but lose position if they sleep or ride slowly and tired.

When you create or interpret a PCP, think about what meaning (if any) line intersections have in the context of the data being shown. We are good at _detecting_ line intersections visually, but _interpreting_ those intersections requires more cognitive effort.

### What are the relative advantages and disadvantages of the PCP compared with the SploM?

For further insight into PCP space, see these [experiments with parallel coordinate projections](https://staff.city.ac.uk/~jwo/datavis2023/session07/parallelCoords.md) (download and place in your session 7 folder to view in VSCode).

## 3. Layout in your datavis project

Good to see many of you thinking carefully about how you are going to support comparison for your CW, and the role layout will be playing in your design. Some of you are thinking about how to link layout with interaction, such as the use of cross-filtering/highlighting, which is a promising way of exploring comparison between juxtaposed charts.

Many of you said "I will be using X" (where X is one of 'juxtaposition', 'superposition' etc.). Remember that you might use a combination of these in your designs, not just one, depending on what you are trying to find out.

The intention of this question on position and layout was to give you practice in reasoning about your design choices; something that is helpful as part of the design process as well as specifically for your coursework.
