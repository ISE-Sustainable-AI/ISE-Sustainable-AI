---
title:Dependency Model
layout: default
nav_order: 4
use_math: true
---


# Dependency Model

{: .no_toc :}
A variety of factors influence observable software qualities in HPO.
We aim to unveil the underlying dependencies of these factors using the structure of the HPO process together with controlled experimentation and observation of these software qualities.
Herein we will review how observable metrics and dependencies can be connected. Towards that, we present a first tree-based dependency model for exploring the sustainability impacts of all dependencies in hyperparameter optimization.

{: .no_toc .text-delta }
1. TOC
{:toc}


<figure>
<img src ="/assets/graphics/dependency_model.drawio.png">
</p>
<figcaption align = "center"><b>Fig.1 - Dependency Tree for hyperparameter optimization.</b></figcaption>
</figure>

The root node of the dependency tree is always a relevant metric. In figure \ref{fig:kWh} this root node is trial-watt-hours, measured in kilowatt-hours, expressing sustainability of \textit{HPO} through average energy consumption per trial, thus with the tree in figure~\ref{fig:kWh} we aim to reveal cause-effect relations between the metric ($TWH$) and HPO decencies, such as used resources.
% Every subnode, except leaf nodes depict categories, which group other categories or leaf nodes.
Leaf nodes are configurable actions that contribute to changes within a measure (the root node) and incorporate a set of states, which can cause unfavorable or favorable outcomes.
These states can be numeric or nominal values (e.g., nominal: deep learning framework - $\{tensorflow, PyTorch, sonnet, mxnet, keras\}$, numeric - batch size: $124$).
We want to highlight that the presented dependency tree captures a fraction of dependencies that potentially influence $kWh$. E.g. for GPU, the respective processor (Tesla V100, Tesla T4) or the vendor (Dell, HP) and the number of GPUs that can be used as resources might also influence the energy consumption.

For instance, deep learning frameworks tend not to utilize CPUs while performing heavy workloads.
However, CPUs still consume energy while being idle \cite{li2016evaluating}, thus concluding a linkage in our dependency tree, leading to the inclusion of resources and resource types in the model.
Furthermore, the used HPO-Framework may also affect the $TWH$, as some frameworks use resource allocation plans in combination with the \textit{sampling} and \textit{pruning} strategies~\cite{dunlap2021elastic, misra2021rubberband} to minimize idle resources.


Higher utilization of computing resources may indicate more extensive consumption of natural resources through higher energy consumption.
Furthermore, these pruning strategies may potentially include natural resources as budget constraints when deciding about the cancellation of individual trials, thus, making them an excellent candidate to be included in the dependency tree.


Parallelization strategies used by different \textit{HPO} frameworks can also cause waste in the form of stragglers~\cite{misra2021rubberband}.
Stragglers are most likely caused by a combination of an unlucky hyperparameter configuration and poor parallelization or work distribution.
GPUs with differing computational power might be used for some already slowly converging hyperparameter configurations and thus lead to problematic trials. These trials are needed to be waited on, resulting in a waste of computing resources leading to potentially avoidable energy consumption, thus, we included parallelization as a key factor in the model.

We are aware that the presented dependency model is incomplete, however, it shall serve as a starting point to investigate the sustainability qualities of HPO frameworks and unveil cause-effect relations in a greater scope, allowing for future refinement of this model.
