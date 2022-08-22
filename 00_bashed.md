---
title: Basht
layout: default
nav_order: 2
use_math: true
---

# Basht - A **B**enchmarking **A**pproach for **S**ustainabile **H**yperparameter **T**uning
{: .no_toc :}
To aid repeatability and relevance, we created Basht a benchmarking tool to evlaute the sustianability impacts of hyperparameter optimization framework configurations.

## Table of contents
{: .no_toc .text-delta }
1. TOC
{:toc}

## Design

We implemented in python available through [pip](TODO) and [GitHub](TODO). It
consists of a core interface representing the *HPO* experiment process
of *deploy, setup, trail, result collection, metrics collection*, and
*undeploy*.

Further, Basht offers a set of metrics collectors and exporters to collect and
store all measurements from Prometheus and measure latencies through a
set of python-based annotation wrappers. Once collected, we compute all
metrics discussed in Table 2.

Basht also includes a set of experiment definitions in terms of workload
configurations that can be used to investigate different aspects of the
cause-effect relations we depicted in the dependency model. To run an
experiment, we run on a dedicated experiment host, which deploys all the
facilities to deploy the experiment and collect metrics. Once the
deployment runs, the host acts as the coordinator, e.g., ensuring that
all components are ready before starting the optimization. We use a
file-based experiment description that describes the workload as a
simple JSON or YAML file. This file, the used implementation, and
results should always be committed back to the -Website.


## Architecture 

Since we include both frameworks with automatic resource deployment and
frameworks that require prior resource deployment and assignment, we
need a process to measure the execution of the optimization and all
necessary steps to enable the optimization.


![](../../assets/process.svg)

{: .text-center :}
**Figure 1**: Basht Benchmarking Process 


For that, we broke down the *HPO* benchmarking process into six distinct
steps, see Figure 1. In the
following, we briefly discuss each step regarding the necessary actions
performed.

#### Deploy
Includes all deployment operations necessary to run the required
components of a *HPO* framework. With the completion of this step, the
desired architecture of the *HPO* Framework should be running on a
platform, e.g., in the case of Kubernetes, it refers to the steps
necessary to deploy all pods and services in Kubernetes. Note that this
depends on the *SUT*s targeted deployment platform, e.g., if the *SUT*
is targeted for Kubernetes, we do not include the time to deploy and
configure the Kubernetes cluster, just the pods to run the *HPO*
framework. If the *SUT* uses virtual machines as the deployment unit, we
include the time to deploy, configure and provision them. **After this
step, the *SUT* should be ready to perform any arbitrary hyperparameter
optimization.**

#### Setup
All operations needed to initialize and start a trial. Thus,
configuration, assignment of workers, distribution of trading data,
configuration, and handover of relevant classes to all workers. **After
this step, the *SUT* should be ready to perform a specific workload as
part of the benchmarking experiment.**

#### Trial
A Trial defines the loading, training, and validation of a model with a
specific hyperparameter configuration. A hyperparameter configuration is
one combination of hyperparameters that can be used to initialize a
model, e.g., $$\{learning-rate=1e^-2, weight-decay=1e^-6\}$$. Note that we
make no assumptions on where or how the trial is performed. Thus, during
the experiment, we need to be able to track each trail carefully.

- Load: Primarily includes all I/O operations needed to provide a trial with the
required data and initializations of the model class with a certain
hyperparameter configuration. This depends largely on the used *SUT*,
e.g., in some cases, data is distributed during setup and directly
available to each trail. In other cases, all trading data is stored in a
central location, e.g., an S3 bucket in which each trail needs to load.

- Train:
The training procedure of a model computes model parameters to solve a
classification or regression problem. Generally, training is repeated
for a fixed number of epochs, whereas each epoch processes data
batch-wise. Here, the workload and used hardware will strongly influence
the time for training.

- Validate:
The model that was trained with a specified hyperparameter configuration
has to be validated on a validation dataset, which is different from the
training data. Metrics that capture the model performance (e.g.,
F1-Score, Mean Squared Error, Accuracy) will be applied together with
the model predictions over the validation dataset. The performance of
this model on the validation set is later used to find the best
hyperparameter configuration as candidate models are compared against
each other based on the defined model performance metric.

#### Result Collection
This step is responsible for the collection of models,
classification/regression results, or other metrics for the problem at
hand of each trial. After running all trials, results must be
consolidated for comparison. Depending on the framework, this step might
be a continuous process that ends once all trails are complete or a goal
condition, e.g., target accuracy, is met. However, for this benchmark,
we always measure the result collection as the time between the last
completed trail and the identification of the best performing
hyperparameter configuration. **After this step, the workload should
result in a final hyperparameter configuration that meets the
optimization goal.**

#### Test
The final evaluation of the model which performed the best of all trials
on the validation set. The test results are thus the final results for
the model with the best fitting hyperparameter configuration. **After
this step, we have the final measure of the quality of the
hyperparameter optimization task.**

#### Metric Collection
Describes the collection of all gathered metrics which are not used by
the *HPO* framework (latencies, CPU resources). This step typically runs
outside the *HPO* Framework and can be done in parallel starting after
the setup phase if the SUT allows for continuous observation.

#### Un-Deploy
The clean-up procedure to un-deploy all *HPO* Framework components
deployed in the Deploy step. **After this step, we assume that we can
repeat this benchmark with no influencing factors from prior runs.**

Any *SUT* included in the benchmark must implement these steps. If a SUT
supports multiple deployment platforms, we must implement these steps
separately for each platform.

## Measurments and Metrics


| Variable | Domain                          | Short-Description                  | 
|:---------|:--------------------------------|:-----------------------------------|
| $$wn$$     | $$\mathbb{N}$$                    | Number of Worker                   |
| $$at$$     | $$\{$$CPU, GPU, TPU$$\}$$           | Accelerator Type                   |
| $$wcpu$$   | $$\mathbb{N}$$                    | vCPU per Worker                    |
| $$wmem$$   | $$\mathbb{N}$$ in MB              | Memory per Worker                  |
| $$amem$$   | $$\mathbb{N}$$ in MB              | Accelerator Memory                 |
| $$t$$      | $$\{$$MNIST$$, ...\}$$              | Dateset                            |
| $$m$$      | github-url                      | Model-Implementation               |
| $$ts$$     | $$\mathbb{N}$$ in MB              | Dataset Size                       |
| $$ta$$     | Text                            | Training Algorithm                 |
| $$sa$$     | Text                            | Sampling Strategy                  |
| $$pa$$     | Text                            | Pruning Strategy                   |
| $$tg$$     | Text                            | Optimization Goal                  |
| $$\Phi$$   | Set                             | Hyperparameter Space               |
| $$pS$$     | $$\mathbb{N}$$                    | $$\|\Phi\|$$                         |

{: .text-center :}
**Table 1**: Basht measrumentes 


| Variable | Domain                          | Short-Description                  | 
|:---------|:--------------------------------|:-----------------------------------|
| $$TWH$$    | $$\mathbb{R}$$ in kWh             | Average Wattage used per Trial     |
| $$CINT$$   | $$\mathbb{R}$$ in $$gCO_{2eq}/kWh$$ | Average Carbon Intensity per Trial |
| $$JCT$$    | $$\mathbb{N}$$ in ms              | Job Completion Time                |
| $$OC$$     | $$\mathbb{R}$$ in USD             | Job Cost                           |
| $$WTC$$    | $$\mathbb{R}$$ in USD             | Waste-Trial Cost                   |
| $$WTT$$    | $$\mathbb{N}$$ in ms              | Wasted-Trial Compute Time          |
| $$WTE$$    | $$\mathbb{R}$$ in kWh             | Wasted-Trial Energy                |
| $$WR$$     | $$\mathbb{R}$$ in Percent         | Wattage used by idle resources     |

{: .text-center :}
**Table 2**: Core Metrics