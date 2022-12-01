---
sidebar_position: 1
id: actions_with_gpu
description: Cirun Documentation - Cloud Authentication.
image: "https://cirun.io/cirun-summary-image-v4.png"
keywords: [Cirun, Gcp, digitalocean, azure, Amazon web services, open stack, Authentication, Oracle]
---
# GitHub Actions with your own GPUs

If you are working with Machine Learning Operations ([MLOps](https://ml-ops.org/)). You want continuous delivery and testing of your Model and its operations. MLOps can be automated using **[GitHub Actions](https://docs.github.com/en/actions)** that is a CI/CD platform, and it is way more fast when you use GPUs.
## What is MLOps.
MLOps stands for Machine Learning Operations. It establishes a smooth connection from the creation of the ML models to the production. MLOps continues by concentrating on the upkeep and overall monitoring of Machine Learning Engineering. MLOps collaborate on every task carried out by data scientists, DevOps engineers, and ML engineers.

## Uses and advantages offer by MLOps.

The development of machine learning and AI solutions can benefit from the usage of MLOps. By utilizing continuous integration and deployment (CI/CD) methods with appropriate monitoring, validation, and execution of ML models, data scientists and machine learning engineers may work together and employ an MLOps technique to speed up the design and production of models.

The major benefits of MLOps are efficiency, scalability, and reduced risk. MLOps allow data teams to create models faster, deliver ML models of better quality, and deploy and produce models more quickly. In particular, MLOps reduces friction with devops and IT, allows better cooperation with data teams, makes ML pipelines reproducible, and speeds up release velocity. Risk reduction: Regulatory reviews and drift checks for machine learning models are common, and MLOps makes it possible to respond to these requests faster and openly while also ensuring that more organization or industry rules are met.

## MLOps are better done on GPUs.
ML operations are better to be done on GPUs. As ML models uses too many mathematical algorithms, GPUs are designed to solve complex mathematics effectively. So, whenever we do ML operations on GPUs, it will be done effectively and also minimize the computing time complexity.

This example uses an example workflow to demonstrate how to use GPUs to do ML operations using [Cirun.io](https://cirun.io/). When this workflow is triggered, it automatically runs a script that creates a self-hosted runner on AWS with GPU and does ML operations on it.

- You can see the Time complexity difference in CPU and GPU [here](https://github.com/vishal9629/MLops_with_Cirun/actions/runs/3452191297/usage)


## MLOps using Cirun

When we do Machine Learning Operations ([MLOps](https://ml-ops.org/)). We want continuous delivery, testing and other operations of your Model. **[CML](https://github.com/iterative/cml#getting-started)**, an open-source CLI tool for implementing CI/CD with a focus on MLOps. Model training, model evaluation, monitoring datasets, and machine provisioning are all included in CML, which is used to automate development workflows. 
CML uses custom Docker images that come pre-installed libraries that are essential for MLOps like NodeJS, Python, DVC and CML set up on an Ubuntu LTS. MLOps can be automated using the **[GitHub Actions](https://docs.github.com/en/actions)** that is a CI/CD platform.

MLOps operates more efficiently on GPUs. Therefore, integrating GPUs is more beneficial if we automate the entire process.

In automation of whole process with GPUs [Cirun.io](https://cirun.io/) provides a solution. Cirun create On-demand Self-Hosted Github Actions Runners on your Cloud!

### Example workflow

The following workflow was created to show how to do MLOps with Cirun. Here we are creating a comparison between MLOps using CPU and GPU. In this workflow we created jobs for two runners, one is for self-hosted using GPU and another one is for GitHub action runner using CPU. You can also reproduce this workflow for single runner by removing one **Runner Configuration** and **Matrix configuration**. To review the latest version of this file in the [GitHub](https://github.com/vishal9629/MLops_with_Cirun/tree/new-example-2/.github/workflows) repository.

##### Also you can directly copy single workflow yml. [MLOps using GPUs yml](https://github.com/vishal9629/MLops_with_Cirun/blob/new-example-2/.github/workflows/MLOps-gpu.yml)


```yml
name: "GitHub Actions with your own GPUs"
on: 
  push:
    branches: [ "none" ] # When a push occurs on a branch workflow will trigger on your desired branch.
  workflow_dispatch: 
jobs:
  CPU_GPU_matrix: # Creating matrix for jobs.
    strategy:
      matrix:
        # Creating a 2d matrix to execute our jobs with respective docker containers.
        os: ["ubuntu-latest", "cirun.gpu"] 
        containers: ["docker://dvcorg/cml-py3:latest" , "ghcr.io/iterative/cml:0-dvc2-base1-gpu" ]
        container_arg: [" " , "--gpus all"]
        # Excluding the unwanted docker container and container_arg in our respective OS.
        exclude:
          - os: "ubuntu-latest"
            containers: "ghcr.io/iterative/cml:0-dvc2-base1-gpu"
          - os: "ubuntu-latest"
            container_arg: "--gpus all"
          - os: "cirun.gpu"
            containers: "docker://dvcorg/cml-py3:latest"
          - os: "cirun.gpu"
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
          # Your ML workflow goes here, you can pass your variable name
          "python train.py"
```

### Understanding the example
- For understanding GitHub Actions you can refer [Actions](https://docs.github.com/en/actions).
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
#### To setup CML with CPU use container.
```yml
image: "docker://dvcorg/cml-py3:latest"
```
#### To setup CML with GPUs use container.
```yml
image: "ghcr.io/iterative/cml:0-dvc2-base1-gpu"
```
- Followed by container argument.
```yml
"--gpus all"
```
- Excluding the unwanted docker container and container_arg from our respective OS. An excluded configuration only has to be a partial match for it to be excluded.

```yml
exclude:
          - os: "ubuntu-latest"
            containers: "ghcr.io/iterative/cml:0-dvc2-base1-gpu"
          - os: "ubuntu-latest"
            container_arg: "--gpus all"
          - os: "cirun.gpu"
            containers: "docker://dvcorg/cml-py3:latest"
          - os: "cirun.gpu"
            container_arg: " "
```
- Here we are Dynamic pass the os, containers, container_agr using matrix. You can have more info [here](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container).

```yml
runs-on: "${{ matrix.os }}" 
container:
      image: "${{ matrix.containers }}"
      options: "${{ matrix.container_arg }}"
```
- This is very important to pass the GITHUB_TOKEN to have a working workflow. So, that authentication is done on behalf of GitHub Actions. You can have look to [GITHUB_TOKEN](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).
```yml
- name: "MLops"
        env:
          repo_token: "${{ secrets.GITHUB_TOKEN }}"
```
- This is our final run command to have our Model is to be run on the runner.

```yml
run: |
      # Your ML workflow goes here, you can pass your variable name
      "python train.py"
```
### Configuration of GPU on self-hosted runner using Cirun

To configure your GPUs using self-hosted runner see [Cirun Configuration](https://docs.cirun.io/reference/yaml#gpu-gpu).

## References
- [GitHub Actions](https://docs.github.com/en/actions)
- [MLOps](https://ml-ops.org/)
- [CML](https://cml.dev/)
- [Example GitHub repository](https://github.com/vishal9629/MLops_with_Cirun/tree/new-example-2)
- [MLOps using GPUs yml](https://github.com/vishal9629/MLops_with_Cirun/blob/new-example-2/.github/workflows/MLOps-gpu.yml)