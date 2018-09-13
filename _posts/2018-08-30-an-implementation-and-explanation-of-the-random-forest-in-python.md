---
published: true
title: An Implementation and Explanation of the Random Forest in Python
---
![](https://cdn-images-1.medium.com/max/1000/1*VQBcSuSfFqSePJaEC0T5yg.jpeg)
*([Source](https://www.pexels.![](https://cdn-images-1.medium.com/max/1000/1*VQBcSuSfFqSePJaEC0T5yg.jpeg)*

## A guide for using and understanding the random forest by building up from a single decision tree.

Fortunately, with libraries such as [Scikit-Learn](http://scikit-learn.org/),
it’s now easy to implement [hundreds of machine learning
algorithms](http://scikit-learn.org/stable/supervised_learning.html) in Python.
It’s so easy that we often don’t need any underlying knowledge of how the model
works in order to use it. While knowing *all* the details is not necessary, it’s
still helpful to have an idea of how a machine learning model works under the
hood. This lets us diagnose the model when it’s underperforming or explain how
it makes decisions, which is crucial if we want to convince others to trust our
models.

In this article, we’ll look at how to build and use the Random Forest in Python.
In addition to seeing the code, we’ll try to get an understanding of how this
model works. Because a random forest in made of many decision trees, we’ll start
by understanding how a single decision tree makes classifications on a simple
problem. Then, we’ll work our way to using a random forest on a real-world data
science problem. The complete code for this article is available as a [Jupyter
Notebook on
GitHub](https://github.com/WillKoehrsen/Machine-Learning-Projects/blob/master/Random
Forest Tutorial.ipynb).

**Note:** this article [originally appeared](https://enlight.nyc/random-forest)
on [enlight](https://enlight.nyc/), a community-driven, open-source platform
with tutorials for those looking to study machine learning.

*****

### Understanding a Decision Tree

A [decision tree](http://scikit-learn.org/stable/modules/tree.html) is the
building block of a random forest and is an intuitive model. We can think of a
decision tree as a series of yes/no questions asked about our data eventually
leading to a predicted class (or continuous value in the case of regression).
This is an interpretable model because it makes classifications much like we do:
we ask a sequence of queries about the available data we have until we arrive at
a decision (in an ideal world).

The technical [details of a decision
tree](https://machinelearningmastery.com/classification-and-regression-trees-for-machine-learning/)
are in how the questions about the data are formed. In the [CART
algorithm](https://www.stat.wisc.edu/~loh/treeprogs/guide/wires11.pdf), a
decision tree is built by determining the questions (called splits of nodes)
that, when answered, lead to the greatest reduction in [Gini
Impurity](https://en.wikipedia.org/wiki/Decision_tree_learning). What this means
is the decision tree tries to form nodes containing a high proportion of samples
(data points) from a single class by finding values in the features that cleanly
divide the data into classes.

We’ll talk in low-level detail about Gini Impurity later, but first, let’s build
a Decision Tree so we can understand it on a high level.

### Decision Tree on Simple Problem

We’ll start with a very simple binary classification problem as shown below:

![](https://cdn-images-1.medium.com/max/800/0*dvVMJdNRzlUqOl2Z)
*The goal is to divide the data points into their respective classes.*

Our data only has two features (predictor variables), `x1` and `x2` with 6 data
points — samples — divided into 2 different labels. Although this problem is
simple, it’s not linearly separable, which means that we can’t draw a single
straight line through the data to classify the points.

We can however draw a series of straight lines that divide the data points into
boxes, which we’ll call nodes. In fact, this is what a decision tree does during
training. Effectively, a [decision tree is a non-linear
model](https://datascience.stackexchange.com/questions/6787/is-decision-tree-algorithm-a-linear-or-nonlinear-algorithm)
built by constructing many linear boundaries.

To create a decision tree and train (`fit`) it on the data, we use Scikit-Learn.

<script src="https://gist.github.com/WillKoehrsen/eaa3f695de592be52bf4aef6819f3383.js" charset="utf-8"></script>

During training we give the model both the features and the labels so it can
learn to classify points based on the features. (We don’t have a testing set for
this simple problem, but when testing, we only give the model the features and
have it make predictions about the labels.)

We can test the accuracy of our model on the training data:

<script src="https://gist.github.com/WillKoehrsen/13797c086fc67bc9b170594976d9f5ec.js" charset="utf-8"></script>

We see that it gets 100% accuracy, which is what we expect because we gave it
the answers (`y`) for training and did not limit the depth of the tree. It turns
out this ability to completely learn the training data can be a downside of a
decision tree because it may lead to *overfitting *as we’ll discuss later.

*****

### Visualizing a Decision Tree

So what’s actually going on when we train a decision tree? I find a helpful way
to understand the decision tree is by visualizing it, which we can do using a
Scikit-Learn function (for details check out the notebook or [this
article](https://medium.com/p/38ad2d75f21c?source=user_profile---------4------------------)).

![](https://cdn-images-1.medium.com/max/800/0*QwJ2oZssAQ2_cchJ)
*Simple decision tree*

All the nodes, except the leaf nodes (colored terminal nodes), have 5 parts:

1.  Question asked about the data based on a value of a feature. Each question has
either a True or False answer that splits the node. Based on the answer to the
question, a data point moves down the tree.
1.  `gini`: The Gini Impurity of the node. The average weighted Gini Impurity
decreases as we move down the tree.
1.  `samples`: The number of observations in the node.
1.  `value`: The number of samples in each class. For example, the top node has 2
samples in class 0 and 4 samples in class 1.
1.  `class`: The majority classification for points in the node. In the case of leaf
nodes, this is the prediction for all samples in the node.

The leaf nodes do not have a question because these are where the final
predictions are made. To classify a new point, simply move down the tree, using
the features of the point to answer the questions until you arrive at a leaf
node where the `class` is the prediction.

To make see the tree in a different way, we can draw the splits built by the
decision tree on the original data.

![](https://cdn-images-1.medium.com/max/800/1*MCQ6yUvb3i2HTCEh-Cuz2Q.png)
*Splits made by the decision tree.*

Each split is a single line that divides data points into nodes based on feature
values. For this simple problem and with no limit on the maximum depth, the
divisions place each point in a node with only points of the same class. (Again,
later we’ll see that this perfect division of the *training *data might not be
what we want because it can lead to *overfitting*.)

*****

### Gini Impurity

At this point it’ll be helpful to dive into the concept of Gini Impurity (the
math is not intimidating!) The [Gini
Impurity](https://en.wikipedia.org/wiki/Decision_tree_learning#Gini_impurity) of
a node is the probability that a randomly chosen sample in a node would be
incorrectly labeled if it was labeled by the distribution of samples in the
node. For example, in the top (root) node, there is a 44.4% chance of
incorrectly classifying a data point chosen at random based on the sample labels
in the node. We arrive at this value using the following equation:

![](https://cdn-images-1.medium.com/max/800/1*mcHzG8OjhQ2ryiBH7MBPUA.png)
*Gini impurity of a node n.*

The Gini Impurity of a node `n` is 1 minus the sum over all the classes `J` (for
a binary classification task this is 2) of the fraction of examples in each
class `p_i` squared. That might be a little confusing in words, so let’s work
out the Gini impurity of the root node.

![](https://cdn-images-1.medium.com/max/1000/1*uAGS042OxMJ4Ic3k4s313Q.png)
*Gini Impurity of the root node*

At each node, the decision tree searches through the features for the value to
split on that results in the *greatest reduction* in Gini Impurity. (An
[alternative for splitting nodes is using the information
gain](https://datascience.stackexchange.com/questions/10228/gini-impurity-vs-entropy),
a related concept).

It then repeats this splitting process in a greedy, [recursive
procedure](http://scikit-learn.org/stable/modules/tree.html#tree) until it
reaches a maximum depth, or each node contains only samples from one class. The
weighted total Gini Impurity at each level of tree must decrease. At the second
level of thetree, the total weighted Gini Impurity is 0.333:

![](https://cdn-images-1.medium.com/max/1000/1*gdMrk7yEPJLio0d0Sixtkg.png)
*Total Weighted Gini Impurity of Tree Second Level*

(The Gini Impurity of each node is weighted by the fraction of points from the
parent node in that node.) You can continue to work out the Gini Impurity for
each node (check the visual for the answers). Out of some basic math, a powerful
model emerges!

Eventually, the weighted total Gini Impurity of the last layer goes to 0 meaning
each node is completely pure and there is no chance that a point randomly
selected from that node would be misclassified. While this may seem like a
positive, it means that the model may potentially be overfitting because the
nodes are constructed only using *training data.*

*****

### Overfitting: Or Why a Forest is better than One Tree

You might be tempted to ask why not just use one decision tree? It seems like
the perfect classifier since it did not make any mistakes! A critical point to
remember is that the tree made no mistakes **on the training data**. We expect
this to be the case since we gave the tree the answers and didn’t limit the max
depth (number of levels). The objective of a machine learning model is to
generalize well to **new data** *it has never seen before.*

**Overfitting** occurs when we have a [very flexible model](http://qr.ae/TUNozZ)
(the model has a high capacity) which essentially **memorizes** the training
data by fitting it closely. The problem is that the model learns not only the
actual relationships in the training data, but also any noise that is present. A
flexible model is said to have high **variance** because the learned parameters
(such as the structure of the decision tree) will vary considerably with the
training data.

On the other hand, an inflexible model is said to have high **bias** because it
makes **assumptions** about the training data (it’s biased towards pre-conceived
ideas of the data.) For example, a linear classifier makes the assumption that
the data is linear and does not have the flexibility to fit non-linear
relationships. An inflexible model may not have the capacity to fit even the
training data and in both cases — high variance and high bias — the model is not
able to generalize well to new data.

> The balance between creating a model that is so flexible it memorizes the
> training data versus an inflexible model that can’t learn the training data is
known as the [bias-variance
tradeoff](https://en.wikipedia.org/wiki/Biasâvariance_tradeoff) and is a
foundational concept in machine learning.

*****

The reason the decision tree is prone to overfitting when we don’t limit the
maximum depth is because it has unlimited flexibility, meaning that it can keep
growing until it has exactly one leaf node for every single observation,
perfectly classifying all of them. If you go back to the image of the decision
tree and limit the maximum depth to 2 (making only a single split), the
classifications are no longer 100% correct. We have reduced the variance of the
decision tree but at the cost of increasing the bias.

As an alternative to limiting the depth of the tree, which reduces variance
(good) and increases bias (bad), we can combine many decision trees into a
single ensemble model known as the random forest.

### Random Forest

The [random
forest](https://www.stat.berkeley.edu/~breiman/RandomForests/cc_home.htm) is a
model made up of many decision trees. Rather than just simply averaging the
prediction of trees (which we could call a “forest”), this model uses [two key
concepts](https://www.stat.berkeley.edu/~breiman/randomforest2001.pdf) that
gives it the name *random*:

1.  Random sampling of training data points when building trees
1.  Random subsets of features considered when splitting nodes

#### Random sampling of training observations

When training, each tree in a random forest learns from a **random** sample of
the data points. The samples are [drawn with
replacement](https://en.wikipedia.org/wiki/Bootstrapping_(statistics)), known as
*bootstrapping,* which means that some samples will be used multiple times in a
single tree. The idea is that by training each tree on different samples,
although each tree might have high variance with respect to a particular set of
the training data, overall, the entire forest will have lower variance but not
at the cost of increasing the bias.

At test time, predictions are made by averaging the predictions of each decision
tree. This procedure of training each individual learner on different
bootstrapped subsets of the data and then averaging the predictions is known as
*bagging*, short for [bootstrap
aggregating](https://machinelearningmastery.com/bagging-and-random-forest-ensemble-algorithms-for-machine-learning/)*.*

#### Random Subsets of features for splitting nodes

The other main concept in the random forest is that only a [subset of all the
features are considered for splitting each
node](https://sebastianraschka.com/faq/docs/random-forest-feature-subsets.html)
in each decision tree. Generally this is set to `sqrt(n_features)` for
classification meaning that if there are 16 features, at each node in each tree,
only 4 random features will be considered for splitting the node. (The random
forest can also be trained considering all the features at every node as is
common in regression. These options can be controlled in the [Scikit-Learn
Random Forest
implementation](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)).

*****

If you can comprehend a single decision tree, the idea of *bagging,* and random
subsets of features, then you have a pretty good understanding of how a random
forest works:

> The random forest combines hundreds or thousands of decision trees, trains each
> one on a slightly different set of the observations, splitting nodes in each
tree considering a limited number of the features. The final predictions of the
random forest are made by averaging the predictions of each individual tree.

To understand why a random forest is better than a single decision tree imagine
the following scenario: you have to decide whether Tesla stock will go up and
you have access to a dozen analysts who have no prior knowledge about the
company. Each analyst has low bias because they don’t come in with any
assumptions, and is allowed to learn from a dataset of news reports.

This might seem like an ideal situation, but the problem is that the reports are
likely to contain noise in addition to real signals. Because the analysts are
basing their predictions entirely on the data — they have high flexibility —
they can be swayed by irrelevant information. The analysts might come up with
differing predictions from the same dataset. Moreover, each individual analyst
has high variance and would come up with drastically different predictions if
given a *different* training set of reports.

The solution is to not rely on any one individual, but pool the votes of each
analyst. Furthermore, like in a random forest, allow each analyst access to only
a section of the reports and hope the effects of the noisy information will be
cancelled out by the sampling. In real life, we rely on multiple sources (never
trust a solitary Amazon review), and therefore, not only is a decision tree
intuitive, but so is the idea of combining them in a random forest.

*****

### Random Forest in Practice

Next, we’ll build a random forest in Python using Scikit-Learn. Instead of
learning a simple problem, we’ll use a real-world dataset split into a training
and testing set. We use a *test set* as an estimate of how the model will
perform on new data which also lets us determine how much the model is
overfitting.

#### Dataset

The problem we’ll solve is a binary classification task with the goal of
predicting an individual’s health. The features are socioeconomic and lifestyle
characteristics of individuals and the label is `0` for poor health and `1` for
good health. This dataset was collected by the [Centers for Disease Control and
Prevention](https://www.cdc.gov/) and is [available
here](https://www.kaggle.com/cdc/behavioral-risk-factor-surveillance-system).

![](https://cdn-images-1.medium.com/max/800/1*-yHB8RZnWA0rRCkQ1i3z_Q.png)
*Sample of Data*

Generally, 80% of a data science project is spent cleaning, exploring, and
making features out of the data. However, for this article, we’ll stick to the
modeling. (For details of the other steps, look at [this
article](https://medium.com/p/c62152f39420?source=your_stories_page---------------------------)).

This is an imbalanced classification problem, so accuracy is not an appropriate
metric. Instead we'll measure the [Receiver Operating Characteristic Area Under
the
Curve](https://developers.google.com/machine-learning/crash-course/classification/roc-and-auc)
(ROC AUC), a measure from 0 (worst) to 1 (best) with a random guess scoring 0.5.
We can also plot the ROC curve to assess a model.

*****

The [notebook
](https://github.com/WillKoehrsen/Machine-Learning-Projects/blob/master/Random
Forest Tutorial.ipynb)contains the implementation for both the decision tree and
the random forest, but here we’ll just focus on the random forest. After reading
in the data, we can instantiate and train a random forest as follows:

<script src="https://gist.github.com/WillKoehrsen/009862ad00ba7f3767c904506a7d8a54.js" charset="utf-8"></script>

After a few minutes to train, the model is ready to make predictions on the
testing data as follows:

<script src="https://gist.github.com/WillKoehrsen/46237dbd5df103c06db998bae28086bf.js" charset="utf-8"></script>

We make class predictions (`predict`) as well as predicted probabilities
(`predict_proba`) to calculate the ROC AUC. Once we have the testing
predictions, we can calculate the ROC AUC.

<script src="https://gist.github.com/WillKoehrsen/28ada9b83b730de17c9e81526a63f9bf.js" charset="utf-8"></script>


### Results

The final testing ROC AUC for the random forest was **0.87** compared to
**0.67** for the single decision tree with an unlimited max depth. If we look at
the training scores, both models achieved 1.0 ROC AUC, which again is as
expected because we gave these models the training answers and did not limit the
maximum depth of each tree.

Although the random forest overfits (doing better on the training data than on
the testing data), it is able to generalize much better to the testing data than
the single decision tree. The random forest has lower variance (good) while
maintaining the same low bias (also good) of a decision tree.

We can also plot the ROC curve for the single decision tree (top) and the random
forest (bottom). A curve to the top and left is a better model:

![](https://cdn-images-1.medium.com/max/800/0*xAy31Lc-hp9XQ4af)
*Decision Tree ROC Curve*

![](https://cdn-images-1.medium.com/max/800/0*G_VKh9jDAvD64eLB)
*Random Forest ROC Curve*

**The random forest significantly outperforms the single decision tree.**

Another diagnostic measure of the model we can take is to plot the confusion
matrix for the testing predictions (see the notebook for details):

![](https://cdn-images-1.medium.com/max/800/0*VHltnYowCdX618Gy)
*Confusion Matrix*

This shows the predictions the model got correct in the top left and bottom
right corners and the predictions missed by the model in the lower left and
upper right. We can use plots such as these to diagnose our model and decide
whether it’s doing well enough to put into production.

*****

#### Feature Importances

The feature importances in a random forest indicate the [sum of the reduction in
Gini
Impurity](http://danielhomola.com/wp-content/uploads/2018/03/DanielHomola_PhdThesis_final.pdf)
over all the nodes that are split on that feature. We can use these to try and
figure out what predictor variables the random forest considers most important.
The feature importances can be extracted from a trained random forest and put
into a Pandas dataframe as follows:

<script src="https://gist.github.com/WillKoehrsen/475f57ab3c61ea46db0833c6708f1702.js" charset="utf-8"></script>

Feature importances can give us insight into a problem by telling us what
variables are the most discerning between classes. For example, here `DIFFWALK,
`indicating whether the patient has difficulty walking, is the most important
feature which makes sense in the problem context.

Feature importances can be used for feature engineering by building additional
features from the most important. We can also use feature importances for
[feature
selection](http://scikit-learn.org/stable/modules/feature_selection.html) by
removing low importance features.

#### Visualize Tree in Forest

Finally, we can visualize a single decision tree in the forest. This time, we
have to limit the depth of the tree otherwise it will be too large to be
converted into an image. To make the figure below, I limited the maximum depth
to 6. This still results in a large tree that we can’t completely parse!
However, given our deep dive into the decision tree, we grasp how our model is
working.

![](https://cdn-images-1.medium.com/max/800/0*2OooR8mNm6jTbm84)
*Single (limited) decision tree in random forest.*

*****

### Next Steps

A further step is to optimize the random forest which we can do through random
search using the `RandomizedSearchCV`[ in
Scikit-Learn.](http://scikit-learn.org/stable/modules/generated/sklearn.model_selection.RandomizedSearchCV.html)
Optimization refers to finding the best hyperparameters for a model on a given
dataset. The best hyperparameters will vary between datasets, so we have to
perform optimization (also called model tuning) separately on each datasets.

I like to think of model tuning as finding the best settings for a machine
learning algorithm. Examples of what we might optimize in a random forest are
the number of decision trees, the maximum depth of each decision tree, the
maximum number of features considered for splitting each node, and the maximum
number of data points required in a leaf node.

For an implementation of random search for model optimization of the random
forest, refer to the [Jupyter
Notebook.](https://github.com/WillKoehrsen/Machine-Learning-Projects/blob/master/Random
Forest Tutorial.ipynb)

### Complete Running Example

The below code is created with [repl.it](http://repl.it/) and presents a
complete interactive running example of the random forest in Python. Feel free
to run and change the code (loading the packages might take a few moments).

<span class="figcaption_hack">Complete Python example of random forest.</span>

### Conclusions

While we can build powerful machine learning models in Python without
understanding anything about them, I find it’s more effective to have knowledge
about what is occurring behind the scenes. In this article, we not only built
and used a random forest in Python, but we also developed an understanding of
the model by starting with the basics.

We first looked at an individual decision tree, the building block of a random
forest, and then saw how we can overcome the high variance of a single decision
tree by combining hundreds of them in an ensemble model known as a random
forest. The random forest uses the concepts of random sampling of observations,
random sampling of features, and averaging predictions.

The key concepts to understand from this article are:

1.  **Decision tree**: an intuitive model that makes decisions based on a sequence
of questions asked about feature values. Has low bias and high variance leading
to overfitting the training data.
1.  **Gini Impurity**: a measure that the decision tree tries to minimize when
splitting each node. Represents the probability that a randomly selected sample
from a node will be incorrectly classified according to the distribution of
samples in the node.
1.  **Bootstrapping**: sampling random sets of observations with replacement.
1.  **Random subsets of features**: selecting a random set of the features when
considering splits for each node in a decision tree.
1.  **Random Forest**: ensemble model made of many decision trees using
bootstrapping, random subsets of features, and average voting to make
predictions. This is an example of a bagging ensemble.
1.  **Bias-variance tradeoff**: a core issue in machine learning describing the
balance between a model with high flexibility (high variance) that learns the
training data very well at the cost of not being able to generalize to new data
, and an inflexible model (high bias) that cannot learn the training data. A
random forest reduces the variance of a single decision tree leading to better
predictions on new data.

Hopefully this article has given you the confidence and understanding needed to
start using the random forest on your projects. The random forest is a powerful
machine learning model, but that should not prevent us from knowing how it
works. The more we know about a model, the better equipped we will be to use it
effectively and explain how it makes predictions.

*****

As always, I welcome comments, feedback, and constructive criticism. I can be
reached on Twitter [@koehrsen_will](http://twitter.com/@koehrsen_will). This
article was originally published on [enlight](https://enlight.nyc/), an
open-source community for studying machine learning. I would like to thank
enlight and also [repl.it ](http://repl.it/)for hosting the code in the article.


<!DOCTYPE html><html lang="en"><head><title class="next-head">Repl.it - Random Forest Tutorial</title><link rel="shortcut icon" href="/public/images/favicon.ico" type="image/vnd.microsoft.icon" class="next-head"/><link rel="alternate" type="application/json+oembed" href="https://repl.it/data/oembed?url=https%3A%2F%2Frepl.it%2F%40WillKoehrsen%2FRandom-Forest-Tutorial" title="repl.it - select language" class="next-head"/><meta property="og:title" content="Random Forest Tutorial" class="next-head"/><meta property="og:description" content="A short implementation of the random forest on a real-world dataset. Includes training / predicting along with visualizations. " class="next-head"/><meta property="og:type" content="article" class="next-head"/><meta property="og:image" content="https://www.gravatar.com/avatar/f85b7564fd35d5c86054b95090052d94?d=https://repl.it/public/images/evalbot/evalbot_17.png&amp;s=256" class="next-head"/><meta property="og:site_name" content="repl.it" class="next-head"/><meta property="fb:app_id" content="1775481339348651" class="next-head"/><meta itemProp="name" content="repl.it" class="next-head"/><meta itemProp="description" content="A short implementation of the random forest on a real-world dataset. Includes training / predicting along with visualizations. " class="next-head"/><meta itemProp="image" content="https://www.gravatar.com/avatar/f85b7564fd35d5c86054b95090052d94?d=https://repl.it/public/images/evalbot/evalbot_17.png&amp;s=256" class="next-head"/><meta name="description" content="A short implementation of the random forest on a real-world dataset. Includes training / predicting along with visualizations. " class="next-head"/><meta name="keywords" content="Interpreters,Compilers,Teach,Learn,Code,REPL,Compiler,Clojure,Haskell,Kotlin (beta),QBasic,Forth,LOLCODE,BrainF,Emoticon,Bloop,Unlambda,JavaScript,CoffeeScript,Scheme,APL,Lua,Python 2.7,Ruby,Roy,PHP,Python,Nodejs,Enzyme,Go,Java,C++,C++11,C,C#,F#,HTML, CSS, JS,Rust,Swift,Python (with Turtle),Jest,Django,Express,Sinatra,Ruby on Rails,R,Next.js,GatsbyJS,React,React Typescript,React Reason,bash,Quil" class="next-head"/><meta name="author" property="og:author" content="repl.it" class="next-head"/><meta name="twitter:card" content="summary" class="next-head"/><meta name="twitter:site" content="@replit" class="next-head"/><meta name="twitter:title" content="Random Forest Tutorial" class="next-head"/><meta name="twitter:description" content="A short implementation of the random forest on a real-world dataset. Includes training / predicting along with visualizations. " class="next-head"/><meta name="twitter:image" content="https://www.gravatar.com/avatar/f85b7564fd35d5c86054b95090052d94?d=https://repl.it/public/images/evalbot/evalbot_17.png&amp;s=256" class="next-head"/><meta name="google" value="notranslate" class="next-head"/><link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/styles/vs.min.css" class="next-head"/><link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/slick-carousel/1.6.0/slick.min.css" class="next-head"/><link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/slick-carousel/1.6.0/slick-theme.min.css" class="next-head"/><link href="//fonts.googleapis.com/css?family=Lato:300,400|Questrial|Raleway:400,100,700" rel="stylesheet" type="text/css" class="next-head"/><link href="//cloud.typenetwork.com/projects/895/fontface.css/" rel="stylesheet" type="text/css" class="next-head"/><meta charSet="utf-8" class="next-head"/><meta name="viewport" content="initial-scale=1.0, width=device-width" class="next-head"/><link href="//cloud.typenetwork.com/projects/895/fontface.css/" rel="stylesheet" type="text/css" class="next-head"/><style>
    * {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  min-height: 100%;
}

button {
  border-radius: 0;
}

a {
  color: #4183C4;
  text-decoration: none;
  cursor: pointer;
  border-bottom: 1px solid transparent;
}

a:hover, a:active, a:focus {
  border-bottom: 1px solid #4183C4;
}

header {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 99;
  height: 60px;
  width: 100%;
  background-color: rgba(255, 255, 255, 0.9);
  border-bottom: 1px solid #ececec;
}

header .logo {
  position: absolute;
  left: 25px;
  height: 60px;
  line-height: 94px;
}

header .logo img {
  width: 116px;
}

#page {
  height: 100%;
  padding-top: 60px;
  padding-bottom: 433px;
}

@media all and (max-width : 720px) {
  #page {
    padding-bottom: 0;
  }
}

/* Blog */

.postItem {
  width: 720px;
  margin: 50px auto 50px;
}

.post-seperator {
  width: 800px;
  margin-top: 20px;
  margin-left: -40px;
  border-top: 1px solid #ececec;
}

.postItem:last-child .post-seperator {
  display: none;
}

.postItem img {
  display: block;
  margin: 50px auto;
  max-width: 100%;
}

.postItem img + em {
  display: block;
  text-align: center;
  margin-top: -40px;
  font-size: 18px;
}

@media all and (max-width : 800px) {
  .post-seperator {
    width: 100%;
    margin-left: auto;
  }
}

.postTitle a {
  font-size: 40px;
  line-height: 1.15;
  font-family: Raleway;
  font-weight: 100;
  color: #807F7F;
}

.postTitle a:hover {
  color: #4183C4
}

.postAuthor {
  font-family: Lato;
  font-weight: 300;
  font-size: 18px;
  margin-top: 15px;
  color: #807F7F;
}

.postContent {
  font-family: "Miller Text Rom", Georgia, Cambria, "Times New Roman", Times, serif;
  font-size: 18px;
  line-height: 1.6;
  margin-top: 45px;
  color: rgb(93, 91, 91);
}
.postContent h1,
.postContent h2,
.postContent h3,
.postContent h4,
.postContent h5 {
  color: rgb(76, 75, 75);
  font-family: Lato;
  font-weight: 400;
  margin-top: 10px;
}

.postContent h1 {
  font-size: 30px;
  color: rgb(63, 64, 63);
  margin-top: 20px;
}

.postContent h2 {
  font-size: 26px;
  margin-top: 20px;
}

.postContent h3 {
  font-size: 24px;
}

.postContent h4 {
  font-size: 22px;
}

.postContent h5 {
  font-size: 20px;
}

@media all and (max-width : 720px) {
  .postItem {
    width: auto;
    margin: 50px 18px;
  }

  .postContent {
    font-size: 18px
  }

  .postAuthor {
    font-size: 16px;
  }

  .postContent h1 {
    font-size: 28px;
  }

  .postContent h2 {
    font-size: 24px;
  }

  .postContent h3 {
    font-size: 22px;
  }

  .postContent h4 {
    font-size: 20px;
  }

  .postContent h5 {
    font-size: 18px;
  }
}

.postContent p {
  margin-top: 20px;
  margin-bottom: 20px;
}

.postContent ul {
  padding-left: 5px;
}

.postContent ol {
  padding-left: 25px
}

.postContent ul {
  list-style: none;
}

.postContent ul, .postContent ol {
  margin: 20px 0;
}

.postContent ul > li:before {
  content: "• ";
  line-height: 0;
  color: #807F7F;
}

/* Nested */
.postContent ul > li > ul > li:before {
  content: "◦ ";
  line-height: 0;
  color: #807F7F;
}

.postContent ul > li {
  padding-left: 1em;
  text-indent: -.7em;
}

.postContent ol > li {
  text-indent: 0;
}

.postContent li > ul,
.postContent li > ol {
  margin-left: 10px;
}

.postContent pre {
  overflow: auto;
  border: 1px solid #e1e1e8;
  padding: 12px;
  margin-top: 20px  ;
}

.postContent code {
  padding: 2px 4px;
}

.postContent pre code {
  display: block;
  overflow-x: auto;
  color: #000;
  border: none;
}

.postContent code, .postContent pre {
  font-size: 12px;
  line-height: 18px;
  font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace;
}

.postContent blockquote {
  font-style: italic;
  border-left: 3px solid black;
  padding-left: 20px;
}

.readMore {
  margin-bottom: 20px;
}

/* Marketing */

.marketingPage {
  font-family: Raleway, sans-serif;
}

.marketingSection {
  text-align: center;
  line-height: 1;
  width: 100%;
  background-color: #fff;
}

.marketingSection:last-child {
  padding-bottom: 100px;
}

.marketingSection h3 {
  font-size: 61px;
  font-weight: 100;
}

.marketingSection > p {
  font-size: 22px;
  margin: 25px auto 0;
  max-width: 700px;
}

.includeTopBorder {
  border-top: 1px solid #ececec;
}

.marketingHeader {
  background-color: #fff;
  color: #807F7F;
  min-height: calc(100vh - 60px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
}

.marketingHeader .button {
  margin-top: 25px;
}

.marketingHeader .scrolldownWrapper {
  width: 40px;
  height: 40px;
  position: absolute;
  bottom: 30px;
  right: 0;
  left: 0;
  margin-right: auto;
  margin-left: auto;
  -webkit-animation-duration: 0.75s;
  animation-duration: 0.75s;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
}

.marketingHeader .scrolldown {
  width: 20px;
  height: 20px;
  border-right: 1px solid #807F7F;
  border-bottom: 1px solid #807F7F;
  transform: rotate(45deg);
  margin: auto;
}

@keyframes fadeInDown {
  0% {
     opacity: 0;
     transform: translateY(-40px);
  }
  100% {
     opacity: 1;
     transform: translateY(0);
  }
}

.fadeInDown {
  -webkit-animation-name: fadeInDown;
  animation-name: fadeInDown;
}

.featuresWrapper {
  max-width: 1200px;
  margin: 0 auto;
}

.marketingFeature {
  margin-top: 50px;
  max-width: 500px;
  text-align: left;
  display: inline-flex;
  line-height: 1.5;
}

.marketingFeature:nth-child(odd) {
  margin-right: 70px;
}

.marketingFeature .img {
  flex: 1;
  height: 100px;
  width: 100px;
  background-size: contain;
  background-repeat: no-repeat;
}

.cloudcode {
  background-image: url('/public/images/cloudcode.png')
}
.languages {
  background-image: url('/public/images/languages2.png')
}
.secure {
  background-image: url('/public/images/lock.png')
}
.console {
  background-image: url('/public/images/console2.png')
}

.marketingFeature .textWrapper {
  flex: 4;
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 20px;
}

.marketingFeature .title {
  font-size: 19px;
  font-weight: 600;
  margin-bottom: 10px;
}

.marketingFeature .details {
  font-size: 17px;
}

.marketingCard {
  display: inline-block;
  vertical-align: top;
  width: 33.33%;
}

.marketingCard .cardImg {
  width: 185px;
  height: 185px;
  margin: auto;
  padding-bottom: 20px;
  background-size: contain;
  background-repeat: no-repeat;
}

.marketingCard h5 {
  color: #807F7F;
  font-size: 18px;
  width: 185px;
  margin: 30px auto 0;
  line-height: 1.5;
}

@media all and (max-width : 720px) {
  .marketingCard {
    display: block;
    width: auto;
  }

  .marketingSection h3 {
    font-size: 50px;
    font-weight: 100;
  }
}

/* Classrooms */

.taheader {
  background-color: #5192b3;
  color: #FFF;
  padding: 0 20px;
}

.taheader .scrolldown {
  border-right: 1px solid #fff;
  border-bottom: 1px solid #fff;
}

.tacontact {
  background-color: #F2F2F3;
  color: #807F7F;
  font-size: 22px;
  margin-bottom: -70px;
  border-bottom: 2px solid rgb(222, 222, 222);
  padding-top: 30px;
}

.tacontact p:first-of-type {
  padding-top: 30px;
  margin-bottom: 50px;
  line-height: 1.5;
}

.tacontact .contactLinkContainer {
  color: #4183C4;
  margin: 20px auto;
  display: block;
}

.tacontact .contactLinkContainer a {
  color: #4183C4;
  text-decoration: none;
  position: relative;
  font-size: 18px;
  margin: 0 30px;
}

.tacontact .twitter, .tacontact .facebook {

}

.aboutContact .twitter, .tacontact .twitter {
  background-image: url('/public/images/twitter.png');
  display: inline-block;
  background-size: contain;
  height: 24px;
  width: 24px;
  background-repeat: no-repeat;
  vertical-align: text-bottom;
}

.aboutContact .facebook, .tacontact .facebook {
  background-image: url('/public/images/facebook.png');
  display: inline-block;
  background-size: contain;
  height: 24px;
  width: 24px;
  background-repeat: no-repeat;
  vertical-align: text-bottom;
}

.taconnect {
  background-color: #FFF;
  color: #807F7F;
  min-height: 600px;
  padding: 50px 0;
}

.taconnect h3 {
  font-size: 40px;
}

.taconnect > p {
  font-size: 22px;
  padding-top: 100px;
}

.tacards {
  margin-top: 110px;
}

.cardImg.student {
  background-image: url('/public/images/student.png');
}

.cardImg.teacher {
  background-image: url('/public/images/teacher.png');
}

.cardImg.progress {
  background-image: url('/public/images/progress.png');
}

.taenvironment {
  color: #807F7F;
  height: 900px;
}

.taenvironment h3 {
  padding-top: 100px;
}

.studentEnvironmentImg {
  background-image: url('/public/images/studentenvironment.png');
  min-height: 500px;
  width: 1250px;
  background-size: cover;
  background-repeat: no-repeat;
  margin: 100px auto 0;
}

@media all and (max-width : 1280px) {
  .studentEnvironmentImg {
    height: 500px;
    width: 100%;
    background-size: contain;
  }
}

.tafeatures {
  color: #807F7F;
  background-color: rgba(242, 242, 242, 1);
  min-height: 500px;
  display: flex;
  align-items: center;
  padding-bottom: 30px;
}

.taschools {
  background-color: #F2F2F3;
  height: 415px;
  padding: 0 15%;
}

.schoolsSlider {
  margin: 0 auto;
}

.taschools > p {
  padding-top: 100px;
  color: #807F7F;
  font-size: 22px;
  margin: 0 auto 100px;
}

.tauser_logo {
  display: inline-block;
  height: 90px;
  width: 100%;
  margin: auto;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}

.alpha {
  background-image: url('/public/images/classrooms_clients/alpha_ps.png');
}
.columbia_uni_ny {
  background-image: url('/public/images/classrooms_clients/columbia_uni.png');
}
.coast_mountain_academy {
  background-image: url('/public/images/classrooms_clients/coast_mountain_academy.png');
}
.flocabulary {
  background-image: url('/public/images/classrooms_clients/flocabulary.png');
}
.guilsborough {
  background-image: url('/public/images/classrooms_clients/guilsborough.png');
}
.lonestar {
  background-image: url('/public/images/classrooms_clients/lonestar.png');
}
.madeira_city_schools {
  background-image: url('/public/images/classrooms_clients/madeira_city_schools.jpg');
}
.nait {
  background-image: url('/public/images/classrooms_clients/nait.png');
}
.perse {
  background-image: url('/public/images/classrooms_clients/perse.png');
}
.richard_college {
  background-image: url('/public/images/classrooms_clients/richard_college.png');
}
.sfusd {
  background-image: url('/public/images/classrooms_clients/sfusd.png');
}
.uni_of_oregon {
  background-image: url('/public/images/classrooms_clients/uni_of_oregon.png');
}
.utc_oxford {
  background-image: url('/public/images/classrooms_clients/utc_oxford.png');
}
.willington_academy {
  background-image: url('/public/images/classrooms_clients/willington_academy.png');
}
.workshop_college {
  background-image: url('/public/images/classrooms_clients/workshop_college.png');
}
.wyncode {
  background-image: url('/public/images/classrooms_clients/wyncode.png');
}

.tatestimonials {
  padding: 100px 15%;
}

.testimonial {
  margin-bottom: 100px;
  position: relative;
  max-width: 700px;
  margin: 0 auto;
}

.testimonialsSlider {
  width: 90%;
  max-width: 700px;
  margin: 0 auto;
}
.testimonial_auth_pic {
  border-radius: 100px;
  margin-bottom: 32px;
  height: 100px;
  width: 100px;
  margin: 0 auto 32px;
  background-size: contain;
  background-repeat: no-repeat;
}

.testimonial_quote {
  min-height: 90px;
  font-size: 20px;
  margin: 0 auto 42px;
  color: rgb(128, 127, 127);
  line-height: 28px;
  font-style: italic;
}

.quote {
  background-image: url('/public/images/quote.png');
  height: 30px;
  width: 22px;
  display: inline-block;
  background-size: contain;
  background-repeat: no-repeat;
  vertical-align: bottom;
}

.quote:nth-child(even) {
  transform: scaleX(-1);
  height: 18px;
  margin-left: 5px;
  vertical-align: text-bottom;
}

.testimonial_author_info {
  font-size: 20px;
  color: rgb(68, 67, 67);
}

.testimonial_arrow {
  position: absolute;
  top: 150px;
}

.prev.testimonial_arrow {
  left: -45px;
}

.next.testimonial_arrow {
  right: -45px;
}

.nguyen {
  background-image: url('/public/images/nguyen.png');
}

.steve {
  background-image: url('/public/images/steve.jpg');
}

.zach {
  background-image: url('/public/images/zach.jpg');
}

.quincy {
  background-image: url('/public/images/quincy.jpg');
}

#thanks {
  display: none;
}

/* About us */

.aboutHeader h3 {
  color: #F06A60;
}

.aboutHeader p {
  line-height: 1.5;
}

.aboutTeam {
  color: #807F7F;
  border-top: 1px solid #ECECEC;
  min-height: 665px;
}

.aboutTeam > h3 {
  padding-top: 100px;
}

.teamMembers {
  margin-top: 100px;
}

.teamMembers .marketingCard {
  width: 16%;
}

@media all and (max-width : 720px) {
  .teamMembers .marketingCard {
    display: block;
    margin: 0 auto;
    margin-bottom: 30px;
    width: auto;
  }
}

.memberImg {
  border-radius: 100%;
}

.faris {
  background-image: url('/public/images/faris.jpg');
}
.amjad {
  background-image: url('/public/images/amjad.jpg');
}
.haya {
  background-image: url('/public/images/haya.jpg?x=1');
  position: relative;
  right: 10px;
}
.mason {
  background-image: url('/public/images/mason.png?x=1');
}
.tim {
  background-image: url('/public/images/tim.png');
}
.rob {
  background-image: url('/public/images/rob.png');
}

.memberName {
  font-size: 40px;
  font-weight: 100;
  margin-top: 20px;
}

.memberDescription {
  width: 100% !important;
  margin-top: 10px !important;
}

.memberTwitter {
  font-weight: normal;
}

.aboutContact {
  background-color: #F2F2F3;
  color: #807F7F;
  height: 400px;
}

.aboutContact h3 {
  padding-top: 100px;
  margin-bottom: 20px;
}

.aboutContact a {
  display: inline-block;
  margin-top: 25px;
  font-size: 18px;
}

@media all and (max-width : 720px) {
  .marketingSection.jobsSection .content {
    width: auto;
    margin: 50px 18px;
  }

  .jobsSection .content h3 {
    font-size: 22px;
  }

  .jobsSection .content p {
    font-size: 18px;
  }
}

/* Feedback */
.feedbackIframe {
  border: 0;
  box-sizing: border-box;
  max-height: 100%;
  border-left: 1px solid silver;
  border-right: 1px solid silver;
}

@media all and (max-width : 480px) {
  .feedbackIframe {
    margin-top: 20px;
    border: 0;
  }
}

/* Bot */

.slack-add {
  margin-right: 20px;
  display: inline-block;
  height: 40px;
  border: none;
}

.slack-add img {
  height: 40px;
  width: 140px;
  margin: 0;
  max-width: none;
}

.fb-msg {
  cursor:pointer;
  text-decoration:none;
  border: none;
}

.slack-add:hover, .fb-msg:hover {
  border: none;
  opacity: 0.8;
}

.fb-msg span {
  height: 40px;
  width: 139px;
  line-height: 40px;
  text-align: center;
  border-radius: 4px;
  background: #0084ff;
  color: #fff;
  font-size: 14px;
  display:inline-block;
  font-family: helvetica, arial, sans-serif;
  white-space: nowrap;
}

#landing-language-cycle{
  display: inline-block;
  text-align: left;
  width: 100px;
  white-space: nowrap;
}

.teacher-start-nav {
  background: #F7F8F9;
  color: #797B7C;
  padding: 0 1em;
}

.teacher-start-nav:hover {
  background: rgb(81, 146, 179);
  color: #fff;
}

.mainLandingHeader {
  padding: 1em 4em 2em 2em;
  font-size: 22px;
  min-height: calc(83vh - 60px);
}

.mainLandingHeader .teachers-start {
  height: 55px;
  width: 380px;
  max-width: calc(100% - 40px);
  font-weight: 400;
  font-size: 19px;
  font-family: 'Questrial';
  padding: 0;
  border: none;

  background: #F7F8F9;
  color: #797B7C;
}

.mainLandingHeader .teachers-start:hover {
  background: rgb(81, 146, 179);
  color: #FFF;
}

.mainLandingHeader .bot-container {
  position: relative;
  width: 380px;
  max-width: calc(100% - 40px);
  height: 35px;
  z-index: 2;
}

.mainLandingHeader .bot {
  position: absolute;
  bottom: -36px;
  right: 4px;
  width: 70px;
}

.mainLandingHeader .language-search-placeholder {
  background: #fff;
  min-height: 55px;
  width: 380px;
  line-height: 55px;
  border: 1px solid #4083C4;
  color: #000;
  font-size: 19px;
  padding-left: 12px;
  outline: none;
}

.mainLandingHeader .heading {
  margin-bottom: .75em;
  color: #2E4457;
}

.mainLandingHeader .heading.roles {
  font-style: italic;
}

.mainLandingHeader h2 {
  font-weight: 500;
}

.mainLandingHeader h4 {
  font-weight: 300;
  font-size: 1.25em;
}

.mainLandingHeader .title-em {
  font-weight: 800;
  font-style: normal;
}

.mainLandingFeatures,
.mainLandingClassroom {
  padding-top: 100px;
  padding-bottom: 100px;
}

.mainLandingFeatures p {
  margin-bottom: 50px;
}

.mainLandingClassroom {
  background: rgba(170,218,245,0.17);
}

.mainLandingClassroom h3 {
  color: rgb(81, 146, 179);
}

.mainLandingClassroom p,
.mainLandingClassroom p a {
  color: #807F7F;
  font-size: 22px;
  text-decoration: none;
}

.mainLandingClassroom .moreInfo {
  font-size: 16px;
}

.mainLandingClassroom .moreInfo:hover {
  border-bottom: 0;
  color: rgb(81, 146, 179);
}

.mainLandingClassroom p a:hover {
  border-bottom: 1px solid #807F7F;
}

.mainLandingClassroom .button {
  margin-top: 50px;
}

.mainLandingFeatures {
  color: #807F7F;
}

.mainLandingBlog {
  border-top: 1px solid #ececec;
}

.mainLandingBlog .subtitle {
  font-size: 22px;
  margin: 0 auto 50px;
  max-width: 700px;
  color: #807F7F;
  text-align: center;
  line-height: 1;
  width: 100%;
  background-color: #fff;
}

@media all and (min-width: 880px) {
  .mainLandingBlog .postItem {
    margin: 0;
    margin-left: 150px;
  }
}

.zero {
  font-size: 25px;
}

.introVideo {
  max-width: 820px;
  display: block;
  margin: 100px auto 100px;
  height: 100%;
  position: relative;
  background: black;
}

.taconnect .introVideo {
  margin: 20px auto 50px;
}

.introVideo .thumbnailWrapper {
  cursor: pointer;
}

.introVideo iframe,
.introVideo .thumb {
  max-width: 100%;
  width: 100%;
  display: block;
  height: 461px;
  border: none;
  top: 0;
  bottom: 0;
  margin: auto 0;
}

.introVideo .thumb {
  max-height: 461px;
  width: auto;
  height: auto;
}

.introVideo .play {
  position: absolute;
  margin: auto;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  display: inline-block;
  background-size: contain;
  height: 80px;
  width: 80px;
  background-repeat: no-repeat;
  background-image: url('/public/images/youtube-play.png');
}

.introVideo .thumbnailWrapper:hover .play{
  background-image: url('/public/images/youtube-play-hover.png');
}

@media all and (max-width: 830px) {
  .introVideo {
    max-width: 90%;
  }
}

@media all and (max-width: 600px) {
  .introVideo iframe {
    height: 320px;
  }

  .introVideo .thumb {
    max-height: 320px
  }

  .introVideo .play {
    height: 45px;
    width: 64px;
  }
}

@media all and (max-width: 480px) {
  .introVideo iframe {
    height: 230px;
  }

  .introVideo .thumb {
    max-height: 230px
  }

  .introVideo .play {
    height: 36px;
    width: 51px;
  }
}

@keyframes loader-fill-animation {
    0% { width: 0%; }
    90% { width: 100%; }
}

@keyframes loading-spinner {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

.loading-spinner {
  width: 64px;
  height: 64px;

  background-image: url('/public/images/logo.svg');
  background-position: center;
  background-size: contain;

  transform: rotate(0deg);

  animation-name: loading-spinner;
  animation-duration: 2s;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
}

.css-typer-character {
  animation-duration: 0.01s;
  animation-name: css-typer-show;
  animation-fill-mode: both;
}

@keyframes css-typer-show {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.page-close {
  position: absolute;
  top: 20px;
  right: 20px;
  background-image: url(/public/images/close.png);
  height: 15px;
  width: 15px;
  background-size: contain;
  background-position: center;
  background-repeat: no-repeat;
  cursor: pointer;
}

.page-close:hover {
  background-image: url(/public/images/close_hover.png);
}
</style><script src="https://cdn.polyfill.io/v2/polyfill.min.js?features=default,Symbol,Symbol.iterator,Array.prototype.@@iterator,Array.prototype.includes,Array.prototype.find,Object.values" class="next-head"></script><script class="next-head">
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

    ga('create', 'UA-25629695-1', 'auto');
    ga('send', 'pageview');</script><script class="next-head">
    !function(){
      var analytics=window.analytics=window.analytics||[];if(!analytics.initialize)if(analytics.invoked)window.console&&console.error&&console.error("Segment snippet included twice.");else{analytics.invoked=!0;analytics.methods=["trackSubmit","trackClick","trackLink","trackForm","pageview","identify","reset","group","track","ready","alias","debug","page","once","off","on"];analytics.factory=function(t){return function(){var e=Array.prototype.slice.call(arguments);e.unshift(t);analytics.push(e);return analytics}};for(var t=0;t<analytics.methods.length;t++){var e=analytics.methods[t];analytics[e]=analytics.factory(e)}analytics.load=function(t){var e=document.createElement("script");e.type="text/javascript";e.async=!0;e.src=("https:"===document.location.protocol?"https://":"http://")+"cdn.segment.com/analytics.js/v1/"+t+"/analytics.min.js";var n=document.getElementsByTagName("script")[0];n.parentNode.insertBefore(e,n)};analytics.SNIPPET_VERSION="4.0.0";
    analytics.load('jdVID8rHoI7wkCBDjKmjApGBGWclWIKJ');
    analytics.page();
    }}();</script><script class="next-head">
    (function (isTouchDevice) {
      if (!isTouchDevice) return;
      var isTouchClass = 'is-touch-device';
      var docElement = document.documentElement;
      docElement.className = docElement.className ? [docElement.className, isTouchClass].join(' ') : isTouchClass;
    })(('ontouchstart' in window) || window.DocumentTouch && document instanceof DocumentTouch);
            </script><link rel="preload" href="/_next/fe48d264-0938-4ab4-a310-5fc686ab952d/page/replEnvironment.js" as="script"/><link rel="preload" href="/_next/fe48d264-0938-4ab4-a310-5fc686ab952d/page/_app.js" as="script"/><link rel="preload" href="/_next/fe48d264-0938-4ab4-a310-5fc686ab952d/page/_error.js" as="script"/><link rel="preload" href="/_next/static/commons/main-bf2a9bbd3aa2f834d2be.js" as="script"/><style id="__jsx-3633663456">.email-verification-banner.jsx-3633663456{font-family:Questrial;border-color:#E2A8A7;border-style:solid;border-width:1px 0px;color:#D56D6D;background-color:#FEF1F1;width:100%;text-align:center;padding:10px;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;-webkit-box-pack:center;-webkit-justify-content:center;-ms-flex-pack:center;justify-content:center;}
.email-verification-banner.jsx-3633663456 a.jsx-3633663456{color:#4183C4;}
.email-verification-banner.jsx-3633663456 a.jsx-3633663456:hover{border-bottom:1px solid #4183C4;cursor:pointer;}</style><style id="__jsx-3203370839">.workspaceWrapper.jsx-3203370839{height:100%;position:relative;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;background-color:#FFFFFF;}
.windowManagerWrapper.jsx-3203370839{-webkit-flex:5;-ms-flex:5;flex:5;height:100%;width:100%;}</style><style id="__jsx-2154740905">.loading-dots-icon.jsx-2154740905 img.jsx-2154740905{width:100%;}</style><style id="__jsx-2861210517">.ws-header-cta.jsx-2861210517{margin-left:20px;font-family:'Questrial',sans-serif;height:calc(100% - 1px);max-width:100px;min-width:100px;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;-webkit-box-pack:center;-webkit-justify-content:center;-ms-flex-pack:center;justify-content:center;background-color:rgb(236, 236, 236);border-bottom:1px solid rgb(236, 236, 236);color:rgb(119, 119, 119);cursor:pointer;position:relative;-webkit-transition:color 0.05s ease-in;transition:color 0.05s ease-in;z-index:1;}
.ws-header-cta.jsx-2861210517:hover{color:rgb(119, 119, 119);}
.ws-header-cta.jsx-2861210517:before{content:"";position:absolute;background-color:#FFF;bottom:0;left:0;right:0;top:100%;z-index:-1;-webkit-transition:top 0.09s ease-in;transition:top 0.09s ease-in;}
.ws-header-cta.jsx-2861210517:hover.jsx-2861210517:before{top:0;}
.ws-header-cta:first-child{margin-left:30px;}
.ws-header-cta-icon.jsx-2861210517{background-size:contain;background-repeat:no-repeat;margin-left:5px;width:20px;height:15px;-webkit-transition:background-image 0.05s ease-in;transition:background-image 0.05s ease-in;}
@media screen and (max-width:950px){.ws-header-cta.jsx-2861210517{max-width:50px;min-width:50px;}.ws-header-cta-label.jsx-2861210517{display:none;}}</style><style id="__jsx-699899063">.ws-header-cta.jsx-699899063{border-bottom:4px solid #78b087 !important;background:#dceede !important;color:#78b087 !important;height:100% !important;}
.ws-header-cta.jsx-699899063:hover{color:white !important;}
.ws-header-cta.jsx-699899063:before{background-color:#78b087 !important;}</style><style id="__jsx-2926474592">.lite-header.jsx-2926474592{background:#f4f4f4;border-bottom:1px solid #e4e4e4;font-family:'Questrial',sans-serif;height:40px;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-box-pack:justify;-webkit-justify-content:space-between;-ms-flex-pack:justify;justify-content:space-between;}
.lite-header-cta.jsx-2926474592{margin-left:auto;margin-right:auto;}
.lite-header-logo-link.jsx-2926474592{display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;-webkit-text-decoration:none;text-decoration:none;border-bottom:none;color:#767777;font-size:14px;}
.lite-header-logo-link.jsx-2926474592:hover{color:#7997a5;}
.lite-header-logo.jsx-2926474592{width:83px;margin-left:11px;margin-right:11px;}</style><style id="__jsx-588867458">.file-icon-svg{width:100%;height:100%;}
.file-icon-svg path{fill:currentColor;}</style><style id="__jsx-942974558">.side-nav-item.jsx-942974558{display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;-webkit-box-pack:center;-webkit-justify-content:center;-ms-flex-pack:center;justify-content:center;width:100%;height:50px;cursor:pointer;}
.side-nav-item-active.jsx-942974558{background:rgb(245, 245, 245);}
.side-nav-item-img.jsx-942974558{width:25px;height:22px;width:22px;color:#b3b4b3;}
.side-nav-item.jsx-942974558:hover .side-nav-item-img.jsx-942974558,.side-nav-item-active.jsx-942974558 .side-nav-item-img.jsx-942974558{color:#7B99A6;}</style><style id="__jsx-340579616">.tooltip-wrap-tooltip{border:1px solid #ecedec;background-color:#FFF;color:rgb(63, 63, 63);}</style><style id="__jsx-1385922659">.tooltip-base.jsx-1385922659{position:relative;}
.tooltip-handle.jsx-1385922659{position:absolute;height:10px;width:16px;z-index:1011;}
.tooltip-wrap-tooltip{padding:12px;font-family:Questrial,sans-serif;box-sizing:border-box;position:absolute;z-index:1010;font-size:14px;line-height:18px;white-space:nowrap;}
.tooltip-wrap-top{left:-100%;bottom:calc(100% + 23px);-webkit-transform:translate(0,0);-ms-transform:translate(0,0);transform:translate(0,0);}
.tooltip-wrap-right{top:50%;left:calc(100% + 14px);-webkit-transform:translate(0,-50%);-ms-transform:translate(0,-50%);transform:translate(0,-50%);}
.tooltip-wrap-bottom{left:calc(50% + 2px);top:calc(100% + 13px);-webkit-transform:translate(-50%,0);-ms-transform:translate(-50%,0);transform:translate(-50%,0);}
.tooltip-wrap-left{top:50%;right:calc(100% + 14px);-webkit-transform:translate(0,-50%);-ms-transform:translate(0,-50%);transform:translate(0,-50%);}
.tooltip-handle-top.jsx-1385922659{bottom:100%;left:50%;-webkit-transform:translate(-50%,-16px);-ms-transform:translate(-50%,-16px);transform:translate(-50%,-16px);}
.tooltip-handle-right.jsx-1385922659{top:50%;left:100%;-webkit-transform:translate(4px,-108%);-ms-transform:translate(4px,-108%);transform:translate(4px,-108%);}
.tooltip-handle-bottom.jsx-1385922659{top:100%;left:50%;-webkit-transform:translate(-50%,1px);-ms-transform:translate(-50%,1px);transform:translate(-50%,1px);}
.tooltip-handle-left.jsx-1385922659{top:50%;right:100%;-webkit-transform:translate(1px,-108%);-ms-transform:translate(1px,-108%);transform:translate(1px,-108%);}</style><style id="__jsx-2654216489">.packages-icon-svg{width:100%;height:100%;}
.packages-icon-svg path{fill:currentColor;}</style><style id="__jsx-2347153736">.debugger-icon-svg{width:100%;height:100%;}
.debugger-icon-svg path{fill:currentColor;fill-rule:evenodd;}</style><style id="__jsx-2554818929">.side-nav.jsx-2554818929{background-color:rgb(245, 245, 245);}
.side-nav-options.jsx-2554818929{border-right:1px solid rgb(236, 236, 236);}</style><style id="__jsx-695095782">.side-nav.jsx-695095782{height:100%;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;}
.side-nav-options.jsx-695095782{-webkit-flex:0 0 50px;-ms-flex:0 0 50px;flex:0 0 50px;height:100%;}
.side-nav-active-pane.jsx-695095782{-webkit-flex:1;-ms-flex:1;flex:1;height:100%;overflow:hidden;}</style><style id="__jsx-3967925514">.file-header-format-button.jsx-3967925514{background-color:#FFF;padding:0 10px;height:100%;width:36px;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;cursor:pointer;}
.file-header-format-button.jsx-3967925514:hover{color:#78b087;}
.file-header-format-icon.jsx-3967925514{height:16px;width:16px;cursor:pointer;}
.file-header-format-button.jsx-3967925514 .cls-1{fill:rgb(119, 119, 119);}
.file-header-format-button.jsx-3967925514:hover .cls-1{fill:#7998a6;}
.file-header-format-button.jsx-3967925514 .cls-2{fill:#FFF;}</style><style id="__jsx-1990457210">.history-icon-svg .cls-1,.history-icon-svg .cls-2{fill:currentColor;}
.history-icon-svg .cls-1{fill-rule:evenodd;}</style><style id="__jsx-1735852555">.save-status.jsx-1735852555{display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;width:80px;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;height:100%;font-family:Questrial;font-size:16px;opacity:0.7;border:0;color:rgb(180, 180, 180);margin-left:15px;}
.save-status.jsx-1735852555 span.jsx-1735852555{border-bottom:1px solid transparent;}
.save-status.jsx-1735852555:hover{color:#7998a6;}
.save-status.jsx-1735852555 .history-icon.jsx-1735852555{width:18px;height:16px;margin-right:5px;}
.save-status.jsx-1735852555:hover span.jsx-1735852555{border-bottom:1px solid #7998a6;}</style><style id="__jsx-3307283349">.file-header.jsx-3307283349{position:relative;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;-webkit-box-pack:start;-webkit-justify-content:flex-start;-ms-flex-pack:start;justify-content:flex-start;background:#FFFFFF;position:relative;overflow:hidden;}
.file-header-name.jsx-3307283349{display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-box-pack:space-around;-webkit-justify-content:space-around;-ms-flex-pack:space-around;justify-content:space-around;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;background-color:rgb(245, 245, 245);height:100%;font-family:'Questrial',sans-serif;text-align:center;color:rgb(119, 119, 119);font-size:14px;}
.file-header-editor-loading.jsx-3307283349{height:23px;width:30px;background-image:url(/public/images/loading_dots.gif);background-position:center center;background-repeat:no-repeat;background-size:contain;}
.file-header-dl-info.jsx-3307283349{width:16px;height:16px;background-image:url(/public/images/information.png);background-position:center center;background-repeat:no-repeat;background-size:contain;cursor:pointer;}
.file-header-dl-info.jsx-3307283349:hover{background-image:url(/public/images/information_hover.png);}
.file-header-loading-bar.jsx-3307283349{width:40px;height:2px;position:absolute;top:0;left:0;background:#2b91af;-webkit-animation:loading-bar-move-jsx-3307283349 2.5s ease-in-out 0s infinite;animation:loading-bar-move-jsx-3307283349 2.5s ease-in-out 0s infinite;-webkit-animation-fill-mode:forward;animation-fill-mode:forward;}
@-webkit-keyframes loading-bar-move-jsx-3307283349{0%{left:-7%;}100%{left:105%;}}
@keyframes loading-bar-move-jsx-3307283349{0%{left:-7%;}100%{left:105%;}}</style><style id="__jsx-2800815994">.static-editor.jsx-2800815994{height:100%;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;font-size:14px;font-family:"Droid Sans Mono",monospace,monospace,"Droid Sans Fallback";line-height:21px;position:relative;outline:0;border:0;background:#FFFFFF;overflow:auto;}
.static-editor.jsx-2800815994:focus.jsx-2800815994::after{position:absolute;top:0;left:0;height:100%;width:100%;padding-top:50px;box-sizing:border-box;background:rgba(255,255,255,0.7);content:"Editor is loading please wait";font-family:"Questrial",sans-serif;font-size:18px;text-align:center;color:rgb(61,61,61);}
.static-editor-gutter.jsx-2800815994{width:68px;white-space:pre-wrap;}
.static-editor-line-num.jsx-2800815994{width:42px;display:inline-block;text-align:right;color:#2b91af;margin-left:0px;}
.static-editor-code.jsx-2800815994{width:calc(100% - 68px);white-space:pre-wrap;tab-size:2;}</style><style id="__jsx-2115772186">.hljs-dark{background:#1E1E1E;color:#DCDCDC;}
.hljs-dark .hljs-keyword,.hljs-dark .hljs-literal,.hljs-dark .hljs-symbol,.hljs-dark .hljs-name{color:#569CD6;}
.hljs-dark .hljs-link{color:#569CD6;-webkit-text-decoration:underline;text-decoration:underline;}
.hljs-dark .hljs-built_in,.hljs-dark .hljs-type{color:#4EC9B0;}
.hljs-dark .hljs-number,.hljs-dark .hljs-class{color:#B8D7A3;}
.hljs-dark .hljs-string,.hljs-dark .hljs-meta-string{color:#D69D85;}
.hljs-dark .hljs-regexp,.hljs-dark .hljs-template-tag{color:#d16969;}
.hljs-dark .hljs-title{color:#dcdcaa;}
.hljs-dark .hljs-subst,.hljs-dark .hljs-function,.hljs-dark .hljs-params,.hljs-dark .hljs-formula{color:#DCDCDC;}
.hljs-dark .hljs-comment,.hljs-dark .hljs-quote{color:#57A64A;}
.hljs-comment{font-style:normal;}
.hljs-dark .hljs-doctag{color:#608B4E;}
.hljs-dark .hljs-meta,.hljs-dark .hljs-meta-keyword,.hljs-dark .hljs-tag{color:#9B9B9B;}
.hljs-dark .hljs-variable,.hljs-dark .hljs-template-variable{color:#BD63C5;}
.hljs-dark .hljs-attr,.hljs-dark .hljs-attribute,.hljs-dark .hljs-builtin-name{color:#9CDCFE;}
.hljs-dark .hljs-section{color:gold;}
.hljs-dark .hljs-emphasis{font-style:italic;}
.hljs-dark .hljs-strong{font-weight:bold;}
.hljs-dark .hljs-bullet,.hljs-dark .hljs-selector-tag,.hljs-dark .hljs-selector-id,.hljs-dark .hljs-selector-class,.hljs-dark .hljs-selector-attr,.hljs-dark .hljs-selector-pseudo{color:#D7BA7D;}
.hljs-dark .hljs-addition{background-color:#144212;display:inline-block;width:100%;}
.hljs-dark .hljs-deletion{background-color:#600;display:inline-block;width:100%;}</style><style id="__jsx-2669800056">.examples-placeholder.jsx-2669800056{position:absolute;z-index:1;font-family:Monaco,Menlo,"Ubuntu Mono",Consolas,source-code-pro,monospace;color:rgb(180, 180, 180);font-size:14px;left:68px;line-height:21px;right:0;top:2px;pointer-events:none;}
.examples-placeholder-link.jsx-2669800056{cursor:pointer;pointer-events:all;color:#5192B3;-webkit-text-decoration:none;text-decoration:none;}
.examples-placeholder-link.jsx-2669800056:hover{-webkit-text-decoration:underline;text-decoration:underline;}
.editor-loading-overlay.jsx-2669800056{position:absolute;right:0;top:0;z-index:2;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-box-pack:center;-webkit-justify-content:center;-ms-flex-pack:center;justify-content:center;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;height:100%;width:100%;background:#FFFFFF;overflow:hidden;}</style><style id="__jsx-1542287934">.workspace-page-wrapper.jsx-1542287934{height:calc(100% - 0px);width:100%;position:fixed;left:0px;right:0px;top:0px;bottom:0px;}
.migrating-spinner.jsx-1542287934{display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-box-pack:center;-webkit-justify-content:center;-ms-flex-pack:center;justify-content:center;-webkit-align-items:center;-webkit-box-align:center;-ms-flex-align:center;align-items:center;position:fixed;left:0;top:0;width:100%;height:100%;background-color:rgba(0,0,0,0.4);z-index:500000;}</style><style id="__jsx-3485800696">#nprogress{pointer-events:none;}
.nprogress-bar{background:#4183C4;position:fixed;z-index:400001;top:0;left:0;width:100%;height:2px;}
.nprogress-static-css-bar{width:0;-webkit-animation:10s ease-out 750ms 1 normal both running nprogress-widen;animation:10s ease-out 750ms 1 normal both running nprogress-widen;}
@-webkit-keyframes nprogress-widen{0%{width:0;}100%{width:90%;}}
@keyframes nprogress-widen{0%{width:0;}100%{width:90%;}}
.nprogress-peg{display:block;position:absolute;right:0px;width:100px;height:100%;box-shadow:0 0 10px #4183C4,0 0 5px #4183C4;opacity:1.0;-webkit-transform:rotate(3deg) translate(0px,-4px);-ms-transform:rotate(3deg) translate(0px,-4px);-webkit-transform:rotate(3deg) translate(0px,-4px);-ms-transform:rotate(3deg) translate(0px,-4px);transform:rotate(3deg) translate(0px,-4px);}
.nprogress-spinner{display:block;position:fixed;z-index:1031;top:15px;right:15px;}
.nprogress-spinner-icon{width:18px;height:18px;box-sizing:border-box;visibility:hidden;border:solid 2px transparent;border-top-color:#29d;border-left-color:#29d;border-radius:50%;-webkit-animation:nprogress-spinner 400ms linear infinite;-webkit-animation:nprogress-spinner 400ms linear infinite;animation:nprogress-spinner 400ms linear infinite;-webkit-animation-delay:11s;animation-delay:11s;}
@-webkit-keyframes nprogress-spinner{0%{-webkit-transform:rotate(0deg);visibility:visible;}100%{-webkit-transform:rotate(360deg);}}
@-webkit-keyframes nprogress-spinner{0%{-webkit-transform:rotate(0deg);visibility:visible;}100%{-webkit-transform:rotate(360deg);-ms-transform:rotate(360deg);transform:rotate(360deg);}}
@keyframes nprogress-spinner{0%{-webkit-transform:rotate(0deg);visibility:visible;}100%{-webkit-transform:rotate(360deg);-ms-transform:rotate(360deg);transform:rotate(360deg);}}</style><script type="text/javascript">KNOWN_LANGUAGES = JSON.parse(atob('eyJjbG9qdXJlIjp7ImRpc3BsYXlOYW1lIjoiQ2xvanVyZSIsInRhZ2xpbmUiOiJBIG1vZGVybiBKVk0tYmFzZWQgTGlzcCBkaWFsZWN0IHdpdGggYSBmb2N1cyBvbiBpbW11dGFiaWxpdHkiLCJrZXkiOiJjbG9qdXJlIiwiZW50cnlwb2ludCI6Im1haW4uY2xqIiwiZXh0IjoiY2xqIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOmZhbHNlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjp0cnVlLCJoYXNMYW5ndWFnZVNlcnZlciI6ZmFsc2UsImhlYWRlciI6IkNsb2p1cmUgMS44LjBcbkphdmEgSG90U3BvdChUTSkgNjQtQml0IFNlcnZlciBWTSAxLjguMF85MS1iMTQiLCJjYXRlZ29yeSI6IlByYWN0aWNhbCIsImljb24iOiJodHRwczovL3JlcGwuaXQvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvY2xvanVyZS5zdmciLCJ0ZW1wbGF0ZSI6IiIsIm1hdGNoaW5ncyI6W10sImVuZ2luZSI6ImdvdmFsIn0sImhhc2tlbGwiOnsiZGlzcGxheU5hbWUiOiJIYXNrZWxsIiwidGFnbGluZSI6IkFuIGFkdmFuY2VkLCBwdXJlbHkgZnVuY3Rpb25hbCBwcm9ncmFtbWluZyBsYW5ndWFnZSIsImtleSI6Imhhc2tlbGwiLCJlbnRyeXBvaW50IjoibWFpbi5ocyIsImV4dCI6ImhzIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOmZhbHNlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjp0cnVlLCJoYXNMYW5ndWFnZVNlcnZlciI6ZmFsc2UsImhlYWRlciI6IkdIQ2ksIHZlcnNpb24gOC4wLjEiLCJjYXRlZ29yeSI6IlByYWN0aWNhbCIsImljb24iOiJodHRwczovL3JlcGwuaXQvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvaGFza2VsbC5zdmciLCJ0ZW1wbGF0ZSI6IiIsIm1hdGNoaW5ncyI6W10sImVuZ2luZSI6ImdvdmFsIn0sImtvdGxpbiI6eyJkaXNwbGF5TmFtZSI6IktvdGxpbiAoYmV0YSkiLCJ0YWdsaW5lIjoiU3RhdGljYWxseSB0eXBlZCBwcm9ncmFtbWluZyBsYW5ndWFnZSBpbnRlcm9wZXJhYmxlIHdpdGggSmF2YSBhbmQgQW5kcm9pZCIsImtleSI6ImtvdGxpbiIsImVudHJ5cG9pbnQiOiJtYWluLmt0IiwiZXh0Ijoia3QiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOmZhbHNlLCJoYXNGb3JtYXQiOmZhbHNlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjp0cnVlLCJoYXNMYW5ndWFnZVNlcnZlciI6ZmFsc2UsImhlYWRlciI6IldlbGNvbWUgdG8gS290bGluIHZlcnNpb24gMS4wLjMiLCJjYXRlZ29yeSI6IlByYWN0aWNhbCIsImljb24iOiJodHRwczovL3JlcGwuaXQvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMva290bGluLnN2ZyIsInRlbXBsYXRlIjoiIiwibWF0Y2hpbmdzIjpbXSwiZW5naW5lIjoiZ292YWwifSwicWJhc2ljIjp7ImRpc3BsYXlOYW1lIjoiUUJhc2ljIiwiZW5naW5lIjoicmVwbGJveCIsInRhZ2xpbmUiOiJTdHJ1Y3R1cmVkIHByb2dyYW1taW5nIGZvciBiZWdpbm5lcnMuIiwiaGVhZGVyIjoiUUJhc2ljIChxYi5qcylcbkNvcHlyaWdodCAoYykgMjAxMCBTdGV2ZSBIYW5vdiIsImNhdGVnb3J5IjoiQ2xhc3NpYyIsImV4dCI6ImJhcyIsImljb24iOiIvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvbGFuZ3VhZ2Uuc3ZnIiwia2V5IjoicWJhc2ljIn0sImZvcnRoIjp7ImRpc3BsYXlOYW1lIjoiRm9ydGgiLCJlbmdpbmUiOiJyZXBsYm94IiwidGFnbGluZSI6IkFuIGludGVyYWN0aXZlIHN0YWNrLW9yaWVudGVkIGxhbmd1YWdlLiIsImhlYWRlciI6IkpTLUZvcnRoIDAuNTIwMDgwNDE3MTM0MlxuaHR0cDovL3d3dy5mb3J0aGZyZWFrLm5ldC9qc2ZvcnRoLmh0bWxcblRoaXMgcHJvZ3JhbSBpcyBwdWJsaXNoZWQgdW5kZXIgdGhlIEdQTC4iLCJjYXRlZ29yeSI6IkNsYXNzaWMiLCJleHQiOiJmdGgiLCJpY29uIjoiL3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2xhbmd1YWdlLnN2ZyIsImtleSI6ImZvcnRoIn0sImxvbGNvZGUiOnsiZGlzcGxheU5hbWUiOiJMT0xDT0RFIiwiZW5naW5lIjoicmVwbGJveCIsInRhZ2xpbmUiOiJUaGUgYmFzaWMgbGFuZ3VhZ2Ugb2YgbG9sY2F0cy4iLCJoZWFkZXIiOiJMT0xDT0RFIHYxLjIgKGxvbC1jb2ZmZWUpXG5Db3B5cmlnaHQgKGMpIDIwMTEgTWF4IFNoYXdhYmtlaCIsImNhdGVnb3J5IjoiRXNvdGVyaWMiLCJleHQiOiJsb2wiLCJpY29uIjoiL3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2xvbGNvZGUuc3ZnIiwia2V5IjoibG9sY29kZSJ9LCJicmFpbmZ1Y2siOnsiZGlzcGxheU5hbWUiOiJCcmFpbkYiLCJlbmdpbmUiOiJyZXBsYm94IiwidGFnbGluZSI6IkEgcHVyZSBUdXJpbmcgbWFjaGluZSBjb250cm9sbGVyLiIsImhlYWRlciI6IkJyYWluRioqKiwgYmZqc1xuQ29weXJpZ2h0IChjKSAyMDExIEFtamFkIE1hc2FkIiwiY2F0ZWdvcnkiOiJFc290ZXJpYyIsImV4dCI6ImJmIiwiaWNvbiI6Ii9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9icmFpbmZ1Y2suc3ZnIiwia2V5IjoiYnJhaW5mdWNrIn0sImVtb3RpY29uIjp7ImRpc3BsYXlOYW1lIjoiRW1vdGljb24iLCJlbmdpbmUiOiJyZXBsYm94IiwidGFnbGluZSI6IlByb2dyYW1taW5nIHdpdGggYW4gZXh0cmEgZG9zZSBvZiBzbWlsZS4iLCJoZWFkZXIiOiJFbW90aWNvbiB2MS41IChlbW90aWNvZmZlZSlcbkNvcHlyaWdodCAoYykgMjAxMSBBbWphZCBNYXNhZCIsImNhdGVnb3J5IjoiRXNvdGVyaWMiLCJleHQiOiJlbW90aWNvbiIsImljb24iOiIvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvbGFuZ3VhZ2Uuc3ZnIiwia2V5IjoiZW1vdGljb24ifSwiYmxvb3AiOnsiZGlzcGxheU5hbWUiOiJCbG9vcCIsImVuZ2luZSI6InJlcGxib3giLCJ0YWdsaW5lIjoiTm90aGluZyBidXQgYm91bmRlZCBsb29wcy4iLCJoZWFkZXIiOiJCbG9vUGpzXG5Db3B5cmlnaHQgKGMpIDIwMDUgVGltIENhbWVyb24gUnlhblxuQmFzZWQgb24gUGVybCBjb2RlIGJ5IEpvaG4gQ293YW4sIDE5OTQiLCJjYXRlZ29yeSI6IkVzb3RlcmljIiwiZXh0IjoiYmxvb3AiLCJpY29uIjoiL3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2xhbmd1YWdlLnN2ZyIsImtleSI6ImJsb29wIn0sInJlYWN0X25hdGl2ZSI6eyJkaXNwbGF5TmFtZSI6IlJlYWN0IE5hdGl2ZSIsInRhZ2xpbmUiOiJDcmVhdGUgbW9iaWxlIGFwcHMgd2l0aCBSZWFjdCBOYXRpdmUgYW5kIEV4cG8iLCJrZXkiOiJyZWFjdF9uYXRpdmUiLCJlbnRyeXBvaW50IjoiaW5kZXguanMiLCJleHQiOiJqcyIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOnRydWUsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOmZhbHNlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJSZWFjdCBOYXRpdmUiLCJjYXRlZ29yeSI6IlBvcHVsYXIiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL3JlYWN0LnN2ZyIsInRlbXBsYXRlIjoiaW1wb3J0IFJlYWN0IGZyb20gJ3JlYWN0JztcbmltcG9ydCB7IEJ1dHRvbiwgVmlldyB9IGZyb20gJ3JlYWN0LW5hdGl2ZSc7XG5cbmNvbnN0IHByZXNzQ291bnQgPSAwO1xuXG5leHBvcnQgZGVmYXVsdCBjbGFzcyBBcHAgZXh0ZW5kcyBSZWFjdC5Db21wb25lbnQge1xuICBjb25zdHJ1Y3Rvcihwcm9wcykge1xuICAgIHN1cGVyKHByb3BzKVxuXG4gICAgdGhpcy5zdGF0ZSA9IHtcbiAgICAgIHByZXNzQ291bnQ6IDAsXG4gICAgfVxuICB9XG5cbiAgY29tcG9uZW50RGlkVXBkYXRlKCkge1xuICAgIGNvbnNvbGUubG9nKCdQcmVzcyBDb3VudDogJywgdGhpcy5zdGF0ZS5wcmVzc0NvdW50KTtcbiAgfVxuXG4gIHJlbmRlcigpIHtcbiAgICByZXR1cm4gKFxuICAgICAgPFZpZXcgc3R5bGU9e3twb3NpdGlvbjogJ3JlbGF0aXZlJywgdG9wOiAxMDB9fT5cbiAgICAgICAgPEJ1dHRvblxuICAgICAgICAgIHRpdGxlPVwiUHJlc3MgTWUhXCJcbiAgICAgICAgICBvblByZXNzPXsoKSA9PiB0aGlzLnNldFN0YXRlKHtwcmVzc0NvdW50OiB0aGlzLnN0YXRlLnByZXNzQ291bnQgKyAxfSl9XG4gICAgICAgIC8+XG4gICAgICA8L1ZpZXc+XG4gICAgKVxuICB9XG59IiwibWF0Y2hpbmdzIjpbXSwiZW5naW5lIjoiZ292YWwifSwidW5sYW1iZGEiOnsiZGlzcGxheU5hbWUiOiJVbmxhbWJkYSIsImVuZ2luZSI6InJlcGxib3giLCJ0YWdsaW5lIjoiRnVuY3Rpb25hbCBwdXJpdHkgZ2l2ZW4gZm9ybS4iLCJoZWFkZXIiOiJVbmxhbWJkYSB2Mi4wICh1bmxhbWJkYS1jb2ZmZWUpXG5Db3B5cmlnaHQgKGMpIDIwMTEgTWF4IFNoYXdhYmtlaCIsImNhdGVnb3J5IjoiRXNvdGVyaWMiLCJleHQiOiJ1bmwiLCJpY29uIjoiL3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2xhbmd1YWdlLnN2ZyIsImtleSI6InVubGFtYmRhIn0sImphdmFzY3JpcHQiOnsiZGlzcGxheU5hbWUiOiJKYXZhU2NyaXB0IiwiZW5naW5lIjoicmVwbGJveCIsInRhZ2xpbmUiOiJUaGUgZGUgZmFjdG8gbGFuZ3VhZ2Ugb2YgdGhlIFdlYi4iLCJjYXRlZ29yeSI6IldlYiIsImhlYWRlciI6Ik5hdGl2ZSBCcm93c2VyIEphdmFTY3JpcHQiLCJleHQiOiJqcyIsImljb24iOiIvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvamF2YXNjcmlwdC5zdmciLCJrZXkiOiJqYXZhc2NyaXB0In0sImJhYmVsIjp7ImRpc3BsYXlOYW1lIjoiRVM2IiwiZW5naW5lIjoicmVwbGJveCIsInRhZ2xpbmUiOiJOZXh0IGdlbmVyYXRpb24gSmF2YVNjcmlwdC4iLCJoZWFkZXIiOiJCYWJlbCBDb21waWxlciB2Ni40LjRcbkNvcHlyaWdodCAoYykgMjAxNC0yMDE1IFNlYmFzdGlhbiBNY0tlbnppZSIsImNhdGVnb3J5IjoiSGlkZGVuIiwiZXh0IjoianMiLCJpY29uIjoiL3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2phdmFzY3JpcHQuc3ZnIiwia2V5IjoiYmFiZWwifSwiY29mZmVlc2NyaXB0Ijp7ImRpc3BsYXlOYW1lIjoiQ29mZmVlU2NyaXB0IiwiZW5naW5lIjoicmVwbGJveCIsInRhZ2xpbmUiOiJVbmZhbmN5IEphdmFTY3JpcHQuIiwiaGVhZGVyIjoiQ29mZmVlU2NyaXB0IHYxLjEwXG5Db3B5cmlnaHQgKGMpIDIwMTYsIEplcmVteSBBc2hrZW5hcyIsImNhdGVnb3J5IjoiV2ViIiwiZXh0IjoiY29mZmVlIiwiaWNvbiI6Ii9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9jb2ZmZWVzY3JpcHQuc3ZnIiwia2V5IjoiY29mZmVlc2NyaXB0In0sInNjaGVtZSI6eyJkaXNwbGF5TmFtZSI6IlNjaGVtZSIsImVuZ2luZSI6InJlcGxib3giLCJ0YWdsaW5lIjoiQW4gZWxlZ2FudCBkeW5hbWljIGRpYWxlY3Qgb2YgTGlzcC4iLCJoZWFkZXIiOiJCaXdhU2NoZW1lIEludGVycHJldGVyIHZlcnNpb24gMC42LjRcbkNvcHlyaWdodCAoQykgMjAwNy0yMDE0IFl1dGFrYSBIQVJBIGFuZCB0aGUgQml3YVNjaGVtZSB0ZWFtIiwiY2F0ZWdvcnkiOiJQcmFjdGljYWwiLCJleHQiOiJzYyIsImljb24iOiIvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvc2NoZW1lLnN2ZyIsImtleSI6InNjaGVtZSJ9LCJhcGwiOnsiZGlzcGxheU5hbWUiOiJBUEwiLCJlbmdpbmUiOiJyZXBsYm94IiwidGFnbGluZSI6IkFuIGFycmF5LW9yaWVudGVkIGxhbmd1YWdlIHVzaW5nIGZ1bm55IGNoYXJhY3RlcnMuIiwiaGVhZGVyIjoibmduL2FwbCIsImNhdGVnb3J5IjoiQ2xhc3NpYyIsImV4dCI6ImFwbCIsImtleSI6ImFwbCJ9LCJsdWEiOnsiZGlzcGxheU5hbWUiOiJMdWEiLCJ0YWdsaW5lIjoiQSBsaWdodHdlaWdodCBtdWx0aS1wYXJhZGlnbSBzY3JpcHRpbmcgbGFuZ3VhZ2UuIiwia2V5IjoibHVhIiwiZW50cnlwb2ludCI6Im1haW4ubHVhIiwiZXh0IjoibHVhIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjpmYWxzZSwiaGFzRm9ybWF0Ijp0cnVlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjp0cnVlLCJoYXNMYW5ndWFnZVNlcnZlciI6ZmFsc2UsImhlYWRlciI6Ikx1YSA1LjEgIENvcHlyaWdodCAoQykgMTk5NC0yMDA2IEx1YS5vcmcsIFBVQy1SaW9cbltHQ0MgNC4yLjEgKExMVk0sIEVtc2NyaXB0ZW4gMS41KV0gb24gbGludXgyIiwiY2F0ZWdvcnkiOiJQcmFjdGljYWwiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2x1YS5zdmciLCJ0ZW1wbGF0ZSI6IiIsIm1hdGNoaW5ncyI6W10sImVuZ2luZSI6ImdvdmFsIn0sInB5dGhvbiI6eyJkaXNwbGF5TmFtZSI6IlB5dGhvbiIsInRhZ2xpbmUiOiJBIGR5bmFtaWMgbGFuZ3VhZ2UgZW1waGFzaXppbmcgcmVhZGFiaWxpdHkuIiwia2V5IjoicHl0aG9uIiwiZW50cnlwb2ludCI6Im1haW4ucHkiLCJleHQiOiJweSIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOnRydWUsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6dHJ1ZSwiaGFzRXZhbCI6dHJ1ZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOnRydWUsImhlYWRlciI6IlB5dGhvbiAyLjcuMTAgKGRlZmF1bHQsIEp1bCAxNCAyMDE1LCAxOTo0NjoyNylcbltHQ0MgNC44LjJdIG9uIGxpbnV4IiwiY2F0ZWdvcnkiOiJQcmFjdGljYWwiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL3B5dGhvbi5zdmciLCJ0ZW1wbGF0ZSI6IiIsIm1hdGNoaW5ncyI6W1sieyIsIn0iXSxbIigiLCIpIl0sWyJbIiwiXSJdXSwiZW5naW5lIjoiZ292YWwifSwicnVieSI6eyJkaXNwbGF5TmFtZSI6IlJ1YnkiLCJ0YWdsaW5lIjoiQSBuYXR1cmFsIGR5bmFtaWMgb2JqZWN0LW9yaWVudGVkIGxhbmd1YWdlLiIsImtleSI6InJ1YnkiLCJlbnRyeXBvaW50IjoibWFpbi5yYiIsImV4dCI6InJiIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6dHJ1ZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6dHJ1ZSwiaGFzTGlicmFyaWVzIjp0cnVlLCJoYXNFdmFsIjp0cnVlLCJoYXNMYW5ndWFnZVNlcnZlciI6dHJ1ZSwiaGVhZGVyIjoicnVieSAyLjUuMHAwICgyMDE3LTEyLTI1IHJldmlzaW9uIDYxNDY4KSBbeDg2XzY0LWxpbnV4XSIsImNhdGVnb3J5IjoiUHJhY3RpY2FsIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9ydWJ5LnN2ZyIsInRlbXBsYXRlIjoiIiwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCJ9LCJyb3kiOnsiZGlzcGxheU5hbWUiOiJSb3kiLCJlbmdpbmUiOiJyZXBsYm94IiwidGFnbGluZSI6IlNtYWxsIGZ1bmN0aW9uYWwgbGFuZ3VhZ2UgdGhhdCBjb21waWxlcyB0byBKYXZhU2NyaXB0LiIsImhlYWRlciI6IlJveSAwLjEuM1xuQ29weXJpZ2h0IChDKSAyMDExIEJyaWFuIE1jS2VubmEiLCJjYXRlZ29yeSI6IldlYiIsImV4dCI6InJveSIsImljb24iOiIvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvcm95LnN2ZyIsImtleSI6InJveSJ9LCJwaHAiOnsiZGlzcGxheU5hbWUiOiJQSFAiLCJ0YWdsaW5lIjoiQSBwb3B1bGFyIGdlbmVyYWwtcHVycG9zZSBzY3JpcHRpbmcgbGFuZ3VhZ2UuIiwia2V5IjoicGhwIiwiZW50cnlwb2ludCI6Im1haW4ucGhwIiwiZXh0IjoicGhwIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjpmYWxzZSwiaGFzRm9ybWF0IjpmYWxzZSwiaGFzTGlicmFyaWVzIjpmYWxzZSwiaGFzRXZhbCI6dHJ1ZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJQSFAgNy4wLjggKGNsaSkgKGJ1aWx0OiBKdW4gMjMgMjAxNiAyMzozOToxNCkgKCBOVFMgKVxuQ29weXJpZ2h0IChjKSAxOTk3LTIwMTYgVGhlIFBIUCBHcm91cFxuWmVuZCBFbmdpbmUgdjMuMC4wLCBDb3B5cmlnaHQgKGMpIDE5OTgtMjAxNiBaZW5kIFRlY2hub2xvZ2llcyIsImNhdGVnb3J5IjoiUHJhY3RpY2FsIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9waHAuc3ZnIiwidGVtcGxhdGUiOiIiLCJtYXRjaGluZ3MiOltbInsiLCJ9Il0sWyIoIiwiKSJdLFsiWyIsIl0iXV0sImVuZ2luZSI6ImdvdmFsIn0sInB5dGhvbjMiOnsiZGlzcGxheU5hbWUiOiJQeXRob24zIiwidGFnbGluZSI6IkEgZHluYW1pYyBsYW5ndWFnZSBlbXBoYXNpemluZyByZWFkYWJpbGl0eS4iLCJrZXkiOiJweXRob24zIiwiZW50cnlwb2ludCI6Im1haW4ucHkiLCJleHQiOiJweSIsImhhc0xpbnQiOnRydWUsImhhc1VuaXRUZXN0cyI6dHJ1ZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6dHJ1ZSwiaGFzTGlicmFyaWVzIjp0cnVlLCJoYXNFdmFsIjp0cnVlLCJoYXNMYW5ndWFnZVNlcnZlciI6dHJ1ZSwiaGVhZGVyIjoiUHl0aG9uIDMuNi4xIChkZWZhdWx0LCBEZWMgMjAxNSwgMTM6MDU6MTEpXG5bR0NDIDQuOC4yXSBvbiBsaW51eCIsImNhdGVnb3J5IjoiUHJhY3RpY2FsIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9weXRob24uc3ZnIiwidGVtcGxhdGUiOiIiLCJtYXRjaGluZ3MiOltbInsiLCJ9Il0sWyIoIiwiKSJdLFsiWyIsIl0iXV0sImVuZ2luZSI6ImdvdmFsIn0sIm5vZGVqcyI6eyJkaXNwbGF5TmFtZSI6Ik5vZGVqcyIsInRhZ2xpbmUiOiJFdmVudGVkIEkvTyBmb3IgdjggSmF2YXNjcmlwdC4iLCJrZXkiOiJub2RlanMiLCJlbnRyeXBvaW50IjoiaW5kZXguanMiLCJleHQiOiJqcyIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOnRydWUsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6dHJ1ZSwiaGFzRXZhbCI6dHJ1ZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJub2RlIHY5LjcuMSBsaW51eC9hbWQ2NCIsImNhdGVnb3J5IjoiUHJhY3RpY2FsIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9ub2RlanMuc3ZnIiwidGVtcGxhdGUiOiIiLCJtYXRjaGluZ3MiOltbInsiLCJ9Il0sWyIoIiwiKSJdLFsiWyIsIl0iXV0sImVuZ2luZSI6ImdvdmFsIn0sImVuenltZSI6eyJkaXNwbGF5TmFtZSI6IkVuenltZSIsInRhZ2xpbmUiOiJBIEphdmFTY3JpcHQgVGVzdGluZyB1dGlsaXR5IGZvciBSZWFjdCIsImtleSI6ImVuenltZSIsImVudHJ5cG9pbnQiOiJpbmRleC5qcyIsImV4dCI6ImpzIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjpmYWxzZSwiaGFzRm9ybWF0IjpmYWxzZSwiaGFzTGlicmFyaWVzIjpmYWxzZSwiaGFzRXZhbCI6dHJ1ZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJub2RlIHY3LjQgbGludXgvYW1kNjQiLCJjYXRlZ29yeSI6IlRlc3RpbmciLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL3JlYWN0LnN2ZyIsInRlbXBsYXRlIjoiaW1wb3J0IHsgc2hhbGxvdywgbW91bnQgfSBmcm9tICdlbnp5bWUnO1xuXG5mdW5jdGlvbiBDb21wb25lbnQoKSB7IHJldHVybiA8ZGl2IC8+OyB9XG5cbmNvbnN0IHdyYXBwZXIgPSBzaGFsbG93KDxDb21wb25lbnQgLz4pO1xuXG5hc3NlcnQod3JhcHBlci5maW5kKCdkaXYnKS5sZW5ndGggPT09IDEpIiwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCJ9LCJnbyI6eyJkaXNwbGF5TmFtZSI6IkdvIiwidGFnbGluZSI6IlN0YXRpY2FsbHkgdHlwZWQgeWV0IGV4cHJlc3NpdmUgbGFuZ3VhZ2Ugd2l0aCBhIGZvY3VzIG9uIGNvbmN1cnJlbmN5LiIsImtleSI6ImdvIiwiZW50cnlwb2ludCI6Im1haW4uZ28iLCJleHQiOiJnbyIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOmZhbHNlLCJoYXNQcm9qZWN0TW9kZSI6dHJ1ZSwiaGFzRm9ybWF0Ijp0cnVlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOnRydWUsImhlYWRlciI6ImdvIHZlcnNpb24gZ28xLjkuNCBsaW51eC9hbWQ2NCIsImNhdGVnb3J5IjoiUHJhY3RpY2FsIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9nby5zdmciLCJ0ZW1wbGF0ZSI6InBhY2thZ2UgbWFpblxuXG5pbXBvcnQgXCJmbXRcIlxuXG5mdW5jIG1haW4oKSB7XG4gIGZtdC5QcmludGxuKFwiSGVsbG8gV29ybGRcIilcbn0iLCJtYXRjaGluZ3MiOltbInsiLCJ9Il0sWyIoIiwiKSJdLFsiWyIsIl0iXV0sImVuZ2luZSI6ImdvdmFsIn0sImphdmEiOnsiZGlzcGxheU5hbWUiOiJKYXZhIiwidGFnbGluZSI6IkEgY29uY3VycmVudCwgY2xhc3MtYmFzZWQsIHN0YXRpY2FsbHkgdHlwZWQgb2JqZWN0LW9yaWVudGVkIGxhbmd1YWdlLiIsImtleSI6ImphdmEiLCJlbnRyeXBvaW50IjoiTWFpbi5qYXZhIiwiZXh0IjoiamF2YSIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOnRydWUsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOmZhbHNlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOnRydWUsImhlYWRlciI6ImphdmEgdmVyc2lvbiBcIjEuOC4wXzMxXCJcbkphdmEoVE0pIFNFIFJ1bnRpbWUgRW52aXJvbm1lbnQgKGJ1aWxkIDEuOC4wXzMxLWIxMylcbkphdmEgSG90U3BvdChUTSkgNjQtQml0IFNlcnZlciBWTSAoYnVpbGQgMjUuMzEtYjA3LCBtaXhlZCBtb2RlKSIsImNhdGVnb3J5IjoiUHJhY3RpY2FsIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9qYXZhLnN2ZyIsInRlbXBsYXRlIjoiY2xhc3MgTWFpbiB7XG4gIHB1YmxpYyBzdGF0aWMgdm9pZCBtYWluKFN0cmluZ1tdIGFyZ3MpIHtcbiAgICBTeXN0ZW0ub3V0LnByaW50bG4oXCJIZWxsbyB3b3JsZCFcIik7XG4gIH1cbn0iLCJtYXRjaGluZ3MiOltbInsiLCJ9Il0sWyIoIiwiKSJdLFsiWyIsIl0iXV0sImVuZ2luZSI6ImdvdmFsIn0sImNwcCI6eyJkaXNwbGF5TmFtZSI6IkMrKyIsInRhZ2xpbmUiOiJBIGdlbmVyYWwgcHVycG9zZSBzeXN0ZW0gcHJvZ3JhbW1pbmcgbGFuZ3VhZ2UuIiwia2V5IjoiY3BwIiwiZW50cnlwb2ludCI6Im1haW4uY3BwIiwiZXh0IjoiY3BwIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6ZmFsc2UsImhhc0V2YWwiOmZhbHNlLCJoYXNMYW5ndWFnZVNlcnZlciI6dHJ1ZSwiaGVhZGVyIjoiZ2NjIHZlcnNpb24gNC42LjMiLCJjYXRlZ29yeSI6IlByYWN0aWNhbCIsImljb24iOiJodHRwczovL3JlcGwuaXQvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvY3BwLnN2ZyIsInRlbXBsYXRlIjoiI2luY2x1ZGUgPGlvc3RyZWFtPlxuXG5pbnQgbWFpbigpIHtcbiAgc3RkOjpjb3V0IDw8IFwiSGVsbG8gV29ybGQhXFxuXCI7XG59IiwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCJ9LCJjcHAxMSI6eyJkaXNwbGF5TmFtZSI6IkMrKzExIiwidGFnbGluZSI6IkEgZ2VuZXJhbCBwdXJwb3NlIHN5c3RlbSBwcm9ncmFtbWluZyBsYW5ndWFnZS4iLCJrZXkiOiJjcHAxMSIsImVudHJ5cG9pbnQiOiJtYWluLmNwcCIsImV4dCI6ImNwcCIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOmZhbHNlLCJoYXNQcm9qZWN0TW9kZSI6dHJ1ZSwiaGFzRm9ybWF0Ijp0cnVlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOnRydWUsImhlYWRlciI6ImdjYyB2ZXJzaW9uIDQuNi4zIiwiY2F0ZWdvcnkiOiJQcmFjdGljYWwiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2NwcC5zdmciLCJ0ZW1wbGF0ZSI6IiNpbmNsdWRlIDxpb3N0cmVhbT5cblxuaW50IG1haW4oKSB7XG4gIHN0ZDo6Y291dCA8PCBcIkhlbGxvIFdvcmxkIVxcblwiO1xufSIsIm1hdGNoaW5ncyI6W1sieyIsIn0iXSxbIigiLCIpIl0sWyJbIiwiXSJdXSwiZW5naW5lIjoiZ292YWwifSwiYyI6eyJkaXNwbGF5TmFtZSI6IkMiLCJ0YWdsaW5lIjoiTG93LWxldmVsIGFuZCBjcm9zcy1wbGF0Zm9ybSBpbXBlcmF0aXZlIGxhbmd1YWdlLiIsImtleSI6ImMiLCJlbnRyeXBvaW50IjoibWFpbi5jIiwiZXh0IjoiYyIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOmZhbHNlLCJoYXNQcm9qZWN0TW9kZSI6dHJ1ZSwiaGFzRm9ybWF0Ijp0cnVlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOnRydWUsImhlYWRlciI6ImdjYyB2ZXJzaW9uIDQuNi4zIiwiY2F0ZWdvcnkiOiJQcmFjdGljYWwiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2Muc3ZnIiwidGVtcGxhdGUiOiIjaW5jbHVkZSA8c3RkaW8uaD5cblxuaW50IG1haW4odm9pZCkge1xuICBwcmludGYoXCJIZWxsbyBXb3JsZFxcblwiKTtcbiAgcmV0dXJuIDA7XG59IiwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCJ9LCJjc2hhcnAiOnsiZGlzcGxheU5hbWUiOiJDIyIsInRhZ2xpbmUiOiJBIE1pY3Jvc29mdCAuTkVUIHByb2dyYW1taW5nIGxhbmd1YWdlLiIsImtleSI6ImNzaGFycCIsImVudHJ5cG9pbnQiOiJtYWluLmNzIiwiZXh0IjoiY3MiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6ZmFsc2UsImhhc0xpYnJhcmllcyI6ZmFsc2UsImhhc0V2YWwiOmZhbHNlLCJoYXNMYW5ndWFnZVNlcnZlciI6ZmFsc2UsImhlYWRlciI6Ik1vbm8gQyMgY29tcGlsZXIgdmVyc2lvbiA0LjAuNC4wIiwiY2F0ZWdvcnkiOiJQcmFjdGljYWwiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2NzaGFycC5zdmciLCJ0ZW1wbGF0ZSI6InVzaW5nIFN5c3RlbTtcblxuY2xhc3MgTWFpbkNsYXNzIHtcbiAgcHVibGljIHN0YXRpYyB2b2lkIE1haW4gKHN0cmluZ1tdIGFyZ3MpIHtcbiAgICBDb25zb2xlLldyaXRlTGluZSAoXCJIZWxsbyBXb3JsZFwiKTtcbiAgfVxufSIsIm1hdGNoaW5ncyI6W1sieyIsIn0iXSxbIigiLCIpIl0sWyJbIiwiXSJdXSwiZW5naW5lIjoiZ292YWwifSwiZnNoYXJwIjp7ImRpc3BsYXlOYW1lIjoiRiMiLCJ0YWdsaW5lIjoiQSBNaWNyb3NvZnQgLk5FVCBmdW5jdGlvbmFsIHByb2dyYW1taW5nIGxhbmd1YWdlLiIsImtleSI6ImZzaGFycCIsImVudHJ5cG9pbnQiOiJtYWluLmZzIiwiZXh0IjoiZnMiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6ZmFsc2UsImhhc0xpYnJhcmllcyI6ZmFsc2UsImhhc0V2YWwiOmZhbHNlLCJoYXNMYW5ndWFnZVNlcnZlciI6ZmFsc2UsImhlYWRlciI6IlwiRiMgQ29tcGlsZXIgZm9yIEYjIDQuMCAoT3BlbiBTb3VyY2UgRWRpdGlvbikiLCJjYXRlZ29yeSI6IlByYWN0aWNhbCIsImljb24iOiJodHRwczovL3JlcGwuaXQvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvZnNoYXJwLnN2ZyIsInRlbXBsYXRlIjoiIiwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCJ9LCJ3ZWJfcHJvamVjdCI6eyJlbmdpbmUiOiJyZXBsYm94IiwiZGlzcGxheU5hbWUiOiJIVE1MLCBDU1MsIEpTIiwidGFnbGluZSI6IlRoZSBsYW5ndWFnZXMgdGhhdCBtYWtlIHVwIHRoZSB3ZWIuIiwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJoZWFkZXIiOiIiLCJjYXRlZ29yeSI6IkhpZGRlbiIsImV4dCI6Imh0bWwiLCJwcm9qZWN0X3RlbXBsYXRlIjpbeyJuYW1lIjoiaW5kZXguaHRtbCIsImNvbnRlbnQiOiI8IURPQ1RZUEUgaHRtbD5cbjxodG1sPlxuICA8aGVhZD5cbiAgICA8bWV0YSBjaGFyc2V0PVwidXRmLThcIj5cbiAgICA8bWV0YSBuYW1lPVwidmlld3BvcnRcIiBjb250ZW50PVwid2lkdGg9ZGV2aWNlLXdpZHRoXCI+XG4gICAgPHRpdGxlPnJlcGwuaXQ8L3RpdGxlPlxuICAgIDxsaW5rIGhyZWY9XCJpbmRleC5jc3NcIiByZWw9XCJzdHlsZXNoZWV0XCIgdHlwZT1cInRleHQvY3NzXCIgLz5cbiAgPC9oZWFkPlxuICA8Ym9keT5cbiAgICA8c2NyaXB0IHNyYz1cImluZGV4LmpzXCI+PC9zY3JpcHQ+XG4gIDwvYm9keT5cbjwvaHRtbD4iLCJpbmRleCI6MH0seyJuYW1lIjoiaW5kZXguanMiLCJjb250ZW50IjoiIiwiaW5kZXgiOjF9LHsibmFtZSI6ImluZGV4LmNzcyIsImNvbnRlbnQiOiIiLCJpbmRleCI6Mn1dLCJpY29uIjoiL3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL3dlYl9wcm9qZWN0LnN2ZyIsImtleSI6IndlYl9wcm9qZWN0In0sImh0bWwiOnsiZW5naW5lIjoicmVwbGJveCIsImRpc3BsYXlOYW1lIjoiSFRNTCwgQ1NTLCBKUyIsInRhZ2xpbmUiOiJUaGUgbGFuZ3VhZ2VzIHRoYXQgbWFrZSB1cCB0aGUgd2ViLiIsIm1hdGNoaW5ncyI6W1sieyIsIn0iXSxbIigiLCIpIl0sWyJbIiwiXSJdXSwiaGVhZGVyIjoiIiwiY2F0ZWdvcnkiOiJXZWIiLCJleHQiOiJodG1sIiwicHJvamVjdF90ZW1wbGF0ZSI6W3sibmFtZSI6ImluZGV4Lmh0bWwiLCJjb250ZW50IjoiPCFET0NUWVBFIGh0bWw+XG48aHRtbD5cbiAgPGhlYWQ+XG4gICAgPG1ldGEgY2hhcnNldD1cInV0Zi04XCI+XG4gICAgPG1ldGEgbmFtZT1cInZpZXdwb3J0XCIgY29udGVudD1cIndpZHRoPWRldmljZS13aWR0aFwiPlxuICAgIDx0aXRsZT5yZXBsLml0PC90aXRsZT5cbiAgICA8bGluayBocmVmPVwiaW5kZXguY3NzXCIgcmVsPVwic3R5bGVzaGVldFwiIHR5cGU9XCJ0ZXh0L2Nzc1wiIC8+XG4gIDwvaGVhZD5cbiAgPGJvZHk+XG4gICAgPHNjcmlwdCBzcmM9XCJpbmRleC5qc1wiPjwvc2NyaXB0PlxuICA8L2JvZHk+XG48L2h0bWw+IiwiaW5kZXgiOjB9LHsibmFtZSI6ImluZGV4LmpzIiwiY29udGVudCI6IiIsImluZGV4IjoxfSx7Im5hbWUiOiJpbmRleC5jc3MiLCJjb250ZW50IjoiIiwiaW5kZXgiOjJ9XSwiaWNvbiI6Ii9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy93ZWJfcHJvamVjdC5zdmciLCJrZXkiOiJodG1sIn0sInJ1c3QiOnsiZGlzcGxheU5hbWUiOiJSdXN0IiwidGFnbGluZSI6IkEgZmFzdCBhbmQgc2FmZSBzeXN0ZW1zIHByb2dyYW1taW5nIGxhbmd1YWdlLiIsImtleSI6InJ1c3QiLCJlbnRyeXBvaW50IjoibWFpbi5ycyIsImV4dCI6InJzIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOmZhbHNlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJydXN0YyAxLjkiLCJjYXRlZ29yeSI6IlByYWN0aWNhbCIsImljb24iOiJodHRwczovL3JlcGwuaXQvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvcnVzdC5zdmciLCJ0ZW1wbGF0ZSI6ImZuIG1haW4oKSB7XG4gIHByaW50bG4hKFwiSGVsbG8gV29ybGQhXCIpO1xufSIsIm1hdGNoaW5ncyI6W1sieyIsIn0iXSxbIigiLCIpIl0sWyJbIiwiXSJdXSwiZW5naW5lIjoiZ292YWwifSwic3dpZnQiOnsiZGlzcGxheU5hbWUiOiJTd2lmdCIsInRhZ2xpbmUiOiJBIG1vZGVybiBnZW5lcmFsLXB1cnBvc2UgcHJvZ3JhbW1pbmcgbGFuZ3VhZ2UgZnJvbSBBcHBsZS4iLCJrZXkiOiJzd2lmdCIsImVudHJ5cG9pbnQiOiJtYWluLnN3aWZ0IiwiZXh0Ijoic3dpZnQiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6ZmFsc2UsImhhc0xpYnJhcmllcyI6ZmFsc2UsImhhc0V2YWwiOmZhbHNlLCJoYXNMYW5ndWFnZVNlcnZlciI6ZmFsc2UsImhlYWRlciI6IlN3aWZ0IHZlcnNpb24gMy4wLjEgKFJlbGVhc2UpIiwiY2F0ZWdvcnkiOiJQcmFjdGljYWwiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL3N3aWZ0LnN2ZyIsInRlbXBsYXRlIjoiIiwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCJ9LCJweXRob25fdHVydGxlIjp7ImRpc3BsYXlOYW1lIjoiUHl0aG9uICh3aXRoIFR1cnRsZSkiLCJ0YWdsaW5lIjoiQSBkeW5hbWljIGxhbmd1YWdlIGVtcGhhc2l6aW5nIHJlYWRhYmlsaXR5LiIsImVuZ2luZSI6InB5dGhvbnR1cnRsZSIsImhlYWRlciI6IiIsImNhdGVnb3J5IjoiUHJhY3RpY2FsIiwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJleHQiOiJweSIsImljb24iOiIvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvcHl0aG9uX3R1cnRsZS5zdmciLCJrZXkiOiJweXRob25fdHVydGxlIn0sImplc3QiOnsiZGlzcGxheU5hbWUiOiJKZXN0IiwidGFnbGluZSI6IlBhaW5sZXNzIEphdmFTY3JpcHQgVGVzdGluZy4iLCJrZXkiOiJqZXN0IiwiZW50cnlwb2ludCI6ImNvbmZpZy5qc29uIiwiZXh0IjoianMiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6ZmFsc2UsImhhc0xpYnJhcmllcyI6ZmFsc2UsImhhc0V2YWwiOmZhbHNlLCJoYXNMYW5ndWFnZVNlcnZlciI6ZmFsc2UsImhlYWRlciI6Ikplc3QgdjIyLjEuMiBub2RlIHY3LjQuMCBsaW51eC9hbWQ2NCIsImNhdGVnb3J5IjoiVGVzdGluZyIsImljb24iOiJodHRwczovL3JlcGwuaXQvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvamVzdC5zdmciLCJwcm9qZWN0X3RlbXBsYXRlIjpbeyJuYW1lIjoiY29uZmlnLmpzb24iLCJjb250ZW50Ijoie1xuICBcInRlc3RSZWdleFwiOiBcIi4qLXRlc3RcXFxcLmpzJFwiLFxuICBcInRlc3RFbnZpcm9ubWVudFwiOiBcIm5vZGVcIlxufSJ9LHsibmFtZSI6ImFkZC10ZXN0LmpzIiwiY29udGVudCI6ImNvbnN0IGFkZCA9IHJlcXVpcmUoJy4vYWRkJyk7XG5kZXNjcmliZSgnYWRkJywgKCkgPT4ge1xuICBpdCgnc2hvdWxkIGFkZCB0d28gbnVtYmVycycsICgpID0+IHtcbiAgICBleHBlY3QoYWRkKDEsIDIpKS50b0JlKDMpO1xuICB9KTtcbn0pOyJ9LHsibmFtZSI6ImFkZC5qcyIsImNvbnRlbnQiOiJmdW5jdGlvbiBhZGQoYSwgYikge1xuICByZXR1cm4gYSArIGI7XG59XG5cbm1vZHVsZS5leHBvcnRzID0gYWRkOyJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCJ9LCJkamFuZ28iOnsiZGlzcGxheU5hbWUiOiJEamFuZ28iLCJ0YWdsaW5lIjoiUHl0aG9uIGZyYW1ld29yayB0aGF0IGVuY291cmFnZXMgcmFwaWQgZGV2ZWxvcG1lbnQuIiwia2V5IjoiZGphbmdvIiwiZW50cnlwb2ludCI6Im1haW4vdmlld3MucHkiLCJleHQiOiJweSIsImhhc0xpbnQiOnRydWUsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6dHJ1ZSwiaGFzRXZhbCI6ZmFsc2UsImhhc0xhbmd1YWdlU2VydmVyIjp0cnVlLCJoZWFkZXIiOiJQeXRob24gMy42LjEgKGRlZmF1bHQsIEp1biAyMSAyMDE3LCAxODo0ODozNSlcbltHQ0MgNC45LjJdIG9uIGxpbnV4IiwiY2F0ZWdvcnkiOiJGcmFtZXdvcmsiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2RqYW5nby5zdmciLCJwcm9qZWN0X3RlbXBsYXRlIjpbeyJuYW1lIjoic3RhdGljL2Nzcy9zdHlsZS5jc3MiLCJjb250ZW50IjoiYm9keSB7XG4gICAgYmFja2dyb3VuZDogI2NlY2VjZTtcbn1cbiJ9LHsibmFtZSI6InRlbXBsYXRlcy9iYXNlLmh0bWwiLCJjb250ZW50IjoieyUgbG9hZCBzdGF0aWNmaWxlcyAlfVxuICAgICAgICA8IURPQ1RZUEUgaHRtbD5cblxuPGh0bWwgbGFuZz1cImVuXCI+XG48aGVhZD5cbiAgICA8bWV0YSBjaGFyc2V0PVwiVVRGLThcIj5cbiAgICA8dGl0bGU+SGVsbG8gRGphbmdvPC90aXRsZT5cbiAgICA8bWV0YSBjaGFyc2V0PVwiVVRGLThcIi8+XG4gICAgPG1ldGEgbmFtZT1cInZpZXdwb3J0XCIgY29udGVudD1cIndpZHRoPWRldmljZS13aWR0aCwgaW5pdGlhbC1zY2FsZT0xXCIvPlxuICAgIDxsaW5rIHJlbD1cInN0eWxlc2hlZXRcIiBocmVmPVwieyUgc3RhdGljIFwiY3NzL3N0eWxlLmNzc1wiICV9XCI+XG48L2hlYWQ+XG48Ym9keT5cbiAgICB7JSBibG9jayBjb250ZW50ICV9eyUgZW5kYmxvY2sgY29udGVudCAlfVxuPC9ib2R5PlxuPC9odG1sPiJ9LHsibmFtZSI6InRlbXBsYXRlcy9tYWluL2luZGV4Lmh0bWwiLCJjb250ZW50IjoieyUgZXh0ZW5kcyBcImJhc2UuaHRtbFwiICV9XG5cbnslIGJsb2NrIGNvbnRlbnQgJX1cbiAgPGgxPkhlbGxvIFJlcGwuaXQ8L2gxPlxueyUgZW5kYmxvY2sgY29udGVudCAlfSJ9LHsibmFtZSI6Im1haW4vbWlncmF0aW9ucy9fX2luaXRfXy5weSIsImNvbnRlbnQiOiIifSx7Im5hbWUiOiJtYWluL3VybHMucHkiLCJjb250ZW50IjoiZnJvbSBkamFuZ28uY29uZi51cmxzIGltcG9ydCB1cmxcbmZyb20gZGphbmdvLmNvbnRyaWIgaW1wb3J0IGFkbWluXG5mcm9tIG1haW4gaW1wb3J0IHZpZXdzXG5cbnVybHBhdHRlcm5zID0gW1xuICB1cmwocideYWRtaW4vJywgYWRtaW4uc2l0ZS51cmxzKSxcbiAgdXJsKHInXiQnLCB2aWV3cy5ob21lLCBuYW1lPSdob21lJyksXG5dXG4ifSx7Im5hbWUiOiJtYWluL3NldHRpbmdzLnB5IiwiY29udGVudCI6ImltcG9ydCBvc1xuXG5CQVNFX0RJUiA9IG9zLnBhdGguZGlybmFtZShvcy5wYXRoLmRpcm5hbWUob3MucGF0aC5hYnNwYXRoKF9fZmlsZV9fKSkpXG5cbiMgU0VDVVJJVFkgV0FSTklORzoga2VlcCB0aGUgc2VjcmV0IGtleSB1c2VkIGluIHByb2R1Y3Rpb24gc2VjcmV0IVxuU0VDUkVUX0tFWSA9ICczM2VsKnZAQCl6aTU3cl9xXzFucmp0YV50cTZuJjhodyh2MXcoPSlhaXcjb2UxcDlkeidcblxuREVCVUcgPSBUcnVlXG5cbiMgU0VDVVJJVFkgV0FSTklORzogbWFrZSBzdXJlIHlvdSB1cGRhdGUgdGhpcyB0byB5b3VyIHdlYnNpdGVzIFVSTFxuQUxMT1dFRF9IT1NUUyA9IFsnKiddXG5YX0ZSQU1FX09QVElPTlMgPSAnQUxMT1cgQUxMJ1xuXG5JTlNUQUxMRURfQVBQUyA9IFtcbiAgICAnZGphbmdvLmNvbnRyaWIuYWRtaW4nLFxuICAgICdkamFuZ28uY29udHJpYi5hdXRoJyxcbiAgICAnZGphbmdvLmNvbnRyaWIuY29udGVudHR5cGVzJyxcbiAgICAnZGphbmdvLmNvbnRyaWIuc2Vzc2lvbnMnLFxuICAgICdkamFuZ28uY29udHJpYi5tZXNzYWdlcycsXG4gICAgJ2RqYW5nby5jb250cmliLnN0YXRpY2ZpbGVzJyxcblxuICAgICdtYWluJyxcbl1cblxuTUlERExFV0FSRSA9IFtcbiAgICAnZGphbmdvLm1pZGRsZXdhcmUuc2VjdXJpdHkuU2VjdXJpdHlNaWRkbGV3YXJlJyxcbiAgICAnZGphbmdvLmNvbnRyaWIuc2Vzc2lvbnMubWlkZGxld2FyZS5TZXNzaW9uTWlkZGxld2FyZScsXG4gICAgJ2RqYW5nby5taWRkbGV3YXJlLmNvbW1vbi5Db21tb25NaWRkbGV3YXJlJyxcbiAgICAnZGphbmdvLm1pZGRsZXdhcmUuY3NyZi5Dc3JmVmlld01pZGRsZXdhcmUnLFxuICAgICdkamFuZ28uY29udHJpYi5hdXRoLm1pZGRsZXdhcmUuQXV0aGVudGljYXRpb25NaWRkbGV3YXJlJyxcbiAgICAnZGphbmdvLmNvbnRyaWIubWVzc2FnZXMubWlkZGxld2FyZS5NZXNzYWdlTWlkZGxld2FyZScsXG4gICAgJ2RqYW5nby5taWRkbGV3YXJlLmNsaWNramFja2luZy5YRnJhbWVPcHRpb25zTWlkZGxld2FyZScsXG5dXG5cblJPT1RfVVJMQ09ORiA9ICdtYWluLnVybHMnXG5cblRFTVBMQVRFUyA9IFtcbiAgICB7XG4gICAgICAgICdCQUNLRU5EJzogJ2RqYW5nby50ZW1wbGF0ZS5iYWNrZW5kcy5kamFuZ28uRGphbmdvVGVtcGxhdGVzJyxcbiAgICAgICAgJ0RJUlMnOiBbb3MucGF0aC5qb2luKEJBU0VfRElSLCAndGVtcGxhdGVzJyldLFxuICAgICAgICAnQVBQX0RJUlMnOiBUcnVlLFxuICAgICAgICAnT1BUSU9OUyc6IHtcbiAgICAgICAgICAgICdjb250ZXh0X3Byb2Nlc3NvcnMnOiBbXG4gICAgICAgICAgICAgICAgJ2RqYW5nby50ZW1wbGF0ZS5jb250ZXh0X3Byb2Nlc3NvcnMuZGVidWcnLFxuICAgICAgICAgICAgICAgICdkamFuZ28udGVtcGxhdGUuY29udGV4dF9wcm9jZXNzb3JzLnJlcXVlc3QnLFxuICAgICAgICAgICAgICAgICdkamFuZ28uY29udHJpYi5hdXRoLmNvbnRleHRfcHJvY2Vzc29ycy5hdXRoJyxcbiAgICAgICAgICAgICAgICAnZGphbmdvLmNvbnRyaWIubWVzc2FnZXMuY29udGV4dF9wcm9jZXNzb3JzLm1lc3NhZ2VzJyxcbiAgICAgICAgICAgIF0sXG4gICAgICAgIH0sXG4gICAgfSxcbl1cblxuV1NHSV9BUFBMSUNBVElPTiA9ICdtYWluLndzZ2kuYXBwbGljYXRpb24nXG5cblxuREFUQUJBU0VTID0ge1xuICAgICdkZWZhdWx0Jzoge1xuICAgICAgICAnRU5HSU5FJzogJ2RqYW5nby5kYi5iYWNrZW5kcy5zcWxpdGUzJyxcbiAgICAgICAgJ05BTUUnOiBvcy5wYXRoLmpvaW4oQkFTRV9ESVIsICdkYi5zcWxpdGUzJyksXG4gICAgfVxufVxuXG5BVVRIX1BBU1NXT1JEX1ZBTElEQVRPUlMgPSBbXG4gICAgeyAnTkFNRSc6ICdkamFuZ28uY29udHJpYi5hdXRoLnBhc3N3b3JkX3ZhbGlkYXRpb24uVXNlckF0dHJpYnV0ZVNpbWlsYXJpdHlWYWxpZGF0b3InIH0sXG4gICAgeyAnTkFNRSc6ICdkamFuZ28uY29udHJpYi5hdXRoLnBhc3N3b3JkX3ZhbGlkYXRpb24uTWluaW11bUxlbmd0aFZhbGlkYXRvcicgfSxcbiAgICB7ICdOQU1FJzogJ2RqYW5nby5jb250cmliLmF1dGgucGFzc3dvcmRfdmFsaWRhdGlvbi5Db21tb25QYXNzd29yZFZhbGlkYXRvcicgfSxcbiAgICB7ICdOQU1FJzogJ2RqYW5nby5jb250cmliLmF1dGgucGFzc3dvcmRfdmFsaWRhdGlvbi5OdW1lcmljUGFzc3dvcmRWYWxpZGF0b3InIH0sXG5dXG5cbkxBTkdVQUdFX0NPREUgPSAnZW4tdXMnXG5USU1FX1pPTkUgPSAnVVRDJ1xuVVNFX0kxOE4gPSBUcnVlXG5VU0VfTDEwTiA9IFRydWVcblVTRV9UWiA9IFRydWVcblxuU1RBVElDX1VSTCA9ICcvc3RhdGljLydcblxuU1RBVElDRklMRVNfRElSUyA9IChcbiAgICBvcy5wYXRoLmpvaW4oQkFTRV9ESVIsICdzdGF0aWMnKSxcbikifSx7Im5hbWUiOiJtYWluL21vZGVscy5weSIsImNvbnRlbnQiOiJmcm9tIGRqYW5nby5kYiBpbXBvcnQgbW9kZWxzXG5cbiMgQ3JlYXRlIHlvdXIgbW9kZWxzIGhlcmUuIn0seyJuYW1lIjoibWFpbi92aWV3cy5weSIsImNvbnRlbnQiOiJmcm9tIGRqYW5nby5zaG9ydGN1dHMgaW1wb3J0IHJlbmRlclxuXG5cbiMgQ3JlYXRlIHlvdXIgdmlld3MgaGVyZS5cbmRlZiBob21lKHJlcXVlc3QpOlxuICAgIHJldHVybiByZW5kZXIocmVxdWVzdCwgJ21haW4vaW5kZXguaHRtbCcpIn1dLCJtYXRjaGluZ3MiOltbInsiLCJ9Il0sWyIoIiwiKSJdLFsiWyIsIl0iXV0sImVuZ2luZSI6ImdvdmFsIiwiY29uZmlnIjp7ImlzU2VydmVyIjp0cnVlfX0sImV4cHJlc3MiOnsiZGlzcGxheU5hbWUiOiJFeHByZXNzIiwidGFnbGluZSI6IkphdmFzY3JpcHQgZnJhbWV3b3JrIGRlc2lnbmVkIGZvciBidWlsZGluZyB3ZWIgYXBwbGljYXRpb25zIGFuZCBBUElzLiIsImtleSI6ImV4cHJlc3MiLCJlbnRyeXBvaW50IjoiaW5kZXguanMiLCJleHQiOiJqcyIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOnRydWUsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6dHJ1ZSwiaGFzRXZhbCI6ZmFsc2UsImhhc0xhbmd1YWdlU2VydmVyIjpmYWxzZSwiaGVhZGVyIjoibm9kZSB2OS43LjEgbGludXgvYW1kNjQiLCJjYXRlZ29yeSI6IkZyYW1ld29yayIsImljb24iOiJodHRwczovL3JlcGwuaXQvcHVibGljL2ltYWdlcy9sYW5ndWFnZXMvZXhwcmVzcy5zdmciLCJwcm9qZWN0X3RlbXBsYXRlIjpbeyJuYW1lIjoiZGIuanMiLCJjb250ZW50IjoiY29uc3Qgc3FsaXRlMyA9IHJlcXVpcmUoJ3NxbGl0ZTMnKS52ZXJib3NlKCk7XG5jb25zdCBkYiA9IG5ldyBzcWxpdGUzLkRhdGFiYXNlKCcuL2RiLnNxbGl0ZTMnKTtcblxuZGIucnVuKCdDUkVBVEUgVEFCTEUgSUYgTk9UIEVYSVNUUyBib29rcyAobmFtZSBURVhULCBhdXRob3IgVEVYVCknKTtcblxubW9kdWxlLmV4cG9ydHMgPSBkYjsifSx7Im5hbWUiOiJwYWNrYWdlLmpzb24iLCJjb250ZW50Ijoie1xuICBcIm5hbWVcIjogXCJhcHBcIixcbiAgXCJ2ZXJzaW9uXCI6IFwiMC4wLjFcIixcbiAgXCJkZXNjcmlwdGlvblwiOiBcIlwiLFxuICBcIm1haW5cIjogXCJpbmRleC5qc1wiLFxuICBcInNjcmlwdHNcIjoge1xuICB9LFxuICBcImF1dGhvclwiOiBcIlwiLFxuICBcImxpY2Vuc2VcIjogXCJNSVRcIixcbiAgXCJkZXBlbmRlbmNpZXNcIjoge1xuICAgIFwiZXhwcmVzc1wiOiBcImxhdGVzdFwiLFxuICAgIFwiYm9keS1wYXJzZXJcIjogXCJsYXRlc3RcIixcbiAgICBcInNxbGl0ZTNcIjogXCJsYXRlc3RcIlxuICB9XG59In0seyJuYW1lIjoiaW5kZXguanMiLCJjb250ZW50IjoiY29uc3QgZXhwcmVzcyA9IHJlcXVpcmUoJ2V4cHJlc3MnKTtcbmNvbnN0IGJvZHlQYXJzZXIgPSByZXF1aXJlKCdib2R5LXBhcnNlcicpO1xuY29uc3QgZGIgPSByZXF1aXJlKCcuL2RiJyk7XG5cbmNvbnN0IGFwcCA9IGV4cHJlc3MoKTtcblxuYXBwLnVzZShib2R5UGFyc2VyLmpzb24oKSk7XG5hcHAudXNlKGJvZHlQYXJzZXIudXJsZW5jb2RlZCh7IGV4dGVuZGVkOiB0cnVlIH0pKTtcblxuYXBwLnVzZShleHByZXNzLnN0YXRpYygncHVibGljJykpO1xuXG5hcHAuZ2V0KCcvJywgKHJlcSwgcmVzKSA9PiB7XG5cdHJlcy5zZW5kKCdIZWxsbyBFeHByZXNzIGFwcCcpO1xufSk7XG5cbmFwcC5saXN0ZW4oMzAwMCwgKCkgPT4gY29uc29sZS5sb2coJ3NlcnZlciBzdGFydGVkJykpOyJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCIsImNvbmZpZyI6eyJpc1NlcnZlciI6dHJ1ZX19LCJzaW5hdHJhIjp7ImRpc3BsYXlOYW1lIjoiU2luYXRyYSIsInRhZ2xpbmUiOiJEU0wgZm9yIHF1aWNrbHkgY3JlYXRpbmcgd2ViIGFwcGxpY2F0aW9ucyBpbiBSdWJ5IHdpdGggbWluaW1hbCBlZmZvcnQiLCJrZXkiOiJzaW5hdHJhIiwiZW50cnlwb2ludCI6Im1haW4ucmIiLCJleHQiOiJyYiIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOnRydWUsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6dHJ1ZSwiaGFzRXZhbCI6ZmFsc2UsImhhc0xhbmd1YWdlU2VydmVyIjpmYWxzZSwiaGVhZGVyIjoicnVieSAyLjUuMHAwICgyMDE3LTEyLTI1IHJldmlzaW9uIDYxNDY4KSBbeDg2XzY0LWxpbnV4XSIsImNhdGVnb3J5IjoiRnJhbWV3b3JrIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9zaW5hdHJhLnBuZyIsInByb2plY3RfdGVtcGxhdGUiOlt7Im5hbWUiOiJ2aWV3cy9pbmRleC5lcmIiLCJjb250ZW50IjoiXG48IURPQ1RZUEUgaHRtbD5cbjxodG1sIGxhbmc9XCJlblwiPlxuPGhlYWQ+XG4gIDx0aXRsZT5IZWxsbyE8L3RpdGxlPlxuPC9oZWFkPlxuPGJvZHk+XG4gIEhlbGxvIGZyb20gPCU9IGhvc3QgJT4uXG48L2JvZHk+XG48L2h0bWw+In0seyJuYW1lIjoiR2VtZmlsZSIsImNvbnRlbnQiOiJzb3VyY2UgJ2h0dHA6Ly9ydWJ5Z2Vtcy5vcmcnXG5cbmdlbSAnc2luYXRyYScifSx7Im5hbWUiOiJtYWluLnJiIiwiY29udGVudCI6InJlcXVpcmUgJ3NpbmF0cmEnXG5cbnNldCA6cHJvdGVjdGlvbiwgOmV4Y2VwdCA9PiA6ZnJhbWVfb3B0aW9uc1xuc2V0IDpiaW5kLCAnMC4wLjAuMCdcblxuZ2V0ICcvJyBkb1xuICBlcmIgOmluZGV4LCA6bG9jYWxzID0+IHsgaG9zdDogcmVxdWVzdC5ob3N0IH1cbmVuZCJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCIsImNvbmZpZyI6eyJpc1NlcnZlciI6dHJ1ZX19LCJyYWlscyI6eyJkaXNwbGF5TmFtZSI6IlJ1Ynkgb24gUmFpbHMiLCJ0YWdsaW5lIjoiQSB3ZWItYXBwbGljYXRpb24gZnJhbWV3b3JrIHRoYXQgaW5jbHVkZXMgZXZlcnl0aGluZyBuZWVkZWQgdG8gY3JlYXRlIHdlYiBhcHBsaWNhdGlvbnMiLCJrZXkiOiJyYWlscyIsImVudHJ5cG9pbnQiOiJjb25maWcvcm91dGVzLnJiIiwiZXh0IjoicmIiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6dHJ1ZSwiaGFzTGlicmFyaWVzIjp0cnVlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJydWJ5IDIuNS4wcDAgKDIwMTctMTItMjUgcmV2aXNpb24gNjE0NjgpIFt4ODZfNjQtbGludXhdIiwiY2F0ZWdvcnkiOiJGcmFtZXdvcmsiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL3JhaWxzLnN2ZyIsInByb2plY3RfdGVtcGxhdGUiOlt7Im5hbWUiOiJHZW1maWxlIiwiY29udGVudCI6InNvdXJjZSAnaHR0cHM6Ly9ydWJ5Z2Vtcy5vcmcnXG5naXRfc291cmNlKDpnaXRodWIpIHsgfHJlcG98IFwiaHR0cHM6Ly9naXRodWIuY29tLyN7cmVwb30uZ2l0XCIgfVxuXG5ydWJ5ICcyLjUuMSdcblxuIyBCdW5kbGUgZWRnZSBSYWlscyBpbnN0ZWFkOiBnZW0gJ3JhaWxzJywgZ2l0aHViOiAncmFpbHMvcmFpbHMnXG5nZW0gJ3JhaWxzJywgJ34+IDUuMi4wJ1xuIyBVc2Ugc3FsaXRlMyBhcyB0aGUgZGF0YWJhc2UgZm9yIEFjdGl2ZSBSZWNvcmRcbmdlbSAnc3FsaXRlMydcbiMgQnVpbGQgSlNPTiBBUElzIHdpdGggZWFzZS4gUmVhZCBtb3JlOiBodHRwczovL2dpdGh1Yi5jb20vcmFpbHMvamJ1aWxkZXJcbmdlbSAnamJ1aWxkZXInLCAnfj4gMi41J1xuIyBVc2UgQWN0aXZlTW9kZWwgaGFzX3NlY3VyZV9wYXNzd29yZFxuIyBnZW0gJ2JjcnlwdCcsICd+PiAzLjEuNydcblxuIyBVc2UgQWN0aXZlU3RvcmFnZSB2YXJpYW50XG4jIGdlbSAnbWluaV9tYWdpY2snLCAnfj4gNC44J1xuXG4jIFVzZSBDYXBpc3RyYW5vIGZvciBkZXBsb3ltZW50XG4jIGdlbSAnY2FwaXN0cmFuby1yYWlscycsIGdyb3VwOiA6ZGV2ZWxvcG1lbnRcblxuIyBSZWR1Y2VzIGJvb3QgdGltZXMgdGhyb3VnaCBjYWNoaW5nOyByZXF1aXJlZCBpbiBjb25maWcvYm9vdC5yYlxuZ2VtICdib290c25hcCcsICc+PSAxLjEuMCcsIHJlcXVpcmU6IGZhbHNlXG5cbmdyb3VwIDpkZXZlbG9wbWVudCwgOnRlc3QgZG9cbiAgIyBDYWxsICdieWVidWcnIGFueXdoZXJlIGluIHRoZSBjb2RlIHRvIHN0b3AgZXhlY3V0aW9uIGFuZCBnZXQgYSBkZWJ1Z2dlciBjb25zb2xlXG4gIGdlbSAnYnllYnVnJywgcGxhdGZvcm1zOiBbOm1yaSwgOm1pbmd3LCA6eDY0X21pbmd3XVxuZW5kXG5cbmdyb3VwIDpkZXZlbG9wbWVudCBkb1xuICAjIEFjY2VzcyBhbiBpbnRlcmFjdGl2ZSBjb25zb2xlIG9uIGV4Y2VwdGlvbiBwYWdlcyBvciBieSBjYWxsaW5nICdjb25zb2xlJyBhbnl3aGVyZSBpbiB0aGUgY29kZS5cbiAgZ2VtICd3ZWItY29uc29sZScsICc+PSAzLjMuMCdcbmVuZFxuXG5cbiMgV2luZG93cyBkb2VzIG5vdCBpbmNsdWRlIHpvbmVpbmZvIGZpbGVzLCBzbyBidW5kbGUgdGhlIHR6aW5mby1kYXRhIGdlbVxuZ2VtICd0emluZm8tZGF0YScsIHBsYXRmb3JtczogWzptaW5ndywgOm1zd2luLCA6eDY0X21pbmd3LCA6anJ1YnldXG4ifSx7Im5hbWUiOiJjb25maWcvYm9vdC5yYiIsImNvbnRlbnQiOiJFTlZbJ0JVTkRMRV9HRU1GSUxFJ10gfHw9IEZpbGUuZXhwYW5kX3BhdGgoJy4uL0dlbWZpbGUnLCBfX2Rpcl9fKVxuXG5yZXF1aXJlICdidW5kbGVyL3NldHVwJyAjIFNldCB1cCBnZW1zIGxpc3RlZCBpbiB0aGUgR2VtZmlsZS5cbnJlcXVpcmUgJ2Jvb3RzbmFwL3NldHVwJyAjIFNwZWVkIHVwIGJvb3QgdGltZSBieSBjYWNoaW5nIGV4cGVuc2l2ZSBvcGVyYXRpb25zLlxuIn0seyJuYW1lIjoiY29uZmlnL3JvdXRlcy5yYiIsImNvbnRlbnQiOiJSYWlscy5hcHBsaWNhdGlvbi5yb3V0ZXMuZHJhdyBkb1xuICAjIEZvciBkZXRhaWxzIG9uIHRoZSBEU0wgYXZhaWxhYmxlIHdpdGhpbiB0aGlzIGZpbGUsIHNlZSBodHRwOi8vZ3VpZGVzLnJ1YnlvbnJhaWxzLm9yZy9yb3V0aW5nLmh0bWxcbmVuZFxuIn0seyJuYW1lIjoiY29uZmlnL2RhdGFiYXNlLnltbCIsImNvbnRlbnQiOiIjIFNRTGl0ZSB2ZXJzaW9uIDMueFxuIyAgIGdlbSBpbnN0YWxsIHNxbGl0ZTNcbiNcbiMgICBFbnN1cmUgdGhlIFNRTGl0ZSAzIGdlbSBpcyBkZWZpbmVkIGluIHlvdXIgR2VtZmlsZVxuIyAgIGdlbSAnc3FsaXRlMydcbiNcbmRlZmF1bHQ6ICZkZWZhdWx0XG4gIGFkYXB0ZXI6IHNxbGl0ZTNcbiAgcG9vbDogPCU9IEVOVi5mZXRjaChcIlJBSUxTX01BWF9USFJFQURTXCIpIHsgNSB9ICU+XG4gIHRpbWVvdXQ6IDUwMDBcblxuZGV2ZWxvcG1lbnQ6XG4gIDw8OiAqZGVmYXVsdFxuICBkYXRhYmFzZTogZGIvZGV2ZWxvcG1lbnQuc3FsaXRlM1xuXG4jIFdhcm5pbmc6IFRoZSBkYXRhYmFzZSBkZWZpbmVkIGFzIFwidGVzdFwiIHdpbGwgYmUgZXJhc2VkIGFuZFxuIyByZS1nZW5lcmF0ZWQgZnJvbSB5b3VyIGRldmVsb3BtZW50IGRhdGFiYXNlIHdoZW4geW91IHJ1biBcInJha2VcIi5cbiMgRG8gbm90IHNldCB0aGlzIGRiIHRvIHRoZSBzYW1lIGFzIGRldmVsb3BtZW50IG9yIHByb2R1Y3Rpb24uXG50ZXN0OlxuICA8PDogKmRlZmF1bHRcbiAgZGF0YWJhc2U6IGRiL3Rlc3Quc3FsaXRlM1xuXG5wcm9kdWN0aW9uOlxuICA8PDogKmRlZmF1bHRcbiAgZGF0YWJhc2U6IGRiL3Byb2R1Y3Rpb24uc3FsaXRlM1xuIn0seyJuYW1lIjoiY29uZmlnL2FwcGxpY2F0aW9uLnJiIiwiY29udGVudCI6InJlcXVpcmVfcmVsYXRpdmUgXCJib290XCJcblxucmVxdWlyZSBcInJhaWxzXCJcbiMgUGljayB0aGUgZnJhbWV3b3JrcyB5b3Ugd2FudDpcbnJlcXVpcmUgXCJhY3RpdmVfbW9kZWwvcmFpbHRpZVwiXG5yZXF1aXJlIFwiYWN0aXZlX2pvYi9yYWlsdGllXCJcbnJlcXVpcmUgXCJhY3RpdmVfcmVjb3JkL3JhaWx0aWVcIlxucmVxdWlyZSBcImFjdGl2ZV9zdG9yYWdlL2VuZ2luZVwiXG5yZXF1aXJlIFwiYWN0aW9uX2NvbnRyb2xsZXIvcmFpbHRpZVwiXG4jIHJlcXVpcmUgXCJhY3Rpb25fbWFpbGVyL3JhaWx0aWVcIlxucmVxdWlyZSBcImFjdGlvbl92aWV3L3JhaWx0aWVcIlxuIyByZXF1aXJlIFwiYWN0aW9uX2NhYmxlL2VuZ2luZVwiXG4jIHJlcXVpcmUgXCJzcHJvY2tldHMvcmFpbHRpZVwiXG4jIHJlcXVpcmUgXCJyYWlscy90ZXN0X3VuaXQvcmFpbHRpZVwiXG5cbiMgUmVxdWlyZSB0aGUgZ2VtcyBsaXN0ZWQgaW4gR2VtZmlsZSwgaW5jbHVkaW5nIGFueSBnZW1zXG4jIHlvdSd2ZSBsaW1pdGVkIHRvIDp0ZXN0LCA6ZGV2ZWxvcG1lbnQsIG9yIDpwcm9kdWN0aW9uLlxuQnVuZGxlci5yZXF1aXJlKCpSYWlscy5ncm91cHMpXG5cbm1vZHVsZSBBcHBcbiAgY2xhc3MgQXBwbGljYXRpb24gPCBSYWlsczo6QXBwbGljYXRpb25cbiAgICBjb25maWcuYWN0aW9uX2Rpc3BhdGNoLmRlZmF1bHRfaGVhZGVycyA9IHtcbiAgICAgIFwiWC1GcmFtZS1PcHRpb25zXCIgPT4gXCJBTExPV0FMTFwiLFxuICAgIH1cblxuICAgICMgSW5pdGlhbGl6ZSBjb25maWd1cmF0aW9uIGRlZmF1bHRzIGZvciBvcmlnaW5hbGx5IGdlbmVyYXRlZCBSYWlscyB2ZXJzaW9uLlxuICAgIGNvbmZpZy5sb2FkX2RlZmF1bHRzIDUuMlxuXG4gICAgIyBTZXR0aW5ncyBpbiBjb25maWcvZW52aXJvbm1lbnRzLyogdGFrZSBwcmVjZWRlbmNlIG92ZXIgdGhvc2Ugc3BlY2lmaWVkIGhlcmUuXG4gICAgIyBBcHBsaWNhdGlvbiBjb25maWd1cmF0aW9uIGNhbiBnbyBpbnRvIGZpbGVzIGluIGNvbmZpZy9pbml0aWFsaXplcnNcbiAgICAjIC0tIGFsbCAucmIgZmlsZXMgaW4gdGhhdCBkaXJlY3RvcnkgYXJlIGF1dG9tYXRpY2FsbHkgbG9hZGVkIGFmdGVyIGxvYWRpbmdcbiAgICAjIHRoZSBmcmFtZXdvcmsgYW5kIGFueSBnZW1zIGluIHlvdXIgYXBwbGljYXRpb24uXG5cbiAgICAjIERvbid0IGdlbmVyYXRlIHN5c3RlbSB0ZXN0IGZpbGVzLlxuICAgIGNvbmZpZy5nZW5lcmF0b3JzLnN5c3RlbV90ZXN0cyA9IG5pbFxuICBlbmRcbmVuZFxuIn0seyJuYW1lIjoicHVibGljLzQwNC5odG1sIiwiY29udGVudCI6IjwhRE9DVFlQRSBodG1sPlxuPGh0bWw+XG48aGVhZD5cbiAgPHRpdGxlPlRoZSBwYWdlIHlvdSB3ZXJlIGxvb2tpbmcgZm9yIGRvZXNuJ3QgZXhpc3QgKDQwNCk8L3RpdGxlPlxuICA8bWV0YSBuYW1lPVwidmlld3BvcnRcIiBjb250ZW50PVwid2lkdGg9ZGV2aWNlLXdpZHRoLGluaXRpYWwtc2NhbGU9MVwiPlxuICA8c3R5bGU+XG4gIC5yYWlscy1kZWZhdWx0LWVycm9yLXBhZ2Uge1xuICAgIGJhY2tncm91bmQtY29sb3I6ICNFRkVGRUY7XG4gICAgY29sb3I6ICMyRTJGMzA7XG4gICAgdGV4dC1hbGlnbjogY2VudGVyO1xuICAgIGZvbnQtZmFtaWx5OiBhcmlhbCwgc2Fucy1zZXJpZjtcbiAgICBtYXJnaW46IDA7XG4gIH1cblxuICAucmFpbHMtZGVmYXVsdC1lcnJvci1wYWdlIGRpdi5kaWFsb2cge1xuICAgIHdpZHRoOiA5NSU7XG4gICAgbWF4LXdpZHRoOiAzM2VtO1xuICAgIG1hcmdpbjogNGVtIGF1dG8gMDtcbiAgfVxuXG4gIC5yYWlscy1kZWZhdWx0LWVycm9yLXBhZ2UgZGl2LmRpYWxvZyA+IGRpdiB7XG4gICAgYm9yZGVyOiAxcHggc29saWQgI0NDQztcbiAgICBib3JkZXItcmlnaHQtY29sb3I6ICM5OTk7XG4gICAgYm9yZGVyLWxlZnQtY29sb3I6ICM5OTk7XG4gICAgYm9yZGVyLWJvdHRvbS1jb2xvcjogI0JCQjtcbiAgICBib3JkZXItdG9wOiAjQjAwMTAwIHNvbGlkIDRweDtcbiAgICBib3JkZXItdG9wLWxlZnQtcmFkaXVzOiA5cHg7XG4gICAgYm9yZGVyLXRvcC1yaWdodC1yYWRpdXM6IDlweDtcbiAgICBiYWNrZ3JvdW5kLWNvbG9yOiB3aGl0ZTtcbiAgICBwYWRkaW5nOiA3cHggMTIlIDA7XG4gICAgYm94LXNoYWRvdzogMCAzcHggOHB4IHJnYmEoNTAsIDUwLCA1MCwgMC4xNyk7XG4gIH1cblxuICAucmFpbHMtZGVmYXVsdC1lcnJvci1wYWdlIGgxIHtcbiAgICBmb250LXNpemU6IDEwMCU7XG4gICAgY29sb3I6ICM3MzBFMTU7XG4gICAgbGluZS1oZWlnaHQ6IDEuNWVtO1xuICB9XG5cbiAgLnJhaWxzLWRlZmF1bHQtZXJyb3ItcGFnZSBkaXYuZGlhbG9nID4gcCB7XG4gICAgbWFyZ2luOiAwIDAgMWVtO1xuICAgIHBhZGRpbmc6IDFlbTtcbiAgICBiYWNrZ3JvdW5kLWNvbG9yOiAjRjdGN0Y3O1xuICAgIGJvcmRlcjogMXB4IHNvbGlkICNDQ0M7XG4gICAgYm9yZGVyLXJpZ2h0LWNvbG9yOiAjOTk5O1xuICAgIGJvcmRlci1sZWZ0LWNvbG9yOiAjOTk5O1xuICAgIGJvcmRlci1ib3R0b20tY29sb3I6ICM5OTk7XG4gICAgYm9yZGVyLWJvdHRvbS1sZWZ0LXJhZGl1czogNHB4O1xuICAgIGJvcmRlci1ib3R0b20tcmlnaHQtcmFkaXVzOiA0cHg7XG4gICAgYm9yZGVyLXRvcC1jb2xvcjogI0RBREFEQTtcbiAgICBjb2xvcjogIzY2NjtcbiAgICBib3gtc2hhZG93OiAwIDNweCA4cHggcmdiYSg1MCwgNTAsIDUwLCAwLjE3KTtcbiAgfVxuICA8L3N0eWxlPlxuPC9oZWFkPlxuXG48Ym9keSBjbGFzcz1cInJhaWxzLWRlZmF1bHQtZXJyb3ItcGFnZVwiPlxuICA8IS0tIFRoaXMgZmlsZSBsaXZlcyBpbiBwdWJsaWMvNDA0Lmh0bWwgLS0+XG4gIDxkaXYgY2xhc3M9XCJkaWFsb2dcIj5cbiAgICA8ZGl2PlxuICAgICAgPGgxPlRoZSBwYWdlIHlvdSB3ZXJlIGxvb2tpbmcgZm9yIGRvZXNuJ3QgZXhpc3QuPC9oMT5cbiAgICAgIDxwPllvdSBtYXkgaGF2ZSBtaXN0eXBlZCB0aGUgYWRkcmVzcyBvciB0aGUgcGFnZSBtYXkgaGF2ZSBtb3ZlZC48L3A+XG4gICAgPC9kaXY+XG4gICAgPHA+SWYgeW91IGFyZSB0aGUgYXBwbGljYXRpb24gb3duZXIgY2hlY2sgdGhlIGxvZ3MgZm9yIG1vcmUgaW5mb3JtYXRpb24uPC9wPlxuICA8L2Rpdj5cbjwvYm9keT5cbjwvaHRtbD5cbiJ9LHsibmFtZSI6InB1YmxpYy81MDAuaHRtbCIsImNvbnRlbnQiOiI8IURPQ1RZUEUgaHRtbD5cbjxodG1sPlxuPGhlYWQ+XG4gIDx0aXRsZT5XZSdyZSBzb3JyeSwgYnV0IHNvbWV0aGluZyB3ZW50IHdyb25nICg1MDApPC90aXRsZT5cbiAgPG1ldGEgbmFtZT1cInZpZXdwb3J0XCIgY29udGVudD1cIndpZHRoPWRldmljZS13aWR0aCxpbml0aWFsLXNjYWxlPTFcIj5cbiAgPHN0eWxlPlxuICAucmFpbHMtZGVmYXVsdC1lcnJvci1wYWdlIHtcbiAgICBiYWNrZ3JvdW5kLWNvbG9yOiAjRUZFRkVGO1xuICAgIGNvbG9yOiAjMkUyRjMwO1xuICAgIHRleHQtYWxpZ246IGNlbnRlcjtcbiAgICBmb250LWZhbWlseTogYXJpYWwsIHNhbnMtc2VyaWY7XG4gICAgbWFyZ2luOiAwO1xuICB9XG5cbiAgLnJhaWxzLWRlZmF1bHQtZXJyb3ItcGFnZSBkaXYuZGlhbG9nIHtcbiAgICB3aWR0aDogOTUlO1xuICAgIG1heC13aWR0aDogMzNlbTtcbiAgICBtYXJnaW46IDRlbSBhdXRvIDA7XG4gIH1cblxuICAucmFpbHMtZGVmYXVsdC1lcnJvci1wYWdlIGRpdi5kaWFsb2cgPiBkaXYge1xuICAgIGJvcmRlcjogMXB4IHNvbGlkICNDQ0M7XG4gICAgYm9yZGVyLXJpZ2h0LWNvbG9yOiAjOTk5O1xuICAgIGJvcmRlci1sZWZ0LWNvbG9yOiAjOTk5O1xuICAgIGJvcmRlci1ib3R0b20tY29sb3I6ICNCQkI7XG4gICAgYm9yZGVyLXRvcDogI0IwMDEwMCBzb2xpZCA0cHg7XG4gICAgYm9yZGVyLXRvcC1sZWZ0LXJhZGl1czogOXB4O1xuICAgIGJvcmRlci10b3AtcmlnaHQtcmFkaXVzOiA5cHg7XG4gICAgYmFja2dyb3VuZC1jb2xvcjogd2hpdGU7XG4gICAgcGFkZGluZzogN3B4IDEyJSAwO1xuICAgIGJveC1zaGFkb3c6IDAgM3B4IDhweCByZ2JhKDUwLCA1MCwgNTAsIDAuMTcpO1xuICB9XG5cbiAgLnJhaWxzLWRlZmF1bHQtZXJyb3ItcGFnZSBoMSB7XG4gICAgZm9udC1zaXplOiAxMDAlO1xuICAgIGNvbG9yOiAjNzMwRTE1O1xuICAgIGxpbmUtaGVpZ2h0OiAxLjVlbTtcbiAgfVxuXG4gIC5yYWlscy1kZWZhdWx0LWVycm9yLXBhZ2UgZGl2LmRpYWxvZyA+IHAge1xuICAgIG1hcmdpbjogMCAwIDFlbTtcbiAgICBwYWRkaW5nOiAxZW07XG4gICAgYmFja2dyb3VuZC1jb2xvcjogI0Y3RjdGNztcbiAgICBib3JkZXI6IDFweCBzb2xpZCAjQ0NDO1xuICAgIGJvcmRlci1yaWdodC1jb2xvcjogIzk5OTtcbiAgICBib3JkZXItbGVmdC1jb2xvcjogIzk5OTtcbiAgICBib3JkZXItYm90dG9tLWNvbG9yOiAjOTk5O1xuICAgIGJvcmRlci1ib3R0b20tbGVmdC1yYWRpdXM6IDRweDtcbiAgICBib3JkZXItYm90dG9tLXJpZ2h0LXJhZGl1czogNHB4O1xuICAgIGJvcmRlci10b3AtY29sb3I6ICNEQURBREE7XG4gICAgY29sb3I6ICM2NjY7XG4gICAgYm94LXNoYWRvdzogMCAzcHggOHB4IHJnYmEoNTAsIDUwLCA1MCwgMC4xNyk7XG4gIH1cbiAgPC9zdHlsZT5cbjwvaGVhZD5cblxuPGJvZHkgY2xhc3M9XCJyYWlscy1kZWZhdWx0LWVycm9yLXBhZ2VcIj5cbiAgPCEtLSBUaGlzIGZpbGUgbGl2ZXMgaW4gcHVibGljLzUwMC5odG1sIC0tPlxuICA8ZGl2IGNsYXNzPVwiZGlhbG9nXCI+XG4gICAgPGRpdj5cbiAgICAgIDxoMT5XZSdyZSBzb3JyeSwgYnV0IHNvbWV0aGluZyB3ZW50IHdyb25nLjwvaDE+XG4gICAgPC9kaXY+XG4gICAgPHA+SWYgeW91IGFyZSB0aGUgYXBwbGljYXRpb24gb3duZXIgY2hlY2sgdGhlIGxvZ3MgZm9yIG1vcmUgaW5mb3JtYXRpb24uPC9wPlxuICA8L2Rpdj5cbjwvYm9keT5cbjwvaHRtbD5cbiJ9LHsibmFtZSI6ImFwcC92aWV3cy9sYXlvdXRzL2FwcGxpY2F0aW9uLmh0bWwuZXJiIiwiY29udGVudCI6IjwhRE9DVFlQRSBodG1sPlxuPGh0bWw+XG4gIDxoZWFkPlxuICAgIDx0aXRsZT5BcHA8L3RpdGxlPlxuICAgIDwlPSBjc3JmX21ldGFfdGFncyAlPlxuICAgIDwlPSBjc3BfbWV0YV90YWcgJT5cblxuICAgIDwlPSBzdHlsZXNoZWV0X2xpbmtfdGFnICAgICdhcHBsaWNhdGlvbicsIG1lZGlhOiAnYWxsJyAlPlxuICAgIDwlPSBqYXZhc2NyaXB0X2luY2x1ZGVfdGFnICdhcHBsaWNhdGlvbicgJT5cbiAgPC9oZWFkPlxuXG4gIDxib2R5PlxuICAgIDwlPSB5aWVsZCAlPlxuICA8L2JvZHk+XG48L2h0bWw+XG4ifSx7Im5hbWUiOiJhcHAvbW9kZWxzL2FwcGxpY2F0aW9uX3JlY29yZC5yYiIsImNvbnRlbnQiOiJjbGFzcyBBcHBsaWNhdGlvblJlY29yZCA8IEFjdGl2ZVJlY29yZDo6QmFzZVxuICBzZWxmLmFic3RyYWN0X2NsYXNzID0gdHJ1ZVxuZW5kXG4ifSx7Im5hbWUiOiJhcHAvYXNzZXRzL2NvbmZpZy9tYW5pZmVzdC5qcyIsImNvbnRlbnQiOiIvLz0gbGlua190cmVlIC4uL2ltYWdlc1xuLy89IGxpbmtfZGlyZWN0b3J5IC4uL2phdmFzY3JpcHRzIC5qc1xuLy89IGxpbmtfZGlyZWN0b3J5IC4uL3N0eWxlc2hlZXRzIC5jc3NcbiJ9LHsibmFtZSI6ImFwcC9hc3NldHMvamF2YXNjcmlwdHMvYXBwbGljYXRpb24uanMiLCJjb250ZW50IjoiLy8gVGhpcyBpcyBhIG1hbmlmZXN0IGZpbGUgdGhhdCdsbCBiZSBjb21waWxlZCBpbnRvIGFwcGxpY2F0aW9uLmpzLCB3aGljaCB3aWxsIGluY2x1ZGUgYWxsIHRoZSBmaWxlc1xuLy8gbGlzdGVkIGJlbG93LlxuLy9cbi8vIEFueSBKYXZhU2NyaXB0L0NvZmZlZSBmaWxlIHdpdGhpbiB0aGlzIGRpcmVjdG9yeSwgbGliL2Fzc2V0cy9qYXZhc2NyaXB0cywgb3IgYW55IHBsdWdpbidzXG4vLyB2ZW5kb3IvYXNzZXRzL2phdmFzY3JpcHRzIGRpcmVjdG9yeSBjYW4gYmUgcmVmZXJlbmNlZCBoZXJlIHVzaW5nIGEgcmVsYXRpdmUgcGF0aC5cbi8vXG4vLyBJdCdzIG5vdCBhZHZpc2FibGUgdG8gYWRkIGNvZGUgZGlyZWN0bHkgaGVyZSwgYnV0IGlmIHlvdSBkbywgaXQnbGwgYXBwZWFyIGF0IHRoZSBib3R0b20gb2YgdGhlXG4vLyBjb21waWxlZCBmaWxlLiBKYXZhU2NyaXB0IGNvZGUgaW4gdGhpcyBmaWxlIHNob3VsZCBiZSBhZGRlZCBhZnRlciB0aGUgbGFzdCByZXF1aXJlXyogc3RhdGVtZW50LlxuLy9cbi8vIFJlYWQgU3Byb2NrZXRzIFJFQURNRSAoaHR0cHM6Ly9naXRodWIuY29tL3JhaWxzL3Nwcm9ja2V0cyNzcHJvY2tldHMtZGlyZWN0aXZlcykgZm9yIGRldGFpbHNcbi8vIGFib3V0IHN1cHBvcnRlZCBkaXJlY3RpdmVzLlxuLy9cbi8vPSByZXF1aXJlIHJhaWxzLXVqc1xuLy89IHJlcXVpcmUgYWN0aXZlc3RvcmFnZVxuLy89IHJlcXVpcmVfdHJlZSAuXG4ifSx7Im5hbWUiOiJhcHAvYXNzZXRzL3N0eWxlc2hlZXRzL2FwcGxpY2F0aW9uLmNzcyIsImNvbnRlbnQiOiIvKlxuICogVGhpcyBpcyBhIG1hbmlmZXN0IGZpbGUgdGhhdCdsbCBiZSBjb21waWxlZCBpbnRvIGFwcGxpY2F0aW9uLmNzcywgd2hpY2ggd2lsbCBpbmNsdWRlIGFsbCB0aGUgZmlsZXNcbiAqIGxpc3RlZCBiZWxvdy5cbiAqXG4gKiBBbnkgQ1NTIGFuZCBTQ1NTIGZpbGUgd2l0aGluIHRoaXMgZGlyZWN0b3J5LCBsaWIvYXNzZXRzL3N0eWxlc2hlZXRzLCBvciBhbnkgcGx1Z2luJ3NcbiAqIHZlbmRvci9hc3NldHMvc3R5bGVzaGVldHMgZGlyZWN0b3J5IGNhbiBiZSByZWZlcmVuY2VkIGhlcmUgdXNpbmcgYSByZWxhdGl2ZSBwYXRoLlxuICpcbiAqIFlvdSdyZSBmcmVlIHRvIGFkZCBhcHBsaWNhdGlvbi13aWRlIHN0eWxlcyB0byB0aGlzIGZpbGUgYW5kIHRoZXknbGwgYXBwZWFyIGF0IHRoZSBib3R0b20gb2YgdGhlXG4gKiBjb21waWxlZCBmaWxlIHNvIHRoZSBzdHlsZXMgeW91IGFkZCBoZXJlIHRha2UgcHJlY2VkZW5jZSBvdmVyIHN0eWxlcyBkZWZpbmVkIGluIGFueSBvdGhlciBDU1MvU0NTU1xuICogZmlsZXMgaW4gdGhpcyBkaXJlY3RvcnkuIFN0eWxlcyBpbiB0aGlzIGZpbGUgc2hvdWxkIGJlIGFkZGVkIGFmdGVyIHRoZSBsYXN0IHJlcXVpcmVfKiBzdGF0ZW1lbnQuXG4gKiBJdCBpcyBnZW5lcmFsbHkgYmV0dGVyIHRvIGNyZWF0ZSBhIG5ldyBmaWxlIHBlciBzdHlsZSBzY29wZS5cbiAqXG4gKj0gcmVxdWlyZV90cmVlIC5cbiAqPSByZXF1aXJlX3NlbGZcbiAqL1xuIn0seyJuYW1lIjoiYXBwL2NvbnRyb2xsZXJzL2FwcGxpY2F0aW9uX2NvbnRyb2xsZXIucmIiLCJjb250ZW50IjoiY2xhc3MgQXBwbGljYXRpb25Db250cm9sbGVyIDwgQWN0aW9uQ29udHJvbGxlcjo6QmFzZVxuZW5kXG4ifSx7Im5hbWUiOiJhcHAvaGVscGVycy9hcHBsaWNhdGlvbl9oZWxwZXIucmIiLCJjb250ZW50IjoibW9kdWxlIEFwcGxpY2F0aW9uSGVscGVyXG5lbmRcbiJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCIsImNvbmZpZyI6eyJpc1NlcnZlciI6dHJ1ZX19LCJybGFuZyI6eyJkaXNwbGF5TmFtZSI6IlIiLCJ0YWdsaW5lIjoiYSBwcm9ncmFtbWluZyBsYW5ndWFnZSBhbmQgZW52aXJvbm1lbnQgZm9yIHN0YXRpc3RpY2FsIGNvbXB1dGluZyBhbmQgZ3JhcGhpY3MiLCJrZXkiOiJybGFuZyIsImVudHJ5cG9pbnQiOiJtYWluLnIiLCJleHQiOiJyIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOmZhbHNlLCJoYXNMaWJyYXJpZXMiOmZhbHNlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJ1c2luZyBHTlUgUiBWZXJzaW9uIDMuNS4wICgyMDE4LTA0LTIzKSIsImNhdGVnb3J5IjoiUHJhY3RpY2FsIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9ybGFuZy5zdmciLCJ0ZW1wbGF0ZSI6IiIsIm1hdGNoaW5ncyI6W10sImVuZ2luZSI6ImdvdmFsIn0sIm5leHRqcyI6eyJkaXNwbGF5TmFtZSI6Ik5leHQuanMiLCJ0YWdsaW5lIjoiQSBsaWdodHdlaWdodCBmcmFtZXdvcmsgZm9yIHN0YXRpYyBhbmQgc2VydmVyXHUyMDExcmVuZGVyZWQgUmVhY3QgYXBwbGljYXRpb25zIiwia2V5IjoibmV4dGpzIiwiZW50cnlwb2ludCI6InBhZ2VzL2luZGV4LmpzIiwiZXh0IjoianMiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6dHJ1ZSwiaGFzTGlicmFyaWVzIjp0cnVlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJOZXh0LmpzIDYuMC4zLCBub2RlIHY5LjcuMSBsaW51eC9hbWQ2NCIsImNhdGVnb3J5IjoiRnJhbWV3b3JrIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9ub2RlanMuc3ZnIiwicHJvamVjdF90ZW1wbGF0ZSI6W3sibmFtZSI6InBhZ2VzL2luZGV4LmpzIiwiY29udGVudCI6ImV4cG9ydCBkZWZhdWx0ICgpID0+IDxkaXY+V2VsY29tZSB0byBuZXh0LmpzITwvZGl2PjsifSx7Im5hbWUiOiJwYWNrYWdlLmpzb24iLCJjb250ZW50Ijoie1xuICBcIm5hbWVcIjogXCJteS1hd2Vzb21lLWFwcFwiLFxuICBcInZlcnNpb25cIjogXCIxLjAuMFwiLFxuICBcImRlc2NyaXB0aW9uXCI6IFwiZWFzaWx5IHRoZSBuZXh0IGZhY2Vib29rXCIsXG4gIFwia2V5d29yZHNcIjogW10sXG4gIFwiZGVwZW5kZW5jaWVzXCI6IHtcbiAgICBcInJlYWN0XCI6IFwiMTYuNC4wXCIsXG4gICAgXCJyZWFjdC1kb21cIjogXCIxNi40XCIsXG4gICAgXCJuZXh0XCI6IFwiNi4wLjNcIlxuICB9LFxuICBcImRldkRlcGVuZGVuY2llc1wiOiB7fSxcbiAgXCJzY3JpcHRzXCI6IHtcbiAgICBcImRldlwiOiBcIm5leHRcIixcbiAgICBcImJ1aWxkXCI6IFwibmV4dCBidWlsZFwiLFxuICAgIFwic3RhcnRcIjogXCJuZXh0IHN0YXJ0XCJcbiAgfVxufSJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCIsImNvbmZpZyI6eyJpc1NlcnZlciI6dHJ1ZX19LCJnYXRzYnlqcyI6eyJkaXNwbGF5TmFtZSI6IkdhdHNieUpTIiwidGFnbGluZSI6IkJsYXppbmctZmFzdCBzdGF0aWMgc2l0ZSBnZW5lcmF0b3IgZm9yIFJlYWN0Iiwia2V5IjoiZ2F0c2J5anMiLCJlbnRyeXBvaW50Ijoic3JjL3BhZ2VzL2luZGV4LmpzIiwiZXh0IjoianMiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6dHJ1ZSwiaGFzTGlicmFyaWVzIjp0cnVlLCJoYXNFdmFsIjpmYWxzZSwiaGFzTGFuZ3VhZ2VTZXJ2ZXIiOmZhbHNlLCJoZWFkZXIiOiJHYXRzYnlKUyAxLjkuMjQ3LCBub2RlIHY5LjcuMSBsaW51eC9hbWQ2NCIsImNhdGVnb3J5IjoiRnJhbWV3b3JrIiwiaWNvbiI6Imh0dHBzOi8vbG9nb3MtLXR1cmJpby5yZXBsLmNvL2dhdHNieWpzLnN2ZyIsInByb2plY3RfdGVtcGxhdGUiOlt7Im5hbWUiOiJnYXRzYnktYnJvd3Nlci5qcyIsImNvbnRlbnQiOiIvKipcbiAqIEltcGxlbWVudCBHYXRzYnkncyBCcm93c2VyIEFQSXMgaW4gdGhpcyBmaWxlLlxuICpcbiAqIFNlZTogaHR0cHM6Ly93d3cuZ2F0c2J5anMub3JnL2RvY3MvYnJvd3Nlci1hcGlzL1xuICovXG5cbiAvLyBZb3UgY2FuIGRlbGV0ZSB0aGlzIGZpbGUgaWYgeW91J3JlIG5vdCB1c2luZyBpdFxuIn0seyJuYW1lIjoiZ2F0c2J5LWNvbmZpZy5qcyIsImNvbnRlbnQiOiJtb2R1bGUuZXhwb3J0cyA9IHtcbiAgc2l0ZU1ldGFkYXRhOiB7XG4gICAgdGl0bGU6ICdHYXRzYnkgRGVmYXVsdCBTdGFydGVyJyxcbiAgfSxcbiAgcGx1Z2luczogWydnYXRzYnktcGx1Z2luLXJlYWN0LWhlbG1ldCddLFxufVxuIn0seyJuYW1lIjoiZ2F0c2J5LW5vZGUuanMiLCJjb250ZW50IjoiLyoqXG4gKiBJbXBsZW1lbnQgR2F0c2J5J3MgTm9kZSBBUElzIGluIHRoaXMgZmlsZS5cbiAqXG4gKiBTZWU6IGh0dHBzOi8vd3d3LmdhdHNieWpzLm9yZy9kb2NzL25vZGUtYXBpcy9cbiAqL1xuXG4gLy8gWW91IGNhbiBkZWxldGUgdGhpcyBmaWxlIGlmIHlvdSdyZSBub3QgdXNpbmcgaXRcbiJ9LHsibmFtZSI6ImdhdHNieS1zc3IuanMiLCJjb250ZW50IjoiLyoqXG4gKiBJbXBsZW1lbnQgR2F0c2J5J3MgU1NSIChTZXJ2ZXIgU2lkZSBSZW5kZXJpbmcpIEFQSXMgaW4gdGhpcyBmaWxlLlxuICpcbiAqIFNlZTogaHR0cHM6Ly93d3cuZ2F0c2J5anMub3JnL2RvY3Mvc3NyLWFwaXMvXG4gKi9cblxuIC8vIFlvdSBjYW4gZGVsZXRlIHRoaXMgZmlsZSBpZiB5b3UncmUgbm90IHVzaW5nIGl0In0seyJuYW1lIjoicGFja2FnZS5qc29uIiwiY29udGVudCI6IntcbiAgXCJuYW1lXCI6IFwibXktZ2F0c2J5LWFwcFwiLFxuICBcImRlc2NyaXB0aW9uXCI6IFwiZWFzaWx5IHRoZSBuZXh0IGluc3RhZ3JhbVwiLFxuICBcInZlcnNpb25cIjogXCIxLjAuMFwiLFxuICBcImRlcGVuZGVuY2llc1wiOiB7XG4gICAgXCJnYXRzYnlcIjogXCJeMS45LjI0N1wiLFxuICAgIFwiZ2F0c2J5LWxpbmtcIjogXCJeMS42LjQwXCIsXG4gICAgXCJnYXRzYnktcGx1Z2luLXJlYWN0LWhlbG1ldFwiOiBcIl4yLjAuMTBcIixcbiAgICBcInJlYWN0LWhlbG1ldFwiOiBcIl41LjIuMFwiXG4gIH0sXG4gIFwia2V5d29yZHNcIjogW1xuICAgIFwiZ2F0c2J5XCJcbiAgXSxcbiAgXCJsaWNlbnNlXCI6IFwiTUlUXCIsXG4gIFwic2NyaXB0c1wiOiB7XG4gICAgXCJidWlsZFwiOiBcImdhdHNieSBidWlsZFwiLFxuICAgIFwiZGV2ZWxvcFwiOiBcImdhdHNieSBkZXZlbG9wIC1IIDAuMC4wLjBcIlxuICB9XG59XG4ifSx7Im5hbWUiOiJzcmMvY29tcG9uZW50cy9oZWFkZXIuanMiLCJjb250ZW50IjoiaW1wb3J0IFJlYWN0IGZyb20gJ3JlYWN0J1xuaW1wb3J0IExpbmsgZnJvbSAnZ2F0c2J5LWxpbmsnXG5cbmNvbnN0IEhlYWRlciA9ICh7IHNpdGVUaXRsZSB9KSA9PiAoXG4gIDxkaXZcbiAgICBzdHlsZT17e1xuICAgICAgYmFja2dyb3VuZDogJ3JlYmVjY2FwdXJwbGUnLFxuICAgICAgbWFyZ2luQm90dG9tOiAnMS40NXJlbScsXG4gICAgfX1cbiAgPlxuICAgIDxkaXZcbiAgICAgIHN0eWxlPXt7XG4gICAgICAgIG1hcmdpbjogJzAgYXV0bycsXG4gICAgICAgIG1heFdpZHRoOiA5NjAsXG4gICAgICAgIHBhZGRpbmc6ICcxLjQ1cmVtIDEuMDg3NXJlbScsXG4gICAgICB9fVxuICAgID5cbiAgICAgIDxoMSBzdHlsZT17eyBtYXJnaW46IDAgfX0+XG4gICAgICAgIDxMaW5rXG4gICAgICAgICAgdG89XCIvXCJcbiAgICAgICAgICBzdHlsZT17e1xuICAgICAgICAgICAgY29sb3I6ICd3aGl0ZScsXG4gICAgICAgICAgICB0ZXh0RGVjb3JhdGlvbjogJ25vbmUnLFxuICAgICAgICAgIH19XG4gICAgICAgID5cbiAgICAgICAgICB7c2l0ZVRpdGxlfVxuICAgICAgICA8L0xpbms+XG4gICAgICA8L2gxPlxuICAgIDwvZGl2PlxuICA8L2Rpdj5cbilcblxuZXhwb3J0IGRlZmF1bHQgSGVhZGVyXG4ifSx7Im5hbWUiOiJzcmMvbGF5b3V0cy9pbmRleC5qcyIsImNvbnRlbnQiOiJpbXBvcnQgUmVhY3QgZnJvbSAncmVhY3QnXG5pbXBvcnQgUHJvcFR5cGVzIGZyb20gJ3Byb3AtdHlwZXMnXG5pbXBvcnQgSGVsbWV0IGZyb20gJ3JlYWN0LWhlbG1ldCdcblxuaW1wb3J0IEhlYWRlciBmcm9tICcuLi9jb21wb25lbnRzL2hlYWRlcidcbmltcG9ydCAnLi9pbmRleC5jc3MnXG5cbmNvbnN0IExheW91dCA9ICh7IGNoaWxkcmVuLCBkYXRhIH0pID0+IChcbiAgPGRpdj5cbiAgICA8SGVsbWV0XG4gICAgICB0aXRsZT17ZGF0YS5zaXRlLnNpdGVNZXRhZGF0YS50aXRsZX1cbiAgICAgIG1ldGE9e1tcbiAgICAgICAgeyBuYW1lOiAnZGVzY3JpcHRpb24nLCBjb250ZW50OiAnU2FtcGxlJyB9LFxuICAgICAgICB7IG5hbWU6ICdrZXl3b3JkcycsIGNvbnRlbnQ6ICdzYW1wbGUsIHNvbWV0aGluZycgfSxcbiAgICAgIF19XG4gICAgLz5cbiAgICA8SGVhZGVyIHNpdGVUaXRsZT17ZGF0YS5zaXRlLnNpdGVNZXRhZGF0YS50aXRsZX0gLz5cbiAgICA8ZGl2XG4gICAgICBzdHlsZT17e1xuICAgICAgICBtYXJnaW46ICcwIGF1dG8nLFxuICAgICAgICBtYXhXaWR0aDogOTYwLFxuICAgICAgICBwYWRkaW5nOiAnMHB4IDEuMDg3NXJlbSAxLjQ1cmVtJyxcbiAgICAgICAgcGFkZGluZ1RvcDogMCxcbiAgICAgIH19XG4gICAgPlxuICAgICAge2NoaWxkcmVuKCl9XG4gICAgPC9kaXY+XG4gIDwvZGl2PlxuKVxuXG5MYXlvdXQucHJvcFR5cGVzID0ge1xuICBjaGlsZHJlbjogUHJvcFR5cGVzLmZ1bmMsXG59XG5cbmV4cG9ydCBkZWZhdWx0IExheW91dFxuXG5leHBvcnQgY29uc3QgcXVlcnkgPSBncmFwaHFsYFxuICBxdWVyeSBTaXRlVGl0bGVRdWVyeSB7XG4gICAgc2l0ZSB7XG4gICAgICBzaXRlTWV0YWRhdGEge1xuICAgICAgICB0aXRsZVxuICAgICAgfVxuICAgIH1cbiAgfVxuYFxuIn0seyJuYW1lIjoic3JjL2xheW91dHMvaW5kZXguY3NzIiwiY29udGVudCI6Imh0bWwge1xuICBmb250LWZhbWlseTogc2Fucy1zZXJpZjtcbiAgLW1zLXRleHQtc2l6ZS1hZGp1c3Q6IDEwMCU7XG4gIC13ZWJraXQtdGV4dC1zaXplLWFkanVzdDogMTAwJTtcbn1cbmJvZHkge1xuICBtYXJnaW46IDA7XG59XG5hcnRpY2xlLFxuYXNpZGUsXG5kZXRhaWxzLFxuZmlnY2FwdGlvbixcbmZpZ3VyZSxcbmZvb3RlcixcbmhlYWRlcixcbm1haW4sXG5tZW51LFxubmF2LFxuc2VjdGlvbixcbnN1bW1hcnkge1xuICBkaXNwbGF5OiBibG9jaztcbn1cbmF1ZGlvLFxuY2FudmFzLFxucHJvZ3Jlc3MsXG52aWRlbyB7XG4gIGRpc3BsYXk6IGlubGluZS1ibG9jaztcbn1cbmF1ZGlvOm5vdChbY29udHJvbHNdKSB7XG4gIGRpc3BsYXk6IG5vbmU7XG4gIGhlaWdodDogMDtcbn1cbnByb2dyZXNzIHtcbiAgdmVydGljYWwtYWxpZ246IGJhc2VsaW5lO1xufVxuW2hpZGRlbl0sXG50ZW1wbGF0ZSB7XG4gIGRpc3BsYXk6IG5vbmU7XG59XG5hIHtcbiAgYmFja2dyb3VuZC1jb2xvcjogdHJhbnNwYXJlbnQ7XG4gIC13ZWJraXQtdGV4dC1kZWNvcmF0aW9uLXNraXA6IG9iamVjdHM7XG59XG5hOmFjdGl2ZSxcbmE6aG92ZXIge1xuICBvdXRsaW5lLXdpZHRoOiAwO1xufVxuYWJiclt0aXRsZV0ge1xuICBib3JkZXItYm90dG9tOiBub25lO1xuICB0ZXh0LWRlY29yYXRpb246IHVuZGVybGluZTtcbiAgdGV4dC1kZWNvcmF0aW9uOiB1bmRlcmxpbmUgZG90dGVkO1xufVxuYixcbnN0cm9uZyB7XG4gIGZvbnQtd2VpZ2h0OiBpbmhlcml0O1xuICBmb250LXdlaWdodDogYm9sZGVyO1xufVxuZGZuIHtcbiAgZm9udC1zdHlsZTogaXRhbGljO1xufVxuaDEge1xuICBmb250LXNpemU6IDJlbTtcbiAgbWFyZ2luOiAuNjdlbSAwO1xufVxubWFyayB7XG4gIGJhY2tncm91bmQtY29sb3I6ICNmZjA7XG4gIGNvbG9yOiAjMDAwO1xufVxuc21hbGwge1xuICBmb250LXNpemU6IDgwJTtcbn1cbnN1YixcbnN1cCB7XG4gIGZvbnQtc2l6ZTogNzUlO1xuICBsaW5lLWhlaWdodDogMDtcbiAgcG9zaXRpb246IHJlbGF0aXZlO1xuICB2ZXJ0aWNhbC1hbGlnbjogYmFzZWxpbmU7XG59XG5zdWIge1xuICBib3R0b206IC0uMjVlbTtcbn1cbnN1cCB7XG4gIHRvcDogLS41ZW07XG59XG5pbWcge1xuICBib3JkZXItc3R5bGU6IG5vbmU7XG59XG5zdmc6bm90KDpyb290KSB7XG4gIG92ZXJmbG93OiBoaWRkZW47XG59XG5jb2RlLFxua2JkLFxucHJlLFxuc2FtcCB7XG4gIGZvbnQtZmFtaWx5OiBtb25vc3BhY2UsIG1vbm9zcGFjZTtcbiAgZm9udC1zaXplOiAxZW07XG59XG5maWd1cmUge1xuICBtYXJnaW46IDFlbSA0MHB4O1xufVxuaHIge1xuICBib3gtc2l6aW5nOiBjb250ZW50LWJveDtcbiAgaGVpZ2h0OiAwO1xuICBvdmVyZmxvdzogdmlzaWJsZTtcbn1cbmJ1dHRvbixcbmlucHV0LFxub3B0Z3JvdXAsXG5zZWxlY3QsXG50ZXh0YXJlYSB7XG4gIGZvbnQ6IGluaGVyaXQ7XG4gIG1hcmdpbjogMDtcbn1cbm9wdGdyb3VwIHtcbiAgZm9udC13ZWlnaHQ6IDcwMDtcbn1cbmJ1dHRvbixcbmlucHV0IHtcbiAgb3ZlcmZsb3c6IHZpc2libGU7XG59XG5idXR0b24sXG5zZWxlY3Qge1xuICB0ZXh0LXRyYW5zZm9ybTogbm9uZTtcbn1cblt0eXBlPXJlc2V0XSxcblt0eXBlPXN1Ym1pdF0sXG5idXR0b24sXG5odG1sIFt0eXBlPWJ1dHRvbl0ge1xuICAtd2Via2l0LWFwcGVhcmFuY2U6IGJ1dHRvbjtcbn1cblt0eXBlPWJ1dHRvbl06Oi1tb3otZm9jdXMtaW5uZXIsXG5bdHlwZT1yZXNldF06Oi1tb3otZm9jdXMtaW5uZXIsXG5bdHlwZT1zdWJtaXRdOjotbW96LWZvY3VzLWlubmVyLFxuYnV0dG9uOjotbW96LWZvY3VzLWlubmVyIHtcbiAgYm9yZGVyLXN0eWxlOiBub25lO1xuICBwYWRkaW5nOiAwO1xufVxuW3R5cGU9YnV0dG9uXTotbW96LWZvY3VzcmluZyxcblt0eXBlPXJlc2V0XTotbW96LWZvY3VzcmluZyxcblt0eXBlPXN1Ym1pdF06LW1vei1mb2N1c3JpbmcsXG5idXR0b246LW1vei1mb2N1c3Jpbmcge1xuICBvdXRsaW5lOiAxcHggZG90dGVkIEJ1dHRvblRleHQ7XG59XG5maWVsZHNldCB7XG4gIGJvcmRlcjogMXB4IHNvbGlkIHNpbHZlcjtcbiAgbWFyZ2luOiAwIDJweDtcbiAgcGFkZGluZzogLjM1ZW0gLjYyNWVtIC43NWVtO1xufVxubGVnZW5kIHtcbiAgYm94LXNpemluZzogYm9yZGVyLWJveDtcbiAgY29sb3I6IGluaGVyaXQ7XG4gIGRpc3BsYXk6IHRhYmxlO1xuICBtYXgtd2lkdGg6IDEwMCU7XG4gIHBhZGRpbmc6IDA7XG4gIHdoaXRlLXNwYWNlOiBub3JtYWw7XG59XG50ZXh0YXJlYSB7XG4gIG92ZXJmbG93OiBhdXRvO1xufVxuW3R5cGU9Y2hlY2tib3hdLFxuW3R5cGU9cmFkaW9dIHtcbiAgYm94LXNpemluZzogYm9yZGVyLWJveDtcbiAgcGFkZGluZzogMDtcbn1cblt0eXBlPW51bWJlcl06Oi13ZWJraXQtaW5uZXItc3Bpbi1idXR0b24sXG5bdHlwZT1udW1iZXJdOjotd2Via2l0LW91dGVyLXNwaW4tYnV0dG9uIHtcbiAgaGVpZ2h0OiBhdXRvO1xufVxuW3R5cGU9c2VhcmNoXSB7XG4gIC13ZWJraXQtYXBwZWFyYW5jZTogdGV4dGZpZWxkO1xuICBvdXRsaW5lLW9mZnNldDogLTJweDtcbn1cblt0eXBlPXNlYXJjaF06Oi13ZWJraXQtc2VhcmNoLWNhbmNlbC1idXR0b24sXG5bdHlwZT1zZWFyY2hdOjotd2Via2l0LXNlYXJjaC1kZWNvcmF0aW9uIHtcbiAgLXdlYmtpdC1hcHBlYXJhbmNlOiBub25lO1xufVxuOjotd2Via2l0LWlucHV0LXBsYWNlaG9sZGVyIHtcbiAgY29sb3I6IGluaGVyaXQ7XG4gIG9wYWNpdHk6IC41NDtcbn1cbjo6LXdlYmtpdC1maWxlLXVwbG9hZC1idXR0b24ge1xuICAtd2Via2l0LWFwcGVhcmFuY2U6IGJ1dHRvbjtcbiAgZm9udDogaW5oZXJpdDtcbn1cbmh0bWwge1xuICBmb250OiAxMTIuNSUvMS40NWVtIGdlb3JnaWEsIHNlcmlmO1xuICBib3gtc2l6aW5nOiBib3JkZXItYm94O1xuICBvdmVyZmxvdy15OiBzY3JvbGw7XG59XG4qIHtcbiAgYm94LXNpemluZzogaW5oZXJpdDtcbn1cbio6YmVmb3JlIHtcbiAgYm94LXNpemluZzogaW5oZXJpdDtcbn1cbio6YWZ0ZXIge1xuICBib3gtc2l6aW5nOiBpbmhlcml0O1xufVxuYm9keSB7XG4gIGNvbG9yOiBoc2xhKDAsIDAlLCAwJSwgMC44KTtcbiAgZm9udC1mYW1pbHk6IGdlb3JnaWEsIHNlcmlmO1xuICBmb250LXdlaWdodDogbm9ybWFsO1xuICB3b3JkLXdyYXA6IGJyZWFrLXdvcmQ7XG4gIGZvbnQta2VybmluZzogbm9ybWFsO1xuICAtbW96LWZvbnQtZmVhdHVyZS1zZXR0aW5nczogXCJrZXJuXCIsIFwibGlnYVwiLCBcImNsaWdcIiwgXCJjYWx0XCI7XG4gIC1tcy1mb250LWZlYXR1cmUtc2V0dGluZ3M6IFwia2VyblwiLCBcImxpZ2FcIiwgXCJjbGlnXCIsIFwiY2FsdFwiO1xuICAtd2Via2l0LWZvbnQtZmVhdHVyZS1zZXR0aW5nczogXCJrZXJuXCIsIFwibGlnYVwiLCBcImNsaWdcIiwgXCJjYWx0XCI7XG4gIGZvbnQtZmVhdHVyZS1zZXR0aW5nczogXCJrZXJuXCIsIFwibGlnYVwiLCBcImNsaWdcIiwgXCJjYWx0XCI7XG59XG5pbWcge1xuICBtYXgtd2lkdGg6IDEwMCU7XG4gIG1hcmdpbi1sZWZ0OiAwO1xuICBtYXJnaW4tcmlnaHQ6IDA7XG4gIG1hcmdpbi10b3A6IDA7XG4gIHBhZGRpbmctYm90dG9tOiAwO1xuICBwYWRkaW5nLWxlZnQ6IDA7XG4gIHBhZGRpbmctcmlnaHQ6IDA7XG4gIHBhZGRpbmctdG9wOiAwO1xuICBtYXJnaW4tYm90dG9tOiAxLjQ1cmVtO1xufVxuaDEge1xuICBtYXJnaW4tbGVmdDogMDtcbiAgbWFyZ2luLXJpZ2h0OiAwO1xuICBtYXJnaW4tdG9wOiAwO1xuICBwYWRkaW5nLWJvdHRvbTogMDtcbiAgcGFkZGluZy1sZWZ0OiAwO1xuICBwYWRkaW5nLXJpZ2h0OiAwO1xuICBwYWRkaW5nLXRvcDogMDtcbiAgbWFyZ2luLWJvdHRvbTogMS40NXJlbTtcbiAgY29sb3I6IGluaGVyaXQ7XG4gIGZvbnQtZmFtaWx5OiAtYXBwbGUtc3lzdGVtLCBCbGlua01hY1N5c3RlbUZvbnQsIFNlZ29lIFVJLCBSb2JvdG8sIE94eWdlbixcbiAgICBVYnVudHUsIENhbnRhcmVsbCwgRmlyYSBTYW5zLCBEcm9pZCBTYW5zLCBIZWx2ZXRpY2EgTmV1ZSwgc2Fucy1zZXJpZjtcbiAgZm9udC13ZWlnaHQ6IGJvbGQ7XG4gIHRleHQtcmVuZGVyaW5nOiBvcHRpbWl6ZUxlZ2liaWxpdHk7XG4gIGZvbnQtc2l6ZTogMi4yNXJlbTtcbiAgbGluZS1oZWlnaHQ6IDEuMTtcbn1cbmgyIHtcbiAgbWFyZ2luLWxlZnQ6IDA7XG4gIG1hcmdpbi1yaWdodDogMDtcbiAgbWFyZ2luLXRvcDogMDtcbiAgcGFkZGluZy1ib3R0b206IDA7XG4gIHBhZGRpbmctbGVmdDogMDtcbiAgcGFkZGluZy1yaWdodDogMDtcbiAgcGFkZGluZy10b3A6IDA7XG4gIG1hcmdpbi1ib3R0b206IDEuNDVyZW07XG4gIGNvbG9yOiBpbmhlcml0O1xuICBmb250LWZhbWlseTogLWFwcGxlLXN5c3RlbSwgQmxpbmtNYWNTeXN0ZW1Gb250LCBTZWdvZSBVSSwgUm9ib3RvLCBPeHlnZW4sXG4gICAgVWJ1bnR1LCBDYW50YXJlbGwsIEZpcmEgU2FucywgRHJvaWQgU2FucywgSGVsdmV0aWNhIE5ldWUsIHNhbnMtc2VyaWY7XG4gIGZvbnQtd2VpZ2h0OiBib2xkO1xuICB0ZXh0LXJlbmRlcmluZzogb3B0aW1pemVMZWdpYmlsaXR5O1xuICBmb250LXNpemU6IDEuNjI2NzFyZW07XG4gIGxpbmUtaGVpZ2h0OiAxLjE7XG59XG5oMyB7XG4gIG1hcmdpbi1sZWZ0OiAwO1xuICBtYXJnaW4tcmlnaHQ6IDA7XG4gIG1hcmdpbi10b3A6IDA7XG4gIHBhZGRpbmctYm90dG9tOiAwO1xuICBwYWRkaW5nLWxlZnQ6IDA7XG4gIHBhZGRpbmctcmlnaHQ6IDA7XG4gIHBhZGRpbmctdG9wOiAwO1xuICBtYXJnaW4tYm90dG9tOiAxLjQ1cmVtO1xuICBjb2xvcjogaW5oZXJpdDtcbiAgZm9udC1mYW1pbHk6IC1hcHBsZS1zeXN0ZW0sIEJsaW5rTWFjU3lzdGVtRm9udCwgU2Vnb2UgVUksIFJvYm90bywgT3h5Z2VuLFxuICAgIFVidW50dSwgQ2FudGFyZWxsLCBGaXJhIFNhbnMsIERyb2lkIFNhbnMsIEhlbHZldGljYSBOZXVlLCBzYW5zLXNlcmlmO1xuICBmb250LXdlaWdodDogYm9sZDtcbiAgdGV4dC1yZW5kZXJpbmc6IG9wdGltaXplTGVnaWJpbGl0eTtcbiAgZm9udC1zaXplOiAxLjM4MzE2cmVtO1xuICBsaW5lLWhlaWdodDogMS4xO1xufVxuaDQge1xuICBtYXJnaW4tbGVmdDogMDtcbiAgbWFyZ2luLXJpZ2h0OiAwO1xuICBtYXJnaW4tdG9wOiAwO1xuICBwYWRkaW5nLWJvdHRvbTogMDtcbiAgcGFkZGluZy1sZWZ0OiAwO1xuICBwYWRkaW5nLXJpZ2h0OiAwO1xuICBwYWRkaW5nLXRvcDogMDtcbiAgbWFyZ2luLWJvdHRvbTogMS40NXJlbTtcbiAgY29sb3I6IGluaGVyaXQ7XG4gIGZvbnQtZmFtaWx5OiAtYXBwbGUtc3lzdGVtLCBCbGlua01hY1N5c3RlbUZvbnQsIFNlZ29lIFVJLCBSb2JvdG8sIE94eWdlbixcbiAgICBVYnVudHUsIENhbnRhcmVsbCwgRmlyYSBTYW5zLCBEcm9pZCBTYW5zLCBIZWx2ZXRpY2EgTmV1ZSwgc2Fucy1zZXJpZjtcbiAgZm9udC13ZWlnaHQ6IGJvbGQ7XG4gIHRleHQtcmVuZGVyaW5nOiBvcHRpbWl6ZUxlZ2liaWxpdHk7XG4gIGZvbnQtc2l6ZTogMXJlbTtcbiAgbGluZS1oZWlnaHQ6IDEuMTtcbn1cbmg1IHtcbiAgbWFyZ2luLWxlZnQ6IDA7XG4gIG1hcmdpbi1yaWdodDogMDtcbiAgbWFyZ2luLXRvcDogMDtcbiAgcGFkZGluZy1ib3R0b206IDA7XG4gIHBhZGRpbmctbGVmdDogMDtcbiAgcGFkZGluZy1yaWdodDogMDtcbiAgcGFkZGluZy10b3A6IDA7XG4gIG1hcmdpbi1ib3R0b206IDEuNDVyZW07XG4gIGNvbG9yOiBpbmhlcml0O1xuICBmb250LWZhbWlseTogLWFwcGxlLXN5c3RlbSwgQmxpbmtNYWNTeXN0ZW1Gb250LCBTZWdvZSBVSSwgUm9ib3RvLCBPeHlnZW4sXG4gICAgVWJ1bnR1LCBDYW50YXJlbGwsIEZpcmEgU2FucywgRHJvaWQgU2FucywgSGVsdmV0aWNhIE5ldWUsIHNhbnMtc2VyaWY7XG4gIGZvbnQtd2VpZ2h0OiBib2xkO1xuICB0ZXh0LXJlbmRlcmluZzogb3B0aW1pemVMZWdpYmlsaXR5O1xuICBmb250LXNpemU6IDAuODUwMjhyZW07XG4gIGxpbmUtaGVpZ2h0OiAxLjE7XG59XG5oNiB7XG4gIG1hcmdpbi1sZWZ0OiAwO1xuICBtYXJnaW4tcmlnaHQ6IDA7XG4gIG1hcmdpbi10b3A6IDA7XG4gIHBhZGRpbmctYm90dG9tOiAwO1xuICBwYWRkaW5nLWxlZnQ6IDA7XG4gIHBhZGRpbmctcmlnaHQ6IDA7XG4gIHBhZGRpbmctdG9wOiAwO1xuICBtYXJnaW4tYm90dG9tOiAxLjQ1cmVtO1xuICBjb2xvcjogaW5oZXJpdDtcbiAgZm9udC1mYW1pbHk6IC1hcHBsZS1zeXN0ZW0sIEJsaW5rTWFjU3lzdGVtRm9udCwgU2Vnb2UgVUksIFJvYm90bywgT3h5Z2VuLFxuICAgIFVidW50dSwgQ2FudGFyZWxsLCBGaXJhIFNhbnMsIERyb2lkIFNhbnMsIEhlbHZldGljYSBOZXVlLCBzYW5zLXNlcmlmO1xuICBmb250LXdlaWdodDogYm9sZDtcbiAgdGV4dC1yZW5kZXJpbmc6IG9wdGltaXplTGVnaWJpbGl0eTtcbiAgZm9udC1zaXplOiAwLjc4NDA1cmVtO1xuICBsaW5lLWhlaWdodDogMS4xO1xufVxuaGdyb3VwIHtcbiAgbWFyZ2luLWxlZnQ6IDA7XG4gIG1hcmdpbi1yaWdodDogMDtcbiAgbWFyZ2luLXRvcDogMDtcbiAgcGFkZGluZy1ib3R0b206IDA7XG4gIHBhZGRpbmctbGVmdDogMDtcbiAgcGFkZGluZy1yaWdodDogMDtcbiAgcGFkZGluZy10b3A6IDA7XG4gIG1hcmdpbi1ib3R0b206IDEuNDVyZW07XG59XG51bCB7XG4gIG1hcmdpbi1sZWZ0OiAxLjQ1cmVtO1xuICBtYXJnaW4tcmlnaHQ6IDA7XG4gIG1hcmdpbi10b3A6IDA7XG4gIHBhZGRpbmctYm90dG9tOiAwO1xuICBwYWRkaW5nLWxlZnQ6IDA7XG4gIHBhZGRpbmctcmlnaHQ6IDA7XG4gIHBhZGRpbmctdG9wOiAwO1xuICBtYXJnaW4tYm90dG9tOiAxLjQ1cmVtO1xuICBsaXN0LXN0eWxlLXBvc2l0aW9uOiBvdXRzaWRlO1xuICBsaXN0LXN0eWxlLWltYWdlOiBub25lO1xufVxub2wge1xuICBtYXJnaW4tbGVmdDogMS40NXJlbTtcbiAgbWFyZ2luLXJpZ2h0OiAwO1xuICBtYXJnaW4tdG9wOiAwO1xuICBwYWRkaW5nLWJvdHRvbTogMDtcbiAgcGFkZGluZy1sZWZ0OiAwO1xuICBwYWRkaW5nLXJpZ2h0OiAwO1xuICBwYWRkaW5nLXRvcDogMDtcbiAgbWFyZ2luLWJvdHRvbTogMS40NXJlbTtcbiAgbGlzdC1zdHlsZS1wb3NpdGlvbjogb3V0c2lkZTtcbiAgbGlzdC1zdHlsZS1pbWFnZTogbm9uZTtcbn1cbmRsIHtcbiAgbWFyZ2luLWxlZnQ6IDA7XG4gIG1hcmdpbi1yaWdodDogMDtcbiAgbWFyZ2luLXRvcDogMDtcbiAgcGFkZGluZy1ib3R0b206IDA7XG4gIHBhZGRpbmctbGVmdDogMDtcbiAgcGFkZGluZy1yaWdodDogMDtcbiAgcGFkZGluZy10b3A6IDA7XG4gIG1hcmdpbi1ib3R0b206IDEuNDVyZW07XG59XG5kZCB7XG4gIG1hcmdpbi1sZWZ0OiAwO1xuICBtYXJnaW4tcmlnaHQ6IDA7XG4gIG1hcmdpbi10b3A6IDA7XG4gIHBhZGRpbmctYm90dG9tOiAwO1xuICBwYWRkaW5nLWxlZnQ6IDA7XG4gIHBhZGRpbmctcmlnaHQ6IDA7XG4gIHBhZGRpbmctdG9wOiAwO1xuICBtYXJnaW4tYm90dG9tOiAxLjQ1cmVtO1xufVxucCB7XG4gIG1hcmdpbi1sZWZ0OiAwO1xuICBtYXJnaW4tcmlnaHQ6IDA7XG4gIG1hcmdpbi10b3A6IDA7XG4gIHBhZGRpbmctYm90dG9tOiAwO1xuICBwYWRkaW5nLWxlZnQ6IDA7XG4gIHBhZGRpbmctcmlnaHQ6IDA7XG4gIHBhZGRpbmctdG9wOiAwO1xuICBtYXJnaW4tYm90dG9tOiAxLjQ1cmVtO1xufVxuZmlndXJlIHtcbiAgbWFyZ2luLWxlZnQ6IDA7XG4gIG1hcmdpbi1yaWdodDogMDtcbiAgbWFyZ2luLXRvcDogMDtcbiAgcGFkZGluZy1ib3R0b206IDA7XG4gIHBhZGRpbmctbGVmdDogMDtcbiAgcGFkZGluZy1yaWdodDogMDtcbiAgcGFkZGluZy10b3A6IDA7XG4gIG1hcmdpbi1ib3R0b206IDEuNDVyZW07XG59XG5wcmUge1xuICBtYXJnaW4tbGVmdDogMDtcbiAgbWFyZ2luLXJpZ2h0OiAwO1xuICBtYXJnaW4tdG9wOiAwO1xuICBwYWRkaW5nLWJvdHRvbTogMDtcbiAgcGFkZGluZy1sZWZ0OiAwO1xuICBwYWRkaW5nLXJpZ2h0OiAwO1xuICBwYWRkaW5nLXRvcDogMDtcbiAgbWFyZ2luLWJvdHRvbTogMS40NXJlbTtcbiAgZm9udC1zaXplOiAwLjg1cmVtO1xuICBsaW5lLWhlaWdodDogMS40MjtcbiAgYmFja2dyb3VuZDogaHNsYSgwLCAwJSwgMCUsIDAuMDQpO1xuICBib3JkZXItcmFkaXVzOiAzcHg7XG4gIG92ZXJmbG93OiBhdXRvO1xuICB3b3JkLXdyYXA6IG5vcm1hbDtcbiAgcGFkZGluZzogMS40NXJlbTtcbn1cbnRhYmxlIHtcbiAgbWFyZ2luLWxlZnQ6IDA7XG4gIG1hcmdpbi1yaWdodDogMDtcbiAgbWFyZ2luLXRvcDogMDtcbiAgcGFkZGluZy1ib3R0b206IDA7XG4gIHBhZGRpbmctbGVmdDogMDtcbiAgcGFkZGluZy1yaWdodDogMDtcbiAgcGFkZGluZy10b3A6IDA7XG4gIG1hcmdpbi1ib3R0b206IDEuNDVyZW07XG4gIGZvbnQtc2l6ZTogMXJlbTtcbiAgbGluZS1oZWlnaHQ6IDEuNDVyZW07XG4gIGJvcmRlci1jb2xsYXBzZTogY29sbGFwc2U7XG4gIHdpZHRoOiAxMDAlO1xufVxuZmllbGRzZXQge1xuICBtYXJnaW4tbGVmdDogMDtcbiAgbWFyZ2luLXJpZ2h0OiAwO1xuICBtYXJnaW4tdG9wOiAwO1xuICBwYWRkaW5nLWJvdHRvbTogMDtcbiAgcGFkZGluZy1sZWZ0OiAwO1xuICBwYWRkaW5nLXJpZ2h0OiAwO1xuICBwYWRkaW5nLXRvcDogMDtcbiAgbWFyZ2luLWJvdHRvbTogMS40NXJlbTtcbn1cbmJsb2NrcXVvdGUge1xuICBtYXJnaW4tbGVmdDogMS40NXJlbTtcbiAgbWFyZ2luLXJpZ2h0OiAxLjQ1cmVtO1xuICBtYXJnaW4tdG9wOiAwO1xuICBwYWRkaW5nLWJvdHRvbTogMDtcbiAgcGFkZGluZy1sZWZ0OiAwO1xuICBwYWRkaW5nLXJpZ2h0OiAwO1xuICBwYWRkaW5nLXRvcDogMDtcbiAgbWFyZ2luLWJvdHRvbTogMS40NXJlbTtcbn1cbmZvcm0ge1xuICBtYXJnaW4tbGVmdDogMDtcbiAgbWFyZ2luLXJpZ2h0OiAwO1xuICBtYXJnaW4tdG9wOiAwO1xuICBwYWRkaW5nLWJvdHRvbTogMDtcbiAgcGFkZGluZy1sZWZ0OiAwO1xuICBwYWRkaW5nLXJpZ2h0OiAwO1xuICBwYWRkaW5nLXRvcDogMDtcbiAgbWFyZ2luLWJvdHRvbTogMS40NXJlbTtcbn1cbm5vc2NyaXB0IHtcbiAgbWFyZ2luLWxlZnQ6IDA7XG4gIG1hcmdpbi1yaWdodDogMDtcbiAgbWFyZ2luLXRvcDogMDtcbiAgcGFkZGluZy1ib3R0b206IDA7XG4gIHBhZGRpbmctbGVmdDogMDtcbiAgcGFkZGluZy1yaWdodDogMDtcbiAgcGFkZGluZy10b3A6IDA7XG4gIG1hcmdpbi1ib3R0b206IDEuNDVyZW07XG59XG5pZnJhbWUge1xuICBtYXJnaW4tbGVmdDogMDtcbiAgbWFyZ2luLXJpZ2h0OiAwO1xuICBtYXJnaW4tdG9wOiAwO1xuICBwYWRkaW5nLWJvdHRvbTogMDtcbiAgcGFkZGluZy1sZWZ0OiAwO1xuICBwYWRkaW5nLXJpZ2h0OiAwO1xuICBwYWRkaW5nLXRvcDogMDtcbiAgbWFyZ2luLWJvdHRvbTogMS40NXJlbTtcbn1cbmhyIHtcbiAgbWFyZ2luLWxlZnQ6IDA7XG4gIG1hcmdpbi1yaWdodDogMDtcbiAgbWFyZ2luLXRvcDogMDtcbiAgcGFkZGluZy1ib3R0b206IDA7XG4gIHBhZGRpbmctbGVmdDogMDtcbiAgcGFkZGluZy1yaWdodDogMDtcbiAgcGFkZGluZy10b3A6IDA7XG4gIG1hcmdpbi1ib3R0b206IGNhbGMoMS40NXJlbSAtIDFweCk7XG4gIGJhY2tncm91bmQ6IGhzbGEoMCwgMCUsIDAlLCAwLjIpO1xuICBib3JkZXI6IG5vbmU7XG4gIGhlaWdodDogMXB4O1xufVxuYWRkcmVzcyB7XG4gIG1hcmdpbi1sZWZ0OiAwO1xuICBtYXJnaW4tcmlnaHQ6IDA7XG4gIG1hcmdpbi10b3A6IDA7XG4gIHBhZGRpbmctYm90dG9tOiAwO1xuICBwYWRkaW5nLWxlZnQ6IDA7XG4gIHBhZGRpbmctcmlnaHQ6IDA7XG4gIHBhZGRpbmctdG9wOiAwO1xuICBtYXJnaW4tYm90dG9tOiAxLjQ1cmVtO1xufVxuYiB7XG4gIGZvbnQtd2VpZ2h0OiBib2xkO1xufVxuc3Ryb25nIHtcbiAgZm9udC13ZWlnaHQ6IGJvbGQ7XG59XG5kdCB7XG4gIGZvbnQtd2VpZ2h0OiBib2xkO1xufVxudGgge1xuICBmb250LXdlaWdodDogYm9sZDtcbn1cbmxpIHtcbiAgbWFyZ2luLWJvdHRvbTogY2FsYygxLjQ1cmVtIC8gMik7XG59XG5vbCBsaSB7XG4gIHBhZGRpbmctbGVmdDogMDtcbn1cbnVsIGxpIHtcbiAgcGFkZGluZy1sZWZ0OiAwO1xufVxubGkgPiBvbCB7XG4gIG1hcmdpbi1sZWZ0OiAxLjQ1cmVtO1xuICBtYXJnaW4tYm90dG9tOiBjYWxjKDEuNDVyZW0gLyAyKTtcbiAgbWFyZ2luLXRvcDogY2FsYygxLjQ1cmVtIC8gMik7XG59XG5saSA+IHVsIHtcbiAgbWFyZ2luLWxlZnQ6IDEuNDVyZW07XG4gIG1hcmdpbi1ib3R0b206IGNhbGMoMS40NXJlbSAvIDIpO1xuICBtYXJnaW4tdG9wOiBjYWxjKDEuNDVyZW0gLyAyKTtcbn1cbmJsb2NrcXVvdGUgKjpsYXN0LWNoaWxkIHtcbiAgbWFyZ2luLWJvdHRvbTogMDtcbn1cbmxpICo6bGFzdC1jaGlsZCB7XG4gIG1hcmdpbi1ib3R0b206IDA7XG59XG5wICo6bGFzdC1jaGlsZCB7XG4gIG1hcmdpbi1ib3R0b206IDA7XG59XG5saSA+IHAge1xuICBtYXJnaW4tYm90dG9tOiBjYWxjKDEuNDVyZW0gLyAyKTtcbn1cbmNvZGUge1xuICBmb250LXNpemU6IDAuODVyZW07XG4gIGxpbmUtaGVpZ2h0OiAxLjQ1cmVtO1xufVxua2JkIHtcbiAgZm9udC1zaXplOiAwLjg1cmVtO1xuICBsaW5lLWhlaWdodDogMS40NXJlbTtcbn1cbnNhbXAge1xuICBmb250LXNpemU6IDAuODVyZW07XG4gIGxpbmUtaGVpZ2h0OiAxLjQ1cmVtO1xufVxuYWJiciB7XG4gIGJvcmRlci1ib3R0b206IDFweCBkb3R0ZWQgaHNsYSgwLCAwJSwgMCUsIDAuNSk7XG4gIGN1cnNvcjogaGVscDtcbn1cbmFjcm9ueW0ge1xuICBib3JkZXItYm90dG9tOiAxcHggZG90dGVkIGhzbGEoMCwgMCUsIDAlLCAwLjUpO1xuICBjdXJzb3I6IGhlbHA7XG59XG5hYmJyW3RpdGxlXSB7XG4gIGJvcmRlci1ib3R0b206IDFweCBkb3R0ZWQgaHNsYSgwLCAwJSwgMCUsIDAuNSk7XG4gIGN1cnNvcjogaGVscDtcbiAgdGV4dC1kZWNvcmF0aW9uOiBub25lO1xufVxudGhlYWQge1xuICB0ZXh0LWFsaWduOiBsZWZ0O1xufVxudGQsXG50aCB7XG4gIHRleHQtYWxpZ246IGxlZnQ7XG4gIGJvcmRlci1ib3R0b206IDFweCBzb2xpZCBoc2xhKDAsIDAlLCAwJSwgMC4xMik7XG4gIGZvbnQtZmVhdHVyZS1zZXR0aW5nczogXCJ0bnVtXCI7XG4gIC1tb3otZm9udC1mZWF0dXJlLXNldHRpbmdzOiBcInRudW1cIjtcbiAgLW1zLWZvbnQtZmVhdHVyZS1zZXR0aW5nczogXCJ0bnVtXCI7XG4gIC13ZWJraXQtZm9udC1mZWF0dXJlLXNldHRpbmdzOiBcInRudW1cIjtcbiAgcGFkZGluZy1sZWZ0OiAwLjk2NjY3cmVtO1xuICBwYWRkaW5nLXJpZ2h0OiAwLjk2NjY3cmVtO1xuICBwYWRkaW5nLXRvcDogMC43MjVyZW07XG4gIHBhZGRpbmctYm90dG9tOiBjYWxjKDAuNzI1cmVtIC0gMXB4KTtcbn1cbnRoOmZpcnN0LWNoaWxkLFxudGQ6Zmlyc3QtY2hpbGQge1xuICBwYWRkaW5nLWxlZnQ6IDA7XG59XG50aDpsYXN0LWNoaWxkLFxudGQ6bGFzdC1jaGlsZCB7XG4gIHBhZGRpbmctcmlnaHQ6IDA7XG59XG50dCxcbmNvZGUge1xuICBiYWNrZ3JvdW5kLWNvbG9yOiBoc2xhKDAsIDAlLCAwJSwgMC4wNCk7XG4gIGJvcmRlci1yYWRpdXM6IDNweDtcbiAgZm9udC1mYW1pbHk6IFwiU0ZNb25vLVJlZ3VsYXJcIiwgQ29uc29sYXMsIFwiUm9ib3RvIE1vbm9cIiwgXCJEcm9pZCBTYW5zIE1vbm9cIixcbiAgICBcIkxpYmVyYXRpb24gTW9ub1wiLCBNZW5sbywgQ291cmllciwgbW9ub3NwYWNlO1xuICBwYWRkaW5nOiAwO1xuICBwYWRkaW5nLXRvcDogMC4yZW07XG4gIHBhZGRpbmctYm90dG9tOiAwLjJlbTtcbn1cbnByZSBjb2RlIHtcbiAgYmFja2dyb3VuZDogbm9uZTtcbiAgbGluZS1oZWlnaHQ6IDEuNDI7XG59XG5jb2RlOmJlZm9yZSxcbmNvZGU6YWZ0ZXIsXG50dDpiZWZvcmUsXG50dDphZnRlciB7XG4gIGxldHRlci1zcGFjaW5nOiAtMC4yZW07XG4gIGNvbnRlbnQ6IFwiIFwiO1xufVxucHJlIGNvZGU6YmVmb3JlLFxucHJlIGNvZGU6YWZ0ZXIsXG5wcmUgdHQ6YmVmb3JlLFxucHJlIHR0OmFmdGVyIHtcbiAgY29udGVudDogXCJcIjtcbn1cbkBtZWRpYSBvbmx5IHNjcmVlbiBhbmQgKG1heC13aWR0aDogNDgwcHgpIHtcbiAgaHRtbCB7XG4gICAgZm9udC1zaXplOiAxMDAlO1xuICB9XG59XG4ifSx7Im5hbWUiOiJzcmMvcGFnZXMvaW5kZXguanMiLCJjb250ZW50IjoiaW1wb3J0IFJlYWN0IGZyb20gJ3JlYWN0J1xuaW1wb3J0IExpbmsgZnJvbSAnZ2F0c2J5LWxpbmsnXG5cbmNvbnN0IEluZGV4UGFnZSA9ICgpID0+IChcbiAgPGRpdj5cbiAgICA8aDE+SGkgcGVvcGxlPC9oMT5cbiAgICA8cD5XZWxjb21lIHRvIHlvdXIgbmV3IEdhdHNieSBzaXRlLjwvcD5cbiAgICA8cD5Ob3cgZ28gYnVpbGQgc29tZXRoaW5nIGdyZWF0LjwvcD5cbiAgICA8TGluayB0bz1cIi9wYWdlLTIvXCI+R28gdG8gcGFnZSAyPC9MaW5rPlxuICA8L2Rpdj5cbilcblxuZXhwb3J0IGRlZmF1bHQgSW5kZXhQYWdlXG4ifSx7Im5hbWUiOiJzcmMvcGFnZXMvNDA0LmpzIiwiY29udGVudCI6ImltcG9ydCBSZWFjdCBmcm9tICdyZWFjdCdcblxuY29uc3QgTm90Rm91bmRQYWdlID0gKCkgPT4gKFxuICA8ZGl2PlxuICAgIDxoMT5OT1QgRk9VTkQ8L2gxPlxuICAgIDxwPllvdSBqdXN0IGhpdCBhIHJvdXRlIHRoYXQgZG9lc24mIzM5O3QgZXhpc3QuLi4gdGhlIHNhZG5lc3MuPC9wPlxuICA8L2Rpdj5cbilcblxuZXhwb3J0IGRlZmF1bHQgTm90Rm91bmRQYWdlXG4ifSx7Im5hbWUiOiJzcmMvcGFnZXMvcGFnZS0yLmpzIiwiY29udGVudCI6ImltcG9ydCBSZWFjdCBmcm9tICdyZWFjdCdcbmltcG9ydCBMaW5rIGZyb20gJ2dhdHNieS1saW5rJ1xuXG5jb25zdCBTZWNvbmRQYWdlID0gKCkgPT4gKFxuICA8ZGl2PlxuICAgIDxoMT5IaSBmcm9tIHRoZSBzZWNvbmQgcGFnZTwvaDE+XG4gICAgPHA+V2VsY29tZSB0byBwYWdlIDI8L3A+XG4gICAgPExpbmsgdG89XCIvXCI+R28gYmFjayB0byB0aGUgaG9tZXBhZ2U8L0xpbms+XG4gIDwvZGl2PlxuKVxuXG5leHBvcnQgZGVmYXVsdCBTZWNvbmRQYWdlXG4ifV0sIm1hdGNoaW5ncyI6W1sieyIsIn0iXSxbIigiLCIpIl0sWyJbIiwiXSJdXSwiZW5naW5lIjoiZ292YWwiLCJjb25maWciOnsiaXNTZXJ2ZXIiOnRydWV9fSwicmVhY3RqcyI6eyJkaXNwbGF5TmFtZSI6IlJlYWN0IiwidGFnbGluZSI6IkEgSmF2YVNjcmlwdCBsaWJyYXJ5IGZvciBidWlsZGluZyB1c2VyIGludGVyZmFjZXMiLCJrZXkiOiJyZWFjdGpzIiwiZW50cnlwb2ludCI6InNyYy9pbmRleC5qcyIsImV4dCI6ImpzIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6dHJ1ZSwiaGFzRXZhbCI6ZmFsc2UsImhhc0xhbmd1YWdlU2VydmVyIjpmYWxzZSwiaGVhZGVyIjoiUmVhY3QgMTYuNC4wLCBub2RlIHY5LjcuMSBsaW51eC9hbWQ2NCIsImNhdGVnb3J5IjoiRnJhbWV3b3JrIiwiaWNvbiI6Imh0dHBzOi8vbG9nb3MtLXR1cmJpby5yZXBsLmNvL3JlYWN0anMuc3ZnIiwicHJvamVjdF90ZW1wbGF0ZSI6W3sibmFtZSI6InBhY2thZ2UuanNvbiIsImNvbnRlbnQiOiJ7XG4gIFwibmFtZVwiOiBcInJ1bm5lclwiLFxuICBcInZlcnNpb25cIjogXCIwLjEuMFwiLFxuICBcInByaXZhdGVcIjogdHJ1ZSxcbiAgXCJkZXBlbmRlbmNpZXNcIjoge1xuICAgIFwicmVhY3RcIjogXCJeMTYuNC4wXCIsXG4gICAgXCJyZWFjdC1kb21cIjogXCJeMTYuNC4wXCIsXG4gICAgXCJyZWFjdC1zY3JpcHRzXCI6IFwiMS4xLjRcIlxuICB9LFxuICBcInNjcmlwdHNcIjoge1xuICAgIFwic3RhcnRcIjogXCJyZWFjdC1zY3JpcHRzIHN0YXJ0XCIsXG4gICAgXCJidWlsZFwiOiBcInJlYWN0LXNjcmlwdHMgYnVpbGRcIixcbiAgICBcInRlc3RcIjogXCJyZWFjdC1zY3JpcHRzIHRlc3QgLS1lbnY9anNkb21cIixcbiAgICBcImVqZWN0XCI6IFwicmVhY3Qtc2NyaXB0cyBlamVjdFwiXG4gIH1cbn0ifSx7Im5hbWUiOiJzcmMvaW5kZXguY3NzIiwiY29udGVudCI6ImJvZHkge1xuICBtYXJnaW46IDA7XG4gIHBhZGRpbmc6IDA7XG4gIGZvbnQtZmFtaWx5OiBzYW5zLXNlcmlmO1xufVxuIn0seyJuYW1lIjoic3JjL2luZGV4LmpzIiwiY29udGVudCI6ImltcG9ydCBSZWFjdCBmcm9tICdyZWFjdCc7XG5pbXBvcnQgUmVhY3RET00gZnJvbSAncmVhY3QtZG9tJztcbmltcG9ydCAnLi9pbmRleC5jc3MnO1xuaW1wb3J0IEFwcCBmcm9tICcuL0FwcCc7XG5cblJlYWN0RE9NLnJlbmRlcig8QXBwIC8+LCBkb2N1bWVudC5nZXRFbGVtZW50QnlJZCgncm9vdCcpKTsifSx7Im5hbWUiOiJzcmMvQXBwLmNzcyIsImNvbnRlbnQiOiIuQXBwIHtcbiAgdGV4dC1hbGlnbjogY2VudGVyO1xufVxuXG4uQXBwLWxvZ28ge1xuICBhbmltYXRpb246IEFwcC1sb2dvLXNwaW4gaW5maW5pdGUgMjBzIGxpbmVhcjtcbiAgaGVpZ2h0OiA4MHB4O1xufVxuXG4uQXBwLWhlYWRlciB7XG4gIGJhY2tncm91bmQtY29sb3I6ICMyMjI7XG4gIGhlaWdodDogMTUwcHg7XG4gIHBhZGRpbmc6IDIwcHg7XG4gIGNvbG9yOiB3aGl0ZTtcbn1cblxuLkFwcC10aXRsZSB7XG4gIGZvbnQtc2l6ZTogMS41ZW07XG59XG5cbi5BcHAtaW50cm8ge1xuICBmb250LXNpemU6IGxhcmdlO1xufVxuXG5Aa2V5ZnJhbWVzIEFwcC1sb2dvLXNwaW4ge1xuICBmcm9tIHsgdHJhbnNmb3JtOiByb3RhdGUoMGRlZyk7IH1cbiAgdG8geyB0cmFuc2Zvcm06IHJvdGF0ZSgzNjBkZWcpOyB9XG59XG4ifSx7Im5hbWUiOiJzcmMvbG9nby5zdmciLCJjb250ZW50IjoiPHN2ZyB4bWxucz1cImh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnXCIgdmlld0JveD1cIjAgMCA4NDEuOSA1OTUuM1wiPlxuICAgIDxnIGZpbGw9XCIjNjFEQUZCXCI+XG4gICAgICAgIDxwYXRoIGQ9XCJNNjY2LjMgMjk2LjVjMC0zMi41LTQwLjctNjMuMy0xMDMuMS04Mi40IDE0LjQtNjMuNiA4LTExNC4yLTIwLjItMTMwLjQtNi41LTMuOC0xNC4xLTUuNi0yMi40LTUuNnYyMi4zYzQuNiAwIDguMy45IDExLjQgMi42IDEzLjYgNy44IDE5LjUgMzcuNSAxNC45IDc1LjctMS4xIDkuNC0yLjkgMTkuMy01LjEgMjkuNC0xOS42LTQuOC00MS04LjUtNjMuNS0xMC45LTEzLjUtMTguNS0yNy41LTM1LjMtNDEuNi01MCAzMi42LTMwLjMgNjMuMi00Ni45IDg0LTQ2LjlWNzhjLTI3LjUgMC02My41IDE5LjYtOTkuOSA1My42LTM2LjQtMzMuOC03Mi40LTUzLjItOTkuOS01My4ydjIyLjNjMjAuNyAwIDUxLjQgMTYuNSA4NCA0Ni42LTE0IDE0LjctMjggMzEuNC00MS4zIDQ5LjktMjIuNiAyLjQtNDQgNi4xLTYzLjYgMTEtMi4zLTEwLTQtMTkuNy01LjItMjktNC43LTM4LjIgMS4xLTY3LjkgMTQuNi03NS44IDMtMS44IDYuOS0yLjYgMTEuNS0yLjZWNzguNWMtOC40IDAtMTYgMS44LTIyLjYgNS42LTI4LjEgMTYuMi0zNC40IDY2LjctMTkuOSAxMzAuMS02Mi4yIDE5LjItMTAyLjcgNDkuOS0xMDIuNyA4Mi4zIDAgMzIuNSA0MC43IDYzLjMgMTAzLjEgODIuNC0xNC40IDYzLjYtOCAxMTQuMiAyMC4yIDEzMC40IDYuNSAzLjggMTQuMSA1LjYgMjIuNSA1LjYgMjcuNSAwIDYzLjUtMTkuNiA5OS45LTUzLjYgMzYuNCAzMy44IDcyLjQgNTMuMiA5OS45IDUzLjIgOC40IDAgMTYtMS44IDIyLjYtNS42IDI4LjEtMTYuMiAzNC40LTY2LjcgMTkuOS0xMzAuMSA2Mi0xOS4xIDEwMi41LTQ5LjkgMTAyLjUtODIuM3ptLTEzMC4yLTY2LjdjLTMuNyAxMi45LTguMyAyNi4yLTEzLjUgMzkuNS00LjEtOC04LjQtMTYtMTMuMS0yNC00LjYtOC05LjUtMTUuOC0xNC40LTIzLjQgMTQuMiAyLjEgMjcuOSA0LjcgNDEgNy45em0tNDUuOCAxMDYuNWMtNy44IDEzLjUtMTUuOCAyNi4zLTI0LjEgMzguMi0xNC45IDEuMy0zMCAyLTQ1LjIgMi0xNS4xIDAtMzAuMi0uNy00NS0xLjktOC4zLTExLjktMTYuNC0yNC42LTI0LjItMzgtNy42LTEzLjEtMTQuNS0yNi40LTIwLjgtMzkuOCA2LjItMTMuNCAxMy4yLTI2LjggMjAuNy0zOS45IDcuOC0xMy41IDE1LjgtMjYuMyAyNC4xLTM4LjIgMTQuOS0xLjMgMzAtMiA0NS4yLTIgMTUuMSAwIDMwLjIuNyA0NSAxLjkgOC4zIDExLjkgMTYuNCAyNC42IDI0LjIgMzggNy42IDEzLjEgMTQuNSAyNi40IDIwLjggMzkuOC02LjMgMTMuNC0xMy4yIDI2LjgtMjAuNyAzOS45em0zMi4zLTEzYzUuNCAxMy40IDEwIDI2LjggMTMuOCAzOS44LTEzLjEgMy4yLTI2LjkgNS45LTQxLjIgOCA0LjktNy43IDkuOC0xNS42IDE0LjQtMjMuNyA0LjYtOCA4LjktMTYuMSAxMy0yNC4xek00MjEuMiA0MzBjLTkuMy05LjYtMTguNi0yMC4zLTI3LjgtMzIgOSAuNCAxOC4yLjcgMjcuNS43IDkuNCAwIDE4LjctLjIgMjcuOC0uNy05IDExLjctMTguMyAyMi40LTI3LjUgMzJ6bS03NC40LTU4LjljLTE0LjItMi4xLTI3LjktNC43LTQxLTcuOSAzLjctMTIuOSA4LjMtMjYuMiAxMy41LTM5LjUgNC4xIDggOC40IDE2IDEzLjEgMjQgNC43IDggOS41IDE1LjggMTQuNCAyMy40ek00MjAuNyAxNjNjOS4zIDkuNiAxOC42IDIwLjMgMjcuOCAzMi05LS40LTE4LjItLjctMjcuNS0uNy05LjQgMC0xOC43LjItMjcuOC43IDktMTEuNyAxOC4zLTIyLjQgMjcuNS0zMnptLTc0IDU4LjljLTQuOSA3LjctOS44IDE1LjYtMTQuNCAyMy43LTQuNiA4LTguOSAxNi0xMyAyNC01LjQtMTMuNC0xMC0yNi44LTEzLjgtMzkuOCAxMy4xLTMuMSAyNi45LTUuOCA0MS4yLTcuOXptLTkwLjUgMTI1LjJjLTM1LjQtMTUuMS01OC4zLTM0LjktNTguMy01MC42IDAtMTUuNyAyMi45LTM1LjYgNTguMy01MC42IDguNi0zLjcgMTgtNyAyNy43LTEwLjEgNS43IDE5LjYgMTMuMiA0MCAyMi41IDYwLjktOS4yIDIwLjgtMTYuNiA0MS4xLTIyLjIgNjAuNi05LjktMy4xLTE5LjMtNi41LTI4LTEwLjJ6TTMxMCA0OTBjLTEzLjYtNy44LTE5LjUtMzcuNS0xNC45LTc1LjcgMS4xLTkuNCAyLjktMTkuMyA1LjEtMjkuNCAxOS42IDQuOCA0MSA4LjUgNjMuNSAxMC45IDEzLjUgMTguNSAyNy41IDM1LjMgNDEuNiA1MC0zMi42IDMwLjMtNjMuMiA0Ni45LTg0IDQ2LjktNC41LS4xLTguMy0xLTExLjMtMi43em0yMzcuMi03Ni4yYzQuNyAzOC4yLTEuMSA2Ny45LTE0LjYgNzUuOC0zIDEuOC02LjkgMi42LTExLjUgMi42LTIwLjcgMC01MS40LTE2LjUtODQtNDYuNiAxNC0xNC43IDI4LTMxLjQgNDEuMy00OS45IDIyLjYtMi40IDQ0LTYuMSA2My42LTExIDIuMyAxMC4xIDQuMSAxOS44IDUuMiAyOS4xem0zOC41LTY2LjdjLTguNiAzLjctMTggNy0yNy43IDEwLjEtNS43LTE5LjYtMTMuMi00MC0yMi41LTYwLjkgOS4yLTIwLjggMTYuNi00MS4xIDIyLjItNjAuNiA5LjkgMy4xIDE5LjMgNi41IDI4LjEgMTAuMiAzNS40IDE1LjEgNTguMyAzNC45IDU4LjMgNTAuNi0uMSAxNS43LTIzIDM1LjYtNTguNCA1MC42ek0zMjAuOCA3OC40elwiLz5cbiAgICAgICAgPGNpcmNsZSBjeD1cIjQyMC45XCIgY3k9XCIyOTYuNVwiIHI9XCI0NS43XCIvPlxuICAgICAgICA8cGF0aCBkPVwiTTUyMC41IDc4LjF6XCIvPlxuICAgIDwvZz5cbjwvc3ZnPlxuIn0seyJuYW1lIjoic3JjL0FwcC5qcyIsImNvbnRlbnQiOiJpbXBvcnQgUmVhY3QsIHsgQ29tcG9uZW50IH0gZnJvbSAncmVhY3QnO1xuaW1wb3J0IGxvZ28gZnJvbSAnLi9sb2dvLnN2Zyc7XG5pbXBvcnQgJy4vQXBwLmNzcyc7XG5cbmNsYXNzIEFwcCBleHRlbmRzIENvbXBvbmVudCB7XG4gIHJlbmRlcigpIHtcbiAgICByZXR1cm4gKFxuICAgICAgPGRpdiBjbGFzc05hbWU9XCJBcHBcIj5cbiAgICAgICAgPGhlYWRlciBjbGFzc05hbWU9XCJBcHAtaGVhZGVyXCI+XG4gICAgICAgICAgPGltZyBzcmM9e2xvZ299IGNsYXNzTmFtZT1cIkFwcC1sb2dvXCIgYWx0PVwibG9nb1wiIC8+XG4gICAgICAgICAgPGgxIGNsYXNzTmFtZT1cIkFwcC10aXRsZVwiPldlbGNvbWUgdG8gUmVhY3Q8L2gxPlxuICAgICAgICA8L2hlYWRlcj5cbiAgICAgICAgPHAgY2xhc3NOYW1lPVwiQXBwLWludHJvXCI+XG4gICAgICAgICAgVG8gZ2V0IHN0YXJ0ZWQsIGVkaXQgPGNvZGU+c3JjL0FwcC5qczwvY29kZT4gYW5kIHNhdmUgdG8gcmVsb2FkLlxuICAgICAgICA8L3A+XG4gICAgICA8L2Rpdj5cbiAgICApO1xuICB9XG59XG5cbmV4cG9ydCBkZWZhdWx0IEFwcDtcbiJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCIsImNvbmZpZyI6eyJpc1NlcnZlciI6dHJ1ZX19LCJyZWFjdHRzIjp7ImRpc3BsYXlOYW1lIjoiUmVhY3QgVHlwZXNjcmlwdCIsInRhZ2xpbmUiOiJBIEphdmFTY3JpcHQgbGlicmFyeSBmb3IgYnVpbGRpbmcgdXNlciBpbnRlcmZhY2VzIiwia2V5IjoicmVhY3R0cyIsImVudHJ5cG9pbnQiOiJzcmMvaW5kZXgudHN4IiwiZXh0IjoidHN4IiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6dHJ1ZSwiaGFzRXZhbCI6ZmFsc2UsImhhc0xhbmd1YWdlU2VydmVyIjpmYWxzZSwiaGVhZGVyIjoiUmVhY3QgMTYuNC4wLCBub2RlIHY5LjcuMSBsaW51eC9hbWQ2NCIsImNhdGVnb3J5IjoiRnJhbWV3b3JrIiwiaWNvbiI6Imh0dHBzOi8vbG9nb3MtLXR1cmJpby5yZXBsLmNvL3JlYWN0dHMuc3ZnIiwicHJvamVjdF90ZW1wbGF0ZSI6W3sibmFtZSI6InBhY2thZ2UuanNvbiIsImNvbnRlbnQiOiJ7XG4gIFwibmFtZVwiOiBcInJ1bm5lclwiLFxuICBcInZlcnNpb25cIjogXCIwLjEuMFwiLFxuICBcInByaXZhdGVcIjogdHJ1ZSxcbiAgXCJkZXBlbmRlbmNpZXNcIjoge1xuICAgIFwicmVhY3RcIjogXCJeMTYuNC4wXCIsXG4gICAgXCJyZWFjdC1kb21cIjogXCJeMTYuNC4wXCIsXG4gICAgXCJyZWFjdC1zY3JpcHRzLXRzXCI6IFwiMi4xNi4wXCJcbiAgfSxcbiAgXCJzY3JpcHRzXCI6IHtcbiAgICBcInN0YXJ0XCI6IFwicmVhY3Qtc2NyaXB0cy10cyBzdGFydFwiLFxuICAgIFwiYnVpbGRcIjogXCJyZWFjdC1zY3JpcHRzLXRzIGJ1aWxkXCIsXG4gICAgXCJ0ZXN0XCI6IFwicmVhY3Qtc2NyaXB0cy10cyB0ZXN0IC0tZW52PWpzZG9tXCIsXG4gICAgXCJlamVjdFwiOiBcInJlYWN0LXNjcmlwdHMtdHMgZWplY3RcIlxuICB9LFxuICBcImRldkRlcGVuZGVuY2llc1wiOiB7XG4gICAgXCJAdHlwZXMvamVzdFwiOiBcIl4yMy4wLjBcIixcbiAgICBcIkB0eXBlcy9ub2RlXCI6IFwiXjEwLjMuMVwiLFxuICAgIFwiQHR5cGVzL3JlYWN0XCI6IFwiXjE2LjMuMTZcIixcbiAgICBcIkB0eXBlcy9yZWFjdC1kb21cIjogXCJeMTYuMC42XCIsXG4gICAgXCJ0eXBlc2NyaXB0XCI6IFwiXjIuOS4xXCJcbiAgfVxufVxuIn0seyJuYW1lIjoiaW1hZ2VzLmQudHMiLCJjb250ZW50IjoiZGVjbGFyZSBtb2R1bGUgJyouc3ZnJ1xuZGVjbGFyZSBtb2R1bGUgJyoucG5nJ1xuZGVjbGFyZSBtb2R1bGUgJyouanBnJ1xuIn0seyJuYW1lIjoic3JjL2luZGV4LmNzcyIsImNvbnRlbnQiOiJib2R5IHtcbiAgbWFyZ2luOiAwO1xuICBwYWRkaW5nOiAwO1xuICBmb250LWZhbWlseTogc2Fucy1zZXJpZjtcbn1cbiJ9LHsibmFtZSI6InNyYy9BcHAuY3NzIiwiY29udGVudCI6Ii5BcHAge1xuICB0ZXh0LWFsaWduOiBjZW50ZXI7XG59XG5cbi5BcHAtbG9nbyB7XG4gIGFuaW1hdGlvbjogQXBwLWxvZ28tc3BpbiBpbmZpbml0ZSAyMHMgbGluZWFyO1xuICBoZWlnaHQ6IDgwcHg7XG59XG5cbi5BcHAtaGVhZGVyIHtcbiAgYmFja2dyb3VuZC1jb2xvcjogIzIyMjtcbiAgaGVpZ2h0OiAxNTBweDtcbiAgcGFkZGluZzogMjBweDtcbiAgY29sb3I6IHdoaXRlO1xufVxuXG4uQXBwLXRpdGxlIHtcbiAgZm9udC1zaXplOiAxLjVlbTtcbn1cblxuLkFwcC1pbnRybyB7XG4gIGZvbnQtc2l6ZTogbGFyZ2U7XG59XG5cbkBrZXlmcmFtZXMgQXBwLWxvZ28tc3BpbiB7XG4gIGZyb20geyB0cmFuc2Zvcm06IHJvdGF0ZSgwZGVnKTsgfVxuICB0byB7IHRyYW5zZm9ybTogcm90YXRlKDM2MGRlZyk7IH1cbn1cbiJ9LHsibmFtZSI6InNyYy9pbmRleC50c3giLCJjb250ZW50IjoiaW1wb3J0ICogYXMgUmVhY3QgZnJvbSAncmVhY3QnO1xuaW1wb3J0ICogYXMgUmVhY3RET00gZnJvbSAncmVhY3QtZG9tJztcbmltcG9ydCBBcHAgZnJvbSAnLi9BcHAnO1xuaW1wb3J0ICcuL2luZGV4LmNzcyc7XG5cblJlYWN0RE9NLnJlbmRlcihcbiAgPEFwcCAvPixcbiAgZG9jdW1lbnQuZ2V0RWxlbWVudEJ5SWQoJ3Jvb3QnKSBhcyBIVE1MRWxlbWVudFxuKTtcbiJ9LHsibmFtZSI6InNyYy9sb2dvLnN2ZyIsImNvbnRlbnQiOiI8c3ZnIHhtbG5zPVwiaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmdcIiB2aWV3Qm94PVwiMCAwIDg0MS45IDU5NS4zXCI+XG4gICAgPGcgZmlsbD1cIiM2MURBRkJcIj5cbiAgICAgICAgPHBhdGggZD1cIk02NjYuMyAyOTYuNWMwLTMyLjUtNDAuNy02My4zLTEwMy4xLTgyLjQgMTQuNC02My42IDgtMTE0LjItMjAuMi0xMzAuNC02LjUtMy44LTE0LjEtNS42LTIyLjQtNS42djIyLjNjNC42IDAgOC4zLjkgMTEuNCAyLjYgMTMuNiA3LjggMTkuNSAzNy41IDE0LjkgNzUuNy0xLjEgOS40LTIuOSAxOS4zLTUuMSAyOS40LTE5LjYtNC44LTQxLTguNS02My41LTEwLjktMTMuNS0xOC41LTI3LjUtMzUuMy00MS42LTUwIDMyLjYtMzAuMyA2My4yLTQ2LjkgODQtNDYuOVY3OGMtMjcuNSAwLTYzLjUgMTkuNi05OS45IDUzLjYtMzYuNC0zMy44LTcyLjQtNTMuMi05OS45LTUzLjJ2MjIuM2MyMC43IDAgNTEuNCAxNi41IDg0IDQ2LjYtMTQgMTQuNy0yOCAzMS40LTQxLjMgNDkuOS0yMi42IDIuNC00NCA2LjEtNjMuNiAxMS0yLjMtMTAtNC0xOS43LTUuMi0yOS00LjctMzguMiAxLjEtNjcuOSAxNC42LTc1LjggMy0xLjggNi45LTIuNiAxMS41LTIuNlY3OC41Yy04LjQgMC0xNiAxLjgtMjIuNiA1LjYtMjguMSAxNi4yLTM0LjQgNjYuNy0xOS45IDEzMC4xLTYyLjIgMTkuMi0xMDIuNyA0OS45LTEwMi43IDgyLjMgMCAzMi41IDQwLjcgNjMuMyAxMDMuMSA4Mi40LTE0LjQgNjMuNi04IDExNC4yIDIwLjIgMTMwLjQgNi41IDMuOCAxNC4xIDUuNiAyMi41IDUuNiAyNy41IDAgNjMuNS0xOS42IDk5LjktNTMuNiAzNi40IDMzLjggNzIuNCA1My4yIDk5LjkgNTMuMiA4LjQgMCAxNi0xLjggMjIuNi01LjYgMjguMS0xNi4yIDM0LjQtNjYuNyAxOS45LTEzMC4xIDYyLTE5LjEgMTAyLjUtNDkuOSAxMDIuNS04Mi4zem0tMTMwLjItNjYuN2MtMy43IDEyLjktOC4zIDI2LjItMTMuNSAzOS41LTQuMS04LTguNC0xNi0xMy4xLTI0LTQuNi04LTkuNS0xNS44LTE0LjQtMjMuNCAxNC4yIDIuMSAyNy45IDQuNyA0MSA3Ljl6bS00NS44IDEwNi41Yy03LjggMTMuNS0xNS44IDI2LjMtMjQuMSAzOC4yLTE0LjkgMS4zLTMwIDItNDUuMiAyLTE1LjEgMC0zMC4yLS43LTQ1LTEuOS04LjMtMTEuOS0xNi40LTI0LjYtMjQuMi0zOC03LjYtMTMuMS0xNC41LTI2LjQtMjAuOC0zOS44IDYuMi0xMy40IDEzLjItMjYuOCAyMC43LTM5LjkgNy44LTEzLjUgMTUuOC0yNi4zIDI0LjEtMzguMiAxNC45LTEuMyAzMC0yIDQ1LjItMiAxNS4xIDAgMzAuMi43IDQ1IDEuOSA4LjMgMTEuOSAxNi40IDI0LjYgMjQuMiAzOCA3LjYgMTMuMSAxNC41IDI2LjQgMjAuOCAzOS44LTYuMyAxMy40LTEzLjIgMjYuOC0yMC43IDM5Ljl6bTMyLjMtMTNjNS40IDEzLjQgMTAgMjYuOCAxMy44IDM5LjgtMTMuMSAzLjItMjYuOSA1LjktNDEuMiA4IDQuOS03LjcgOS44LTE1LjYgMTQuNC0yMy43IDQuNi04IDguOS0xNi4xIDEzLTI0LjF6TTQyMS4yIDQzMGMtOS4zLTkuNi0xOC42LTIwLjMtMjcuOC0zMiA5IC40IDE4LjIuNyAyNy41LjcgOS40IDAgMTguNy0uMiAyNy44LS43LTkgMTEuNy0xOC4zIDIyLjQtMjcuNSAzMnptLTc0LjQtNTguOWMtMTQuMi0yLjEtMjcuOS00LjctNDEtNy45IDMuNy0xMi45IDguMy0yNi4yIDEzLjUtMzkuNSA0LjEgOCA4LjQgMTYgMTMuMSAyNCA0LjcgOCA5LjUgMTUuOCAxNC40IDIzLjR6TTQyMC43IDE2M2M5LjMgOS42IDE4LjYgMjAuMyAyNy44IDMyLTktLjQtMTguMi0uNy0yNy41LS43LTkuNCAwLTE4LjcuMi0yNy44LjcgOS0xMS43IDE4LjMtMjIuNCAyNy41LTMyem0tNzQgNTguOWMtNC45IDcuNy05LjggMTUuNi0xNC40IDIzLjctNC42IDgtOC45IDE2LTEzIDI0LTUuNC0xMy40LTEwLTI2LjgtMTMuOC0zOS44IDEzLjEtMy4xIDI2LjktNS44IDQxLjItNy45em0tOTAuNSAxMjUuMmMtMzUuNC0xNS4xLTU4LjMtMzQuOS01OC4zLTUwLjYgMC0xNS43IDIyLjktMzUuNiA1OC4zLTUwLjYgOC42LTMuNyAxOC03IDI3LjctMTAuMSA1LjcgMTkuNiAxMy4yIDQwIDIyLjUgNjAuOS05LjIgMjAuOC0xNi42IDQxLjEtMjIuMiA2MC42LTkuOS0zLjEtMTkuMy02LjUtMjgtMTAuMnpNMzEwIDQ5MGMtMTMuNi03LjgtMTkuNS0zNy41LTE0LjktNzUuNyAxLjEtOS40IDIuOS0xOS4zIDUuMS0yOS40IDE5LjYgNC44IDQxIDguNSA2My41IDEwLjkgMTMuNSAxOC41IDI3LjUgMzUuMyA0MS42IDUwLTMyLjYgMzAuMy02My4yIDQ2LjktODQgNDYuOS00LjUtLjEtOC4zLTEtMTEuMy0yLjd6bTIzNy4yLTc2LjJjNC43IDM4LjItMS4xIDY3LjktMTQuNiA3NS44LTMgMS44LTYuOSAyLjYtMTEuNSAyLjYtMjAuNyAwLTUxLjQtMTYuNS04NC00Ni42IDE0LTE0LjcgMjgtMzEuNCA0MS4zLTQ5LjkgMjIuNi0yLjQgNDQtNi4xIDYzLjYtMTEgMi4zIDEwLjEgNC4xIDE5LjggNS4yIDI5LjF6bTM4LjUtNjYuN2MtOC42IDMuNy0xOCA3LTI3LjcgMTAuMS01LjctMTkuNi0xMy4yLTQwLTIyLjUtNjAuOSA5LjItMjAuOCAxNi42LTQxLjEgMjIuMi02MC42IDkuOSAzLjEgMTkuMyA2LjUgMjguMSAxMC4yIDM1LjQgMTUuMSA1OC4zIDM0LjkgNTguMyA1MC42LS4xIDE1LjctMjMgMzUuNi01OC40IDUwLjZ6TTMyMC44IDc4LjR6XCIvPlxuICAgICAgICA8Y2lyY2xlIGN4PVwiNDIwLjlcIiBjeT1cIjI5Ni41XCIgcj1cIjQ1LjdcIi8+XG4gICAgICAgIDxwYXRoIGQ9XCJNNTIwLjUgNzguMXpcIi8+XG4gICAgPC9nPlxuPC9zdmc+XG4ifSx7Im5hbWUiOiJzcmMvQXBwLnRzeCIsImNvbnRlbnQiOiJpbXBvcnQgKiBhcyBSZWFjdCBmcm9tICdyZWFjdCc7XG5pbXBvcnQgJy4vQXBwLmNzcyc7XG5cbmltcG9ydCBsb2dvIGZyb20gJy4vbG9nby5zdmcnO1xuXG5jbGFzcyBBcHAgZXh0ZW5kcyBSZWFjdC5Db21wb25lbnQge1xuICBwdWJsaWMgcmVuZGVyKCkge1xuICAgIHJldHVybiAoXG4gICAgICA8ZGl2IGNsYXNzTmFtZT1cIkFwcFwiPlxuICAgICAgICA8aGVhZGVyIGNsYXNzTmFtZT1cIkFwcC1oZWFkZXJcIj5cbiAgICAgICAgICA8aW1nIHNyYz17bG9nb30gY2xhc3NOYW1lPVwiQXBwLWxvZ29cIiBhbHQ9XCJsb2dvXCIgLz5cbiAgICAgICAgICA8aDEgY2xhc3NOYW1lPVwiQXBwLXRpdGxlXCI+V2VsY29tZSB0byBSZWFjdDwvaDE+XG4gICAgICAgIDwvaGVhZGVyPlxuICAgICAgICA8cCBjbGFzc05hbWU9XCJBcHAtaW50cm9cIj5cbiAgICAgICAgICBUbyBnZXQgc3RhcnRlZCwgZWRpdCA8Y29kZT5zcmMvQXBwLnRzeDwvY29kZT4gYW5kIHNhdmUgdG8gcmVsb2FkLlxuICAgICAgICA8L3A+XG4gICAgICA8L2Rpdj5cbiAgICApO1xuICB9XG59XG5cbmV4cG9ydCBkZWZhdWx0IEFwcDtcbiJ9LHsibmFtZSI6InRzY29uZmlnLmpzb24iLCJjb250ZW50Ijoie1xuICBcImNvbXBpbGVyT3B0aW9uc1wiOiB7XG4gICAgXCJiYXNlVXJsXCI6IFwiLlwiLFxuICAgIFwib3V0RGlyXCI6IFwiYnVpbGQvZGlzdFwiLFxuICAgIFwibW9kdWxlXCI6IFwiZXNuZXh0XCIsXG4gICAgXCJ0YXJnZXRcIjogXCJlczVcIixcbiAgICBcImxpYlwiOiBbXCJlczZcIiwgXCJkb21cIl0sXG4gICAgXCJzb3VyY2VNYXBcIjogdHJ1ZSxcbiAgICBcImFsbG93SnNcIjogdHJ1ZSxcbiAgICBcImpzeFwiOiBcInJlYWN0XCIsXG4gICAgXCJtb2R1bGVSZXNvbHV0aW9uXCI6IFwibm9kZVwiLFxuICAgIFwicm9vdERpclwiOiBcInNyY1wiLFxuICAgIFwiZm9yY2VDb25zaXN0ZW50Q2FzaW5nSW5GaWxlTmFtZXNcIjogdHJ1ZSxcbiAgICBcIm5vSW1wbGljaXRSZXR1cm5zXCI6IHRydWUsXG4gICAgXCJub0ltcGxpY2l0VGhpc1wiOiB0cnVlLFxuICAgIFwibm9JbXBsaWNpdEFueVwiOiB0cnVlLFxuICAgIFwic3RyaWN0TnVsbENoZWNrc1wiOiB0cnVlLFxuICAgIFwic3VwcHJlc3NJbXBsaWNpdEFueUluZGV4RXJyb3JzXCI6IHRydWUsXG4gICAgXCJub1VudXNlZExvY2Fsc1wiOiB0cnVlXG4gIH0sXG4gIFwiZXhjbHVkZVwiOiBbXG4gICAgXCJub2RlX21vZHVsZXNcIixcbiAgICBcImJ1aWxkXCIsXG4gICAgXCJzY3JpcHRzXCIsXG4gICAgXCJhY2NlcHRhbmNlLXRlc3RzXCIsXG4gICAgXCJ3ZWJwYWNrXCIsXG4gICAgXCJqZXN0XCIsXG4gICAgXCJzcmMvc2V0dXBUZXN0cy50c1wiXG4gIF1cbn1cbiJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCIsImNvbmZpZyI6eyJpc1NlcnZlciI6dHJ1ZX19LCJyZWFjdHJlIjp7ImRpc3BsYXlOYW1lIjoiUmVhY3QgUmVhc29uIiwidGFnbGluZSI6IlJlYXNvbiBiaW5kaW5ncyBmb3IgUmVhY3RKUyIsImtleSI6InJlYWN0cmUiLCJlbnRyeXBvaW50Ijoic3JjL2luZGV4LnJlIiwiZXh0IjoicmUiLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6dHJ1ZSwiaGFzTGlicmFyaWVzIjpmYWxzZSwiaGFzRXZhbCI6ZmFsc2UsImhhc0xhbmd1YWdlU2VydmVyIjp0cnVlLCJoZWFkZXIiOiJSZWFzb24gMy4xLjUsIG5vZGUgdjkuNy4xIGxpbnV4L2FtZDY0IiwiY2F0ZWdvcnkiOiJGcmFtZXdvcmsiLCJpY29uIjoiaHR0cHM6Ly9sb2dvcy0tdHVyYmlvLnJlcGwuY28vcmVhY3RyZS5zdmciLCJwcm9qZWN0X3RlbXBsYXRlIjpbeyJuYW1lIjoicGFja2FnZS5qc29uIiwiY29udGVudCI6IntcbiAgXCJuYW1lXCI6IFwiYXBwXCIsXG4gIFwidmVyc2lvblwiOiBcIjAuMS4wXCIsXG4gIFwicHJpdmF0ZVwiOiB0cnVlLFxuICBcImRlcGVuZGVuY2llc1wiOiB7XG4gICAgXCJyZWFjdFwiOiBcIl4xNi40LjBcIixcbiAgICBcInJlYWN0LWRvbVwiOiBcIl4xNi40LjBcIixcbiAgICBcInJlYXNvbi1zY3JpcHRzXCI6IFwiMC45LjBcIlxuICB9LFxuICBcInNjcmlwdHNcIjoge1xuICAgIFwic3RhcnRcIjogXCJyZWFjdC1zY3JpcHRzIHN0YXJ0XCIsXG4gICAgXCJidWlsZFwiOiBcInJlYWN0LXNjcmlwdHMgYnVpbGRcIixcbiAgICBcInRlc3RcIjogXCJyZWFjdC1zY3JpcHRzIHRlc3QgLS1lbnY9anNkb21cIixcbiAgICBcImVqZWN0XCI6IFwicmVhY3Qtc2NyaXB0cyBlamVjdFwiXG4gIH0sXG4gIFwiZGV2RGVwZW5kZW5jaWVzXCI6IHtcbiAgICBcIkBnbGVubnNsL2JzLWplc3RcIjogXCJeMC40LjJcIixcbiAgICBcInJlYXNvbi1yZWFjdFwiOiBcIl4wLjQuMlwiXG4gIH1cbn1cbiJ9LHsibmFtZSI6InNyYy9pbmRleC5yZSIsImNvbnRlbnQiOiJbJWJzLnJhdyB7fHJlcXVpcmUoJy4vaW5kZXguY3NzJyl8fV07XG5cblJlYWN0RE9NUmUucmVuZGVyVG9FbGVtZW50V2l0aElkKFxuICA8QXBwIG1lc3NhZ2U9XCJXZWxjb21lIHRvIFJlYWN0IGFuZCBSZWFzb25cIiAvPixcbiAgXCJyb290XCIsXG4pOyJ9LHsibmFtZSI6InNyYy9pbmRleC5jc3MiLCJjb250ZW50IjoiYm9keSB7XG4gIG1hcmdpbjogMDtcbiAgcGFkZGluZzogMDtcbiAgZm9udC1mYW1pbHk6IHNhbnMtc2VyaWY7XG59XG4ifSx7Im5hbWUiOiJzcmMvQXBwLmNzcyIsImNvbnRlbnQiOiIuQXBwIHtcbiAgdGV4dC1hbGlnbjogY2VudGVyO1xufVxuXG4uQXBwLWxvZ28ge1xuICBhbmltYXRpb246IEFwcC1sb2dvLXNwaW4gaW5maW5pdGUgMjBzIGxpbmVhcjtcbiAgaGVpZ2h0OiA4MHB4O1xufVxuXG4uQXBwLWhlYWRlciB7XG4gIGJhY2tncm91bmQtY29sb3I6ICMyMjI7XG4gIGhlaWdodDogMTUwcHg7XG4gIHBhZGRpbmc6IDIwcHg7XG4gIGNvbG9yOiB3aGl0ZTtcbn1cblxuLkFwcC1pbnRybyB7XG4gIGZvbnQtc2l6ZTogbGFyZ2U7XG59XG5cbkBrZXlmcmFtZXMgQXBwLWxvZ28tc3BpbiB7XG4gIGZyb20ge1xuICAgIHRyYW5zZm9ybTogcm90YXRlKDBkZWcpO1xuICB9XG4gIHRvIHtcbiAgICB0cmFuc2Zvcm06IHJvdGF0ZSgzNjBkZWcpO1xuICB9XG59XG4ifSx7Im5hbWUiOiJzcmMvbG9nby5zdmciLCJjb250ZW50IjoiPHN2ZyB4bWxucz1cImh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnXCIgdmlld0JveD1cIjAgMCA4NDEuOSA1OTUuM1wiPlxuICAgIDxnIGZpbGw9XCIjREI0RDNGXCI+XG4gICAgICAgIDxwYXRoIGQ9XCJNNjY2LjMgMjk2LjVjMC0zMi41LTQwLjctNjMuMy0xMDMuMS04Mi40IDE0LjQtNjMuNiA4LTExNC4yLTIwLjItMTMwLjQtNi41LTMuOC0xNC4xLTUuNi0yMi40LTUuNnYyMi4zYzQuNiAwIDguMy45IDExLjQgMi42IDEzLjYgNy44IDE5LjUgMzcuNSAxNC45IDc1LjctMS4xIDkuNC0yLjkgMTkuMy01LjEgMjkuNC0xOS42LTQuOC00MS04LjUtNjMuNS0xMC45LTEzLjUtMTguNS0yNy41LTM1LjMtNDEuNi01MCAzMi42LTMwLjMgNjMuMi00Ni45IDg0LTQ2LjlWNzhjLTI3LjUgMC02My41IDE5LjYtOTkuOSA1My42LTM2LjQtMzMuOC03Mi40LTUzLjItOTkuOS01My4ydjIyLjNjMjAuNyAwIDUxLjQgMTYuNSA4NCA0Ni42LTE0IDE0LjctMjggMzEuNC00MS4zIDQ5LjktMjIuNiAyLjQtNDQgNi4xLTYzLjYgMTEtMi4zLTEwLTQtMTkuNy01LjItMjktNC43LTM4LjIgMS4xLTY3LjkgMTQuNi03NS44IDMtMS44IDYuOS0yLjYgMTEuNS0yLjZWNzguNWMtOC40IDAtMTYgMS44LTIyLjYgNS42LTI4LjEgMTYuMi0zNC40IDY2LjctMTkuOSAxMzAuMS02Mi4yIDE5LjItMTAyLjcgNDkuOS0xMDIuNyA4Mi4zIDAgMzIuNSA0MC43IDYzLjMgMTAzLjEgODIuNC0xNC40IDYzLjYtOCAxMTQuMiAyMC4yIDEzMC40IDYuNSAzLjggMTQuMSA1LjYgMjIuNSA1LjYgMjcuNSAwIDYzLjUtMTkuNiA5OS45LTUzLjYgMzYuNCAzMy44IDcyLjQgNTMuMiA5OS45IDUzLjIgOC40IDAgMTYtMS44IDIyLjYtNS42IDI4LjEtMTYuMiAzNC40LTY2LjcgMTkuOS0xMzAuMSA2Mi0xOS4xIDEwMi41LTQ5LjkgMTAyLjUtODIuM3ptLTEzMC4yLTY2LjdjLTMuNyAxMi45LTguMyAyNi4yLTEzLjUgMzkuNS00LjEtOC04LjQtMTYtMTMuMS0yNC00LjYtOC05LjUtMTUuOC0xNC40LTIzLjQgMTQuMiAyLjEgMjcuOSA0LjcgNDEgNy45em0tNDUuOCAxMDYuNWMtNy44IDEzLjUtMTUuOCAyNi4zLTI0LjEgMzguMi0xNC45IDEuMy0zMCAyLTQ1LjIgMi0xNS4xIDAtMzAuMi0uNy00NS0xLjktOC4zLTExLjktMTYuNC0yNC42LTI0LjItMzgtNy42LTEzLjEtMTQuNS0yNi40LTIwLjgtMzkuOCA2LjItMTMuNCAxMy4yLTI2LjggMjAuNy0zOS45IDcuOC0xMy41IDE1LjgtMjYuMyAyNC4xLTM4LjIgMTQuOS0xLjMgMzAtMiA0NS4yLTIgMTUuMSAwIDMwLjIuNyA0NSAxLjkgOC4zIDExLjkgMTYuNCAyNC42IDI0LjIgMzggNy42IDEzLjEgMTQuNSAyNi40IDIwLjggMzkuOC02LjMgMTMuNC0xMy4yIDI2LjgtMjAuNyAzOS45em0zMi4zLTEzYzUuNCAxMy40IDEwIDI2LjggMTMuOCAzOS44LTEzLjEgMy4yLTI2LjkgNS45LTQxLjIgOCA0LjktNy43IDkuOC0xNS42IDE0LjQtMjMuNyA0LjYtOCA4LjktMTYuMSAxMy0yNC4xek00MjEuMiA0MzBjLTkuMy05LjYtMTguNi0yMC4zLTI3LjgtMzIgOSAuNCAxOC4yLjcgMjcuNS43IDkuNCAwIDE4LjctLjIgMjcuOC0uNy05IDExLjctMTguMyAyMi40LTI3LjUgMzJ6bS03NC40LTU4LjljLTE0LjItMi4xLTI3LjktNC43LTQxLTcuOSAzLjctMTIuOSA4LjMtMjYuMiAxMy41LTM5LjUgNC4xIDggOC40IDE2IDEzLjEgMjQgNC43IDggOS41IDE1LjggMTQuNCAyMy40ek00MjAuNyAxNjNjOS4zIDkuNiAxOC42IDIwLjMgMjcuOCAzMi05LS40LTE4LjItLjctMjcuNS0uNy05LjQgMC0xOC43LjItMjcuOC43IDktMTEuNyAxOC4zLTIyLjQgMjcuNS0zMnptLTc0IDU4LjljLTQuOSA3LjctOS44IDE1LjYtMTQuNCAyMy43LTQuNiA4LTguOSAxNi0xMyAyNC01LjQtMTMuNC0xMC0yNi44LTEzLjgtMzkuOCAxMy4xLTMuMSAyNi45LTUuOCA0MS4yLTcuOXptLTkwLjUgMTI1LjJjLTM1LjQtMTUuMS01OC4zLTM0LjktNTguMy01MC42IDAtMTUuNyAyMi45LTM1LjYgNTguMy01MC42IDguNi0zLjcgMTgtNyAyNy43LTEwLjEgNS43IDE5LjYgMTMuMiA0MCAyMi41IDYwLjktOS4yIDIwLjgtMTYuNiA0MS4xLTIyLjIgNjAuNi05LjktMy4xLTE5LjMtNi41LTI4LTEwLjJ6TTMxMCA0OTBjLTEzLjYtNy44LTE5LjUtMzcuNS0xNC45LTc1LjcgMS4xLTkuNCAyLjktMTkuMyA1LjEtMjkuNCAxOS42IDQuOCA0MSA4LjUgNjMuNSAxMC45IDEzLjUgMTguNSAyNy41IDM1LjMgNDEuNiA1MC0zMi42IDMwLjMtNjMuMiA0Ni45LTg0IDQ2LjktNC41LS4xLTguMy0xLTExLjMtMi43em0yMzcuMi03Ni4yYzQuNyAzOC4yLTEuMSA2Ny45LTE0LjYgNzUuOC0zIDEuOC02LjkgMi42LTExLjUgMi42LTIwLjcgMC01MS40LTE2LjUtODQtNDYuNiAxNC0xNC43IDI4LTMxLjQgNDEuMy00OS45IDIyLjYtMi40IDQ0LTYuMSA2My42LTExIDIuMyAxMC4xIDQuMSAxOS44IDUuMiAyOS4xem0zOC41LTY2LjdjLTguNiAzLjctMTggNy0yNy43IDEwLjEtNS43LTE5LjYtMTMuMi00MC0yMi41LTYwLjkgOS4yLTIwLjggMTYuNi00MS4xIDIyLjItNjAuNiA5LjkgMy4xIDE5LjMgNi41IDI4LjEgMTAuMiAzNS40IDE1LjEgNTguMyAzNC45IDU4LjMgNTAuNi0uMSAxNS43LTIzIDM1LjYtNTguNCA1MC42ek0zMjAuOCA3OC40elwiLz5cbiAgICAgICAgPGNpcmNsZSBjeD1cIjQyMC45XCIgY3k9XCIyOTYuNVwiIHI9XCI0NS43XCIvPlxuICAgICAgICA8cGF0aCBkPVwiTTUyMC41IDc4LjF6XCIvPlxuICAgIDwvZz5cbjwvc3ZnPlxuIn0seyJuYW1lIjoic3JjL0FwcC5yZSIsImNvbnRlbnQiOiJbJWJzLnJhdyB7fHJlcXVpcmUoJy4vQXBwLmNzcycpfH1dO1xuXG5bQGJzLm1vZHVsZV0gZXh0ZXJuYWwgbG9nbyA6IHN0cmluZyA9IFwiLi9sb2dvLnN2Z1wiO1xuXG5sZXQgY29tcG9uZW50ID0gUmVhc29uUmVhY3Quc3RhdGVsZXNzQ29tcG9uZW50KFwiQXBwXCIpO1xuXG5sZXQgbWFrZSA9ICh+bWVzc2FnZSwgX2NoaWxkcmVuKSA9PiB7XG4gIC4uLmNvbXBvbmVudCxcbiAgcmVuZGVyOiBfc2VsZiA9PlxuICAgIDxkaXYgY2xhc3NOYW1lPVwiQXBwXCI+XG4gICAgICA8ZGl2IGNsYXNzTmFtZT1cIkFwcC1oZWFkZXJcIj5cbiAgICAgICAgPGltZyBzcmM9bG9nbyBjbGFzc05hbWU9XCJBcHAtbG9nb1wiIGFsdD1cImxvZ29cIiAvPlxuICAgICAgICA8aDI+IChSZWFzb25SZWFjdC5zdHJpbmcobWVzc2FnZSkpIDwvaDI+XG4gICAgICA8L2Rpdj5cbiAgICAgIDxwIGNsYXNzTmFtZT1cIkFwcC1pbnRyb1wiPlxuICAgICAgICAoUmVhc29uUmVhY3Quc3RyaW5nKFwiVG8gZ2V0IHN0YXJ0ZWQsIGVkaXRcIikpXG4gICAgICAgIDxjb2RlPiAoUmVhc29uUmVhY3Quc3RyaW5nKFwiIHNyYy9BcHAucmUgXCIpKSA8L2NvZGU+XG4gICAgICAgIChSZWFzb25SZWFjdC5zdHJpbmcoXCJhbmQgc2F2ZSB0byByZWxvYWQuXCIpKVxuICAgICAgPC9wPlxuICAgIDwvZGl2Pixcbn07XG4ifSx7Im5hbWUiOiJwdWJsaWMvaW5kZXguaHRtbCIsImNvbnRlbnQiOiI8IURPQ1RZUEUgaHRtbD5cbjxodG1sIGxhbmc9XCJlblwiPlxuICA8aGVhZD5cbiAgICA8bWV0YSBjaGFyc2V0PVwidXRmLThcIj5cbiAgICA8bWV0YSBuYW1lPVwidmlld3BvcnRcIiBjb250ZW50PVwid2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEsIHNocmluay10by1maXQ9bm9cIj5cbiAgICA8bWV0YSBuYW1lPVwidGhlbWUtY29sb3JcIiBjb250ZW50PVwiIzAwMDAwMFwiPlxuICAgIDwhLS1cbiAgICAgIG1hbmlmZXN0Lmpzb24gcHJvdmlkZXMgbWV0YWRhdGEgdXNlZCB3aGVuIHlvdXIgd2ViIGFwcCBpcyBhZGRlZCB0byB0aGVcbiAgICAgIGhvbWVzY3JlZW4gb24gQW5kcm9pZC4gU2VlIGh0dHBzOi8vZGV2ZWxvcGVycy5nb29nbGUuY29tL3dlYi9mdW5kYW1lbnRhbHMvZW5nYWdlLWFuZC1yZXRhaW4vd2ViLWFwcC1tYW5pZmVzdC9cbiAgICAtLT5cbiAgICA8bGluayByZWw9XCJtYW5pZmVzdFwiIGhyZWY9XCIlUFVCTElDX1VSTCUvbWFuaWZlc3QuanNvblwiPlxuICAgIDxsaW5rIHJlbD1cInNob3J0Y3V0IGljb25cIiBocmVmPVwiJVBVQkxJQ19VUkwlL2Zhdmljb24uaWNvXCI+XG4gICAgPCEtLVxuICAgICAgTm90aWNlIHRoZSB1c2Ugb2YgJVBVQkxJQ19VUkwlIGluIHRoZSB0YWdzIGFib3ZlLlxuICAgICAgSXQgd2lsbCBiZSByZXBsYWNlZCB3aXRoIHRoZSBVUkwgb2YgdGhlIGBwdWJsaWNgIGZvbGRlciBkdXJpbmcgdGhlIGJ1aWxkLlxuICAgICAgT25seSBmaWxlcyBpbnNpZGUgdGhlIGBwdWJsaWNgIGZvbGRlciBjYW4gYmUgcmVmZXJlbmNlZCBmcm9tIHRoZSBIVE1MLlxuXG4gICAgICBVbmxpa2UgXCIvZmF2aWNvbi5pY29cIiBvciBcImZhdmljb24uaWNvXCIsIFwiJVBVQkxJQ19VUkwlL2Zhdmljb24uaWNvXCIgd2lsbFxuICAgICAgd29yayBjb3JyZWN0bHkgYm90aCB3aXRoIGNsaWVudC1zaWRlIHJvdXRpbmcgYW5kIGEgbm9uLXJvb3QgcHVibGljIFVSTC5cbiAgICAgIExlYXJuIGhvdyB0byBjb25maWd1cmUgYSBub24tcm9vdCBwdWJsaWMgVVJMIGJ5IHJ1bm5pbmcgYG5wbSBydW4gYnVpbGRgLlxuICAgIC0tPlxuICAgIDx0aXRsZT5SZWFjdCBBcHA8L3RpdGxlPlxuICA8L2hlYWQ+XG4gIDxib2R5PlxuICAgIDxub3NjcmlwdD5cbiAgICAgIFlvdSBuZWVkIHRvIGVuYWJsZSBKYXZhU2NyaXB0IHRvIHJ1biB0aGlzIGFwcC5cbiAgICA8L25vc2NyaXB0PlxuICAgIDxkaXYgaWQ9XCJyb290XCI+PC9kaXY+XG4gICAgPCEtLVxuICAgICAgVGhpcyBIVE1MIGZpbGUgaXMgYSB0ZW1wbGF0ZS5cbiAgICAgIElmIHlvdSBvcGVuIGl0IGRpcmVjdGx5IGluIHRoZSBicm93c2VyLCB5b3Ugd2lsbCBzZWUgYW4gZW1wdHkgcGFnZS5cblxuICAgICAgWW91IGNhbiBhZGQgd2ViZm9udHMsIG1ldGEgdGFncywgb3IgYW5hbHl0aWNzIHRvIHRoaXMgZmlsZS5cbiAgICAgIFRoZSBidWlsZCBzdGVwIHdpbGwgcGxhY2UgdGhlIGJ1bmRsZWQgc2NyaXB0cyBpbnRvIHRoZSA8Ym9keT4gdGFnLlxuXG4gICAgICBUbyBiZWdpbiB0aGUgZGV2ZWxvcG1lbnQsIHJ1biBgbnBtIHN0YXJ0YCBvciBgeWFybiBzdGFydGAuXG4gICAgICBUbyBjcmVhdGUgYSBwcm9kdWN0aW9uIGJ1bmRsZSwgdXNlIGBucG0gcnVuIGJ1aWxkYCBvciBgeWFybiBidWlsZGAuXG4gICAgLS0+XG4gIDwvYm9keT5cbjwvaHRtbD5cbiJ9LHsibmFtZSI6ImJzY29uZmlnLmpzb24iLCJjb250ZW50Ijoie1xuICBcIm5hbWVcIjogXCJyZWFzb24tc2NyaXB0c1wiLFxuICBcInNvdXJjZXNcIjogW1wic3JjXCJdLFxuICBcImJzLWRlcGVuZGVuY2llc1wiOiBbXCJyZWFzb24tcmVhY3RcIiwgXCJAZ2xlbm5zbC9icy1qZXN0XCJdLFxuICBcInJlYXNvblwiOiB7XG4gICAgXCJyZWFjdC1qc3hcIjogMlxuICB9LFxuICBcImJzYy1mbGFnc1wiOiBbXCItYnMtc3VwZXItZXJyb3JzXCJdLFxuICBcInJlZm10XCI6IDMsXG4gIFwicGFja2FnZS1zcGVjc1wiOiB7XG4gICAgXCJtb2R1bGVcIjogXCJlczZcIixcbiAgICBcImluLXNvdXJjZVwiOiB0cnVlXG4gIH0sXG4gIFwic3VmZml4XCI6IFwiLmJzLmpzXCJcbn1cbiJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCIsImNvbmZpZyI6eyJpc1NlcnZlciI6dHJ1ZX19LCJmbG93Ijp7ImRpc3BsYXlOYW1lIjoiRmxvdyIsInRhZ2xpbmUiOiJBIHN0YXRpYyB0eXBlIGNoZWNrZXIgZm9yIEphdmFTY3JpcHQiLCJrZXkiOiJmbG93IiwiZW50cnlwb2ludCI6InNyYy9pbmRleC5qcyIsImV4dCI6ImpzIiwiaGFzTGludCI6ZmFsc2UsImhhc1VuaXRUZXN0cyI6ZmFsc2UsImhhc1Byb2plY3RNb2RlIjp0cnVlLCJoYXNGb3JtYXQiOnRydWUsImhhc0xpYnJhcmllcyI6dHJ1ZSwiaGFzRXZhbCI6ZmFsc2UsImhhc0xhbmd1YWdlU2VydmVyIjp0cnVlLCJoZWFkZXIiOiJub2RlIHY5LjcuMSBsaW51eC9hbWQ2NCIsImNhdGVnb3J5IjoiSGlkZGVuIiwiaWNvbiI6Imh0dHBzOi8vcmVwbC5pdC9wdWJsaWMvaW1hZ2VzL2xhbmd1YWdlcy9mbG93LnN2ZyIsInByb2plY3RfdGVtcGxhdGUiOlt7Im5hbWUiOiJzcmMvaW5kZXguanMiLCJjb250ZW50IjoiLy8gQGZsb3dcbmNvbnN0IG1hdGggPSByZXF1aXJlKCcuL21hdGgnKTtcblxuY29uc29sZS5sb2cobWF0aC5zcXVhcmUoNykpO1xuY29uc29sZS5sb2cobWF0aC5hZGQoXCJhXCIsIFwiYlwiKSk7XG4ifSx7Im5hbWUiOiJzcmMvbWF0aC5qcyIsImNvbnRlbnQiOiIvLyBAZmxvd1xuXG5leHBvcnRzLmFkZCA9ICh4OiBudW1iZXIsIHk6IG51bWJlcikgPT4geCArIHk7XG5leHBvcnRzLnNxdWFyZSA9ICh4OiBudW1iZXIpID0+IHggKiB4O1xuIn0seyJuYW1lIjoicGFja2FnZS5qc29uIiwiY29udGVudCI6IntcbiAgXCJuYW1lXCI6IFwibXktcHJvamVjdFwiLFxuICBcIm1haW5cIjogXCJzcmMvaW5kZXguanNcIixcbiAgXCJzY3JpcHRzXCI6IHtcbiAgICBcInN0YXJ0XCI6IFwiZmxvdy1ub2RlIC1hIHNyYy9pbmRleC5qc1wiXG4gIH0sXG4gIFwiZGVwZW5kZW5jaWVzXCI6IHtcbiAgICBcImZsb3ctcmVtb3ZlLXR5cGVzXCI6IFwiXjEuMi4zXCIsXG4gICAgXCJmbG93LWJpblwiOiBcIl4wLjc1LjBcIlxuICB9XG59In0seyJuYW1lIjoiLmZsb3djb25maWciLCJjb250ZW50IjoiW2lnbm9yZV1cblxuW2luY2x1ZGVdXG5cbltsaWJzXVxuXG5bbGludHNdXG5cbltvcHRpb25zXVxuXG5bc3RyaWN0XSJ9XSwibWF0Y2hpbmdzIjpbWyJ7IiwifSJdLFsiKCIsIikiXSxbIlsiLCJdIl1dLCJlbmdpbmUiOiJnb3ZhbCJ9LCJiYXNoIjp7ImRpc3BsYXlOYW1lIjoiYmFzaCIsInRhZ2xpbmUiOiJUaGUgY2xhc3NpYyBVbml4IHNoZWxsIiwia2V5IjoiYmFzaCIsImVudHJ5cG9pbnQiOiJtYWluLnNoIiwiZXh0Ijoic2giLCJoYXNMaW50IjpmYWxzZSwiaGFzVW5pdFRlc3RzIjpmYWxzZSwiaGFzUHJvamVjdE1vZGUiOnRydWUsImhhc0Zvcm1hdCI6dHJ1ZSwiaGFzTGlicmFyaWVzIjpmYWxzZSwiaGFzRXZhbCI6ZmFsc2UsImhhc0xhbmd1YWdlU2VydmVyIjpmYWxzZSwiaGVhZGVyIjoiR05VIGJhc2gsIHZlcnNpb24gNC40LjEyKDEpLXJlbGVhc2UgKHg4Nl82NC1wYy1saW51eC1nbnUpIiwiY2F0ZWdvcnkiOiJQcmFjdGljYWwiLCJpY29uIjoiaHR0cHM6Ly9yZXBsLml0L3B1YmxpYy9pbWFnZXMvbGFuZ3VhZ2VzL2xhbmd1YWdlLnN2ZyIsInRlbXBsYXRlIjoiZWNobyBIZWxsbyBXb3JsZCIsIm1hdGNoaW5ncyI6W1sieyIsIn0iXSxbIigiLCIpIl0sWyJbIiwiXSJdXSwiZW5naW5lIjoiZ292YWwifSwicXVpbCI6eyJkaXNwbGF5TmFtZSI6IlF1aWwiLCJ0YWdsaW5lIjoiQSBxdWFudHVtIGluc3RydWN0aW9uIGxhbmd1YWdlLiIsImtleSI6InF1aWwiLCJlbnRyeXBvaW50IjoibWFpbi5xdWlsIiwiZXh0IjoicXVpbCIsImhhc0xpbnQiOmZhbHNlLCJoYXNVbml0VGVzdHMiOmZhbHNlLCJoYXNQcm9qZWN0TW9kZSI6ZmFsc2UsImhhc0Zvcm1hdCI6ZmFsc2UsImhhc0xpYnJhcmllcyI6ZmFsc2UsImhhc0V2YWwiOnRydWUsImhhc0xhbmd1YWdlU2VydmVyIjpmYWxzZSwiaGVhZGVyIjoiUHlxdWlsIDEuOS4wLCBQeXRob24gMy42LjEiLCJjYXRlZ29yeSI6IlF1YW50dW0iLCJpY29uIjoiIiwidGVtcGxhdGUiOiIiLCJtYXRjaGluZ3MiOltdLCJlbmdpbmUiOiJnb3ZhbCJ9fQ=='))</script></head><body class="custom_class"><div id="__next"><div style="position:relative;min-height:100vh"><div id="modal-root"></div><div id="page" style="padding-bottom:0;padding-top:0"><div class="jsx-3633663456"></div><div class="jsx-1542287934 workspace-page-wrapper"><div class="jsx-3203370839 workspaceWrapper"><div class="jsx-3203370839 windowManagerWrapper"><div style="height:100%"><div style="height:100%;display:flex;flex-direction:column"><div style="height:40px;width:100%" class="jsx-2926474592 lite-header"><div class="jsx-2926474592 lite-header-cta"><div class="jsx-2861210517 ws-header-cta "><span class="jsx-2861210517 ws-header-cta-label ">loading</span><span class="jsx-2861210517 ws-header-cta-icon "><div class="jsx-2154740905 loading-dots-icon"><img src="/public/images/loading_dots.gif" class="jsx-2154740905"/></div></span></div></div><a href="https://repl.it/@WillKoehrsen/Random-Forest-Tutorial" target="_blank" class="jsx-2926474592 lite-header-logo-link">open in<!-- --> <img src="/public/images/logo-small.png" class="jsx-2926474592 lite-header-logo"/></a></div><div style="height:calc(100% - 40px);width:100%;display:flex;flex-direction:row"><div style="width:50px;height:100%;min-width:50px" class="jsx-695095782 jsx-2554818929 side-nav"><div class="jsx-695095782 jsx-2554818929 side-nav-options"><div class="jsx-1385922659 jsx-340579616 tooltip-base"><div class="jsx-1385922659 jsx-340579616"><div class="jsx-942974558 side-nav-item"><div class="jsx-942974558 side-nav-item-img"><svg class="file-icon-svg" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 17.908 20.614"><defs class="jsx-588867458"></defs><g><path d="M10.4.937H.9v18.74H17.094V7.608ZM15.6,18.1H2.422V2.51H9.554V8.124H15.6Z"></path></g></svg></div></div></div></div><div class="jsx-1385922659 jsx-340579616 tooltip-base"><div class="jsx-1385922659 jsx-340579616"><div class="jsx-942974558 side-nav-item"><div class="jsx-942974558 side-nav-item-img"><svg class="packages-icon-svg" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16.54"><defs class="jsx-2654216489"></defs><g><path d="M1.6,5.093a.4.4,0,0,0-.616.385v6.451a1.247,1.247,0,0,0,.608,1l4.863,2.538a.382.382,0,0,0,.608-.369V8.447a1.2,1.2,0,0,0-.616-.987Z"></path><path d="M14.523,5.093a.4.4,0,0,1,.616.385v6.451a1.247,1.247,0,0,1-.608,1L9.669,15.471a.382.382,0,0,1-.608-.369V8.447a1.2,1.2,0,0,1,.616-.987Z"></path><path d="M7.413,1.041A1.776,1.776,0,0,1,8.7,1.047l4.491,2.067c.353.163.35.422-.007.576L8.707,5.629a1.9,1.9,0,0,1-1.3.006L2.793,3.685c-.358-.151-.361-.405-.006-.565Z"></path></g></svg></div></div></div></div><div class="jsx-1385922659 jsx-340579616 tooltip-base"><div class="jsx-1385922659 jsx-340579616"><div class="jsx-942974558 side-nav-item"><div class="jsx-942974558 side-nav-item-img"><svg class="debugger-icon-svg" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16.54"><defs class="jsx-2347153736"></defs><g><path d="M.626,8.244q0-2.428,0-4.856a.853.853,0,0,1,.041-.3.567.567,0,0,1,.857-.266c.522.3,1.038.614,1.555.922Q6.248,5.634,9.416,7.521a.851.851,0,0,1,.148,1.344.761.761,0,0,1-.14.1L1.6,13.627a.7.7,0,0,1-.514.13.576.576,0,0,1-.454-.585q0-1.314,0-2.63Z"></path><path d="M14.466,13.925h-.61a.867.867,0,0,1-.867-.867l.041-9.64A.867.867,0,0,1,13.9,2.55h.61a.867.867,0,0,1,.867.867l-.041,9.64A.867.867,0,0,1,14.466,13.925Z"></path></g></svg></div></div></div></div></div><div class="jsx-695095782 jsx-2554818929 side-nav-active-pane"></div></div><div style="width:calc(100% - 50px);height:100%;display:flex;flex-direction:column"><div style="height:calc(50% - 5.5px);width:100%;display:flex;flex-direction:column"><div style="height:32px;width:100%" class="jsx-3307283349 file-header"><div class="jsx-3307283349 file-header-name"><div style="margin:0 30px;padding-right:0" class="jsx-3307283349">main.py</div><div style="height:100%" class="jsx-1385922659 jsx-340579616 tooltip-base"><div style="height:100%" class="jsx-1385922659 jsx-340579616"><div class="jsx-3967925514 file-header-format-button"><div class="jsx-3967925514 file-header-format-icon"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16.581 16.428" class="jsx-3967925514"><path d="M13.26.747H3.323A2.569,2.569,0,0,0,.754,3.316h0v9.795a2.569,2.569,0,0,0,2.568,2.57h9.937a2.569,2.569,0,0,0,2.569-2.569h0V3.316A2.569,2.569,0,0,0,13.26.747Z" class="jsx-3967925514 cls-1"></path><path d="M13.118,11.269a.759.759,0,0,1-.759.759H3.921a.759.759,0,0,1-.759-.759V11.23a.671.671,0,0,1,.689-.653l.07.006h8.438a.671.671,0,0,1,.753.578.7.7,0,0,1,.006.069ZM3.161,5.111V5.073a.759.759,0,0,1,.759-.759h8.439a.759.759,0,0,1,.759.759v.038a.759.759,0,0,1-.759.759H3.921a.759.759,0,0,1-.76-.758ZM3.148,8.2a.759.759,0,0,1,.758-.76H8.4a.759.759,0,0,1,.759.759v.038a.759.759,0,0,1-.759.759H3.907a.759.759,0,0,1-.759-.759Z" class="jsx-3967925514 cls-2"></path></svg></div></div></div></div></div><div class="jsx-1735852555"><a class="jsx-1735852555 save-status" href="/@WillKoehrsen/Random-Forest-Tutorial/history"><div class="jsx-1735852555 history-icon"><svg class="history-icon-svg" data-name="Layer 1 copy 5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 21.051 18.415"><defs class="jsx-1990457210"></defs><g><path class="cls-1" d="M9.813,6.732a.485.485,0,0,1,.02-.154.262.262,0,0,1,.409-.139c.249.159,2.758,1.8,3.766,2.453a.47.47,0,0,1,.071.7.366.366,0,0,1-.067.052l-3.736,2.436a.314.314,0,0,1-.246.068.3.3,0,0,1-.217-.306C9.813,11.386,9.813,7.578,9.813,6.732Z"></path><path class="cls-2" d="M19.595,7.041A8.324,8.324,0,0,0,18.053,4a8.49,8.49,0,0,0-2.638-2.208A8.3,8.3,0,0,0,12.246.837,8.286,8.286,0,0,0,5.918,2.96,10.2,10.2,0,0,0,3.3,7.162C2.879,6.6,2.362,5.925,2.093,5.568,1.9,5.312,1.9,5.312,1.652,5.5c-.114.087-.228.174-.342.26-.229.171-.229.171-.059.4C2.062,7.23,2.877,8.3,3.679,9.38c.123.165.208.2.394.09,1.184-.683,2.375-1.354,3.564-2.025.032-.018.062-.032.094-.048a.106.106,0,0,0,.045-.147c-.14-.247-.232-.405-.331-.6-.084-.169-.173-.187-.334-.092-.643.375-2.035,1.154-2.083,1.178l-.263.15A10.5,10.5,0,0,1,7.7,3.694a6.579,6.579,0,0,1,3.271-1.133,6.85,6.85,0,0,1,2.847.38,6.353,6.353,0,0,1,2.039,1.195A6.53,6.53,0,0,1,18.1,8.382a6.415,6.415,0,0,1-.465,3.4,6.472,6.472,0,0,1-5.083,3.976,6.43,6.43,0,0,1-2.325-.037,6.538,6.538,0,0,1-2.111-.788,6.094,6.094,0,0,1-1.643-1.4.62.62,0,0,0-.948-.08,2.874,2.874,0,0,1-.278.221.622.622,0,0,0-.124.968,8.162,8.162,0,0,0,1.5,1.377A8.57,8.57,0,0,0,9.387,17.3a8.145,8.145,0,0,0,2.076.273h.089a8.23,8.23,0,0,0,2.72-.487,8.452,8.452,0,0,0,3.954-2.945,8.266,8.266,0,0,0,1.369-7.1ZM4.5,8.032l0-.006h.014Z"></path></g></svg></div><span class="jsx-1735852555">history</span></a></div></div><div style="height:calc(100% - 32px);width:100%;position:relative" class="jsx-2669800056"><div tabindex="0" class="jsx-2115772186 jsx-2800815994 static-editor"><pre class="jsx-2115772186 jsx-2800815994 static-editor-gutter"><span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">1</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">2</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">3</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">4</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">5</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">6</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">7</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">8</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">9</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">10</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">11</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">12</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">13</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">14</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">15</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">16</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">17</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">18</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">19</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">20</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">21</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">22</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">23</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">24</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">25</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">26</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">27</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">28</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">29</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">30</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">31</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">32</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">33</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">34</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">35</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">36</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">37</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">38</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">39</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">40</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">41</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">42</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">43</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">44</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">45</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">46</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">47</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">48</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">49</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">50</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">51</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">52</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">53</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">54</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">55</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">56</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">57</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">58</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">59</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">60</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">61</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">62</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">63</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">64</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">65</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">66</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">67</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">68</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">69</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">70</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">71</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">72</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">73</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">74</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">75</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">76</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">77</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">78</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">79</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">80</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">81</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">82</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">83</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">84</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">85</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">86</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">87</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">88</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">89</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">90</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">91</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">92</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">93</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">94</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">95</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">96</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">97</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">98</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">99</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">100</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">101</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">102</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">103</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">104</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">105</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">106</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">107</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">108</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">109</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">110</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">111</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">112</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">113</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">114</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">115</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">116</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">117</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">118</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">119</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">120</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">121</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">122</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">123</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">124</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">125</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">126</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">127</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">128</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">129</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">130</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">131</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">132</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">133</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">134</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">135</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">136</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">137</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">138</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">139</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">140</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">141</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">142</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">143</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">144</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">145</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">146</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">147</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">148</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">149</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">150</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">151</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">152</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">153</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">154</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">155</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">156</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">157</span>
<span class="jsx-2115772186 jsx-2800815994 static-editor-line-num">158</span>
</pre><pre class="jsx-2115772186 jsx-2800815994 static-editor-code"><span class="hljs-keyword">import</span> pandas <span class="hljs-keyword">as</span> pd
<span class="hljs-keyword">import</span> numpy <span class="hljs-keyword">as</span> np
<span class="hljs-keyword">from</span> sklearn.model_selection <span class="hljs-keyword">import</span> train_test_split
<span class="hljs-keyword">from</span> sklearn.ensemble <span class="hljs-keyword">import</span> RandomForestClassifier

RSEED = <span class="hljs-number">50</span>

<span class="hljs-comment"># Load in data</span>
df = pd.read_csv(<span class="hljs-string">'https://s3.amazonaws.com/projects-rf/clean_data.csv'</span>)

<span class="hljs-comment"># Full dataset: https://www.kaggle.com/cdc/behavioral-risk-factor-surveillance-system</span>

<span class="hljs-comment"># Extract the labels</span>
labels = np.array(df.pop(<span class="hljs-string">'label'</span>))

<span class="hljs-comment"># 30% examples in test data</span>
train, test, train_labels, test_labels = train_test_split(df,
                                         labels, 
                                         stratify = labels,
                                         test_size = <span class="hljs-number">0.3</span>, 
                                         random_state = RSEED)

<span class="hljs-comment"># Imputation of missing values</span>
train = train.fillna(train.mean())
test = test.fillna(test.mean())

<span class="hljs-comment"># Features for feature importances</span>
features = list(train.columns)

<span class="hljs-comment"># Create the model with 100 trees</span>
model = RandomForestClassifier(n_estimators=<span class="hljs-number">100</span>, 
                               random_state=RSEED, 
                               max_features = <span class="hljs-string">'sqrt'</span>,
                               n_jobs=<span class="hljs-number">-1</span>, verbose = <span class="hljs-number">1</span>)

<span class="hljs-comment"># Fit on training data</span>
model.fit(train, train_labels)


n_nodes = []
max_depths = []

<span class="hljs-comment"># Stats about the trees in random forest</span>
<span class="hljs-keyword">for</span> ind_tree <span class="hljs-keyword">in</span> model.estimators_:
    n_nodes.append(ind_tree.tree_.node_count)
    max_depths.append(ind_tree.tree_.max_depth)
    
print(<span class="hljs-string">f'Average number of nodes <span class="hljs-subst">{int(np.mean(n_nodes))}</span>'</span>)
print(<span class="hljs-string">f'Average maximum depth <span class="hljs-subst">{int(np.mean(max_depths))}</span>'</span>)

<span class="hljs-comment"># Training predictions (to demonstrate overfitting)</span>
train_rf_predictions = model.predict(train)
train_rf_probs = model.predict_proba(train)[:, <span class="hljs-number">1</span>]

<span class="hljs-comment"># Testing predictions (to determine performance)</span>
rf_predictions = model.predict(test)
rf_probs = model.predict_proba(test)[:, <span class="hljs-number">1</span>]

<span class="hljs-keyword">from</span> sklearn.metrics <span class="hljs-keyword">import</span> precision_score, recall_score, roc_auc_score, roc_curve
<span class="hljs-keyword">import</span> matplotlib.pyplot <span class="hljs-keyword">as</span> plt

<span class="hljs-comment"># Plot formatting</span>
plt.style.use(<span class="hljs-string">'fivethirtyeight'</span>)
plt.rcParams[<span class="hljs-string">'font.size'</span>] = <span class="hljs-number">18</span>

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">evaluate_model</span><span class="hljs-params">(predictions, probs, train_predictions, train_probs)</span>:</span>
    <span class="hljs-string">"""Compare machine learning model to baseline performance.
    Computes statistics and shows ROC curve."""</span>
    
    baseline = {}
    
    baseline[<span class="hljs-string">'recall'</span>] = recall_score(test_labels, 
                                     [<span class="hljs-number">1</span> <span class="hljs-keyword">for</span> _ <span class="hljs-keyword">in</span> range(len(test_labels))])
    baseline[<span class="hljs-string">'precision'</span>] = precision_score(test_labels, 
                                      [<span class="hljs-number">1</span> <span class="hljs-keyword">for</span> _ <span class="hljs-keyword">in</span> range(len(test_labels))])
    baseline[<span class="hljs-string">'roc'</span>] = <span class="hljs-number">0.5</span>
    
    results = {}
    
    results[<span class="hljs-string">'recall'</span>] = recall_score(test_labels, predictions)
    results[<span class="hljs-string">'precision'</span>] = precision_score(test_labels, predictions)
    results[<span class="hljs-string">'roc'</span>] = roc_auc_score(test_labels, probs)
    
    train_results = {}
    train_results[<span class="hljs-string">'recall'</span>] = recall_score(train_labels, train_predictions)
    train_results[<span class="hljs-string">'precision'</span>] = precision_score(train_labels, train_predictions)
    train_results[<span class="hljs-string">'roc'</span>] = roc_auc_score(train_labels, train_probs)
    
    <span class="hljs-keyword">for</span> metric <span class="hljs-keyword">in</span> [<span class="hljs-string">'recall'</span>, <span class="hljs-string">'precision'</span>, <span class="hljs-string">'roc'</span>]:
        print(<span class="hljs-string">f'<span class="hljs-subst">{metric.capitalize()}</span> Baseline: <span class="hljs-subst">{round(baseline[metric], <span class="hljs-number">2</span>)}</span> Test: <span class="hljs-subst">{round(results[metric], <span class="hljs-number">2</span>)}</span> Train: <span class="hljs-subst">{round(train_results[metric], <span class="hljs-number">2</span>)}</span>'</span>)
    
    <span class="hljs-comment"># Calculate false positive rates and true positive rates</span>
    base_fpr, base_tpr, _ = roc_curve(test_labels, [<span class="hljs-number">1</span> <span class="hljs-keyword">for</span> _ <span class="hljs-keyword">in</span> range(len(test_labels))])
    model_fpr, model_tpr, _ = roc_curve(test_labels, probs)

    plt.figure(figsize = (<span class="hljs-number">8</span>, <span class="hljs-number">6</span>))
    plt.rcParams[<span class="hljs-string">'font.size'</span>] = <span class="hljs-number">16</span>
    
    <span class="hljs-comment"># Plot both curves</span>
    plt.plot(base_fpr, base_tpr, <span class="hljs-string">'b'</span>, label = <span class="hljs-string">'baseline'</span>)
    plt.plot(model_fpr, model_tpr, <span class="hljs-string">'r'</span>, label = <span class="hljs-string">'model'</span>)
    plt.legend();
    plt.xlabel(<span class="hljs-string">'False Positive Rate'</span>); 
    plt.ylabel(<span class="hljs-string">'True Positive Rate'</span>); plt.title(<span class="hljs-string">'ROC Curves'</span>);
    plt.show();

evaluate_model(rf_predictions, rf_probs, train_rf_predictions, train_rf_probs)
plt.savefig(<span class="hljs-string">'roc_auc_curve.png'</span>)

<span class="hljs-keyword">from</span> sklearn.metrics <span class="hljs-keyword">import</span> confusion_matrix
<span class="hljs-keyword">import</span> itertools

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">plot_confusion_matrix</span><span class="hljs-params">(cm, classes,
                          normalize=False,
                          title=<span class="hljs-string">'Confusion matrix'</span>,
                          cmap=plt.cm.Oranges)</span>:</span>
    <span class="hljs-string">"""
    This function prints and plots the confusion matrix.
    Normalization can be applied by setting `normalize=True`.
    Source: http://scikit-learn.org/stable/auto_examples/model_selection/plot_confusion_matrix.html
    """</span>
    <span class="hljs-keyword">if</span> normalize:
        cm = cm.astype(<span class="hljs-string">'float'</span>) / cm.sum(axis=<span class="hljs-number">1</span>)[:, np.newaxis]
        print(<span class="hljs-string">"Normalized confusion matrix"</span>)
    <span class="hljs-keyword">else</span>:
        print(<span class="hljs-string">'Confusion matrix, without normalization'</span>)

    print(cm)

    <span class="hljs-comment"># Plot the confusion matrix</span>
    plt.figure(figsize = (<span class="hljs-number">10</span>, <span class="hljs-number">10</span>))
    plt.imshow(cm, interpolation=<span class="hljs-string">'nearest'</span>, cmap=cmap)
    plt.title(title, size = <span class="hljs-number">24</span>)
    plt.colorbar(aspect=<span class="hljs-number">4</span>)
    tick_marks = np.arange(len(classes))
    plt.xticks(tick_marks, classes, rotation=<span class="hljs-number">45</span>, size = <span class="hljs-number">14</span>)
    plt.yticks(tick_marks, classes, size = <span class="hljs-number">14</span>)

    fmt = <span class="hljs-string">'.2f'</span> <span class="hljs-keyword">if</span> normalize <span class="hljs-keyword">else</span> <span class="hljs-string">'d'</span>
    thresh = cm.max() / <span class="hljs-number">2.</span>
    
    <span class="hljs-comment"># Labeling the plot</span>
    <span class="hljs-keyword">for</span> i, j <span class="hljs-keyword">in</span> itertools.product(range(cm.shape[<span class="hljs-number">0</span>]), range(cm.shape[<span class="hljs-number">1</span>])):
        plt.text(j, i, format(cm[i, j], fmt), fontsize = <span class="hljs-number">20</span>,
                 horizontalalignment=<span class="hljs-string">"center"</span>,
                 color=<span class="hljs-string">"white"</span> <span class="hljs-keyword">if</span> cm[i, j] &gt; thresh <span class="hljs-keyword">else</span> <span class="hljs-string">"black"</span>)
        
    plt.grid(<span class="hljs-keyword">None</span>)
    plt.tight_layout()
    plt.ylabel(<span class="hljs-string">'True label'</span>, size = <span class="hljs-number">18</span>)
    plt.xlabel(<span class="hljs-string">'Predicted label'</span>, size = <span class="hljs-number">18</span>)

<span class="hljs-comment"># Confusion matrix</span>
cm = confusion_matrix(test_labels, rf_predictions)
plot_confusion_matrix(cm, classes = [<span class="hljs-string">'Poor Health'</span>, <span class="hljs-string">'Good Health'</span>],
                      title = <span class="hljs-string">'Health Confusion Matrix'</span>)

plt.savefig(<span class="hljs-string">'cm.png'</span>)</pre></div></div></div><div style="height:11px;position:relative;display:flex;justify-content:center;align-items:center;flex:1 0 auto;overflow:hidden;cursor:row-resize;width:100%;background:#DDD"><div style="width:25px;border-top:solid 1px;border-top-color:#777879"></div></div><div style="height:calc(50% - 5.5px);width:100%;background-color:#0D1323;font-size:14px;padding:15px;box-sizing:border-box"><span style="white-space:pre-wrap;font-size:14px;line-height:1em;word-wrap:break-word;font-family:monospace;color:#fff;position:relative;left:0px">Python 3.6.1 (default, Dec 2015, 13:05:11)
[GCC 4.8.2] on linux
</span><div style="margin-left:-8px;background:url(/public/images/sprite.png) -13px -583px no-repeat;width:24px;height:16px"></div></div></div></div></div></div></div></div></div></div></div></div><div id="__next-error"></div><script>
          __NEXT_DATA__ = {"props":{"pageProps":{"isServer":true,"store":{},"initialState":{"replEnvironment":{"isForking":false,"activeWid":"/@WillKoehrsen/Random-Forest-Tutorial"},"repls":{"data":{"824d8d3a-8a57-4f16-a56f-1a2337806711":{"id":"824d8d3a-8a57-4f16-a56f-1a2337806711","userId":1063233,"title":"Random Forest Tutorial","description":"A short implementation of the random forest on a real-world dataset. Includes training / predicting along with visualizations. ","isProject":false,"isPrivate":false,"timeCreated":"2018-08-27T21:45:07.603Z","timeUpdated":"2018-08-27T22:05:25.921Z","views":0,"contentLength":0,"language":"python3","originId":null,"slug":"Random-Forest-Tutorial","config":{},"url":"/@WillKoehrsen/Random-Forest-Tutorial","fileNames":["main.py","requirements.txt"],"isOwner":true}}},"replsByUrl":{"/@WillKoehrsen/Random-Forest-Tutorial":"824d8d3a-8a57-4f16-a56f-1a2337806711"},"replsByUser":{"1063233":["824d8d3a-8a57-4f16-a56f-1a2337806711"]},"notifications":{"data":[],"isFetching":false},"plans":{"data":{}},"workspace":{"/@WillKoehrsen/Random-Forest-Tutorial":{"language":"python3","layout":{"floating":[],"tiling":{"type":"split","fixed":true,"leftSize":"40px","hideHandle":true,"stacked":true,"left":{"type":"pane","pud":"header"},"right":{"type":"split","stacked":false,"fixed":true,"hideHandle":true,"leftSize":"50px","leftMinSize":50,"left":{"type":"pane","pud":"sidenav"},"right":{"type":"split","stacked":1,"fixed":false,"left":{"type":"split","stacked":true,"fixed":true,"hideHandle":true,"leftSize":"32px","left":{"type":"pane","pud":"fileheader"},"right":{"type":"pane","pud":"file-df86bb98-2a44-4218-81af-f78500d2db06"}},"right":{"type":"pane","pud":"output"}}}}},"plugins":{"fileheader":{"type":"fileheader","title":"fileheader","forceRender":false,"alertStatus":"none"},"libs":{"type":"libsearch","title":"library search","state":{"libraries":[],"query":"","selectedLib":null},"forceRender":false,"alertStatus":"none"},"examples":{"type":"examples","title":"examples","state":{"loading":false},"forceRender":false,"alertStatus":"none"},"header":{"type":"header","title":"header","state":{"running":false,"loading":true,"lite":true},"forceRender":false,"alertStatus":"none"},"output":{"type":"console","title":"console","state":{"consoleMessages":[],"loadingEngine":false,"running":false,"presetStdin":"","promptValue":""},"forceRender":false,"alertStatus":"none"},"repl":{"type":"goval","title":"repl","state":{"dotStyleDomains":false,"engineHandle":null,"debuggerHandle":null,"loadingEngine":false,"running":false,"debugging":false},"forceRender":false,"alertStatus":"none"},"debugger":{"type":"debugger","title":"debugger","state":{"stack":[],"activeLine":null,"debugging":false},"forceRender":false,"alertStatus":"none"},"file-df86bb98-2a44-4218-81af-f78500d2db06":{"type":"editor","title":"main.py","state":{"filePath":"824d8d3a-8a57-4f16-a56f-1a2337806711/main.py","readonly":false,"isMac":true,"ace":{"sessionHandle":null,"readonly":false,"filePath":"824d8d3a-8a57-4f16-a56f-1a2337806711/main.py","annotations":[],"activeLine":null,"running":false,"isMac":true},"monaco":{"filePath":"824d8d3a-8a57-4f16-a56f-1a2337806711/main.py","modelHandle":null,"editorHandle":null,"onChangeHandle":null,"viewState":null,"running":false,"activeLineDecorations":[],"langServerReady":false,"readonly":false,"isMac":true}},"forceRender":false,"alertStatus":"none"},"file-99041b83-88a1-4774-b38b-ec03f47a6ecb":{"type":"editor","title":"requirements.txt","state":{"filePath":"824d8d3a-8a57-4f16-a56f-1a2337806711/requirements.txt","readonly":false,"isMac":true,"ace":{"sessionHandle":null,"readonly":false,"filePath":"824d8d3a-8a57-4f16-a56f-1a2337806711/requirements.txt","annotations":[],"activeLine":null,"running":false,"isMac":true},"monaco":{"filePath":"824d8d3a-8a57-4f16-a56f-1a2337806711/requirements.txt","modelHandle":null,"editorHandle":null,"onChangeHandle":null,"viewState":null,"running":false,"activeLineDecorations":[],"langServerReady":false,"readonly":false,"isMac":true}},"forceRender":false,"alertStatus":"none"},"filetree":{"type":"filetree","title":"filetree","state":{"tree":{"824d8d3a-8a57-4f16-a56f-1a2337806711":{"path":"824d8d3a-8a57-4f16-a56f-1a2337806711","isExpanded":true,"isFolder":true},"824d8d3a-8a57-4f16-a56f-1a2337806711/main.py":{"path":"824d8d3a-8a57-4f16-a56f-1a2337806711/main.py","isExpanded":false,"isFolder":false},"824d8d3a-8a57-4f16-a56f-1a2337806711/requirements.txt":{"path":"824d8d3a-8a57-4f16-a56f-1a2337806711/requirements.txt","isExpanded":false,"isFolder":false}},"activeNode":"824d8d3a-8a57-4f16-a56f-1a2337806711/main.py","creatingNodeType":null,"allowActions":true,"showDropTooltip":false},"forceRender":false,"alertStatus":"none"},"sidenav":{"type":"sidenav","title":"side navigation","state":{"navItems":[{"pud":"filetree","pluginType":"filetree","tooltip":"Files","svg":"File"},{"pud":"libs","pluginType":"libsearch","tooltip":"Packages","svg":"Packages"},{"pud":"debugger","pluginType":"debugger","tooltip":"Debugger","svg":"Debugger"}],"activePanePud":null,"savedResizerPosition":25},"forceRender":false,"alertStatus":"none"}},"files":["824d8d3a-8a57-4f16-a56f-1a2337806711/main.py","824d8d3a-8a57-4f16-a56f-1a2337806711/requirements.txt"],"activeFile":"824d8d3a-8a57-4f16-a56f-1a2337806711/main.py","preferences":{"isLayoutStacked":false,"theme":"replitLight","fontSize":14,"indentIsSpaces":true,"indentSize":2,"keyboardHandler":null,"wrapping":true,"primarySplitPosition":60,"infiniteLoopProtection":true,"hideSnackModal":false,"hideFeaturedClassrooms":false,"hideSuggestedClassrooms":false,"codeIntelligence":false},"isMobile":false,"lite":true}},"files":{"824d8d3a-8a57-4f16-a56f-1a2337806711/main.py":{"content":{"asEncoding":{"base64":"aW1wb3J0IHBhbmRhcyBhcyBwZAppbXBvcnQgbnVtcHkgYXMgbnAKZnJvbSBza2xlYXJuLm1vZGVsX3NlbGVjdGlvbiBpbXBvcnQgdHJhaW5fdGVzdF9zcGxpdApmcm9tIHNrbGVhcm4uZW5zZW1ibGUgaW1wb3J0IFJhbmRvbUZvcmVzdENsYXNzaWZpZXIKClJTRUVEID0gNTAKCiMgTG9hZCBpbiBkYXRhCmRmID0gcGQucmVhZF9jc3YoJ2h0dHBzOi8vczMuYW1hem9uYXdzLmNvbS9wcm9qZWN0cy1yZi9jbGVhbl9kYXRhLmNzdicpCgojIEZ1bGwgZGF0YXNldDogaHR0cHM6Ly93d3cua2FnZ2xlLmNvbS9jZGMvYmVoYXZpb3JhbC1yaXNrLWZhY3Rvci1zdXJ2ZWlsbGFuY2Utc3lzdGVtCgojIEV4dHJhY3QgdGhlIGxhYmVscwpsYWJlbHMgPSBucC5hcnJheShkZi5wb3AoJ2xhYmVsJykpCgojIDMwJSBleGFtcGxlcyBpbiB0ZXN0IGRhdGEKdHJhaW4sIHRlc3QsIHRyYWluX2xhYmVscywgdGVzdF9sYWJlbHMgPSB0cmFpbl90ZXN0X3NwbGl0KGRmLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGxhYmVscywgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgc3RyYXRpZnkgPSBsYWJlbHMsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgdGVzdF9zaXplID0gMC4zLCAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICByYW5kb21fc3RhdGUgPSBSU0VFRCkKCiMgSW1wdXRhdGlvbiBvZiBtaXNzaW5nIHZhbHVlcwp0cmFpbiA9IHRyYWluLmZpbGxuYSh0cmFpbi5tZWFuKCkpCnRlc3QgPSB0ZXN0LmZpbGxuYSh0ZXN0Lm1lYW4oKSkKCiMgRmVhdHVyZXMgZm9yIGZlYXR1cmUgaW1wb3J0YW5jZXMKZmVhdHVyZXMgPSBsaXN0KHRyYWluLmNvbHVtbnMpCgojIENyZWF0ZSB0aGUgbW9kZWwgd2l0aCAxMDAgdHJlZXMKbW9kZWwgPSBSYW5kb21Gb3Jlc3RDbGFzc2lmaWVyKG5fZXN0aW1hdG9ycz0xMDAsIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgcmFuZG9tX3N0YXRlPVJTRUVELCAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heF9mZWF0dXJlcyA9ICdzcXJ0JywKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG5fam9icz0tMSwgdmVyYm9zZSA9IDEpCgojIEZpdCBvbiB0cmFpbmluZyBkYXRhCm1vZGVsLmZpdCh0cmFpbiwgdHJhaW5fbGFiZWxzKQoKCm5fbm9kZXMgPSBbXQptYXhfZGVwdGhzID0gW10KCiMgU3RhdHMgYWJvdXQgdGhlIHRyZWVzIGluIHJhbmRvbSBmb3Jlc3QKZm9yIGluZF90cmVlIGluIG1vZGVsLmVzdGltYXRvcnNfOgogICAgbl9ub2Rlcy5hcHBlbmQoaW5kX3RyZWUudHJlZV8ubm9kZV9jb3VudCkKICAgIG1heF9kZXB0aHMuYXBwZW5kKGluZF90cmVlLnRyZWVfLm1heF9kZXB0aCkKICAgIApwcmludChmJ0F2ZXJhZ2UgbnVtYmVyIG9mIG5vZGVzIHtpbnQobnAubWVhbihuX25vZGVzKSl9JykKcHJpbnQoZidBdmVyYWdlIG1heGltdW0gZGVwdGgge2ludChucC5tZWFuKG1heF9kZXB0aHMpKX0nKQoKIyBUcmFpbmluZyBwcmVkaWN0aW9ucyAodG8gZGVtb25zdHJhdGUgb3ZlcmZpdHRpbmcpCnRyYWluX3JmX3ByZWRpY3Rpb25zID0gbW9kZWwucHJlZGljdCh0cmFpbikKdHJhaW5fcmZfcHJvYnMgPSBtb2RlbC5wcmVkaWN0X3Byb2JhKHRyYWluKVs6LCAxXQoKIyBUZXN0aW5nIHByZWRpY3Rpb25zICh0byBkZXRlcm1pbmUgcGVyZm9ybWFuY2UpCnJmX3ByZWRpY3Rpb25zID0gbW9kZWwucHJlZGljdCh0ZXN0KQpyZl9wcm9icyA9IG1vZGVsLnByZWRpY3RfcHJvYmEodGVzdClbOiwgMV0KCmZyb20gc2tsZWFybi5tZXRyaWNzIGltcG9ydCBwcmVjaXNpb25fc2NvcmUsIHJlY2FsbF9zY29yZSwgcm9jX2F1Y19zY29yZSwgcm9jX2N1cnZlCmltcG9ydCBtYXRwbG90bGliLnB5cGxvdCBhcyBwbHQKCiMgUGxvdCBmb3JtYXR0aW5nCnBsdC5zdHlsZS51c2UoJ2ZpdmV0aGlydHllaWdodCcpCnBsdC5yY1BhcmFtc1snZm9udC5zaXplJ10gPSAxOAoKZGVmIGV2YWx1YXRlX21vZGVsKHByZWRpY3Rpb25zLCBwcm9icywgdHJhaW5fcHJlZGljdGlvbnMsIHRyYWluX3Byb2JzKToKICAgICIiIkNvbXBhcmUgbWFjaGluZSBsZWFybmluZyBtb2RlbCB0byBiYXNlbGluZSBwZXJmb3JtYW5jZS4KICAgIENvbXB1dGVzIHN0YXRpc3RpY3MgYW5kIHNob3dzIFJPQyBjdXJ2ZS4iIiIKICAgIAogICAgYmFzZWxpbmUgPSB7fQogICAgCiAgICBiYXNlbGluZVsncmVjYWxsJ10gPSByZWNhbGxfc2NvcmUodGVzdF9sYWJlbHMsIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgWzEgZm9yIF8gaW4gcmFuZ2UobGVuKHRlc3RfbGFiZWxzKSldKQogICAgYmFzZWxpbmVbJ3ByZWNpc2lvbiddID0gcHJlY2lzaW9uX3Njb3JlKHRlc3RfbGFiZWxzLCAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBbMSBmb3IgXyBpbiByYW5nZShsZW4odGVzdF9sYWJlbHMpKV0pCiAgICBiYXNlbGluZVsncm9jJ10gPSAwLjUKICAgIAogICAgcmVzdWx0cyA9IHt9CiAgICAKICAgIHJlc3VsdHNbJ3JlY2FsbCddID0gcmVjYWxsX3Njb3JlKHRlc3RfbGFiZWxzLCBwcmVkaWN0aW9ucykKICAgIHJlc3VsdHNbJ3ByZWNpc2lvbiddID0gcHJlY2lzaW9uX3Njb3JlKHRlc3RfbGFiZWxzLCBwcmVkaWN0aW9ucykKICAgIHJlc3VsdHNbJ3JvYyddID0gcm9jX2F1Y19zY29yZSh0ZXN0X2xhYmVscywgcHJvYnMpCiAgICAKICAgIHRyYWluX3Jlc3VsdHMgPSB7fQogICAgdHJhaW5fcmVzdWx0c1sncmVjYWxsJ10gPSByZWNhbGxfc2NvcmUodHJhaW5fbGFiZWxzLCB0cmFpbl9wcmVkaWN0aW9ucykKICAgIHRyYWluX3Jlc3VsdHNbJ3ByZWNpc2lvbiddID0gcHJlY2lzaW9uX3Njb3JlKHRyYWluX2xhYmVscywgdHJhaW5fcHJlZGljdGlvbnMpCiAgICB0cmFpbl9yZXN1bHRzWydyb2MnXSA9IHJvY19hdWNfc2NvcmUodHJhaW5fbGFiZWxzLCB0cmFpbl9wcm9icykKICAgIAogICAgZm9yIG1ldHJpYyBpbiBbJ3JlY2FsbCcsICdwcmVjaXNpb24nLCAncm9jJ106CiAgICAgICAgcHJpbnQoZid7bWV0cmljLmNhcGl0YWxpemUoKX0gQmFzZWxpbmU6IHtyb3VuZChiYXNlbGluZVttZXRyaWNdLCAyKX0gVGVzdDoge3JvdW5kKHJlc3VsdHNbbWV0cmljXSwgMil9IFRyYWluOiB7cm91bmQodHJhaW5fcmVzdWx0c1ttZXRyaWNdLCAyKX0nKQogICAgCiAgICAjIENhbGN1bGF0ZSBmYWxzZSBwb3NpdGl2ZSByYXRlcyBhbmQgdHJ1ZSBwb3NpdGl2ZSByYXRlcwogICAgYmFzZV9mcHIsIGJhc2VfdHByLCBfID0gcm9jX2N1cnZlKHRlc3RfbGFiZWxzLCBbMSBmb3IgXyBpbiByYW5nZShsZW4odGVzdF9sYWJlbHMpKV0pCiAgICBtb2RlbF9mcHIsIG1vZGVsX3RwciwgXyA9IHJvY19jdXJ2ZSh0ZXN0X2xhYmVscywgcHJvYnMpCgogICAgcGx0LmZpZ3VyZShmaWdzaXplID0gKDgsIDYpKQogICAgcGx0LnJjUGFyYW1zWydmb250LnNpemUnXSA9IDE2CiAgICAKICAgICMgUGxvdCBib3RoIGN1cnZlcwogICAgcGx0LnBsb3QoYmFzZV9mcHIsIGJhc2VfdHByLCAnYicsIGxhYmVsID0gJ2Jhc2VsaW5lJykKICAgIHBsdC5wbG90KG1vZGVsX2ZwciwgbW9kZWxfdHByLCAncicsIGxhYmVsID0gJ21vZGVsJykKICAgIHBsdC5sZWdlbmQoKTsKICAgIHBsdC54bGFiZWwoJ0ZhbHNlIFBvc2l0aXZlIFJhdGUnKTsgCiAgICBwbHQueWxhYmVsKCdUcnVlIFBvc2l0aXZlIFJhdGUnKTsgcGx0LnRpdGxlKCdST0MgQ3VydmVzJyk7CiAgICBwbHQuc2hvdygpOwoKZXZhbHVhdGVfbW9kZWwocmZfcHJlZGljdGlvbnMsIHJmX3Byb2JzLCB0cmFpbl9yZl9wcmVkaWN0aW9ucywgdHJhaW5fcmZfcHJvYnMpCnBsdC5zYXZlZmlnKCdyb2NfYXVjX2N1cnZlLnBuZycpCgpmcm9tIHNrbGVhcm4ubWV0cmljcyBpbXBvcnQgY29uZnVzaW9uX21hdHJpeAppbXBvcnQgaXRlcnRvb2xzCgpkZWYgcGxvdF9jb25mdXNpb25fbWF0cml4KGNtLCBjbGFzc2VzLAogICAgICAgICAgICAgICAgICAgICAgICAgIG5vcm1hbGl6ZT1GYWxzZSwKICAgICAgICAgICAgICAgICAgICAgICAgICB0aXRsZT0nQ29uZnVzaW9uIG1hdHJpeCcsCiAgICAgICAgICAgICAgICAgICAgICAgICAgY21hcD1wbHQuY20uT3Jhbmdlcyk6CiAgICAiIiIKICAgIFRoaXMgZnVuY3Rpb24gcHJpbnRzIGFuZCBwbG90cyB0aGUgY29uZnVzaW9uIG1hdHJpeC4KICAgIE5vcm1hbGl6YXRpb24gY2FuIGJlIGFwcGxpZWQgYnkgc2V0dGluZyBgbm9ybWFsaXplPVRydWVgLgogICAgU291cmNlOiBodHRwOi8vc2Npa2l0LWxlYXJuLm9yZy9zdGFibGUvYXV0b19leGFtcGxlcy9tb2RlbF9zZWxlY3Rpb24vcGxvdF9jb25mdXNpb25fbWF0cml4Lmh0bWwKICAgICIiIgogICAgaWYgbm9ybWFsaXplOgogICAgICAgIGNtID0gY20uYXN0eXBlKCdmbG9hdCcpIC8gY20uc3VtKGF4aXM9MSlbOiwgbnAubmV3YXhpc10KICAgICAgICBwcmludCgiTm9ybWFsaXplZCBjb25mdXNpb24gbWF0cml4IikKICAgIGVsc2U6CiAgICAgICAgcHJpbnQoJ0NvbmZ1c2lvbiBtYXRyaXgsIHdpdGhvdXQgbm9ybWFsaXphdGlvbicpCgogICAgcHJpbnQoY20pCgogICAgIyBQbG90IHRoZSBjb25mdXNpb24gbWF0cml4CiAgICBwbHQuZmlndXJlKGZpZ3NpemUgPSAoMTAsIDEwKSkKICAgIHBsdC5pbXNob3coY20sIGludGVycG9sYXRpb249J25lYXJlc3QnLCBjbWFwPWNtYXApCiAgICBwbHQudGl0bGUodGl0bGUsIHNpemUgPSAyNCkKICAgIHBsdC5jb2xvcmJhcihhc3BlY3Q9NCkKICAgIHRpY2tfbWFya3MgPSBucC5hcmFuZ2UobGVuKGNsYXNzZXMpKQogICAgcGx0Lnh0aWNrcyh0aWNrX21hcmtzLCBjbGFzc2VzLCByb3RhdGlvbj00NSwgc2l6ZSA9IDE0KQogICAgcGx0Lnl0aWNrcyh0aWNrX21hcmtzLCBjbGFzc2VzLCBzaXplID0gMTQpCgogICAgZm10ID0gJy4yZicgaWYgbm9ybWFsaXplIGVsc2UgJ2QnCiAgICB0aHJlc2ggPSBjbS5tYXgoKSAvIDIuCiAgICAKICAgICMgTGFiZWxpbmcgdGhlIHBsb3QKICAgIGZvciBpLCBqIGluIGl0ZXJ0b29scy5wcm9kdWN0KHJhbmdlKGNtLnNoYXBlWzBdKSwgcmFuZ2UoY20uc2hhcGVbMV0pKToKICAgICAgICBwbHQudGV4dChqLCBpLCBmb3JtYXQoY21baSwgal0sIGZtdCksIGZvbnRzaXplID0gMjAsCiAgICAgICAgICAgICAgICAgaG9yaXpvbnRhbGFsaWdubWVudD0iY2VudGVyIiwKICAgICAgICAgICAgICAgICBjb2xvcj0id2hpdGUiIGlmIGNtW2ksIGpdID4gdGhyZXNoIGVsc2UgImJsYWNrIikKICAgICAgICAKICAgIHBsdC5ncmlkKE5vbmUpCiAgICBwbHQudGlnaHRfbGF5b3V0KCkKICAgIHBsdC55bGFiZWwoJ1RydWUgbGFiZWwnLCBzaXplID0gMTgpCiAgICBwbHQueGxhYmVsKCdQcmVkaWN0ZWQgbGFiZWwnLCBzaXplID0gMTgpCgojIENvbmZ1c2lvbiBtYXRyaXgKY20gPSBjb25mdXNpb25fbWF0cml4KHRlc3RfbGFiZWxzLCByZl9wcmVkaWN0aW9ucykKcGxvdF9jb25mdXNpb25fbWF0cml4KGNtLCBjbGFzc2VzID0gWydQb29yIEhlYWx0aCcsICdHb29kIEhlYWx0aCddLAogICAgICAgICAgICAgICAgICAgICAgdGl0bGUgPSAnSGVhbHRoIENvbmZ1c2lvbiBNYXRyaXgnKQoKcGx0LnNhdmVmaWcoJ2NtLnBuZycp"},"asBuffer":null},"synced":true,"deleted":false}},"messages":{"queue":[]},"user":{"userInfo":{"fetchState":"done","fetchingPromise":null,"username":"WillKoehrsen","email":"wjk68@case.edu","id":1063233,"firstName":"Will","lastName":"Koehrsen","organizationId":null,"editorPreferences":{"isLayoutStacked":false,"theme":"replitDark","fontSize":14,"indentIsSpaces":true,"indentSize":4,"keyboardHandler":null,"wrapping":true,"primarySplitPosition":60,"infiniteLoopProtection":true,"hideSnackModal":false,"hideFeaturedClassrooms":false,"hideSuggestedClassrooms":false,"codeIntelligence":true},"timeCreated":"2018-08-27T21:44:56.000Z","emailNotifications":true,"icon":null,"karma":1,"emailHash":"f85b7564fd35d5c86054b95090052d94","organization":null,"roles":["teacher","self_learner"],"subscription":null,"bio":null,"isVerified":true,"gating":[{"controlName":"web-migration","type":"boolean","value":true},{"controlName":"new-workspace","type":"boolean","value":true},{"controlName":"dashboard-search","type":"boolean","value":true},{"controlName":"monaco","type":"boolean","value":true},{"controlName":"repls-filters","type":"boolean","value":true},{"controlName":"classroom-projects","type":"boolean","value":true},{"controlName":"language-server","type":"boolean","value":true},{"controlName":"verify-email","type":"boolean","value":true},{"controlName":"sidebar-lib-search","type":"boolean","value":true},{"controlName":"attach","type":"boolean","value":true},{"controlName":"fork-repls-dashboard","type":"multivariate","value":"hidden"},{"controlName":"file-tree","type":"multivariate","value":"open"}],"error":"","isLoggedIn":true},"billingInfo":{"isFetching":false},"authModal":{"promptCount":0,"dismissed":false,"show":false}},"banners":{"message":""},"profiles":{"data":{"1063233":{"classrooms":null,"emailHash":"f85b7564fd35d5c86054b95090052d94","organization":null,"repls":null,"id":1063233,"username":"WillKoehrsen","bio":null,"icon":null,"topLanguages":null}}}},"initialProps":{"serverState":{"apollo":{"data":{}}},"location":"/replEnvironment","lite":1,"isMobile":false,"isMonaco":true,"user":{"fetchState":"done","fetchingPromise":null,"username":"WillKoehrsen","email":"wjk68@case.edu","id":1063233,"firstName":"Will","lastName":"Koehrsen","organizationId":null,"editorPreferences":{"isLayoutStacked":false,"theme":"replitDark","fontSize":14,"indentIsSpaces":true,"indentSize":4,"keyboardHandler":null,"wrapping":true,"primarySplitPosition":60,"infiniteLoopProtection":true,"hideSnackModal":false,"hideFeaturedClassrooms":false,"hideSuggestedClassrooms":false,"codeIntelligence":true},"timeCreated":"2018-08-27T21:44:56.000Z","emailNotifications":true,"icon":null,"karma":1,"emailHash":"f85b7564fd35d5c86054b95090052d94","organization":null,"roles":["teacher","self_learner"],"subscription":null,"bio":null,"isVerified":true,"gating":[{"controlName":"web-migration","type":"boolean","value":true},{"controlName":"new-workspace","type":"boolean","value":true},{"controlName":"dashboard-search","type":"boolean","value":true},{"controlName":"monaco","type":"boolean","value":true},{"controlName":"repls-filters","type":"boolean","value":true},{"controlName":"classroom-projects","type":"boolean","value":true},{"controlName":"language-server","type":"boolean","value":true},{"controlName":"verify-email","type":"boolean","value":true},{"controlName":"sidebar-lib-search","type":"boolean","value":true},{"controlName":"attach","type":"boolean","value":true},{"controlName":"fork-repls-dashboard","type":"multivariate","value":"hidden"},{"controlName":"file-tree","type":"multivariate","value":"open"}],"error":"","isLoggedIn":true},"languageServer":true}}},"page":"/replEnvironment","pathname":"/replEnvironment","query":{"lite":1},"buildId":"fe48d264-0938-4ab4-a310-5fc686ab952d","assetPrefix":"","nextExport":false,"err":null,"chunks":[]}
          module={}
          __NEXT_LOADED_PAGES__ = []
          __NEXT_LOADED_CHUNKS__ = []

          __NEXT_REGISTER_PAGE = function (route, fn) {
            __NEXT_LOADED_PAGES__.push({ route: route, fn: fn })
          }

          __NEXT_REGISTER_CHUNK = function (chunkName, fn) {
            __NEXT_LOADED_CHUNKS__.push({ chunkName: chunkName, fn: fn })
          }

          false
        </script><script async="" id="__NEXT_PAGE__/replEnvironment" src="/_next/fe48d264-0938-4ab4-a310-5fc686ab952d/page/replEnvironment.js"></script><script async="" id="__NEXT_PAGE__/_app" src="/_next/fe48d264-0938-4ab4-a310-5fc686ab952d/page/_app.js"></script><script async="" id="__NEXT_PAGE__/_error" src="/_next/fe48d264-0938-4ab4-a310-5fc686ab952d/page/_error.js"></script><script src="/_next/static/commons/main-bf2a9bbd3aa2f834d2be.js" async=""></script></body></html>