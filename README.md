# A/B Testing Final Project


## Experiment Overview: Free Trial Screener

At the time of this experiment, Udacity courses currently have two options on the home page: "start free trial", and "access course materials". If the student clicks "start free trial", they will be asked to enter their credit card information, and then they will be enrolled in a free trial for the paid version of the course. After 14 days, they will automatically be charged unless they cancel first. If the student clicks "access course materials", they will be able to view the videos and take the quizzes for free, but they will not receive coaching support or a verified certificate, and they will not submit their final project for feedback.

In the experiment, Udacity tested a change where if the student clicked "start free trial", they were asked how much time they had available to devote to the course. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. If they indicated fewer than 5 hours per week, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead. [This screenshot](https://drive.google.com/file/d/0ByAfiG8HpNUMakVrS0s4cGN2TjQ/view) shows what the experiment looks like.

The hypothesis was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time without significantly reducing the number of students to continue past the free trial and eventually complete the course. If this hypothesis held true, Udacity could improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course.

The unit of diversion is a cookie, although if the student enrolls in the free trial, they are tracked by user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

## Metric Choice

There are several metrics to choose from as either invariant or evaluation metrics. The practical significance boundary for each metric, that is, the difference that would have to be observed before that was a meaningful change for the business, is given in parentheses. All practical significance boundaries are given as absolute changes. The metrics include:

* **Number of cookies**: That is, number of unique cookies to view the course overview page *(dmin=3000)*. This metrics serves as **invariant metric** as users in the experimental group do not see the change yet when the view the course page, therefore they are not affected by the change at this stage.

* **Number of user-ids**: That is, number of users who enroll in the free trial *(dmin=50)*. The metric **is not invariant** as user-ids are tracked after the enrollment. In my opinion it also does not serve as a good evaluation metric, as number of user-ids depends on the number of "start free trial" clicks, which is affected by the change in rendering. This dependency might obscure our results and another metrics below are more preferable.

* **Number of clicks**: That is, number of unique cookies to click the "Start free trial" button *(dmin=240)*. As with the number of cookies, this metric is not affected by the treatment as it is measured before the free trial screener is triggered. Therefore, it serves as an **invariant metric**.

* **Click-through-probability** (CTP): That is, number of unique cookies to click the "Start free trial" button divided by number of unique cookies to view the course overview page *(dmin=0.01)*. Another **invariant metric** in the experiment, because CTP does not depend on the treatment, because users have not seen the trigger-page before the click the button.

* **Gross conversion**: That is, number of user-ids to complete checkout and enroll in the free trial divided by number of unique cookies to click the "Start free trial" button *(dmin= 0.01)*. I preliminarly choose it to be an **evaluation metric**, because the value is exposed to treatment. `(number of enrolments / number of cookies to click)`

* **Retention**: That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by number of user-ids to complete checkout *(dmin=0.01)*. Similarly, I choose this one to be an **evaluation metric** as it potentially might show whether more people are staying after the trial period, if the treatment is applied. `(number of paid users / number of enrollemnts)`

* **Net conversion**: That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by the number of unique cookies to click the "Start free trial" button *(dmin= 0.0075)*. This metric measures whether the treatment helps to convert users who clicked on the button into committed students. Therefore, it serves as an **evaluation metric**. `(number of paid users / number of cookies to click)`

### To summarize:
* **Invariant metrics**: **Number of cookies**, **Number of clicks** and **Click-through-probability**.

* **Evaluation metrics**: **Gross conversion**, **Retention** and **Net conversion**.



## Measuring Variability


As a first step I compute analytical variability of the evaluation metrics. Assuming the binomial distribution of the number of clicks and enrollments, we use the following formula to compute standard deviations: *sigma = sqrt(p(1-p)/N)*, where *p* is the baseline probability value and *N* is the number of clicks or enrollments, depending on the metric.

| Evaluation Metric | Baseline Prob. | N | Analytical Standard Deviation |
|:-------------------:|:--------------------:|:--------------------:|:--------------------:|
| Gross Conversion  | 0.2063 |400| 0.0202 |
| Retention         | 0.5300 |82.5| 0.0549 |
| Net Conversion    | 0.1093 |400| 0.0156 |

N values are obtained by calculating a scaling factor for the given sample size of 5000 cookies visiting the course overview page: `5000 / 40000 = 0.125`

In terms of comparing analytical versus empirical estimates of variability, they are likely to coincide for gross conversion and net conversion as the unit of analysis and unit of diversion are the same in both cases: cookies. As for retention, analytical and empirical estimates are not likely to concide as the unit of analysis is different here: number of enrollments.

## Sizing

### Choosing Number of Samples given Power

I chose not to use Bonferroni correction initially, because this method is too conservative.

I compute the number of pageviews needed for each metric for each group with the help of an [online calculator](http://www.evanmiller.org/ab-testing/sample-size.html), using `alpha = 5%` and `100% - beta = 80%`. The baseline conversion rates and practical significance boundary (*dmin*) are seen in the table below. The result needs to be doubled as we have 2 groups.

| | Gross Conversion | Retention | Net Conversion |
|:-------------------:|:--------------------:|:--------------------:|:--------------------:|
| conversion  | 20.625% | 53% | 10.9313% |
| dmin | 1% | 5% | 0.75% |
| alpha | 5% | 5% | 5% |
| 1 - beta | 80% | 80% | 80% |
| sample size | 25,835 | 39,155 | 27,413 |
| total size | 51,670 | 78,230 | 54,826 |
| scaling factor | 0.08 | 0.0165 | 0.08 |
| pageviews needed | 645,875 | **4,741,212** | 685,325 |

Clearly, almost 5mln pageviews required for the retention metric is too much and I therefore drop it.

### Choosing Duration vs. Exposure

Given that daily traffic is 40,000 pageviews, allocating **50%** of the traffic to the experiment would require ca. `685,325 / 20,000 = 35` days of exposure. In this case 25% of the users will be exposed to the change, which is a considerable amount. However, the change is not dramatic and is not likely make a catastrophic impact. On the other hand, exposing more users allows to reduce the duration of the experiment, something that is highly desirable.


## Analysis

The experimental data is found [here](https://docs.google.com/spreadsheets/d/1Mu5u9GrybDdska-ljPXyBjTpdZIUev_6i7t4LRDfXM8/edit#gid=0).

### Sanity Checks

Assuming binomial distribution with probability 0.5 the standard deviation is `sqrt(0.5 * 0.5 / (N_c + N_e))` and margin of error is `z-score * st.dev.`. Results of calculation for invariant metrics are presented below:

| | # of cookies | # of clicks | 
|:-------------------:|:--------------------:|:--------------------:|
| N_c  | 345543 | 28378 | 
| N_e | 344660 | 28325 | 
| alpha | 5% | 5% | 5% |
| z-score | 1.96 | 1.96 | 
| st.dev | 0.0006018 | 0.0021 |
| margin | 0.0011796 | 0.0041 |
| lower bound | 0.4988 | 0.4959 |
| upper bound | 0.5012 | 0.5041 |
| observed | 0.5006 | 0.5005 |
| passed? | yes | yes |

As for the click-through-rate (CTR), we expect to get a similar value in both groups. Computing the observed value for the control, I construct a confidence interval and then check if the observed value for the experimental group falls within it:

* CTR_c = 28378 / 345543 = 0.0821
* st.dev = sqrt(0.0821(1-0.0821)/345543) = 0.00047
* margin = 1.96 * 0.00047 = 0.000915
* lower bound = 0.0811
* upper bound = 0.0830
* CTR_e = 0.0822

Thus, CTR also passes the sanity check and I can proceed with analysing evaluation metrics.

### Check for Practical and Statistical Significance

The notation used below is the following:

* Nc and Ne - sample sizes of control/experimental groups (denominator)
* Xc and Xe - number of target values (nominator)
* pp - pooled probability
* sterror_p - pooled standard error
* diff - difference in metrics between groups

Then I compute standard errors, margin of errors and confidence intervals using the formulas:

* pp = (Xc + Xe) / (Nc + Ne)
* sterror_p = sqrt(pp * (1-pp) * (1/Nc + 1/Ne))
* d = Xe / Ne - Xc / Nc

The results for both evaluation metrics are:

| | Gross Conversion | Net Conversion | 
|:-------------------:|:--------------------:|:--------------------:|
| Nc  | 17293 | 17293 | 
| Ne | 17260 | 17260 | 
| Xc  | 3785 | 2033 | 
| Xe | 3423 | 1945 | 
| pp | 0.2086 | 0.1151 |
| sterror_p | 0.00437 | 0.00343 |
| lower bound |  -0.0291 | -0.0116 |
| upper bound | -0.0120 | 0.0019 |
| dmin | abs(0.01) | abs(0.0075) |
| diff | -0.02055 | -0.0048 |
| significant? | yes | no |
| practical? | yes | no |

### Sign Tests

For sign tests the [online calculator](http://graphpad.com/quickcalcs/binomial1.cfm) is used.

#### Gross Conversion
Out of 23 days of experiment we observe 4 instances of lower gross conversion in the experimental group, meaning fewer students decide to enroll if asked the question. Assuming the probability of 0.5, the p-value is 0.0026, thus lower than required 0.05. Therefore, the null hypothesis of no change is rejected in the sign test for gross conversion.

#### Net Conversion
Out of 23 days there are 10 instances of lower net conversion in the experimental group. The p-value therefore is 0.6776 and the change is not significant at 95% confidence level.

## Summary

In the analysis, I choose not to use Bonferroni correction as the method might be too conservative in the presence of high correlation between metrics.
The results of both size and sign tests indicate that treatment had statistically and practically significant effects on the gross conversion. The opposite is true for the net conversion.

## Recommendation

Based on the results of the analysis, **I would not recommend launching the change**. The question-screen was effective at reducing the number of students who click and enroll, but it was **not effective in increasing the amount of paying students**. Actually, there appeared to be an opposite effect, albeit statistically and practically insignificant.

## Follow-Up Experiment

A minor change could be potentially added to the present experiment. Namely, in the experimental group the question screen could be modified as to remind the potential long-term students that the amount of effective tuition paid depends on them to a large extent. Also a remaider that 50% of the tuition will be returned if the degree is completed within a short period of time (6 months). 

This might additionally motivate students, who need a little push and are concerned about the cost of tuition. The idea came to me because it was one of the reasons I decided to sign up for the degree.

In this iteration, the same metrics and the unit of diversion could still be chosen.
