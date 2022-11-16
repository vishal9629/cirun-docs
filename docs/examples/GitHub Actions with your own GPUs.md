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

As ML operations are better to be done on GPUs. As ML models used too much mathematical algorithms. GPUs are designed to solve the complex mathematics in effective way. So, whenever we do ML operations on GPUs, it will be done in an effective manner and also minimize the computing time complexity.

This article uses an example workflow to demonstrate how to use GPUs to do ML operations using [Cirun.io](https://cirun.io/). When this workflow is triggered, it automatically runs a script that creates a self-hosted runner on AWS with GPU and do ML operations on it.

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

