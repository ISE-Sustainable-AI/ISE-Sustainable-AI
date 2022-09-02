---
title: Experiments
layout: default
nav_order: 3
use_math: true
---

# Experiments
{: .no_toc }
As part of the project we continuesly collect and publish results form the hyperparameter optimization benchmarks.
In the following we preesnt a brief overview of all currently availble experimentes we perfom using [**basht**](./00_bashed). 
The first set of experiments aims to evaluate the  effects on changing
parallelization, execution environment, trail duration, and sampling
strategies. Through the unified tooling, we can foster *repeatability,
fairness, understandability* and *portability* for all proposed
experiments. We publish all used experiment configurations here.



## Table of contents
{: .no_toc .text-delta }
1. TOC
{:toc}
### Base-Workload

We compare all experimentes with a baseline workload.
For that we the MNIST  dataset $$(t={MNIST})$$ for
digit classification performed by a simple feed-forward network
($$m={ff}$$), with the ADAM optimizer ($$ta=ADAM$$) . The network uses
linear layers varying in size $$ls$$ and number $$ln$$, weight-decay
($$decay$$) and learning rate ($$lr$$) as its hyperparameters. The parameter
search will be conducted using grid search ($$sa={gridsearch}$$) without
pruning ($$pa=\emptyset$$) and the equivalent of four similar-sized
workers with: $$wn=4, wcpu=2, wmem=2048, at=CPU$$, each without any
accelerator hardware configured. We want to stress that the workload
configuration shall not be mistaken with a hyperparameter configuration
$$\phi$$.

### Resource Impact Experimentes
To explore the effects of changing resources and parallelization, we
utilize base-workload under changing the scale of available worker
resources. We perform this experiment in four ways: 
1. **R-MEM:**
vertical memory scalability, we keep four workers but increase $$wmem$$ in
128MB intervals until *JCT* converges. 
2. **R-CPU:** vertical CPU
scalability, we increase vCPU per worker until *JCT* converges 
3. 
**R-ACL** accelerator scalability, we add different types of
accelerators (TPUs, GPUs) to each worker and 
4. **R-Node:** horizontal
scalability, we increase the number of available workers. 

For each experiment we are interested changes in *Trail-Wattage* in combination
with changes in *optimization cost* and *JCT*. As a hypothesis, we
assume that: *The target deployment platform for each SUT strongly
influences the ideal scaling configuration regarding energy efficiency.*

### Search Impact Experimentes
One impacting factor on the *HPO* process is the used sampling and
pruning strategies. Thus, we propose three experiments:
1. **S-Sam**
sampling impact, we vary the sampling strategy[^1] ($$sa$$) 
2.  **S-Pru**
pruning impact by enabling available pruning strategies ($$pa$$) 
1. **S-$$\Phi$$** search space impacts, by reducing and increasing the search parameter space ($$ps$$). 

For each experiment we are
interested changes in *Waste* in combination with changes in *JCT* and
*optimization cost*. As a hypothesis, we assume that: *Changes in the
search space, sampling, and pruning approach strongly impact the waste
and thus overall energy efficiency of HPO processes.*

<br/>

These experiments are by no means complete, and we foresee adding more
targeted experiments once we explore a large number of SUT. However,
these already enable the first approach to correlate impacts on energy
efficiency, carbon emissions, and wasted operations to steps and
configurations in the HPO process.

----
[^1]: dependes on availability for each SUT
