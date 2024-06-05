---
layout: post
title:  "Follow-up on expected goals project: improvements on model and comparisons"
date:   2024-06-04 10:00:00 -0400
categories: jekyll update
---
# Improvements on model

#### TLDR
As shown in the previous post, the model used to predict the expected goals was promising, but still performed worse than StatsBomb measure of expected goals. Following some of the suggestions in the previous post, we will show how our model performs as good, if not better, than StatsBomb's.

### What changed from last results?
- We used all the event data available in database that fulfill two conditions. (1) it's male football, and (2) the entire competition is available (therefore we didn't use data where only a subset of games from a competition was available)
- We modified slightly the architecture of our model.
- We added some features available in the event data regarding the nature of the shot.
- We removed some events that were not relevant to our model (e.g. penalties, free-kicks).


### Improvements in the architecture

As mentioned, we are using a Graph Convolutional Neural Network of the subtype message passing. This kind of architecture are fairly small compared to state of the art LLMs, but still contains tens of thousands of parameters. The constraint on the amount of data for training and validation limited severely the amount of layers and the architecture decision in the design of the model.
We slightly modified the flow of information inside the model so that the output of model is calculated from the shoter's embedding as well as from the goalkeeper's embedding (previously was calculated only from shoter's embedding). 


### Adding of "global" features

We called "global" features to those that don't correspond to a single node of the graph, but to the whole graph as a whole. In this case, these features are of categorical natural present in the dataset, that attempt to add information about the nature of the play that preceded the shot. These features are:
- Play pattern: description of origin of the play (e.g. regular play, from corner, from throw-in, etc.)
- Technique: description of the technique used to make the shot (e.g. normal, volley, diving-header, etc.)
- Body part: part of the body used to make the shot (e.g. head, right foot, etc.)
- Aereal won (boolean): whether the shot was product of winning an aereal duel.
- Follows dribble (boolean): whether the player performing the shot performed a dribble right before.
- First time (boolean): whether the shot is the first touch of the player performing the shot.
- Open goal (boolean): whether the shot if performed with an open goal.




