---
title: Support Groups Finder
---

This is a project I worked on with two other data scientists. I was responsible for creating the model we would use as the basis for an application that would recommend online support groups for users based on the text they input into an online form on our website. We had gathered data from posts on the Reddit website. We identified 22 subreddits we could classify as support groups. With a simple model that would guess the support group that appeared in the data the most, we would have only 7% accuracy. I was tasked with building a model with far greater accuracy than that.

After splitting the data into a training set and a testing set and transforming the data to ready it for modeling, I fed the training data into one of the simpler classifiers, a naïve Bayes classifier, and achieved 44% accuracy when comparing the predictions to the actual support groups in the testing set. That was a decent result compared to the baseline, but I wanted to do better. I then tried a more advanced classifier, a support vector machine (SVM) classifier, and achieved a 58% accuracy. I then tested to see if tuning hyperparameters (adjusting various settings in the classifiers) would make a difference. I used a grid search to test several combinations of hyperparameters, first with the naïve Bayes classifier and then with the SVM classifier. Tuning hyperparameters improved the performance of the naïve Bayes classifier from 44% to 54%, but did not make much of a difference with the SVM classifier. After performing additional transformations to clean up the data and writing code that would allow me to test several classifiers with various hyperparameters, I eventually ended up with a model that had 63% accuracy, a 9X improvement over the baseline. That was enough to get good results when recommending a list of 5-10 support groups based on text from each user.


Based on a user's text input, the application recommends online support groups by leveraging a Natural Language Processing text classification algorithm.

See the web application here: <https://nlpsupportgroupsfinder.web.app/>

See the code for this project here: <https://github.com/labs12-support-group-nlp/DS>
