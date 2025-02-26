---
author: Christian Kaestner
title: "17-445: Experimentation"
semester: Fall 2019
footer: "17-445 Software Engineering for AI-Enabled Systems, Christian Kaestner"
license: Creative Commons Attribution 4.0 International (CC BY 4.0)
---

# Experimentation

Christian Kaestner

<!-- references -->

Required reading: 
* tbd

---

# Learning Goals

* Plan and execute experiments (chaos, A/B, ...) in production
* Conduct and evaluate multiple concurrent A/B tests in a system
* Examine experimental results with statistical rigor
* Perform sensitivity analysis in large configuration/design spaces


---
# Science


> The scientific method is an empirical method of acquiring knowledge that has characterized the development of science since at least the 17th century. It involves careful observation, applying rigorous skepticism about what is observed, given that cognitive assumptions can distort how one interprets the observation. It involves formulating hypotheses, via induction, based on such observations; experimental and measurement-based testing of deductions drawn from the hypotheses; and refinement (or elimination) of the hypotheses based on the experimental findings. -- [Wikipedia](https://en.wikipedia.org/wiki/Scientific_method)

----
## Excursion: The Seven Years' War (1754-63)

Britain loses 1,512 sailors to enemy action...

...and almost 100,000 to scurvy

![Capture of the French ships Alcide and Lys in 1755 off Louisbourg by Boscawen's squadron](war.jpg "Capture of the French ships Alcide and Lys in 1755 off Louisbourg by Boscawen's squadron.")
<!-- .element: class="stretch" -->


(American part of [Seven Years' War](https://en.wikipedia.org/wiki/Seven_Years%27_War) known as [French and Indian War](https://en.wikipedia.org/wiki/French_and_Indian_War))

----
## James Lind (1716-94)

<!-- colstart -->
Possibly first ever controlled medical experiment in 1747

Sailors on different ships supplemented with
- [ ] hard cider
- [ ] sulfric acid
- [ ] vinegar
- [ ] sea water
- [x] oranges
- [x] lemons
- [ ] mixture of balsam of Peru, garlic, myrrh, mustard seed and radish root

(initially dismissed as anecdotes since it didn't fit models of the disease; nobody paid attention until a proper Englishman repeated the experiment in 1794)
<!-- col -->
![James Lind](lind.jpg)
<!-- colend -->
----
## Evidence-Based Medicine

Randomized double-blind trials accepted as gold standard in medical research

![](placebo.jpg)
<!-- .element: class="stretch" -->

Long path towards evidence-based medicine (term coined in 1992)

Notes: 
Discuss what double-blind trial entails and why it is important

Picture by [frolicsomepl](https://pixabay.com/photos/cure-drug-cold-dose-the-disease-1006810/)

----
## Example Research Questions

<!-- colstart -->
Does obtaining a CMU degree lead to a higher salary after graduation?
![](graduation.jpg)
<!-- col -->
Do DNN produce higher accuracy models than random forests for forecasting the weather?
![](weather.jpg)
<!-- colend -->

----
## Controlled Experiment Design

Start with hypothesis

Goal: Testing the impact of independent variable X (treatment or no treatment) on outcome Y, ideally controlling all other influences that may affect Y

Setup: Diving test subjects into two groups: with X (treatment group) and without X (control group), observing outcomes Y

Results: Analyzing whether outcomes differ among the groups

*What are X, Y, and test subjects for the graduation and weather forecast research questions?*
----
## Causal Inference with Controlled Experiments

If both groups differ *only* in X, X causes difference in Y.

***

In practice, we usually cannot exactly repeat the same thing with and without X:
* Don't have identical copies of test subjects (e.g., humans)
* Subjects in different groups may have different characteristics, observable or not (e.g., one group mostly male, one group more intelligent)
* Random other factors may influence Y (e.g., pure chance, weather in each subject's location)

▶ Observed difference may be caused by factors other than X, called confounding variables


----
## Confounding Variables

```mermaid
graph TD;
Coffee --> Cancer
Coffee -.- Smoking
Smoking ==> Cancer
```

Correlation != Causation

Notes: Coffee drinking has a strong correlation with cancer, but not a causal relation. The causal relationship is between smoking and cancer, but coffee drinking is correlated with smoking, hence the effect for smoking.

----
## Confounds for Example Research Questions?
<!-- colstart -->
![](graduation.jpg)
<!-- col -->
![](weather.jpg)
<!-- colend -->


---
## Correlation vs Causation

[![xkcd.com/552](https://imgs.xkcd.com/comics/correlation.png)](https://www.xkcd.com/552/)

---
## Handling Confounds

Strategies:
* Keep constant, matching
    - e.g., same room, same demographic, same tasks
    - eliminates influence, enables causal reasoning
    - limits results to specific subject/conditions
* Randomize
    - given large enough random group, characteristics will balance out
    - use statistics to assess chance of random effects
* Measure and filter out later
    - check whether groups were unbalanced
    - regression modeling, exploring multiple factors, not just X
    - commonly used in natural/quasi experiments where group assignment cannot be controlled


**Examples?**

Notes:
Without human subjects (e.g., comparing models) and with reliable measures for Y, we often have comparably little noise and can keep most factors constant.

We'll come back to statistics in a bit.

---
# Offline Experimentation

----
## Data Science is Exploratory and Iterative

![Data Science Loop](datascienceloop.jpg)

<!-- references -->
Philip Guo. [Data Science Workflow: Overview and Challenges](https://cacm.acm.org/blogs/blog-cacm/169199-data-science-workflow-overview-and-challenges/fulltext). BLOG@CACM, 2013


----
## Constant Experimentation

Which features improve the model? Which encoding? Normalization?

Which modeling techniques might work better?

How to tune the learning algorithm?

Try more or different data?


*Both: Ad-hoc testing & hypothesis-driven exploration*



<!-- references -->

Further reading: Kery, M. B., Radensky, M., Arya, M., John, B. E., & Myers, B. A. (2018, April). [The story in the notebook: Exploratory data science using a literate programming tool](https://dl.acm.org/citation.cfm?id=3173748). In Proceedings of the 2018 CHI Conference on Human Factors in Computing Systems (p. 174). ACM.

----
## Iterative Model Refinement

```mermaid
graph TD;
init[Build initial model] --> measure[Measure model quality]
measure --> feature[Add/modify features]
feature --> measure
measure --> param[Tweak parameters]
param --> measure
```

Repeated evaluation of model quality until no further improvement or good enough

Often on static learning and evaluation set

----
## Experimentation Challenges

* Notebooks allow lightweight experimentation, but 
    * do not track history or rationale
    * no easy merging
    * comparison of many experiments challenging
    * pervasive copy + paste editing
* Experiments may be expensive (time + resources, learning + evaluation)
* Overfitting despite separate evaluation set
* TODO

<!-- references -->

Further reading: Kery, M. B., Radensky, M., Arya, M., John, B. E., & Myers, B. A. (2018, April). [The story in the notebook: Exploratory data science using a literate programming tool](https://dl.acm.org/citation.cfm?id=3173748). In Proceedings of the 2018 CHI Conference on Human Factors in Computing Systems (p. 174). ACM.


---
# Sensitivity Analysis

<!-- references -->

Further reading: Saltelli, Andrea, et al. Global sensitivity analysis: the primer. John Wiley & Sons, 2008.

----
## Sensitivity Analysis / What-If Analysis

> The study of how uncertainty in the output of a model (numerical or otherwise) can be apportioned to different sources of uncertainty in the model input (Saltelli et al., 2008)

Old idea for analyzing models: Which inputs are important?

Example: Policy discussion for a new road -- Lots of stakeholders with conflicting opinions:
* Which inputs make significant changes to the outcome and should be prioritized (e.g., in data collection and discussions)?
* Which basic assumptions may affect the outcome if we revisited them?
* Usually only few inputs create most of the variation.

----
## Case Discussion: Policy Model

![Construction side](construction.jpg)

Notes:
What factors might influence the policy decision to build new train tracks in a city or extend an airport?
Which parameters are important for making an informed decision, which are not?

Photo by [Michael Gaida](https://pixabay.com/photos/site-demolition-work-demolition-3688262/)

----
## Classic Sensitivity Analysis Settings

* Models of the real world (economics, policy, physics)
* Lots of input parameters and constants 
    - too many combinations to explore all (exponential explosion)
* Models are often long running simulations (may take days to evaluate)
    - e.g., climate change models, chemistry kinetics computer models
* Many inputs are estimates or assumptions with some uncertainty
    - e.g., future interest rates, carbon absorption of trees
* Identify important parameters by testing which input changes affect the output's uncertainty most
    * sampling and testing
    * analytical methods

----
## Sensitivity Analysis Goals

* Test the robustness of a model in the presence of uncertainty
* Understand relationships between inputs and outputs
* Uncertainty reduction and focus on important parameters
* Debugging models, checking plausibility 
* (Optimization)
* Simplify the model
* Enhance communication between modelers and decision makers

> "For models to be used in impact assessment or other regulatory settings, it might be advisable to have a back-of-the-envelope version of the general model for the purpose of negotiating assumptions and inferences with stakeholders" (Saltelli et al., 2008.)

Notes: Further reading: https://en.wikipedia.org/wiki/Sensitivity_analysis

----
## Sensitivity Analysis and AI

* Models in machine learning and symbolic AI are often highly complex, hard to (fully) understand?
* Often hundreds of features (inputs) are used, but which are important?
* Features can be computed in different ways (e.g., normalization), but how robust is the model to such decisions?
* Can complicated models be replaced with simpler ones?
* How much does hyperparameter tuning affect results?
* Is the model biased by input X (e.g., gender)?
* 
* Challenge: Often lots and lots of inputs
* Opportunity: Evaluation of ML/AI models is relatively fast


----
## Sensitivity Analysis Techniques in a Nutshell

* Simple models: Analytical methods  (e.g., derivative)
* Commonly: Sampling-based approach
    - Repeated speculative "what-if" changes to observe outcome
    - Typically guided sampling strategy ("design of experiments")
    - Calculating parameter influence on outcomes
* Special handling for dependent/correlated inputs and interactions

----
## Plotted Influence of 4 Parameters

100 random samples. Which parameter ($Z_1, Z_2, Z_3, Z_4$) has the most influence on $Y$?

![](scatter.jpg)

Notes:

Source: https://en.wikipedia.org/wiki/File:Scatter_plots_for_sensitivity_analysis_bis.jpg

Sampling-based sensitivity analysis by scatterplots. Y (vertical axis) is a function of four factors. The points in the four scatterplots are always the same though sorted differently, i.e. by Z1, Z2, Z3, Z4 in turn. Note that the abscissa is different for each plot: (−5, +5) for Z1, (−8, +8) for Z2, (−10, +10) for Z3 and Z4. Z4 is most important in influencing Y as it imparts more 'shape' on Y.

----
## Exhaustive Search (Grid Search)

* Explore all combinations of all variations of all inputs
* Frequently used for hyperparameter optimization in small search spaces
* Exponential search space
* Covers all interactions, ideal for finding optimum
* Readily implemented in many frameworks
* Not feasible for most scenarios 

----
## One-at-a-time

* Sampling:
    * $S_0$ default assignment to all inputs
    * For each input, create one sample that differs from $S_0$ only in that input (or multiple)
* Compute influence as partial derivative or using linear regression
* Simple, fast, practical, but cannot discover interactions
*
* Useful also for *screening*: identifying few significant inputs
----
## Regression analysis

* Random sampling or other strategies
* Fit linear regression model over findings
    - optionally, use feature selection to keep model simple or explore interactions
    - e.g. $3·z_1+.2·z_3-14.2·z_3·z_4$
* Interpret sensitivity from model coefficients
* Simple, computationally efficient, but limited to linear relationships
*
* **General strategy: Replacing one model with a simpler, less accurate, but more explainable model.**
----
![](splconqueror0.png)

<!-- references -->

Futher reading: N. Siegmund, A. Grebhahn, C. Kästner, and S. Apel. [Performance-Influence Models for Highly Configurable Systems](https://www.cs.cmu.edu/~ckaestne/pdf/fse15_influence.pdf). In Proceedings Symposium on the Foundations of Software Engineering (ESEC/FSE), pages 284--294, 2015.

Notes:
Inputs are configuration options here and outputs performance measurements for specific configurations. In this case all inputs are boolean, 5 are shown. After sampling a set of configurations, a linear regression model can be fit to identify which options have the largest performance influence.
Note that the regression model models all options but no interactions.

----
![](splconqueror.png)

<!-- references -->

Futher reading: N. Siegmund, A. Grebhahn, C. Kästner, and S. Apel. [Performance-Influence Models for Highly Configurable Systems](https://www.cs.cmu.edu/~ckaestne/pdf/fse15_influence.pdf). In Proceedings Symposium on the Foundations of Software Engineering (ESEC/FSE), pages 284--294, 2015.

Notes:
The approach can be refined by adding interaction terms to the regression, typically as precomputed synthetic inputs. Since the number of potential interactions is quadratic for pairwise interactions and exponential for all interactions, typically only a select number of interactions is tried. This can be done with domain knowledge or with search strategies, such as trying pairwise combinations among all options that have a significant influence on their own.
----
## Sensitivity Analysis for Duplicate PR Detector

* Project: ML classifier to detect duplicate pull requests on GitHub

<!-- colstart -->
![](intrude-pr.png)
<!-- col -->
![](intrude-sa.png)
<!-- colend -->

<!-- references -->
L. Ren, S. Zhou, C. Kästner, and A. Wąsowski. [Identifying Redundancies in Fork-based Development](https://www.cs.cmu.edu/~ckaestne/pdf/saner19.pdf). In Proceedings of the 27th IEEE International Conference on Software Analysis, Evolution and Reengineering (SANER), pages 230--241, 2019.

----
## Robustness

* How much does a model depend on a specific operationalization of a feature?
    - can we measure delivery time with a different method and still get similar results?
    - does a feature (e.g. code quality) need to be very accurate for our model to work effectively?
    - how sensitive is our model to outliers?
* Evaluate model accuracy with different features or versions of feature extractors


----
## Robustness Example

Project: Explain which GitHub projects adopt badges in their repositories

Modeling "code quality": (a) linter warnings, (b) commits with "fix" in description, (c) number of tests, (d) number of test files, (e) size of test files

Output measure: model fit / stability of effects in models

<!-- colstart -->
![](badges.png)
<!-- col -->
![](badges-model.png)
<!-- colend -->

<!-- references -->

A. Trockman, S. Zhou, C. Kästner, and B. Vasilescu. [Adding Sparkle to Social Coding: An Empirical Study of Repository Badges in the npm Ecosystem](https://www.cs.cmu.edu/~ckaestne/pdf/icse18badges.pdf). In Proceedings of the 40th International Conference on Software Engineering (ICSE), pages 511--522, 2018.

Notes: 
In this data analytics study, there were multiple ways of operationalizing and measuring an outcome (same robustness analysis can be done on inputs). A simple robustness check is whether the results change drastically when using a different measure that is supposed to measure the same underlying effect.

If different proxy measures produce widely different answers, the model may pick up on characteristics of the measure more than the underlying effect. Conversely with multiple proxy measures for the same effect come to the same conclusion one can gain confidence that the results actually relate to the studied phenomenon.

----
## Sampling Strategies (Design of Experiments)

* Sampling in high-dimensional spaces has been studied in detail
* Random sampling as a baseline
* Many designs exist that can explore spaces more systematically than random sampling
* Constraints among inputs complicate all strategies

<!-- colstart -->

![](bb.gif)

([Box-Behnken](https://en.wikipedia.org/wiki/Box%E2%80%93Behnken_design))

<!-- col -->

![](pb.gif)

([Plackett-Burman](https://en.wikipedia.org/wiki/Plackett%E2%80%93Burman_design))

<!-- colend -->

----
## Sensitivity Analysis in Soccer Commentary and Summary Project?
For what purposes could sensitivity analysis be used? How?

![Soccer](soccer.jpg) 

Notes: See the case study from the previous lecture.
----
## Other Examples of Sensitivity Analysis in AI Projects?

<!-- discussion -->

Notes: Collect potential scenarios and concrete examples.

Back to the list earlier:
* Models in machine learning and symbolic AI are often highly complex, hard to (fully) understand?
* Often hundreds of features (inputs) are used, but which are important?
* Features can be computed in different ways (e.g., normalization), but how robust is the model to such decisions?
* Can complicated models be replaced with simpler ones?
* How much does hyperparameter tuning affect results?
* Is the model biased by input X (e.g., gender)?

----
## Tool Support for Sensitivity Analysis?

<!-- discussion -->




---
# Online Experimentation

----
## What if...?
 
* ... we hand plenty of subjects for experiments
* ... we could randomly assign subjects to treatment and control group without them knowing
* ... we could analyze small individual changes and keep everything else constant


▶ Ideal conditions for controlled experiments

![Amazon.com front page](amazon.png)

----
## A/B Testing for Usability

* In running system, random sample of X users are shown modified version
* Outcomes (e.g., sales, time on site) compared among groups

![A/B test example](ab-groove.jpg)

Notes: Picture source: https://www.designforfounders.com/ab-testing-examples/

----
![A/B test example](ab-prescr.jpg)

Notes: Picture source: https://www.designforfounders.com/ab-testing-examples/
----
## Experiment Size

* With enough subjects (users), we can run many many experiments
* Even very small experiments become feasible
* Toward causal inference

![A/B test example of a single button's color](ab-button.png)


----


* Introduction to the scientific method (experimental design, statistical tests, causality)
* Offline experimentation
  * Introduction to *sensitivity analysis*
  * Use cases regarding robustness, bias, performance and hyperparameter tuning, ...
  * Sampling in high-dimensional spaces
* Online experimentation
  * Testing in production, chaos engineering
  * *A/B testing*
  * Necessary statistics foundation
  * Concurrent A/B tests
* Infrastructure for experimentation, planning and tracking experiments
* Interacting with and supporting data scientists


---
# Summary

