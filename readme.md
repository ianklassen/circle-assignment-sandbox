# Circle Assignment Sandbox

Experiment design sizing for a product where the billable unit is a household rather than a person.

**Live demo:** https://ianklassen.github.io/circle-assignment-sandbox

Built by Ian Klassen. 

---

## The problem

Life360 sells to Circles but experiments are typically conducted on people. A paywall or a price is visible to everyone in the household, so assigning treatment at the user level means a parent sees one offer and their teenager may see another. That breaks the product for the person already paying and it contaminates both arms of the test.

Assigning at the Circle level fixes it, but it may have a cost, because you now have fewer independent units and their sizes vary, which inflates variance and lengthens the test. 

This tool sizes both costs before the test ships.

## What comes out

Three designs measured side by side:

- **Each person independently.** Fastest on paper, and only honest when the change is invisible to everyone else in the Circle.
- **Each Circle as a unit.** Valid for anything household visible. Slower.
- **Each Circle, adjusting for past behaviour.** Identical assignment. The difference is in the analysis: what a Circle did before the test starts is used to subtract differences that were already there, so what is left is a cleaner read on the change itself. Less noise means a shorter test. The technique is usually called Controlled-Experiment Using Pre-Existing Data (CUPED). It only helps if past behaviour actually predicts the metric, and it does nothing for Circles with no history.

For each: how many households get split, how many people receive conflicting assignment, the clustering penalty, Circles required, and weeks to a readable result.

Then an independent validity gate rules on the design you chose. It never sees which design was fastest.

## Method

- Design effect for unequal cluster sizes: `1 + ((CV² + 1) · m̄ − 1) · ICC`, where m̄ is mean Circle size and CV its coefficient of variation. Equal size clusters collapse this to the familiar `1 + (m̄ − 1) · ICC`.
- Sample size per arm, two proportion test at 5% two sided and 80% power: `2(z_α/2 + z_β)² · p̄(1−p̄) / δ²`, multiplied by the design effect.
- Adjusting for past behaviour scales required sample by `1 − ρ²`, where ρ is the correlation between a pre period covariate and the metric. It changes the analysis, never the assignment, which is why that column differs from plain Circle assignment on runtime alone.
- Probability that a Circle of size m is split by person level assignment: `1 − 0.5^(m−1)`. The same form gives the probability that a person belonging to m Circles receives conflicting assignments.

Household size and Circle overlap distributions are illustrative, not Life360 data. The structure is what transfers. Replace the distributions with real ones and every number sharpens.

## The validity gate

| Rule | Test |
| --- | --- |
| V1 Split households | Person level assignment on a change the whole Circle can see |
| V2 Runtime | Required weeks exceed the planning window |
| V3 Conflicting assignment | Multi Circle members landing in treatment and control at once |
| V4 Exclusion bias | Removing multi Circle members, and what that does to who is left |
| V5 Effect below the noise floor | Target lift smaller than seasonality, novelty, and between Circle spillover |
| V6 Assumed clustering cost | Design effect above 2x while the within Circle correlation is a guess |

Any stop produces NOT VALID. Any handle without a stop produces VALID WITH CAVEATS. Otherwise VALID.

### Why the gate is separate

A sizing tool answers how long. It cannot answer whether the answer will mean anything, and the two pull in opposite directions, because every shortcut that buys weeks costs validity. Under a deadline, a single combined score will always find a way to trade validity for speed, since speed is the thing being measured. Putting validity in a layer that never sees the runtime is the only arrangement where it survives.

This is the same two stage pattern as the [Tier Placement Desk](https://github.com/ianklassen/tier-placement-desk): a deterministic layer proposes, an independent layer rules, and the second one never sees the argument made by the first.

## One finding worth the tool

Larger Circles carry a bigger clustering penalty, so they look worse. They are usually still faster to test on, because every Circle entering the experiment brings more people with it, and that outweighs the variance cost.

Not always. Past a certain level of within Circle similarity the ordering reverses, because members stop carrying independent information and a large Circle becomes one observation wearing six coats. The tool computes where that crossover sits for your inputs, which on the default settings lands around 40%.

That is the practical question. Whether large households are an asset or a tax is not a fact about households, it is a fact about how independently their members behave, and it is measurable from any past experiment.

## Running it

```
git clone https://github.com/ianklassen/circle-assignment-sandbox
open index.html
```

No build step, no server. To deploy, enable GitHub Pages on the repository root.

## Contact

Ian Klassen
ian@ihk.ca
