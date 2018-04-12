# Notes on Week 1

## Why Data Science? What is it?

Data Science is about taking large data sets and trying to make them useful and meaningful. This involves a combination of computing, statistics, and domain knowledge in the area of your data set. What this results in is a science that is used to draw useful conclusions from data using computation.

The practice of data science has three core activities: 1. Exploration - figuring out what patterns exist in the data through analysis or visualizations 2. Inference - some patterns are there by chance and others are deeply meaningful. Inference is the process of differentiating between the two 3. Prediction - when we have partial information about what we want to know, prediction is making educated guesses about what will occur. Or, training a machine to make predictions for us.

This course will explore all three of these activities in order.

## Cause and Effect

One of the most important functions of a data scientist is determine what is the cause of the data patterns that are being seen. In order to explain these things, we need a common language for describing data.

When it comes to observation of the effects of data, we have:

* individuals - study subjects, participants, units, etc
* treatment - something that is applied to an individual or already exists
* outcome - the observed result

To describe relationships that we see in data, we use the words:

1. association - any relation or liink between two or more things
2. causality - when one thing leads to another

To use an example of these two terms, a recent study linked eating chocolate to a reduced chance of heart disease. Among those with heart disease, 12% say they ate a lot chocolate every day, and 17.4% said they did not each much chocolate. Based on that, there is a link, or association, between eating chocolate and reduced heart disease. However, the scientists in that same study were unable to establish causality - that eating chocolate definitely reduces your chances of getting heart disease.

## Association and Causation

Back in the 1850s in London, Cholera was rampant. The leading health officials at the time were miasmists, meaning that they believed the disease was caused by bad smells. It sounds ridiculous, but that was the case at the time. It wasn't until John Snow mapped out the outbreak locations and established an association between the broad street water pump and the outbreak. He hypothesized that the water from that pump was spreading the disease and made enough of a case to get it turned off. Next, he needed to establish causation

To find causality, he studied of region a London that were supplied by two different water companies. The households and environment were completely the same, except for the water supply. In other words, the individuals were the same, and the treatment was different. It is very important that the individuals be the same in order to establish causality.

## Confounding

If the treatment and control groups have systematic differences other than the treatment, then it might be difficult to identify causality. Such differences are often present in observational studies.

Consider studying the infant size of women who drink during pregnancy and those who do not. You can't get women to drink during their pregnancy for the purposes of your study, so it has to be an observational study. When this happens, you can't control the other decisions that the women who drink during their pregnancy are making. These are caused confounding factors.

In order to avoid confounding in observational studies, you can randomize the individuals who will be the control and who will get the treadment. This is caused a randomized control experiment. Be careful though, random does not mean haphazard.

