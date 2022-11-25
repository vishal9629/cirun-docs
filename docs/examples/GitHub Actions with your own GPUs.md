---
sidebar_position: 1
id: actions_with_gpu
description: Cirun Documentation - Cloud Authentication.
image: "https://cirun.io/cirun-summary-image-v4.png"
keywords: [Cirun, Gcp, digitalocean, azure, Amazon web services, open stack, Authentication, Oracle]
---
# GitHub Actions with your own GPUs

If you are working with Machine Learning Operations ([MLOps](https://ml-ops.org/)). You want continuous delivery and testing of your Model.
## What is MLOps.
MLOps stand for Machine Learning Operations. It establishes a smooth connection from the creation of the ML models to the production. MLOps continues by concentrating on upkeep and overall monitoring of the Machine Learning Engineering. MLOps collaborate on every task carried out by data scientists, Devops engineers, and ML engineers.

## Uses and advantages offer by MLOps.

The development of machine learning and AI solutions can benefit from the usage of MLOps. By utilizing continuous integration and deployment (CI/CD) methods with appropriate monitoring, validation, and execution of ML models, data scientists and machine learning engineers may work together and employ an MLOps technique to speed up the design and production of models.

The major benefits of MLOps are efficiency, scalability, and reducing risk. MLOps allow data teams to create models more fast, deliver ML models of better quality, and deploy and produce models more quickly. MLOps also allows for very massive scalability and management, making it possible to continuously combine, deliver, and deploy thousands of models. In particular, MLOps reduces friction with devops and IT, allows better cooperation with data teams, makes ML pipelines reproducible, and speeds up release velocity. Risk reduction: Regulatory reviews and drift checks for machine learning models are common, and MLOps makes it possible to respond to these requests more fast and openly while also ensure that more organization or industry rules are met.

## MLOps are better done on GPUs.
ML operations are better to be done on GPUs. As ML models used too much mathematical algorithms. GPUs are designed to solve the complex mathematics in effective way. So, whenever we do ML operations on GPUs, it will be done in an effective manner and also minimize the computing time complexity.

This example uses an example workflow to demonstrate how to use GPUs to do ML operations using [Cirun.io](https://cirun.io/). When this workflow is triggered, it automatically runs a script that creates a self-hosted runner on AWS with GPU and do ML operations on it.

- You can see the Time complexity difference in CPU and GPU [here](https://github.com/vishal9629/MLops_with_Cirun/actions/runs/3452191297)


## MLOps using Cirun

When we do Machine Learning Operations ([MLOps](https://ml-ops.org/)). We want continuous delivery, testing and other operations of your Model. For CI/CD of Machine Learning we using [CML](https://github.com/iterative/cml#getting-started) which is an open-source CLI tool for implementing continuous integration and delivery (CI/CD) with focus on MLOps. CML is use to automate the development workflows, it includes the all ML Operations that includes model training, model evaluation, monitoring dataset and also includes machine provisioning. CML uses custom Docker images that come pre-installed libraries that are essential for MLOps like NodeJS, Python, DVC (Data Version Control) and CML set up on am Ubuntu LTS. MLOps can be automated using the [GitHub Actions](https://docs.github.com/en/actions) that is a continuous integration and continuous delivery (CI/CD) platform.

MLOps operates more efficiently on GPUs. Therefore, integrating GPUs is more beneficial if we automate the entire process.

In automation of whole process with GPUs [Cirun.io](https://cirun.io/) provides a solution. Cirun create On-demand Self-Hosted Github Actions Runners on your Cloud!

## Example workflow

The following workflow was created to show how to do MLOps with Cirun. Here we are creating a comparison between MLOps using CPU and GPU. In this workflow we created jobs for two runners, one is for self-hosted using GPU and another one is for GitHub action runner using CPU. You can also reproduce this workflow for single runner by removing one **Runner Configuration** and **Matrix configuration**. To review the latest version of this file in the [GitHub](https://github.com/vishal9629/MLops_with_Cirun/tree/new-example-2/.github/workflows) repository.

```yml
name: "GitHub Actions with your own GPUs"
on: 
  push:
    branches: [ "none" ] # When a push occurs on a branch workflow will trigger.
  workflow_dispatch: 
jobs:
  CPU_GPU_matrix: # Creating matrix for jobs.
    strategy:
      matrix:
        # Creating a 2d matrix to execute our jobs with respective docker containers.
        os: ["ubuntu-latest", "self-hosted"] 
        containers: ["docker://dvcorg/cml-py3:latest" , "ghcr.io/iterative/cml:0-dvc2-base1-gpu" ]
        container_arg: [" " , "--gpus all"]
        # Excluding the unwanted docker container and container_arg in our respective OS.
        exclude:
          - os: "ubuntu-latest"
            containers: "ghcr.io/iterative/cml:0-dvc2-base1-gpu"
          - os: "ubuntu-latest"
            container_arg: "--gpus all"
          - os: "self-hosted"
            containers: "docker://dvcorg/cml-py3:latest"
          - os: "self-hosted"
            container_arg: " "
    # Workflow to run commands in OS.
    # Dynamic passing of os, containers, container_agr using matrix 
    runs-on: "${{ matrix.os }}" 
    container:
      image: "${{ matrix.containers }}"
      options: "${{ matrix.container_arg }}"
    # Steps that we want to have with OS.
    steps:
      - uses: "actions/checkout@v3"
      - name: "Dependency Install"
        run: "pip install -r requirements.txt"
      - name: "MLops"
        env:
          repo_token: "${{ secrets.GITHUB_TOKEN }}"
        run: |
          # Your ML workflow goes here
          "python train.py"
```

## Understanding the example

```yml
CPU_GPU_matrix: # Creating matrix for jobs.
    strategy:
      matrix:
        # Creating a 2d matrix to execute our jobs with respective docker containers.
        os: ["ubuntu-latest", "self-hosted"] 
        containers: ["docker://dvcorg/cml-py3:latest" , "ghcr.io/iterative/cml:0-dvc2-base1-gpu" ]
        container_arg: [" " , "--gpus all"]
```
- Added matrix strategy which enables us to create multiple job runs that are based on the matrix variables. In this example we are creating a matrix that provides multiple combination of os, docker containers and container arguments. These combinations are responsible for to create jobs for CPU and GPU individually using a single workflow. For more info see [matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs).
- Docker container comes with pre-installed dependencies which are important for full-stack data science.
```yml
"docker://dvcorg/cml-py3:latest"
```
- To setup CML with CPU use container.
- We don't need any other container argument for CPU.

```yml
"ghcr.io/iterative/cml:0-dvc2-base1-gpu"
```
- To setup CML with GPUs use container.

```yml
"--gpus all"
```
- Also needs a container argument to configure container for all GPUs.

```yml
exclude:
          - os: "ubuntu-latest"
            containers: "ghcr.io/iterative/cml:0-dvc2-base1-gpu"
          - os: "ubuntu-latest"
            container_arg: "--gpus all"
          - os: "self-hosted"
            containers: "docker://dvcorg/cml-py3:latest"
          - os: "self-hosted"
            container_arg: " "
```
- Excluding the unwanted docker container and container_arg from our respective OS. An excluded configuration only has to be a partial match for it to be excluded.

```yml
runs-on: "${{ matrix.os }}" 
container:
      image: "${{ matrix.containers }}"
      options: "${{ matrix.container_arg }}"
```
- Here we are Dynamic pass the os, containers, container_agr using matrix. You can have more info [here](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container).

```yml
- name: "MLops"
        env:
          repo_token: "${{ secrets.GITHUB_TOKEN }}"
```
- This is very important to pass the GITHUB_TOKEN to have a working workflow. So, that authentication is done on behalf of GitHub Actions. You can have look to [GITHUB_TOKEN](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

```yml
run: |
      "python train.py"
```
- This is our final run command to have our MLOps is to be running on the runner.

## Configuration of GPU on self-hosted runner using Cirun

To configure your GPUs using self-hosted runner see [Cirun Configuration](https://docs.cirun.io/reference/yaml#gpu-gpu).