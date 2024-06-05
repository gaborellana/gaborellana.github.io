---
layout: post
title:  "Follow-up on expected goals project: improvements on model and comparisons"
date:   2024-06-04 10:00:00 -0400
categories: jekyll update
---
# Improvements on model

#### TLDR
As shown in the previous post, the model used to predict the expected goals was promising, but still performed worse than the StatsBomb measure of expected goals. Following some of the suggestions in the previous post, we will show how our model performs as well, if not better, than StatsBomb's.

## What changed from the last results?
- We used all the event data available in the database that fulfill two conditions. (1) it's male football, and (2) the entire competition is available (therefore we didn't use data where only a subset of games from a competition was available)
- We slightly modified the architecture of our model.
- We added some features available in the event data regarding the nature of the shot.
- We removed some events that were not relevant to our model (e.g. penalties, free-kicks).
- We made the validation subset bigger (previously the sizes of subsets were 0.8-0.1-0.1 for training, validation and test sets, now is 0.7-0.2-0.1).


### Improvements in the architecture

As mentioned, we are using a Graph Convolutional Neural Network of the subtype message passing. This kind of architecture is fairly small compared to state-of-the-art LLMs, but still contains tens of thousands of parameters. The constraint on the amount of data for training and validation limited severely the amount of layers and the architecture decision in the design of the model.
We slightly modified the flow of information inside the model so that the output of the model is calculated from the shooter's embedding as well as from the goalkeeper's embedding (previously it was only calculated from the shooter's embedding). 


### Adding of "global" features

We called "global" features those that don't correspond to a single node of the graph but to the graph as a whole. In this case, these features are categorical and attempt to add information about the nature of the play that preceded the shot. These features are:
- Play pattern: description of the origin of the play (e.g. regular play, from corner, from throw-in, etc.)
- Technique: description of the technique used to make the shot (e.g. normal, volley, diving-header, etc.)
- Body part: part of the body used to make the shot (e.g. head, right foot, etc.)
- Aerial won (boolean): whether the shot was the product of winning an aerial duel.
- Follows dribble (boolean): whether the player performing the shot performed a dribble right before.
- First time (boolean): whether the shot is the first touch of the player performing the shot.
- Open goal (boolean): whether the shot if performed with an open goal.



### Removing events that were not relevant to our model

We noticed that our model had a lot of trouble dealing with some particular shots. Once we analyzed them we noticed that they were penalty and direct free-kicks. For this model we are only interested in open plays, so we decided to remove all the events that were not related to open play.

We might integrate them in the future, but we have a few issues with them in the current implementation:
- We don't have enough data on them to train a model successfully.
- The model inputs the global features at the end of the data flow, and it's processed only by linear layers (and non-linear activation function). Therefore, the expressivity of the model regarding these features is limited, given the complexity of the model itself.


## Results

Results on the test subset can be seen in the following scatter plot.

![correlation](/images/correlation_xg_2.png){:class="img-responsive"}

As it can be seen, both metrics are heavily correlated. 

sets|Spearman corr|Pearson corr
---|:---:|:---:
All shots|0.900|0.858
Just goals|0.818|0.813
Just not goals|0.893|0.837



### Comparison with StatsBomb's xG

We also compared our model's output with StatsBomb's xG measure. We used the same data as in the previous post, i.e. the test subset extracted for the whole dataset.

As in the previous post, we use Brier score and AUC to compare the two metrics. Here we also introduce the Crossentropy between both approximations of xG and the empirical distribution of goals ([Wikipedia page](https://en.wikipedia.org/wiki/Receiver_operating_characteristic)).

model|Brier score|Crossentropy|AUC
---|:---:|:---:|:---:
StatsBomb xG|0.0730|0.2561|0.8128
our XG|0.0727|0.2535|0.8207

These results do not allow to state that one model is better than the other, but they do show that our model is not worse than StatsBomb's xG. We will explore the differences:

#### Distribution of data

![distribution_all](/images/dist_all_2.png)

![distribution_goals](/images/dist_goals_2.png)

![distribution_non_goals](/images/dist_nongoals_2.png)


#### examples


Let's considerer some cases from top left corner of scatter plot:

Goals:

1. Lyon - Monaco 2015-16\
StatsBomb xG: 0.60025674\
our xG: 0.89113533\
[link](https://www.youtube.com/watch?v=7p2QWIr7bjU)\
Fifth goal, second half from corner\
![goal_lyon](/images/goal_lyon_monaco.png)


2. Lazio - Udinese 2015-16\
StatsBomb xG: 0.3183885\
our xG: 0.8764055\
[link](https://www.youtube.com/watch?v=u3L97sVMy-U)\
First goal\
![goal_lazio](/images/goal_lazio_udinese.png)


3. Crystal Palace - Chelsea 2015-16\
StatsBomb xG: 0.59007293\
our xG: 0.8907189\
[link](https://youtu.be/d14Xk6c_fU8?si=CmuxwdyHR96uqU9i&t=4116)\
Third goal by Diego Costa\
![goal_chelsea](/images/goal_chelsea_crystal_palace.png)


4. Arsenal - Watforf 2015/16\
StatsBomb xG: 0.47959393\
our xG: 0.9349653\
[link](https://www.watfordfc.com/video/match-highlights/highlights-arsenal-4-0-watford-premier-league-201516#play)\
Last goal by walcott\
![goal_arsenal](/images/goal_arsenal_watford.png)


Not goals:

1. Atalanta - Roma 2015/16\
StatsBomb xG: 0.3948238\
our xG: 0.8656543\
(couldn't find video from it)\
Džeko shot at minute 81\


2. Crystal Palace - Aston Villa 2015/16\
StatsBomb xG: 0.12405383\
our xG: 0.86449665\
(couldn't find video from it)\
Bakary Sako shot, minute 47\



