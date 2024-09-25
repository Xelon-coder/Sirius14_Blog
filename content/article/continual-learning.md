---
title: "📝 Continual Learning: Introduction"
date: 2024-01-29T14:46:56+01:00
tags: ["AI","Article","Pytorch","Python","Continual Learning"]
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Accelerating AI/ML Workloads with continual learning"
summary: "Accelerating AI/ML Workloads with continual learning"
disableHLJS: false # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: false
ShowWordCount: false
ShowRssButtonInSectionTermList: false
UseHugoToc: true
pygmentsUseClasses: true
pygmentsCodeFences: true
---

This article is a summary of continual learning algorithms explaining how it could be a game changer over the next few years.

## Intro

Artificial intelligence (AI) is a vast domain which covers plenty of usage and various needs. Some
sectors as intelligent cars are quickly evolving with the emergence of new heavy datasets and new
training data which enable to improve the prediction of the model. In this domain, predictions need to
be precise, indeed cyclist should not be recognize as moto otherwise it could be difficult for the model
to choose the good decision and it may cause accidents. With the aim of improving the prediction, we
need to constantly relearn with new data, a new example of the situation.

However, even though these deep learning (DL) models have better results, training time is just
increasing due to a new complex datasets. Moreover, prediction could be enhanced with dataset generated in real time. Considering this fact DL models are not conceived to be trained with continual
new data, they need to train from scratch with only one big task containing all the datasets. Consequently, finding a model which enables to learn continuously and to accelerate the learning process.
Incremental learning has been implemented, nevertheless, utilizing CL to acquire new data results in
catastrophic forgetting. Naturally, as the model learns new data, it endeavors to center its predictions
around the newly provided data. DL doesn’t have this problem because it learns from scratch each
time and concatenates the new data with the previous ones. Thus, it could be interesting to find How
to accelerate learning process and avoid catastrophic forgetting?

## Continual Learning

Before explaining what does involve continual learning (CL), I need to introduce some vocabulary.
A task includes several classes, in DL case we only have one big task. On the different point of view,
CL using the same models, structures, and goal. However, instead of learning one unique task as DL,
we want to learn a set of tasks. The goal is to split learning in several tasks.

![Catastrophic Forgetting](/Sirius14_Blog/img/article/cl_1.png)
*Figure 1: Schema explaining catastrophic forgetting problem*

This simple variation in DL system changes a lot the results. During the first task no major difference with DL, but when the after the second task we evaluate the model, we encounter a catastrophic
forgetting (CF), and this is the most problematic thing in CL. Indeed, CF exist because during a task
the parameters will be moving toward a local minimum of the loss function. Furthermore, when the
model begins to learn a new task, this local minimum will be moved to another part since input data
are not the same. This is represented on Figure 1, red cross is the local minimum, the blue arrow is the transition of the minimum between 2 tasks. The second part of the figure shows that for the
classification of classes belongs to the preceding task, the local minimum is no longer center so the
classification will be worse. This behaviour is what we want to avoid, this is a crucial point in CL
research, and this is why nowadays we need to use a DL model.

Having a learning process separated in different tasks each one containing class can reduce the time
to learn the model, in addition with distributed workflows it will be better. Thus, if we want to use
CL, we need algorithms which avoid CF, our goal is to bring closer DL results.

## State of Art

There are two main approaches to conceive CL algorithm:

+ Rehearsal: this method is a very common way of thinking, if the class of the first task has bad
predictions, we can create a buffer which can be added to the input data. This buffer gives older
data from previous tasks, to sum up this method consists of store raw data samples in a buffer,
to be replayed when training on a new task.

+ Regularisation: this approach is based on avoiding overwriting the knowledge previously acquired in adding some constraint to guide the model on the weights, on the probabilities, on the
gradients, on the features.

It exists other method but, in this internship, I only use a rehearsal or regularisation or a blend of
them. So, I begin to study some scientific papers on these types of algorithms, this paper was very
recently from 2017 to 2021. Each one detail the how does it work with numbers and compares to each
other, however it was hard to have the same result on the same datasets, conditions of experimentation
always change and it was complicated to find algorithms better than the other.

### Experience Replay ([ER](https://arxiv.org/pdf/1811.11682.pdf))

So as Figure 3, show this implementation consist of concatenated the actual train mini batch (batch form the current task) with
a buffer mini batch. For each new task, we store in the buffer some representatives of previous class
from the previous task. This percentage can be changed to add more diverse example, this is possible
to write the number of buffers mini batch which is concatenated. This algorithm was like a baseline
for me to evaluate performance and assess the potential for improvement over this version.

![Er](/Sirius14_Blog/img/article/cl_3.png)
*Figure 3: Er schema*

###  Averaged Gradient Episodic Memory ([A-GEM](https://arxiv.org/pdf/1812.00420.pdf))

This time, this is a new algorithm in its methods, A-GEM is a regularization based so the model
update rule is modified. Even though we can see a buffer in Figure 4, his role, his just to guide the model and not to train.
It begins like DL, the model is trained on the current mini batch Mtask (or
NNxy), then a copy of this model is trained on some representatives from the buffer, and we called
this model Mbuffer (or NNrb). With these two models, we perform a scalar product in order to view if
there are similar, to know if the knowledge doesn’t disappear.With a negative scalar product, we keep
the current task, otherwise we make a projection of the Mtask model on Mbuffer. So this time buffer
is used for evaluating and correct the update current model. To compare with ER model, we have:

+ Constraint when updating the parameters of the Mtask, mitigating catastrophic forgetting.

+ No proper rehearsal but a rehearsal buffer is still needed to obtain Mbuffer.

![Agem](/Sirius14_Blog/img/article/cl_4.png)
*Figure 4: Agem schema*

###  Hindsight Anchor Learning ([HAL](https://arxiv.org/pdf/2002.08165.pdf))

This approach seems to be relatively good, resembling an enhancement of
ER. This algorithm is both rehearsal and regularization based, indeed we keep ER implementation
with the buffer, but we are adding a constraint during update of the model. We want to limit the
forgetting on historical data points representative of a class (called anchors).For each class of each
task, we have one anchor, it encodes points that would maximize the catastrophic forgetting. Thanks
to the anchors, the model cannot diverge too much using this method. These are used in the update
calculation and are calculated with at the end of each task. Creating anchors is linear because you
don’t need to re-calculate them, nevertheless all the anchors are used to update the model. If we
compare to ER contrary to A-Gem, we have also similarities:

+ System of anchors differs to ER and bring the regularization method to this algorithm.

+ The buffer is still concatenated to the current task buffer so the same thing as ER.

![Hal](/Sirius14_Blog/img/article/cl_5.png)
*Figure 5: Hal schema*

### Dark Experience Replay and improvement ([DER](https://arxiv.org/pdf/2004.07211.pdf)/[DER++](https://arxiv.org/pdf/2201.00766.pdf))

Before explaining the how it works, I need to briefly explain what knowledge distillation is.

#### Knowledge distillation (KD)

KD is separated in two models, teacher and student. The student
model delivers predictions. There are 2 kinds of predictions:

+ Soft predictions are a tensor of probabilities also called lofts, in fact the last layer of our model,
the step before we predict the class.

+ Hard predictions are the step after, the class predicted by our model.

![Knowledge distillation](/Sirius14_Blog/img/article/cl_6.png)
*Figure 6: Knowledge distillation schema*

Then we can calculate the loss of each one of these predictions, we combine distillation loss and student
loss into an only loss value using coefficient to regulate if we prefer to highlight one loss compared to
the other ones. As you can observe in Figure 6 we have a green circle and a red circle, this means that
DER algorithm only used distillation loss and not combine to student loss in green. In red we have
DER++ which add student loss.

#### [DER](https://arxiv.org/pdf/2004.07211.pdf)

This algorithm is rehearsal and regularization based. It complements rehearsal with an
additional knowledge distillation objective, to the difference between ER is that we store logits (soft
labels) alongside its representative in the buffer instead of relying on a single class output (hard label).
Thus, we need to modify the actual buffer implementation (which is the buffer’s one). Regularization
aspect is brought by distillation loss that we add to current mini batch loss. We still have difference
with an ER model like:

+ Representatives are stored alongside their logits (activations of last layer).

+ Evaluate the model on both older logits and current ones (knowledge distillation).

![Der](/Sirius14_Blog/img/article/cl_7.png)
*Figure 7: Der schema*

#### [DER++](https://arxiv.org/pdf/2201.00766.pdf)

DER++ This is an improvement of DER, this algorithm is still rehearsal and regularization
based, in a nutshell, ER uses hard labels only (class outputs), DER uses soft labels only (logits).
DER++ uses both. Conceptually nothing changes between DER and DER++ except the buffer
implementation as a matter of the fact we need to store labels and logits, so in contrary of DER we
store an additional component, which leave us less size for representatives.

![Der++](/Sirius14_Blog/img/article/cl_8.png)
*Figure 8: Der++ schema*

## Conclusion

Continual Learning algorithms concept is still young and Machine learning algorithms continue to outperform them in terms of performance. However usage is not the same, and may be in 10 years catastrophic forgetting will no longer be an issue. Then CL could be very useful in our daily lives. This area of research is vast and each year new algorithms arrive, but we are still in the early
stages. Over this report I explore the implementation and deployment part of a distributed testbed.
The impact of these presented algorithms is still underperforming against DL but with parallelisation
of calculations and a learning process which don’t incrementally this type of machine learning could
have a real impact on domain like medicine, finance, or autonomous car. With a faster training phase
and incremental learning, it’s also cheaper, and we can create new data in real time to improve the
model without training from scratch.

## Source

Every explaination of this article was based on research paper and represent the culmination of an internship I had the opportunity to undertake. The result I obtained as well as implementation are not available in this blog. If you want more info you can DM me in Discord.

ER : https://arxiv.org/pdf/1811.11682.pdf

A-GEM : https://arxiv.org/pdf/1812.00420.pdf

HAL : https://arxiv.org/pdf/2002.08165.pdf

DER : https://arxiv.org/pdf/2004.07211.pdf

DER ++ : https://arxiv.org/pdf/2201.00766.pdf

Continual Learning Pytorch Framework : https://github.com/aimagelab/mammoth