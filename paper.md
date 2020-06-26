---
title: 'DARE Platform\: a Developer-Friendly and Self-Optimising Workflows-as-a-Service Framework for e-Science on the Cloud'
tags:
  - Python
  - CWL
  - Worfklows-as-a-Service
  - Kubernetes
  - Cloud
  - e-infrastructures
authors:
  - name: Iraklis A. Klampanos
    orcid: 0000-0003-0478-4300
    affiliation: "1"
  - name: Chrysoula Themeli
    orcid: 0000-0002-6759-4136
    affiliation: "1"
  - name: Vangelis Karkaletsis
    orcid: 0000-0000-0000-0000
    affiliation: "1"
affiliations:
 - name: National Centre for Scientific Research “Demokritos”
   index: 1
date: 26 June 2020
bibliography: bibliography.bib
---

# Introduction

In recent years, modern science has relied more than ever on large-scale data as well as on distributed computing and human resources. Scientists and research engineers in fields such as climate, atmospheric sciences and computational seismology, constantly strive to make good use of remote and largely heterogeneous computing resources (high-performance computing -HPC- facilities, clouds, private institutional or local resources, etc.), process, archive and analyse results stored in different locations and collaborate effectively with other scientists.


In this environment, scientists and research engineers need user-friendly, self-optimising environments to develop and execute their scientific workflows without being exposed to the implementation detail and complexity of the underlying systems. They need efficient and easy-to-use environments to store and manage their workflows, working datasets and results in order to reuse, share them with colleagues or encourage attribution. Research engineers in particular need to be able to create user-facing prototypes and solutions efficiently and accurately. Last, users need to feel in control and be able to track the progress of execution, data transformation, at a fine level.


In this article we present the DARE platform, an open source project developed as part of the EU H2020 project "DARE: Delivering Agile Research Excellence on European e-Infrastructures"[^1]. The main goal of the DARE platform is to support research scientists and research engineers/developers to transparently make use of research resources and research infrastructures, platforms and software in order to create data- and computationally-intensive domain-oriented applications. The DARE platform enables seamless development and reusability of scientific workflows / applications and the reproducibility of the experiments. It is also designed to be reflective, in order to enable monitoring, reproducibility and optimisations. DARE primarily focuses on the European e-infrastructures ecosystem, but it can also be used independently, e.g. as part of local or specialised installations. More information on DARE can be found in [@klampanos2019dare, @atkinson2019comprehensible, @atkinson_malcolm_2020_3697898].


[^1]: This work has been supported by the EU H2020 research and innovation programme under grant agreement No 777413.


# The DARE Platform


The DARE platform is designed to live in-between user applications and the underlying computing resources. It is built on top of containerisation as well as parallelisation technologies, e.g. Kubernetes and MPI. Interfacing with client systems and end-users is achieved via RESTful APIs. The execution of scientific workflows is achieved via a Workflows-as-a-service layer, which can handle workflows described in either the dispel4py Python library(@Filgueira2017), or in the Common Workflow Language (CWL)(@amstutz2016common). Based on the Microservices Software Engineering Architecture paradigm, it consists of multiple decoupled but integrated, dockerized components, managed and orchestrated by Kubernetes. The components provide all the necessary tools to research developers to develop, execute and share data-intensive workflows. Currently, the DARE platform supports two types of workflow specification: dispel4py and CWL:

* dispel4py (@Filgueira2017) is a Python framework for the specification of fine-grained, streaming workflows for data-intensive applications. It comes with mappings to selected enactment targets (e.g. Apache Storm, MPI, Multiprocessing, etc.), which are triggered at run-time. As a result, workflows described in dispel4py can be executed in fundamentally different execution contexts.
* CWL (@amstutz2016common) is a YAML-based specification language for describing task-oriented workflows and tools in a way that makes them portable and scalable across a variety of software and hardware environments, from workstations to cluster, cloud, and HPC environments. Like dispel4py, CWL has been designed to meet the needs of data-intensive science, while targeting a different set of requirements.

DARE views workflows and their constituent components (processing elements or *PEs*) as potentially valuable, reusable and shareable research tokens. In order to support workflow reusability, sharing and versioning, the platform provides all the necessary tools for research scientists and engineers to register and describe their applications. Subsequently it allows reference and invocation of workflows and applications by name, via the RESTful APIs provided. Moreover, the platform includes provenance tools to monitor the execution of workflows in real time, or to mine information on past workflow executions.

![DARE Overview](dare_overview_2020.png)

The main logical components of the DARE platform, along with their main corresponding constituent components are the following:

1. Workflow interpretation
   1. Dispel4py interpretation and execution mappings
   2. CWL interpretation and execution
2. DARE knowledge base
   1. Dispel4py Information Registry, which allows storage and versioning of dispel4py processing elements and workflows
   2. CWL Workflow Registry, providing similar functionality to the dispel4py Registry above, in order to register CWL workflows
   3. Provenance store, which allows interfacing with a number of components within the DARE platform via an API
3. Provenance capture and visualisation component
   1. SProvFlow, which allows filtering and visualisation of costs and resources used, data usage frequency, etc.
   2. CWLProv, which allows for capturing provenance information in CWL workflows based on @khan2019sharing.
4. DARE API
   1. Exposition of part of components’ APIs, making use of [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/).
   2. Execution API: a RESTful Web Service exposing important functionality to the users such as dispel4py and CWL workflow execution, while also providing basic file handling functionality.
   3. Keycloak API for AAI: Exposes AAI endpoints based on [Keycloak](https://www.keycloak.org/), for modularity and ease of integration with external authentication mechanisms.


# Developer-friendly WaaS


DARE offers user-friendly development, execution and monitoring of two different types of workflows, namely dispel4py and CWL. The platform accommodates the execution of such workflows in dynamically created and destroyed contexts, in the Cloud. Once a workflow has been developed, registered and debugged, it can be executed by name via parameterised RESTful endpoints. Especially for the earlier development stages, DARE offers a *Playground* module as well as interactive tools, to ease testing and debugging before a workflow can be considered operational.


## Dynamic Loading of Execution Contexts


Execution environments are loaded on-demand, based on access of specific endpoints of the Execution API. Currently, the platform offers three different execution environments, for  dispel4py workflows, CWL workflows and [SPECFEM3D-Cartesian](https://geodynamics.org/cig/software/specfem3d/) - a well-received code for simulating seismic wave propagation[^2]. In this section, we focus on the CWL execution environment, as it is more widely recognisable and generic, however the procedure is also similar to the case of dispel4py.


[^2]: The 3rd endpoint is specialised and now obsolete, as it can be implemented as a CWL workflow, however we include it for completeness and also because it is used by some of the use-cases outlined below.


Before scientists and research engineers can make use of arbitrary, CWL-based execution environments they need to have the corresponding docker containers registered on the platform. The DARE platform installation administrator is responsible for testing and registering such environments, ensuring they are free from malicious software and that they behave as intended. This registration takes place on the CWL Workflow Registry as, at this stage, generic execution environments are usable via CWL workflows requested via the DARE Execution API.


Once the execution environment, realised by a docker container, has been registered, scientists and research engineers can make use of it for their scientific workflows. The CWL Workflow Registry allows users to register multiple bash and python scripts and associate them with specific docker containers. In addition, users can also register CWL workflows in the same registry and also associate them to specific docker containers. Within a DARE platform installation, execution environments and CWL workflows are identified by name and version.


Once environments and associated executables have been successfully registered, users can start a CWL workflows execution via the DARE Execution API, by specifying the name and version of the workflow. The Execution API then dynamically loads the corresponding execution environment and starts the workflow by making use of the underlying Kubernetes container orchestration layer. As this execution takes place within the DARE platform, users are shielded by the underlying implementation details as well as they can take advantage of the built-in registration and provenance tools in order to share, re-run or monitor their workflow-based application.


## Ease of use and monitoring
In order to address the requirements of modern science, the DARE platform offers a user-friendly environment providing Workflows-as-a-Service to scientists and research developers and engineers. We provide a testing environment, the Playground module, where users can develop and test their workflows. Through the Playground API, we provide a simulation of the dispel4py workflow execution giving users immediate access to the logs and output files. In addition, users are provided with interactive tools to register and describe processing elements and complete workflows, which allows sharing, findability and reusability of methods. The platform also provides interactive provenance tools, enabling users to track workflow execution during run-time. Each run in the platform is assigned an ID which is used to retrieve provenance records regarding execution time, cost, resources and data used, etc.


# DARE Platform Use-cases


The DARE platform is currently used in the following domain applications.


## Seismology: Rapid Assessment


The Rapid Assessment (RA) [use-case](https://gitlab.com/project-dare/WP6_EPOS) produces on-demand estimates of ground motion parameters (peak values of ground velocity or acceleration), when large earthquakes occur, by analysing seismic wavefields (@saleh2018epos). It creates maps that compare observation and synthetic data for better understanding of the ground behaviour. The application collects observation data from stations through the Federation of Digital Seismographic Network (FDSN) and executes a SPECFEM3D-Cartesian waveform simulation so as to generate synthetic data. After a pre-processing step on both datasets, the application calculates, compares and plots the ground motion parameters in the observed and synthetic data. RA is implemented as a dispel4py workflow, also making use of the specialised SPECFEM3D-Cartesian endpoint.  


## Seismology: Moment Tensor 3D (MT3D)


As an application, MT3D is a generalisation of the Rapid Assessment [use-case](https://gitlab.com/project-dare/WP6_EPOS). In this case, research developers execute 10 SPECFEM3D-Cartesian simulations before the dispel4py workflow execution. The first simulation is used as the starting solution, the next 6 to describe changes in the mechanism at fixed location while the last 3 serve as a location change description at fixed mechanism. MT3D is similarly implemented as the RA use-case with the difference of multiple parallel SPECFEM3D-Cartesian runs.


## Volcanology: Ash fall hazard modelling


The [volcanology use case](https://gitlab.com/project-dare/wp6_volcanology) aims to model ash-fall as a result of a volcanic eruption. It makes use of a custom model, weather and volcanic data to create synthetic data about deposition thickness, ground load and airborne mass, simulating different scenarios. The volcanology use-case is split in two separate workflows both manageable from the same DARE instance. The first workflow is implemented using dispel4py and it involves data downloading and storage into the DARE platform. The second workflow is implemented in CWL, making use of the CWL WaaS functionality, which allows associating CWL workflows with execution containers.




## Climate-change: Extending Climate4Impact


[Climate4Impact](https://climate4impact.eu)(C4I) is a user friendly platform of services to enable access to climate data for the climate impact community. It has been developed as part of the EU funded IS-ENES series of projects since 2009. This [use case](https://gitlab.com/project-dare/WP7_IS-ENES_Climate4Impact) aims to enable the delegation of on-demand computational-intensive workflows to the DARE platform, from the IS-ENES C4I interface, also making use of data from the Earth System Grid Federation (ESGF). The DARE platform's objective is to provide efficient and transparent access to diverse computing resources through an easy-to-use API to easily deploy execution environments. Climate4Impact aims to increase and widen the use of climate modelling data [@page2018leveraging, @page2019ease, @page2019enabling]. The C4I use-case is implemented as a single dispel4py workflow.


## Atmospheric sciences: Cyclone tracking application


Cyclone Tracking is our new [use-case](https://gitlab.com/project-dare/wp7_cyclone-tracking) in the Climate domain. The use-case is based on a binary Fortran-compiled program (https://github.com/cerfacs-globc/cyclone_tracking) which implements a tracking method for tropical centre and transition diagnostic and for locating pressure / vorticity centres. The Cyclone workflow is developed exclusively in CWL, and it makes use of a cyclone tracking model to produce weather data and visualise the results.


## The Contribution of the DARE Platform


In the use-case listed above the DARE platform achieves the following:
1. It interfaces with users and external systems via a comprehensive and secure RESTful API
2. It facilitates the development of modular, reusable and shareable solutions via its workflow registries
3. It allows for the combination of different workflow approaches, dispel4py and CWL, within the same platform and development environment
4. Via its execution API it orchestrates the dynamic spawning and closing of MPI clusters on the cloud for MPI-enabled components, e.g. the SPECFEM3D-Cartesian model and the dispel4py MPI mapping
5. It provides a flexible environment which local administrators can parametrise by adding custom docker-based environments and user interfaces
6. It collects, mines and visualises provenance information from previous runs


# Software


The DARE platform is available on [GitLab](https://gitlab.com/project-dare/dare-platform) with our latest release so far (v3.2). We also have a [GitLab page](https://project-dare.gitlab.io/dare-platform/) with installation instructions, API documentation and a short demo. The demo is available in the [DARE Execution API GitLab Repository](https://gitlab.com/project-dare/exec-api/-/tree/master/examples/mySplitMerge) and can be used as an integration test.


Each DARE component may include its own tests, client-side helper functions or a short jupyter notebook demo. As example, we include some GitLab urls to client-side helper functions or demos:

i. [Dispel4py Information Registry](https://gitlab.com/project-dare/d4p-registry/-/tree/master/cli        ii. [Workflow Registry](https://gitlab.com/project-dare/workflow-registry/-/tree/master/workflow_client)
iii. [Execution Registry](https://gitlab.com/project-dare/exec-registry/-/tree/master/client)


# Conclusions and Future Work

The main goal of the DARE platform is to support research developers to implement, track, maintain and execute complex, data-intensive applications in a unifying and self-optimising environment. The platform tries to hide as much infrastructure complexity as possible from the users, who can interact with the platform through the DARE API, as well as via helper Python functions and interactive tools.

Directions for future work include the following:

1. Make use of provenance data and workflow metadata to further automate the optimisation of workflow execution.
2. Provide wider-ranging search facilities to users for data, components and containerised environments.
3. Provide of-the-shelf integration with domain-specific as well as generic repositories (e.g. with [Zenodo](https://zenodo.org/) in order to facilitate better Open Science best practices.

# References
