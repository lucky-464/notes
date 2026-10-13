Panel Data Regression Models
Notes from Gujarati, Basic Econometrics, Chapter 16. Mapped to the Digital Adoption project.

1. Pooled OLS: The Lazy Estimator
Concept
Pooled OLS is the simplest thing you can do with panel data. You take every observation — every country-year — and throw it into one regression. India 2014, India 2017, Argentina 2019, all in the same pot. The regression does not know that two rows are from the same country. It treats them as independent.

The person running it is essentially being lazy. They are ignoring the panel structure that makes the data special.

The trade-off with ignoring structure
There is a plus and a minus to treating India 2014 and India 2017 as separate people.

The plus: you capture real change. Between 2014 and 2017, India went through the JIO revolution. Internet cost collapsed. Internet users exploded. Pooled OLS does pick up that variation because India 2017 is in the regression and it has different values than India 2014.

The minus: India 2014 and India 2017 share things that do not change — government structure, institutions, legal system, culture, geography. These are time-invariant within a country. The regression does not have them as variables. So they sit in the error term.

Two separate things I had to untangle
Thing 1: Error terms within a country are correlated.

India 2014 error and India 2017 error share the same India-specific unobservable. Call it α_India. That α_India is the government, institutions, legal system, culture. It is the same in 2014 and 2017 because it does not change quickly.

text
error_India_2014 = α_India + u_2014
error_India_2017 = α_India + u_2017
Both errors contain the same α_India. That makes them partially correlated across time within India. This is always true. It is mechanical. Gujarati's low Durbin-Watson statistic is the tell. This is serial correlation within the country.

Thing 2: Is that α_India correlated with the regressor?

Separate question. α_India could be correlated with internet use. Rich, well-governed countries have both better institutions and more internet. So India's institutional quality might be pulling both its internet use up and its productivity up.

If that correlation exists, the classical OLS assumption Cov(X, error) = 0 fails. The coefficient on internet use is biased and inconsistent. Not just noisy — wrong.

This is not a consequence of Thing 1. Thing 1 is always true. Thing 2 is an empirical question about the specific variables in the model.

Why this matters for my project
If I ran productivity ~ IT_lag1 + controls on 847 rows without country fixed effects, I would be doing pooled OLS. I would be assuming Albania and Argentina are interchangeable except for the controls. They are not.

Country institutions — which are time-invariant within a country — sit in the error term. If they correlate with internet use, the IT coefficient is biased.

Same for year effects. If I do not include year fixed effects, the global rise of the internet over 2011-2024 sits in the error term alongside the global rise in productivity. The coefficient picks up the common trend, not the causal effect.

Diagnostics I will look for
Low Durbin-Watson is the symptom of Thing 1. Gujarati's pooled airline regression has DW = 0.43.

Difference between pooled OLS coefficient and fixed effects coefficient is the symptom of Thing 2.

How I will use it
Run pooled OLS first as the naive benchmark. Then run fixed effects. Compare. If they differ, report it as evidence that pooled OLS is biased.

Why errors are correlated but bias is not automatic
Error terms sharing α_India are always correlated within a country. Structural. Only affects standard errors, not point estimates.

Bias requires the extra step: Cov(α_India, internet) ≠ 0. Only then is the point estimate itself wrong.

Vocabulary
Endogenous: unobserved effect is correlated with the regressor. Bad for pooled OLS.

Exogenous: unobserved effect is uncorrelated with the regressor.

Serial correlation: errors within a unit are correlated across time. Fixed by clustering.

Bias: point estimate is wrong. Present only if Cov(α, X) ≠ 0. Fixed by fixed effects.

2. LSDV: Giving Each Country Its Own Intercept
Concept
LSDV stands for Least Squares Dummy Variable. Give each unit its own intercept.

Pooled OLS gave every country-year row the same baseline. LSDV gives every country its own baseline. Albania starts at α_Albania. Argentina starts at α_Argentina. Same slopes for all countries, different intercepts.

You implement this by adding dummy variables. One dummy per unit, except the first (the base). With 6 airlines, 5 dummies. With 86 countries, 85 dummies.

Why this matters for my project
If I ran productivity ~ IT_lag1 + controls with a single intercept, I would be doing pooled OLS. Albania and Argentina get the same baseline. But their baseline productivity is not the same.

If those country characteristics are correlated with internet use — and they are, because richer, better-governed countries have both more internet and more productivity — the IT coefficient is biased. LSDV absorbs those characteristics into the country intercept.

What actually changes
Gujarati's airlines: output coefficient goes from 2.03 to 3.32 million. Fuel goes from 1.225 to 0.773. That is a different answer, not just a tighter one.

For my project: when I run pooled OLS vs country FE, I expect IT_lag1 coefficient to shift. If it does, that shift is the bias.

Why the shift happens
In pooled OLS, the country-specific unobservable sat in the error term. In LSDV, it is absorbed into the intercept. The regressor is no longer contaminated by it.

Cost of LSDV
One parameter per unit. With 86 countries, 85 country intercepts plus 13 year intercepts plus slopes. PanelOLS handles it without dummies written by hand, but the cost is real.

Bigger cost: time-invariant variables die. "Is this country in Africa" or "was this country a former colony" is constant within a country. Country dummies absorb it completely. The coefficient is not identified.

The F-test
Gujarati tests whether the country intercepts matter. Null: all intercepts are equal. His F = 14.99, highly significant. The restriction fails. Pooled OLS is the wrong model.

I can run a similar test comparing pooled OLS to country FE.

Code anchor
PanelOLS with entity_effects=True, time_effects=True, cluster_entity=True.

Vocabulary
LSDV: fixed effects via explicit dummies.

Differential intercept: coefficient on a dummy, measures how much a unit differs from the base.

Dummy variable trap: perfect collinearity from a dummy for every unit plus a common intercept.

Time-invariant variable: constant within a unit. Absorbed by unit FE.

3. Within-Group Estimator: Same Answer, Cheaper Route
Concept
LSDV gave every country its own intercept via dummies. Within-group does the same job without estimating any dummy. Subtract each country's own mean from every variable, then run OLS on the de-meaned data.

The country intercept is a constant. When I compute the country mean, that mean contains the intercept. When I subtract the mean, the intercept goes with it. Not estimated, just gone.

What de-meaning looks like
Albania's productivity: 38,000, 40,000, 42,000, 41,000, 44,000. Mean is 41,000.

Subtract 41,000 from each row: −3,000, −1,000, +1,000, 0, +3,000.

Now every Albania row is a deviation from Albania's own mean. The average of the de-meaned values is zero.

Why the country effect disappears
Albania's intercept is the same in every year. When I subtract Albania's mean, the intercept is subtracted too.

So the within-group estimator asks: when Albania's internet use is above its own average, is its productivity also above its own average? That is within-country variation.

Why the slopes are identical to LSDV
LSDV absorbs the intercept with a dummy. Within-group absorbs it by subtracting the mean. Both remove the same thing. What remains is the same de-meaned data. So OLS gives the same slopes.

Why it is cheaper
LSDV estimates 5 dummies in the airline example. Within-group estimates 0. Same slopes. Big deal when N is large.

For my project: LSDV would estimate 85 country dummies plus 13 year dummies. Within-group estimates none.

Why PanelOLS(entity_effects=True) is the within-group estimator
I never write dummies. I never de-mean by hand. The function does it internally. That is what "entity effects" means.

R's fixest, Stata's reghdfe, Python's PanelOLS — three packages, three names, one estimator.

Disadvantage 1: time-invariant variables die
Country FE subtracts the country mean from every variable. If a variable is constant within a country, its value equals its mean. Subtract, get zero. Column of zeros cannot explain anything.

Example: "is this country a former British colony?" India is 1 in every year. After de-meaning, every India row for that variable is 0. Coefficient not identified.

Same for region, legal origin, geography, latitude.

README limitations line: "Country fixed effects absorb all time-invariant country characteristics. Region, legal origin, colonial history, and geography are controlled for but not separately identified."

Disadvantage 2: long-run effects get removed
Imagine a country with internet use growing by exactly 2 points every year:

text
2011: 20, 2012: 22, 2013: 24, ..., 2020: 38
Mean is 29. De-mean: −9, −7, −5, −3, −1, +1, +3, +5, +7, +9.

The steady 2-point growth is gone. What remains is position relative to the country's own average.

So the coefficient I estimate is not "the effect of growing internet use." It is "the effect of being above or below your own average internet use in a given year."

What this means for my claim
Cannot say: "A permanent increase in internet adoption raises productivity by X."

Can say: "A deviation from a country's own trend in internet adoption is associated with a deviation in productivity of X."

The honest sentence
Within countries, a 1 percentage point deviation above a country's own internet-adoption trend is associated with a 0.4 percent deviation above that country's own productivity trend. This is a short-run, within-country association, not the effect of a permanent increase.

The sentence to avoid
A 1 percentage point increase in internet adoption raises productivity by 0.4 percent.

That overstates three ways. Causes vs associates. Level vs deviation. Long-run vs short-run.

Rule for every FE coefficient
Include "within countries," include "deviation from trend" or "change," use "associated with" not "causes."

First-difference, briefly
Instead of subtracting the country mean, subtract last year's value from this year's. Same purpose, different math. When T = 2, first-difference and within-group are identical. When T > 2, they differ.

I will not use it. My panel has up to 14 years. But I should know the name.

Vocabulary
Within-group: fixed effects estimator via de-meaning.

De-meaning: subtracting each unit's mean from every variable.

Time-invariant variable: constant within a unit. Absorbed by FE.

Short-run deviation effect: what FE estimates.

Project relevance
PanelOLS(entity_effects=True, time_effects=True, cluster_entity=True) is the within-group estimator. The coefficient on IT_lag1 is a within-country, short-run deviation effect. Every FE coefficient I report will include "within countries," "deviation from trend," and "associated with."

4. Random Effects Model
Concept
Fixed effects treated each country's intercept as a fixed number. Random effects treats each country's intercept as a random draw from a distribution of country intercepts.

Instead of estimating 86 separate intercepts, RE estimates a small number of distributional parameters: mean intercept, variance of the country component, variance of the idiosyncratic component. Three parameters instead of 85.

Why this is attractive
Degrees of freedom: FE uses N−1 degrees of freedom just on intercepts. RE uses 3.

Efficiency: when RE assumptions hold, tighter standard errors than FE.

The catch
RE assumes the country intercepts are uncorrelated with the regressors. If it fails, RE is biased. FE does not need that assumption.

Why this assumption fails in my project
Albania's institutional quality is a country intercept. Albania's internet use is a regressor. Better-governed countries have more internet. Same institutional quality drives both.

My prior: RE assumption fails. RE coefficient is biased. FE is preferred.

Hausman test
Compares FE and RE coefficients. Null: they are the same (RE assumption holds). Alternative: they differ (RE is biased).

Chi-squared. Small p-value means reject. Reject means use FE.

My expectation: reject RE. That justifies my choice of FE.

Code anchor
PanelOLS(entity_effects=True) for FE. RandomEffects for RE. Hausman via linearmodels.panel.compare.

Vocabulary
Random effects: country intercepts as draws from a distribution.

Hausman test: compares FE and RE coefficients.

Distributional parameters: mean and variances, instead of N intercepts.

Efficiency: tighter standard errors when assumptions hold.

5. Properties of Estimators: Which One Do I Trust?
The key word: consistency
Consistency means: as the sample grows, the estimate converges to the truth. If inconsistent, no amount of data saves you. The estimate is wrong even asymptotically.

The three estimators
Pooled OLS. Consistent only if the error term is uncorrelated with the regressors. If the country-specific unobservable correlates with internet use, pooled OLS is inconsistent.

Fixed effects. Consistent. Always. Regardless of whether the true model is pooled, fixed, or random.

Random effects. Consistent if the true model is pooled or random. Inconsistent if the true model is fixed effects.

The table
True model    Pooled OLS    FE    RE
Pooled    Consistent    Consistent    Consistent
FE    Inconsistent    Consistent    Inconsistent
RE    Consistent    Consistent    Consistent
FE is consistent in every row. Pooled OLS and RE fail in the row where the true model has fixed effects — which is the case in cross-country macro.

The tradeoff
FE is always consistent but less efficient than RE when the RE assumption holds. RE gives precision but only if I believe the assumption. In cross-country macro, the assumption rarely holds. I take the precision hit.

The pragmatic argument
Even if the true model is pooled or random, running FE is still fine. Lose efficiency, gain robustness. FE handles all cases.

Vocabulary
Consistency: estimator converges to truth as sample grows.

Efficiency: smaller standard errors, given the same consistency.

Safe choice: FE.

Precision hit: the cost of using FE instead of RE.

Project relevance
This section justifies my specification. I use FE because it is always consistent. I do not have to prove the RE assumption fails. I just have to show FE is safe.

6. FE vs RE: Practical Guidelines
The core question
Is the country-specific unobservable correlated with the regressors?

Yes → FE.

No → RE is fine and more efficient.

Three guidelines that apply to my project
Short panel. When N is large and T is small, FE and RE can differ a lot. Which to use depends on whether the 86 countries are treated as random draws from a larger population. They are not. They are the countries with data. So FE.

The killer. If the country unobservable correlates with any regressor, RE is biased, FE is not. In my project, institutional quality correlates with internet use. RE is biased.

The warning. Gujarati quotes Johnston and DiNardo: "there is no simple rule to help the researcher navigate past the Scylla of fixed effects and the Charybdis of measurement error and dynamic selection."

Translation: FE handles one problem — correlated country unobservables. It does not handle measurement error or reverse causality. Those go in limitations.

One guideline that does not bind here
Time-invariant variables. FE cannot estimate them. RE can. But I do not care about estimating region, legal origin, or colonial history. They are controls, not the focus.

One sentence summary
In a short panel of countries that are not random draws, and where country unobservables likely correlate with the regressors, use FE. RE is only appropriate when that correlation is absent, which is rare in macro. FE does not fix everything — measurement error and reverse causality remain — but it fixes the specific problem of correlated country effects.

Vocabulary
Short panel: N > T.

Correlated random effects: the situation RE assumes away.

Scylla and Charybdis: Gujarati's way of saying no perfect estimator exists.

Project relevance
I cite guidelines 2 and 3 in the README. I cite the closing warning in the limitations section.


