---
title: "📝 Seminar Artificial Intelligence"
date: 2024-09-25T13:45:31+02:00
author: "Sirius14"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Continual Learning: Mitigating Catastrophic Forgetting and Expanding Applications in Distributed Systems"
summary: "Continual Learning: Mitigating Catastrophic Forgetting and Expanding Applications in Distributed Systems"
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

University of Klagenfurt, Faculty of Technical Sciences,Summer Semester 2024, Seminar in Artifical Intelligence

## Abstract

In recent years, artificial intelligence has been used in several areas such as autonomous cars, finance, and medicine. Nonetheless, datasets are becoming increasingly important, their size increases and the time to learn machine learning models increases too. Furthermore, generate new datasets in real time and has trained at any moments could be very efficient for capability of the model’s prediction. To improve them, we need to find other techniques which enable us to go faster while maintaining a good capacity for predictions. Although principles of incremental learning appear to be simple, we are confronted with a catastrophic forgetting problem. All the previous knowledge is lost when a new learning process begins. From a case study of algorithms to real-world impact, this paper explores solutions to avoid catastrophic forgetting problems using specific algorithms and some improvements to make continual learning an alternative to deep learning, ending with a discussion of how continual learning can emerge as a viable alternative to deep learning in certain contexts and their potential limitations.

## Introduction

Artificial intelligence is a vast domain that covers plenty of usage and various needs. Some sectors like intelligent cars are quickly evolving with the emergence of new heavy datasets and new training data which enable to improve the prediction of the model. In this domain, predictions need to be precise, indeed cyclists should not be recognized as motorbikes otherwise it could be difficult for the model to make a good decision and it may cause accidents. Intending to improve the prediction, we need to constantly relearn with new data, a new example of the situation.

AI and more precisely Deep Learning have already changed several aspects of our lives. Whether it is recommendation algorithms transforming how we discover content or personalized services adapting to our preferences or in healthcare with to detect disease and supporting doctors in their decision-making \[1\]\[2\].

However, even though these deep learning models have better results, training time is just increasing due to a new complex dataset. Moreover, the prediction could be enhanced with a dataset generated in real-time. Considering this fact deep learning models are not conceived to be trained with continual new data, they need to train from scratch with only one big task containing all the datasets.

Continual learning, also referred to as task incremental learning or lifelong learning, emerged as a response to this challenge. Consequently, we need to find a model that allows for continuous learning and accelerates the learning process.

Incremental learning has been implemented, but using continuous learning to acquire new data leads to catastrophic forgetting. Naturally, as the model learns new data, it endeavors to center its predictions around the newly provided data. Deep learning does not have this problem since it learns from scratch each time and concatenates the new data with the previous ones.

The seminar paper will be a state-of-the-art of all the advancement of Incremental Learning. Explaining which strategies are used by algorithms to avoid catastrophic forgetting problems. Then it will focus on distributed system opportunities and all the benefits it brings compared to the classic Deep Learning model. Next, we will study the hyperparameter's impact on the incremental learning model and if there are any methods to improve the learning process by choosing the correct ones. Finally, we will discuss the future of the Continual Learning model, and the impact and challenges it should bring to scientific research.

## Background

### Deep Learning

To understand the stakes of Continual Learning, we need to explain how DNN works briefly. Deep Learning belongs to the machine learning family based on neural networks (NN) and has existed since more than 50 years. Deep neural networks are used for plenty of usage, in this section, explanations will be focused on how DL works with supervised learning. Frequently in AI, we are confronted with the classification problem, we have plenty of data that we want to class, for instance in autonomous cars we can have pedestrian, car, bus, or bicycle classes. The goal is to determine to which classes belong the data, to do this job we use neural networks which give in output a prediction for each class. Then by taking the most probable class, we can solve the classification problem. Of course, these predictions could be wrong, so the model needs to be trained to avoid an error.

DL networks repose on human brain behavior, the model includes different types of layers:
+ Input layer which is the first one and takes some data represented in a vector of numerical values.
+ Output layer which is the last one and is also called the activation layer, this includes results of predictions on each neuron.
+ Hidden layer(s) which do the bridge between input, output, or other hidden layers, their goal is to concentrate and generalize the main component of an information

For example, an image with a 224 square size has a 224 * 224 * 3 input vector size (224 width, 224 height, and 3 colors per pixel). If we want to predict the class of this image between a car or a bus, we only have 2 outputs, so we need hidden layers between input and output layers to extract the main components.

Each layer is interconnected by parameters with weight. To improve the model, we need to update the weights. The goal of training the model is to feed him with data previously transformed on a numerical vector. When we compare the prediction to the ground truth, we obtain a loss which indicates how the model prediction is far from the ground truth. With this value, we can update the model weights using the backpropagation of the gradients.

![Deep Learning](/Sirius14_Blog/img/article/sem-ai-1.png)
*Figure 1: Representation of an artificial neuron (a) and Deep neural networks (b)*

One of the important things about NN is the function used to calculate neuron value, according to the context we can choose a ReLU function (between 2 hidden layers for instance) or a sigmoid function (principally used as an activation function which is also called logits) in particular this one discretizes the result and produce a value between 0 and 1 which can be used as a probability and enable to find the predicted class. Then we can add a bias that enables improved convergence and is independent of the preceding layer. 

The more we feed the model, the more it can be precise, but data needs to be different for avoiding overfitting of the model. On the contrary, not enough data could introduce an underfitting. Even though each one of NN structures is different (not the same number of parameters,…) and has various usages like image data input or text, the learning process still be the same, all the training data are concatenated in one big task. This causes a large time in the learning process, even if we can parallelize him to run on multiple nodes.

### Continual Learning and Catastrophic Forgetting

In the context of this seminar paper, we will be using various symbols to represent different mathematical entities. Note that, other symbols particular to a special algorithm will be introduced later. Here's an explanation of some of these symbols:

+ B: This symbol represents an entity called buffer B or episodic memory. In certain algorithms, a "buffer B" may be used to temporarily store information.
+ M: In this context, this symbol represents the model. The model used can vary but it involves a neural network model.
+ T: It represents a task, which consists of a set of classes presented through multiple mini-batches to the model during continual learning.

Before explaining what does involve continual learning (CL), let's introduce some vocabulary. A task includes several classes, in the deep learning (DL) case we only have one big task. On the different point of view, CL using the same models, structures, and goals. However, instead of learning one unique task as DL, we want to learn a set of tasks. The goal is to split learning into several tasks. This simple variation in DL system changes a lot the results. During the first task no major difference with DL, but when the after the second task we evaluate the model, we encounter a catastrophic forgetting \[3\], and this is the most problematic thing in CL. 

Indeed, catastrophic forgetting exists because during a task the parameters will be moving toward a local minimum of the loss function Fig. 2. Schema explaining catastrophic forgetting problem Furthermore, when the model begins to learn a new task, this local minimum will be moved to another part since input data is not the same. This is represented in Fig. 2. red cross is the local minimum, the blue arrow is the transition of the minimum between 2 tasks. 

![Catastrophic Forgetting](/Sirius14_Blog/img/article/sem-ai-2.png)
*Figure 2: Schema explaining catastrophic forgetting problem*

The second part of the figure shows that for the classification of classes belongs to the preceding task, the local minimum is no longer center so the classification will be worse. This behavior is what we want to avoid, this is a crucial point in CL research, and this is why nowadays we need to use a DL model. Having a learning process separated into different tasks each one containing a class can reduce the time to learn the model, in addition with distributed workflows it will be better. Thus, if we want to use CL, we need algorithms that avoid CF, our goal is to bring closer DL results.

Moreover, during the learning process we can split CL between streamed CL and batched CL. On the first one, the model is allowed to visit a training sample only once, so there is no idea of the epoch, it’s like a continuous stream of data that we can’t store each sample for a single use. Conversely, in batched CL data can be replayed as many times as we want, there a multiple epochs per task.

In addition to all this information, CL is splitting into a different type of scenarios first introduce \[4\], each one has its specificities, this will be illustrated with dogs and cats classification example:

#### Task Incremental Learning

The model needs to learn a new task on fixed output space without forgetting previous ones, new tasks add new examples and classes. This aligns with situations in which the identity of the task to be completed is provided both throughout the learning process and during inference. This is the only scenario that knows to which task a sample belongs. When it comes to dogs and cats that change their hue, the model is taught to know whether it must differentiate between white cats and black dogs or between black cats and white dogs, and this information is also provided at inference. This is the most straightforward option because it would be possible to use multiple models for the various jobs.

#### Class-Incremental Learning

The model needs to learn different classes in each new task, this new class does not appear in the previous task, this scenario is more complex than task incremental. For instance, a model for classifying animals in images could be taught with reptiles after first learning about mammals.

#### Domain-Incremental Learning

This scenario is focused on the context, the model learns the same kind of problem, and it needs to generalize and adapt itself through several environments. This scenario can be divided into two sub-scenarios based on whether or not the data distribution change is known during the learning process and each distinct distribution is therefore considered a task. It's critical to distinguish between approaches that require this knowledge and those that do not, as the latter could be utilized to improve model training. This situation is the same as the one with the color-changing dogs and cats that were previously described instances are first taken from one subset of all possible examples, and then they are taken from a second subset, causing the distribution of the examples to change.

### Metrics

Among plenty of metrics, we will focus on continual accuracy predictions 1 and 5 first, which refer respectively to the first and 5 predictions accuracy achieved for each new task or learning scenario encountered during the continual learning process. In other words, we take data from each class from every past task including the present task, and we compare the number of good predictions which has the highest percentage value from wrong predictions.

![Metrics](/Sirius14_Blog/img/article/sem-ai-3.png)
*Figure 3: Representation of an output layer (a), with several tasks for explaining metrics (b)*

To illustrate how continual accuracy prediction 1 metrics is, we will use Figure 3.

+ On the one hand (a), its simple output layer from a class is represented. Prediction 1 accuracy is the highest probability in red compared to the correct class in green, in our case (a) we have 100%. If we want to consider prediction 5 accuracy, it will be the 5 highest probability, so in this example each neuron (since there are only 5 neurons in this figure).

+ On the other hand (b), to obtain continual accuracy metrics, we compute the average of the highest probabilities for each class within each task in red compared to the correct class in green. Therefore, for the most part (b) we have two correct predictions and two wrong predictions, the average is 50%, and this is our metric value. Through this process, we achieve a continual accuracy prediction of 1. To attain a continual accuracy prediction of 5, we similarly average out but for the 5 highest probabilities.

By averaging the continual accuracy prediction for each task, we obtain the Final Average Accuracy (FAA). This metric is the most used to compare algorithms between them.

Another important metric is Final Forgetting (FF), which evaluates the amount of information lost during the learning process. It’s calculated for a task Ti by taking the difference between the final accuracy of the task and the accuracy measured when the task Ti was initially learned. By averaging the forgetting for each task, we obtain FF.

FAA and FF are complementary, ideally, we need to find an algorithm that maximizes FAA and minimizes FF.

## Current Methods and Approaches

### Approaches

As shown in \[5\], most of these existing methods can be grouped into four general strategies which are Regularization, Dynamic Architectures, Rehearsal, and Generative Replay:

#### Rehearsal

This method is a very common way of thinking, if the class of the first task has bad predictions, we can create a buffer B which can be added to the input data. This buffer B gives older data from previous tasks, to sum up, this method consists of storing raw data samples in a buffer B, to be replayed when training on a new task.

#### Regularisation

The idea of regularization is to adjust the parameters update rule as a task is learned. One way to achieve this is to discontinue the learning process before convergence while learning a new task or to use a low learning rate for specific parameters. The goal is to restrict how much the DNN adapts to the new task in order to prevent prior duties from being forgotten. Using knowledge distillation or using a model's outputs as target vectors for another model, is another regularization technique. Typically, it is applied by using the model's outputs for a few examples before learning a new task, and then using those same examples as target vectors for the model during the learning process.

#### Architectural based

Parameter isolation or architectural-based method focuses on the dynamic change of the model architecture. This approach includes progressive neural networks (PNN), and each task has sequential architecture, after the training process, this task creates a column T1 and is added to the PNN model. Then, for the task, we froze the weights from the previous columns and added column Tn to the model. For each new task, we add adapter layers, which are non-linear lateral connections enabling dimensionality reduction and enhancing initial conditioning. This method has the advantage of keeping intact results from previous tasks, reducing CF. It does not require a buffer B to store previous inputs. Nonetheless, this approach could increase resources during the learning process because the model becomes more and more complex, asking for more computational power and time, and the model tends to grow the number of parameters. This method is most suited to task-IL scenarios to identify which part of the DNN should be used.

#### Generative based

In order to create fake examples of the previous tasks, generative models are trained concurrently with the main model. Then, these made-up examples can be applied like the rest of the examples saved in rehearsal-based techniques. This strategy requires less memory than rehearsal-based techniques but has the drawback of being more computationally expensive. An interesting solution based on this approach is a replay through feedback RtF \[6\].

### Algorithms

For this part look at the following article : https://xelon-coder.github.io/Sirius14_Blog/article/continual-learning/

#### eXtended Dark Experience Replay (XDER)

According to DER and DER++, extended dark experience replay is an improvement and is mainly made for Class-IL scenarios. One of the drawbacks of CL algorithms is that they overstate the classes of the current tasks \[7\]. If the model learns the eighth task, it would have better results on Tc-1 th task than Tc-n with n > 1 and Tc current Task. To avoid this issue and spread the amount of information retained for each task, instead of simply adding previous logic. They re-scale logits from the current task to avoid that they delete or degrading information from previous tasks. XDER implements a new approach by preparing future output layers to improve new tasks' arrival and unseen classes. The buffer B is used for updating the model on seen and unseen tasks trying to predict its value before training the model. Another point implemented with this algorithm is to restrict activation of the soft-max layer on current task classes, with this method we prevent bias on previous tasks because of heavier gradients (there is no real impact on their logits). By comparing XDER with DER++ they obtained results that outperformed other approaches on a given dataset. However, it is important to note that when buffer B size increases, the gap gets smaller.

The results obtained with miniImageNet with a buffer size of 2000 \[8\] are 28.19% of FAA and 20.45% of FF.

## Cloud Integration and Hyperparameter Optimization in Continual Learning

### Leveraging Cloud Resources

Continual learning is close to the online learning notion, whenever it’s two different paradigms they still have the same concept of data arriving in a continuous stream. This commonality underscores the importance of real-time adaptation in machine learning systems. With the advancement of distributed systems and cloud computing \[9\] \[10\], lifelong learning is ready to enhance performance and scalability during the training process. Having great accuracy is not the only goal of continual learning, using the incremental learning process, there is a performance aspect contrary to Deep learning where the learning process with amount of data and can take a lot of time to be trained. Less data opens to more flexibility, scalability, and low data resources. This is ideal for IoT applications.

In a recent paper, Thomas and al \[11\] studied the advantage of a rehearsal-based approach for distributed systems. Firstly, they are using an innovative strategy to train the model with data parallelism. Instead of using one model, they create multiple replicas, each one running on a different GPU. Each model is trained with a mini-batch and updating the model is done by computing the average gradient. With this method, the learning process is performed in a synchronous way reducing the learning time. Secondly, they created an augmented mini-batch, this kind of mini-batch is composed of examples from previous tasks called representatives and examples from the new task. Then they choose a small sample from this augmented mini batch called candidates and put them into the buffer B, replacing considering a given strategy (random, LIFO, FIFO,…) the buffer B if it is full. Finally, they combined this idea of an augmented mini-batch for each node (each instance of the model) to have a distributed rehearsal buffer B, enabling more memory space than a centralized buffer B. Results obtained with ER algorithm are interesting, with (ResNet-50 running on ThetaGPU with dataset ImageNet-1K (4 tasks), with |B| fixed to 30% of the input dataset on 32 nodes (GPUs)) reducing the learning process by 2 and performing 80.23% of Top5 value accuracy against 90.90 for the non-continual learning approach (from scratch).

![Metrics](/Sirius14_Blog/img/article/sem-ai-4.png)
*Figure 4: Asynchronous updates of the rehearsal buffers and global augmentations*

### Hyperparamters Optimization

One of the most important things not to overlook in Neural Network models is hyperparameters. Especially in Continual Learning where the number of hyperparameters is high due to algorithms implemented against catastrophic forgetting. That one controls the training process and has a huge impact on the models. Among these hyperparameters some of them like the learning rate are very important to fix it define how quickly the model ought to adjust to data changes. If the learning rate is too high, then the model will have a quick adaptation but will forget quickly too, and in continual learning, we want to avoid forgetting. So, we must find a perfect value that is small enough to avoid forgetting and large enough for our model to adapt to changing data. The size of the Mini batch has also a significant impact, if the size is too large this could lead the model to converge to a local optimum introducing a lack of generalization. On the other hand, small size has a strong impact on the learning process, reducing parallel computation and making loss efficiencies through material resources.

Additionally, finding a correct combination of several hyperparameters could be very hard and long. Normally to optimize them: 
> solutions are typically designed to operate under the assumption of independent and identically distributed samples obtained all at once. \[12\].

Nonetheless, we are supposed to be on an incrementally training-based, thus we don’t have one big task, so in order to fine-tune hyperparameters we must find another approach. A recently adopted strategy explained on \[12\], assumes to use of adaptive hyperparameters depending on the previous task. Foremost, on the first few tasks they apply a classic optimization using fANOVA a framework shown by Hunter and al. \[13\]. The examination of hyperparameter relevance made possible by this methodology provides insightful information about the variables affecting performance. For the rest of the tasks, they 
> select the k most important parameters to be changed and keeps fixed the others with the optimal value computed in the previous task \[12\].

Results obtained are quite encouraging, even though this strategy does not outperform classic hyperparameter optimization in terms of accuracy and has sometimes worse results than the latter. It looks much better about the running time during both the learning process and optimization, compared to the classic approach, the adaptive one takes almost half of the time depending on the algorithm and dataset. As always in continual learning, it’s a compromise to be made.

## Discussion

### Future and Impact

This area of research is vast and each year new algorithms arrive, but we are still in the early stages. In this paper, we explored how to combat the catastrophic forgetting issue with dedicated algorithms. The impact of these presented algorithms is still underperforming against DL but with parallelization of calculations and a learning process which don’t incrementally this type of machine learning could have a real impact on the domains like medicine, finance, or autonomous car. With a faster training phase and incremental learning, it’s also cheaper, and we can create new data in real-time to improve the model without training from scratch.

Other challenges \[14\] that incomes are the buffer B size, with new tasks we need to fill the buffer B constantly with new data. However, in some scenarios, to keep the speed and memory efficiency during the training, the buffer B size ideally does not grow.

Medicine is a promising area for continual learning. Indeed, with deep learning detection one model is specified for one disease. CL can bring new ways to create models that can classify several diseases and learn to find new ones. In \[15\] Xiaojie Li and all explore generative replay method for Task-IL and Class-IL scenarios. Training the either generator or classifier with the previous task. Combined with a feature extractor already pre-trained but not updated during the learning process, they obtained encouraging results with higher accuracy and lower forgetting than EWC and DGR. This paper only focuses on reducing CF and not improving knowledge transfer.

We can see that almost all methods are inspired by how biology works and how humans learn. It can pass with replay functionality common to a lot of approaches, or the use of penalty to rescale the model.

### Evaluation and Limitation

The existence of several algorithms to avoid CF is necessary. Independently of the given results they can offer, they proposed different strategies. The replay (or rehearsal) method needs memory but seems to have better accuracy than generative replay approaches. The regularization method seems to enhance the number of updates and parameters. Considering \[16\] REMIND appears to be a good trade-off between accuracy, memory, and updates. Nevertheless, these results depend on the given dataset chosen, in other conditions or with other major goals other strategies could be adopted.

Regarding, scientific works, Continual Learning is not so simple to understand, each paper introduces new notions and compares their approaches with different datasets (between 2 papers). Some papers like \[17\] \[18\], give their hyperparameters and other information used for reproducing results whereas others just explain the baseline with brief comparison. Due to different implementations, we should keep in mind that the outcomes may not yield directly equivalent results. The fact that CL covers several tasks and domains adds to this complexity, necessitating customized strategies for every situation.

There is not only a CF issue that limits CL. To avoid CF and make sure that the model has a good learning curve, it may tend to overfit the model to old data which can prevent the model to generalized. The complexity of setting up special algorithms that which makes the model complex, and which can become difficult to implement. Eventually, keeping old and new knowledge balanced between previous and new tasks is quite challenging.

## Conclusion

This area of research is still recent; therefore, it does not have a lot of articles published. Despite the numerous challenges to overcome, CL opens new opportunities for several domains including medicine, cybersecurity \[19\], and autonomous cars. Each year, new algorithms are created and try to bring another solution to reduce CF. In this paper, we have explored various challenges of CL including improvements using cloud computing or hyperparameters optimal. Continual learning can have a real impact on our society, reducing the can aid in lowering deep learning's financial and environmental expenses. It can be helpful for the adaptability in real life. In this paper we explore CL challenges and their impacts, highlighting the need for various strategies to combat Catastrophic Forgetting the complexity of research and implementation, and the potential benefits for fields such as medicine, cybersecurity, and autonomous cars.

## Source

\[1\] N. D. Thong Tran, C. K. Leung, E. W. R. Madill and P. T. Binh, "A Deep Learning Based Predictive Model for Healthcare Analytics," 2022 IEEE 10th International Conference on Healthcare Informatics (ICHI), Rochester, MN, USA, 2022, pp. 547-549, doi: 10.1109/ICHI54592.2022.00106.

\[2\] R. Sharma, S. Rani and S. Tanwar, "Machine Learning Algorithms for building Recommender Systems," 2019 International Conference on Intelligent Computing and Control Systems (ICCS), Madurai, India, 2019, pp. 785-790, doi: 10.1109/ICCS45141.2019.9065538.

\[3\] T. Elvira, J. O. Couder, T. T. Procko and O. Ochoa, "Impacts of Catastrophic Forgetting: From a Machine Learning Engineering Perspective," 2023 Congress in Computer Science, Computer Engineering, & Applied Computing (CSCE), Las Vegas, NV, USA, 2023, pp. 01-08, doi: 10.1109/CSCE60160.2023.00010.

\[4\] van de Ven, G. M. and Tolias, A. S, "Three scenarios for continual learning", 2019, https://arxiv.org/abs/1904.07734

\[5\] T. Lesort, "Continual learning: Tackling catastrophic forgetting in deep neural", 2020, https://arxiv.org/abs/2007.00487
networks with replay processes

\[6\] Gido M. van de Ven, Andreas S. Tolias, "Generative replay with feedback connections as a general strategy for continual learning" 2019, https://doi.org/10.48550/arXiv.1809.10635

\[7\] M. Boschini, L. Bonicelli, P. Buzzega, A. Porrello and S. Calderara, "Class-Incremental Continual Learning Into the eXtended DER-Verse," in IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 45, no. 5, pp. 5497-5512, 1 May 2023, doi: 10.1109/TPAMI.2022.3206549.

\[8\] Arslan Chaudhry1, Albert Gordo, Puneet K. Dokania, Philip Torr, David Lopez-Paz. “Using Hindsight to Anchor Past Knowledge in Continual Learning”. In: 2021, https://doi.org/10.48550/arXiv.2002.08165

\[9\] C. Lee, S. -H. Kim and C. -H. Youn, "An Accelerated Continual Learning with Demand Prediction based Scheduling in Edge-Cloud Computing," 2020 International Conference on Data Mining Workshops (ICDMW), Sorrento, Italy, 2020, pp. 717-722, doi: 10.1109/ICDMW51313.2020.00103.

\[10\] C. Chen, K. I. -K. Wang, P. Li and K. Sakurai, "Flexibility and Privacy: A Multi-Head Federated Continual Learning Framework for Dynamic Edge Environments," 2023 Eleventh International Symposium on Computing and Networking (CANDAR), Matsue, Japan, 2023, pp. 1-10, doi: 10.1109/CANDAR60563.2023.00009.

\[11\] Thomas Bouvier, Bogdan Nicolae, Hugo Chaugier, Alexandru Costan, Ian Foster, et al.. Efficient Data-Parallel Continual Learning with Asynchronous Distributed Rehearsal Buffers. 2024 IEEE 24th International Symposium on Cluster, Cloud and Internet Computing (CCGrid), May 2024, Philadelphia (PA), United States. ⟨10.1109/CCGrid59990.2024.00036⟩. ⟨hal-04600107⟩

\[12\] Rudy Semola, Julio Hurtado, Vincenzo Lomonaco, Davide Bacciu, "Adaptive Hyperparameter Optimization for Continual Learning Scenarios" 2024, https://doi.org/10.48550/arXiv.2403.07015

\[13\] Frank Hutter, Holger Hoos, and Kevin LeytonBrown, "An efficient approach for assessing hyperparameter importance". in International conference on machine learning, pages 754–762. PMLR, 2014.

\[14\] H. Bae, S. Song and J. Park, "The Present and Future of Continual Learning," 2020 International Conference on Information and Communication Technology Convergence (ICTC), Jeju, Korea (South), 2020, pp. 1193-1195, doi: 10.1109/ICTC49870.2020.9289549.

\[15\] X. Li, H. Li and L. Ma, "Continual Learning of Medical Image Classification Based on Feature Replay," 2022 16th IEEE International Conference on Signal Processing (ICSP), Beijing, China, 2022, pp. 426-430, doi: 10.1109/ICSP56322.2022.9965230.

\[16\] M. Y. Harun, J. Gallardo, T. L. Hayes and C. Kanan, "How Efficient Are Today’s Continual Learning Algorithms?," 2023 IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops (CVPRW), Vancouver, BC, Canada, 2023, pp. 2431-2436, doi: 10.1109/CVPRW59228.2023.00241.

\[17\] Arslan Chaudhry, Marc’Aurelio Ranzato, Marcus Rohrbach, Mohamed Elhoseiny. “Efficient lifelong learning with A-GEM”. In: 2019, https://doi.org/10.48550/arXiv.1812.00420

\[18\] Pietro Buzzega et al. “Dark Experience for General Continual Learning: a Strong, Simple Baseline”. In: Advances in Neural Information Processing Systems. Ed. by H. Larochelle et al. Vol. 33. Curran Associates, Inc., 2020,pp. 15920–15930.13, https://proceedings.neurips.cc/paper/2020/file/b704ea2c39778f07c617f6b7ce480e9e-Paper.pdf

\[19\] J. Kozal, J. Zwolińska, M. Klonowski and M. Woźniak, "Defending Network IDS against Adversarial Examples with Continual Learning," 2023 IEEE International Conference on Data Mining Workshops (ICDMW), Shanghai, China, 2023, pp. 60-69, doi: 10.1109/ICDMW60847.2023.00017.