---
sidebar_position: 1
id: actions_with_gpu
description: Cirun Documentation - Cloud Authentication.
image: "https://cirun.io/cirun-summary-image-v4.png"
keywords: [Cirun, Gcp, digitalocean, azure, Amazon web services, open stack, Authentication, Oracle]
---
# GitHub Actions with your own GPUs

If you are working with Machine Learning Operations ([MLOps](https://ml-ops.org/)). You want continous delivery and testing of your Model.

For Continous Integration of machine learning you can use [CML](https://github.com/iterative/cml#getting-started) which is an open-source CLI tool for implementing continous integration and delivery (CI/CD) with focus on MLOps.

MLOps are more effectively works with GPUs. So, if we automate the whole process using GPUs is more helpful.

[GitHub Actions](https://docs.github.com/en/actions) is a continous integration and continous delivery (CI/CD) platform that allows to automate build, test and deployment.

In automation of whole process [Cirun.io](https://cirun.io/) provides a solution. Cirun create On-demand Self-Hosted Github Actions Runners on your Cloud! 

## Example overview

ML operations are better to be done on GPUs. As ML models used too much mathematical algorithms. GPUs are designed to solve the complex mathematics in effective way. So, whenever we do ML operations on GPUs, it will be done in an effective manner and also minimize the computing time complexity.

This article uses an example workflow to demonstrate how to use GPUs to do ML operations using [Cirun.io](https://cirun.io/). When this workflow is triggered, it automatically runs a script that creates a self-hosted runner on AWS with GPU and do ML operations on it.

- You can see the Time complexity difference in CPU and GPU [here](https://github.com/vishal9629/MLops_with_Cirun/actions/runs/3452191297)

## Things must have in GitHub Actions workflow to work with MLOps.

- Create a Workflow.
- We need to configure CML- [Continous machine learning](https://github.com/iterative/cml#getting-started) with repository to setup the enviornment for training.
- CML come preinstalled in Docker Images.
- To setup CML with GitHub actions use docker container.
```yml
"docker://dvcorg/cml-py3:latest"
```
- We don't need any other container argument for CPU.
- To setup CML with GPUs use container.
```yml
"ghcr.io/iterative/cml:0-dvc2-base1-gpu"
```
- Also needs a container argument to configure container for all GPUs.
```yml
"--gpus all"
```
- We also have to pass our repository's [GITHUB_TOKEN](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) so, that authentication is done on behalf of GitHub Actions.
- Then we have steps what ever we want to do with our Workflow.

## Features used in this example

The example workflow demonstrates how to use GitHub Actions with your own GPUs.

| Feature     | Implementation |
| ----------- | ----------- |
| Triggering a workflow to run automatically | [push](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push)       |
| Manually running a workflow from the UI  | [workflow_dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch)        |
| Setting permissions for the token | [permissions](https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs) |
|  Running the job on different runners | [runs-on](https://docs.github.com/en/actions/using-jobs/choosing-the-runner-for-a-job) |
| Running jobs in a container | [container](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container)|
| Cloning your repository to the runner| [actions/checkout](https://github.com/actions/checkout) |

## Example workflow

The following workflow was created to show how to do MLOps with Cirun. In this workflow we created run the MLOps for two runners, ons is self-hosted using GPU and another one is GitHub action runner using CPU. You can also reproduce this workflow for single runner by removing one **Runner** and **Matrix**. To review the latest version of this file in the [GitHub](https://github.com/vishal9629/MLops_with_Cirun/tree/new-example-2/.github/workflows) repository.

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

The following explains how each of these features are used when creating a GitHub Actions workflow.

 ```yml 
name: "GitHub Actions with your own GPUs"
```  
The name of the workflow as it will appear in the "Actions" tab of the GitHub repository.

```yml 
on: 
  push:
    branches: [ "none" ]
```
Add the push event, so that the workflow runs automatically every time a commit is pushed to a branch called main. For more information, see [push](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push).

```yml
workflow_dispatch: 
```
Add the **workflow_dispatch** event if you want to be able to manually run this workflow from the UI. For more information, see [workflow_dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch).

```yml
jobs:
```
Groups together all the jobs that run in the workflow file.

```yml
CPU_GPU_matrix: # Creating matrix for jobs.
    strategy:
      matrix:
        # Creating a 2d matrix to execute our jobs with respective docker containers.
        os: ["ubuntu-latest", "self-hosted"] 
        containers: ["docker://dvcorg/cml-py3:latest" , "ghcr.io/iterative/cml:0-dvc2-base1-gpu" ]
        container_arg: [" " , "--gpus all"]
```
Add matrix strategy that lets you use variables in a single job definition to automatically create multiple job runs that are based on the combinations of the variables. For more info see [matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs).

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
Excluding the unwanted docker container and container_arg in our respective OS. An excluded configuration only has to be a partial match for it to be excluded.

```yml
runs-on: "${{ matrix.os }}"
```
Configures the job to run on a GitHub-hosted runner or a self-hosted runner, depending on the repository running the workflow. In this example, the job will run on a self-hosted runner and as well as for ubuntu-latest. For more information on these options see "[Choosing the runner for a job.](https://docs.github.com/en/actions/using-jobs/choosing-the-runner-for-a-job)"

```yml
container:
      image: "${{ matrix.containers }}"
      options: "${{ matrix.container_arg }}"
```
Dynamic passing of os, containers, container_agr using matrix. You can have more info [here](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container).

```yml
steps:
```
Groups together all the steps that will run as part of the job. Each job in a workflow has its own steps section.

```yml
- uses: "actions/checkout@v3"
```
This is an action that checks out your repository and downloads it to the runner, allowing you to run actions against your code (such as testing tools).

```yml
- name: "Dependency Install"
        run: "pip install -r requirements.txt"
```
The uses keyword tells the job to retrieve the action named "pip install -r requirements.txt".

```yml
- name: "MLops"
        env:
          repo_token: "${{ secrets.GITHUB_TOKEN }}"
```
This is very important to pass the GITHUB_TOKEN to have a working workflow. You can have look to [GITHUB_TOKEN](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

```yml
run: |
      "python train.py"
```
This is our final run command to have our MLOps is to be running on the runner.


## Configuration of GPU on self-hosted runner using Cirun

```yml
# Self-Hosted Github Action Runners on AWS via Cirun.io
runners:
  - name: "gpu-runner"
    # Cloud Provider: AWS
    cloud: "aws"
    # VM on AWS
    instance_type: "g4dn.xlarge"
    # Ubuntu, ami image, region
    region: "eu-west-1"
    machine_image: "ami-00ac0c28c01352e53"
    # Add this label in the "runs-on" param in .github/workflows/<workflow-name>.yml
    # So that this runner is created for running the workflow
    workflow: ".github/workflows/MLops.yml"
    labels:
      - "cirun.gpu"
```