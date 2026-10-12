
Advantages, mapped to the Digital Adoption project.

Heterogeneity across units. Albania is not Argentina. Country fixed effects absorb time-invariant country characteristics — geography, legal system, colonial history — so the estimate is not contaminated by differences between countries.

More data and variation. 86 countries × ~10 years = 876 rows. Two sources of variation: across countries and within countries over time. Less collinearity between controls, more degrees of freedom.

Better suited to study change. The research question is whether a rise in internet use raises productivity. That is a change question. Only panel data can capture it.

Detects effects invisible in pure cross-section or time series. A cross-section of 86 countries in 2024 would only show "rich countries have more internet." It could not isolate the effect of a change. Panel data can.

Handles multi-year adjustment. Digital adoption does not affect productivity instantly. Workers learn, firms reorganise. The lagged regressor IT_lag1 allows for this.
Code anchor in the notebook.

panel_task2.csv — 86 countries, 847 rows

IT_lag1 — lagged digital adoption, non-null for 761 rows

gender_gap — male minus female internet use

Panel Data: An Illustrative Example
Concept. Gujarati sets up the running example: six airline firms, 1970-1984, 90 observations. Balanced panel. Introduces the four estimator options.

Vocabulary that applies to the project.

Balanced panel. Every unit has the same number of observations. Gujarati's airlines: 6 × 15 = 90, complete.

Unbalanced panel. Units have different numbers of observations. The Digital Adoption panel is unbalanced: 86 countries, 3 to 14 years each, 847 rows.

Short panel. N > T. Digital Adoption: 86 countries, 14 years. Short.

Long panel. T > N. Gujarati's airlines: 15 years, 6 firms. Long.

Four estimator options.

Pooled OLS — ignores panel structure. Used as biased benchmark.

LSDV — fixed effects via dummies. Equivalent to within-group, consumes N degrees of freedom.

Within-group — de-mean by unit, run OLS. This is what PanelOLS(entity_effects=True) does.

Random effects — intercepts as random draws. Rejected via Hausman test in favour of FE.

Code anchor.

PanelOLS with entity_effects=True, time_effects=True, cluster_entity=True

panel_task2.csv — 847 rows, 86 countries

Implication for the project. Gujarati's formulas assume balance. Yours is unbalanced. Coefficients are still consistent, but degrees of freedom accounting and some diagnostics differ.

Pooled OLS: The Lazy Estimator

Pooled OLS is the simplest thing you can do with panel data. You take every observation — every country-year — and throw it into one regression. India 2014, India 2017, Argentina 2019, all in the same pot. The regression does not know that two rows are from the same country. It treats them as independent.

The person running it is essentially being lazy. They are ignoring the panel structure that makes the data special.

The trade-off with ignoring structure.

There is a plus and a minus to treating India 2014 and India 2017 as separate people.

The plus: you capture real change. Between 2014 and 2017, India went through the JIO revolution. Internet cost collapsed. Internet users exploded. Pooled OLS does pick up that variation because India 2017 is in the regression and it has different values than India 2014.

The minus: India 2014 and India 2017 share things that do not change — government structure, institutions, legal system, culture, geography. These are time-invariant within a country. The regression does not have them as variables. So they sit in the error term.

Two separate things I had to untangle.

Thing 1: Error terms within a country are correlated.

India 2014 error and India 2017 error share the same India-specific unobservable. Call it α_India. That α_India is the government, institutions, legal system, culture. It is the same in 2014 and 2017 because it does not change quickly.

So:

error_India_2014 = α_India + u_2014
error_India_2017 = α_India + u_2017

Both errors contain the same α_India. That makes them partially correlated across time within India. This is always true. It is mechanical. Gujarati's low Durbin-Watson statistic is the tell. This is serial correlation within the country.

Thing 2: Is that α_India correlated with the regressor?

Separate question. α_India could be correlated with internet use. Rich, well-governed countries have both better institutions AND more internet. So India's institutional quality might be pulling both its internet use up and its productivity up.

If that correlation exists, the classical OLS assumption Cov(X, error) = 0 fails. The coefficient on internet use is biased and inconsistent. Not just noisy — wrong.

This is not a consequence of Thing 1. Thing 1 is always true. Thing 2 is an empirical question about the specific variables in the model.

Why this matters for my project.

If I ran productivity ~ IT_lag1 + controls on 847 rows without country fixed effects, I would be doing pooled OLS. I would be assuming Albania and Argentina are interchangeable except for the controls. They are not.

Country institutions — which are time-invariant within a country — sit in the error term. If they correlate with internet use (plausible in cross-country macro), the IT coefficient is biased.

Same for year effects. If I do not include year fixed effects, the global rise of the internet over 2011-2024 sits in the error term alongside the global rise in productivity. The coefficient picks up the common trend, not the causal effect.

Diagnostics I will look for.

Low Durbin-Watson is the symptom of Thing 1. Gujarati's pooled airline regression has DW = 0.43. Very low.

Difference between pooled OLS coefficient and fixed effects coefficient is the symptom of Thing 2. If they differ a lot, bias was present.

How I will use it.

Run pooled OLS first as the naive benchmark. Then run fixed effects. Compare. If they differ, I report it as evidence that pooled OLS is biased, and fixed effects is preferred.

Why errors are correlated but bias is not automatic.

Error terms sharing α_India are always correlated within a country. That is structural. It only affects the standard errors, not the point estimates.

Bias requires the extra step: Cov(α_India, internet) ≠ 0. Only then is the point estimate itself wrong. Gujarati asserts this for the airline case. I assert it for the country case. In both, fixed effects is the fix.

Vocabulary I picked up.

Endogenous: unobserved effect is correlated with the regressor. Bad for pooled OLS.

Exogenous: unobserved effect is uncorrelated with the regressor. Pooled OLS is fine in this case.

Serial correlation: errors within a unit are correlated across time. Always present if α is ignored. Fixed by clustering.

Bias: point estimate is wrong. Present only if Cov(α, X) ≠ 0. Fixed by fixed effects.

LSDV: Giving Each Country Its Own Intercept

LSDV stands for Least Squares Dummy Variable. The move is simple: give each unit its own intercept.

Pooled OLS gave every country-year row the same baseline. LSDV gives every country its own baseline. Albania starts at α_Albania. Argentina starts at α_Argentina. Same slopes for all countries, different intercepts.

You implement this by adding dummy variables. One dummy per unit, except the first. The first is the base. With 6 airlines, that is 5 dummies. With 86 countries, that would be 85 dummies.

Why this matters for my project.

If I ran productivity ~ IT_lag1 + controls with a single intercept, I would be doing pooled OLS. Albania and Argentina get the same baseline. But their baseline productivity is not the same. Albania's institutions, geography, legal system, culture are different from Argentina's. Those things do not change much year to year, so they cannot be captured by year controls. They sit in the error term.

If those country characteristics are correlated with internet use — and they are, because richer, better-governed countries have both more internet and more productivity — the IT coefficient is biased. LSDV absorbs those characteristics into the country intercept. The IT coefficient stops being contaminated.
What actually changes.

Gujarati's airlines: the coefficients on output, fuel, and load all shift compared to pooled OLS. Not tiny shifts. Output goes from 2.03 to 3.32 million. Fuel goes from 1.225 to 0.773. That is a different answer, not just a tighter one.

For my project: when I run pooled OLS vs country FE, I expect IT_lag1 coefficient to shift. If it does, that shift is the bias. I will report both and say pooled OLS is biased.

Why the shift happens, in my mental model.

In pooled OLS, the country-specific unobservable sat in the error term. Because India 2014 and India 2017 share that unobservable, the errors for those rows are correlated. That is Thing 1 from the 16.3 notes.

But bias requires more. It requires that the unobservable is also correlated with the regressor. If India's institutional quality is correlated with its internet use, then the regression is picking up "India is well-governed" and calling it "India has internet." That is Thing 2. LSDV cuts the link by absorbing the unobservable into the intercept.
Cost of LSDV.

One parameter per unit. With 86 countries, that is 85 country intercepts plus 13 year intercepts plus the slopes. Lots of parameters, lots of degrees of freedom used. PanelOLS handles it without me writing dummies by hand, but the cost is real.

The bigger cost: time-invariant variables die. If I wanted to estimate the effect of "is this country in Africa" or "was this country a former colony," those do not change within a country. The country dummies would absorb them completely. The coefficients would not be identified. I can control for them implicitly, but I cannot estimate them.

This matters for my project. Any country characteristic that does not vary over time is not identified once I add country FE. I should say this in the limitations section.

The F-test.

Gujarati tests whether the country intercepts actually matter. Null: all country intercepts are equal. If true, pooled OLS is fine. If false, LSDV is needed. His F is 14.99, highly significant. The restriction fails.

For my project: I can run a similar test. Compare pooled OLS to country FE. The F-test (or the corresponding comparison) will tell me whether the country dummies matter. My prior is yes, they do. But I should verify.

How I will use this in the assignment.

Pooled OLS is the benchmark. Country FE is the preferred specification. The gap between them is the bias. I will report both and say: pooled OLS is biased because country unobservables are correlated with internet use, FE removes that bias, the coefficient shifts from X to Y.

The code anchor: PanelOLS with entity_effects=True, time_effects=True, cluster_entity=True. The last is for standard errors. The first two are the  LSDV equivalent.

Within-Group Estimator: Same Answer, Cheaper Route

LSDV gave every country its own intercept by adding dummy variables. Six airlines, five dummies. 86 countries, 85 dummies. It works, but it is expensive. Every dummy is one estimated parameter.

Within-group does the same job without estimating any dummy. The trick is to subtract each country's own mean from every variable. Then run OLS on the de-meaned data.

The country intercept is a constant. It is the same in every year for that country. When I compute the country mean, that mean contains the intercept. When I subtract the mean, the intercept goes with it. Not estimated, just gone.

What de-meaning looks like with India.

Take Albania's productivity: 38,000, 40,000, 42,000, 41,000, 44,000 over five years. The mean is 41,000.

Subtract 41,000 from each row: −3,000, −1,000, +1,000, 0, +3,000.

Now every Albania row is a deviation from Albania's own mean. The average of the de-meaned values is zero.

Do the same for internet use, GDP, trade, every variable. Then run OLS.

Why the country effect disappears.

Albania's intercept is the same in every year. When I subtract Albania's mean, the intercept is subtracted too. It is not in the regression anymore.

So the within-group estimator asks: when Albania's internet use is above its own average, is its productivity also above its own average? That is within-country variation. That is the question FE answers.

Why the slopes are identical to LSDV.

LSDV absorbs the intercept with a dummy. Within-group absorbs the intercept by subtracting the mean. Both remove the same thing. What remains is the same de-meaned data. So OLS gives the same slopes.

Gujarati's tables 16.3 and 16.4: identical coefficients on Q, PF, LF. Only the intercepts differ in how they are reported.

Why it is cheaper.

LSDV estimates 5 dummies in the airline example. Within-group estimates 0. Same slopes. Big deal when N is large.

For my project: LSDV would estimate 85 country dummies plus 13 year dummies. Within-group estimates none. It subtracts country means and year means, runs OLS on what is left. Same slopes, way fewer parameters.

Why PanelOLS(entity_effects=True) is really the within-group estimator.

I never write dummies. I never de-mean by hand. The function does it internally. That is what "entity effects" means. It is the within-group estimator under a friendly name.

R's fixest says "country fixed effects." Stata's reghdfe says "absorb country." Three packages, three names, one estimator. Same slopes.

Disadvantage 1: time-invariant variables die.

Country FE subtracts the country mean from every variable. If a variable is constant within a country, its value equals its mean. Subtract, get zero. Column of zeros cannot explain anything.

Example: "is this country a former British colony?" India is 1 in 2011, 1 in 2015, 1 in 2024. Constant. After de-meaning, every India row for that variable is 0. Coefficient not identified.

So country FE controls for colonial history implicitly, because India's intercept contains it. But I cannot report a coefficient for colonial history. It is absorbed.

Same for region, legal origin, geography, latitude. All constant. All absorbed.

For the README limitations section: "Country fixed effects absorb all time-invariant country characteristics. Region, legal origin, colonial history, and geography are controlled for but not separately identified."

Disadvantage 2: long-run effects get removed.

Subtle but important. Let me build it.

Imagine a country with internet use growing by exactly 2 points every year:

text
2011: 20, 2012: 22, 2013: 24, ..., 2020: 38
Mean is 29. De-mean: −9, −7, −5, −3, −1, +1, +3, +5, +7, +9.

The steady 2-point growth is gone. What remains is position relative to the country's own average. The long-run trend was inside the mean, and the mean was subtracted.

So the coefficient I estimate is not "the effect of growing internet use." It is "the effect of being above or below your own average internet use in a given year."

What this means for my claim.

I cannot say: "A permanent increase in internet adoption raises productivity by X."

I can say: "A deviation from a country's own trend in internet adoption is associated with a deviation in productivity of X."

Narrower but honest. The FE coefficient is a within-country deviation effect.

If the coefficient on IT_lag1 is 0.4, the honest sentence is:

"Within countries, a 1 percentage point deviation above a country's own internet-adoption trend is associated with a 0.4 percent deviation above that country's own productivity trend. This is a short-run, within-country association, not the effect of a permanent increase."

The sentence I must avoid:

"A 1 percentage point increase in internet adoption raises productivity by 0.4 percent."

That overstates three ways. Causes versus associates. Level versus deviation. Long-run versus short-run.

Rule for every FE coefficient I report: include "within countries," include "deviation from trend" or "change," use "associated with" not "causes." All three, or it is overclaiming.

First-difference, briefly.

Gujarati mentions an alternative: first-difference. Instead of subtracting the country mean, subtract last year's value from this year's.

Albania 2021 minus Albania 2020

Albania 2022 minus Albania 2021

Same purpose. Removes the country level. But different math. When T = 2, first-difference and within-group are identical. When T > 2, they differ.

I will not use it. My panel has up to 14 years. But I should know the name.

One sentence summary.

Within-group gives the same slopes as LSDV without estimating any dummies. It subtracts each country's mean from every variable and runs OLS on what is left. It is what PanelOLS(entity_effects=True) does internally.

Vocabulary I picked up.

Within-group: fixed effects estimator via de-meaning.

De-meaning: subtracting each unit's mean from every variable.

Time-invariant variable: constant within a unit. Gets absorbed by FE.

Short-run deviation effect: the coefficient FE estimates. Not a long-run level effect.

Project relevance.

PanelOLS(entity_effects=True, time_effects=True, cluster_entity=True) is the within-group estimator. I do not write dummies. I do not de-mean by hand. But this is what the function is doing under the hood.

The coefficient on IT_lag1 is a within-country, short-run deviation effect. My README will say so. Every FE coefficient I report will include "within countries," "deviation from trend," and "associated with." No exceptions.

Country FE absorbs region, legal origin, colonial history, geography. I cannot estimate their effects. I state this in the limitations section
