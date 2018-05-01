---
title: "Coming to $terms in R"
date: 2018-04-15T16:16:07-07:00
author: Steven Slezak
draft: true
---

A recent analysis I worked on involved building a log regression and some ensemble methods using a data set with about 25 features, in addition to the target. It was an analysis of customer churn in the telecom industry. If you are interested, you can find the problem statement [here][1], the annotated code [here][2], and the raw code in [my GitHub repository][3].

The problem I ran into came up later, when I wanted to reproduce a step in this analysis. I forgot the *R* command I used! [*@DarrinLRogers*][4] came through with the reminder so I thought I should write it all down in a post. You know, for next time I forget.

>Working with Regression Models

I was doing some feature selection and feature engineering, so I expected to reduce the number of features significantly. Certain features were highly correlated, and there was also some redundancy, both of which meant pruning back the number of features. The number increased to 32, by the way, because almost all features had to be replaced with dummy variables.

I took advantage of the neat shortcut in *R* that allows you to use the `Y ~ .,` argument to pass all the features into a regression function. I was using `glm()` but it is the same syntax for `lm()`. The code looked like this:

    LogModel2 <- glm(Churn ~ ., family = binomial(link = 'logit'), data = training)
    

Running the `summary()` command on a `glm` or `lm` object gives an indication of the relative importance of the features in the data set, which are now terms in the regression equation. Importance is given by the number of asterisks to the right of the column of p-values. You can use these to determine which terms to prune from the equation.

A similar selection analysis can be done running the `anova()` command. The results won't necessarily be the same as the `summary()` command's so comparisons can be helpful.

About 12 terms had to be cut out of the regression model. They weren't adding much to performance. Having a simpler model is best, so cutting out these features made sense. But with that many to remove, rewriting the `glm()` command gets kind of tedious. Fortunately, I came across an *R* tip I found very useful. It made it possible to output the full regression model *with all terms expressed*, not abbreviated in the `Y ~ .,` syntax.

All I had to do was copy the full equation and remove the terms I didn't need. I ended up with this:

    LogModel2 <- glm(Churn ~ TotalCharges + PhoneService_Yes + MultipleLines_Yes + InternetService_DSL + 
                             `InternetService_Fiber optic` + OnlineSecurity_Yes + 
                             TechSupport_Yes + StreamingTV_Yes + StreamingMovies_Yes + 
                             `Contract_Month-to-month` + `Contract_One year` + PaperlessBilling_Yes +
                             `PaymentMethod_Electronic check` + `tenure_group_0 - 6 Mos`,
                             family = binomial(link = 'logit'), 
                             data = training)
    

If there is another way of doing this (and I'm sure there is) I would like to hear about it.

The call that yields this result is, in this case `LogModel2$terms`. It produces a lot of information about the object but the output of the full equation with all terms was most useful in this analysis.

>The Takeaway

Later, I wanted to do something similar with another project I was working on, but I forgot how I did it the first time! I drew a total blank. I spent hours googling and going through my history in *R-Studio* but I couldn't find the code. It was terribly frustrating.

Finally, in desperation, I turned to the *#rstats* channel in Twitter and the *R4ds* Slack group. It was a mistake. I should have gone there first.

I got a lot of good responses and helpful ideas on both channels but none was exactly what I was looking for. I realized this `object$terms` call might not be well known in the community. So I vowed when I figured it out, or if someone pointed me in the right direction, I would pass along the knowledge in a blog post.

Fortunately, *@DarrinLRogers* came through with the solution I was looking for. Thanks to him and all the others who pitched in with good ideas.

You can learn more about the *R4ds* Slack channel [from Jesse Meagan][5]. Sign up for *R4ds* Slack [here][6].

Mission accomplished.

 [1]: https://seslezak.github.io/R-Code/Telco_Churn_Reg_PS.html
 [2]: https://seslezak.github.io/R-Code/Telco_Churn_Reg.html
 [3]: https://github.com/seslezak/R-Code/blob/master/Telco%20Customer%20Churn%20Regression
 [4]: https://twitter.com/DarrinLRogers
 [5]: https://medium.com/@kierisi/r4ds-the-next-iteration-d51e0a1b0b82
 [6]: https://docs.google.com/forms/d/e/1FAIpQLSeT3zfzjWxoaQ6RmUEdT9n0xtvkuSaMeBetDQLpzNJvGUB6IQ/viewform
