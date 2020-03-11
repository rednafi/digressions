---
title: Python Workflow for Sanity
toc: false
layout: post
description:
categories: [python]
---

# Python Workflow

I use debian linux (Ubuntu, PopOS) as my primary operating system for development. Python's versioning, virtual environment creation, switching between multiple versions, package management can feel a bit overwhelming. Here's how I keep my sanity while working on a new python projet:

## Install Python

Run the following commands one by one:

```bash
# update the packages list and install the prerequisites
sudo apt update
sudo apt install software-properties-common

# add deadsnakes ppa to your sources' list (When prompted press Enter to continue)
sudo add-apt-repository ppa:deadsnakes/ppa

# install python3.7
sudo apt install python3.7

# verify python installation
python3.7 --version
```

## Create Virtual Environment

There are multiple ways creating and switching between different environments can be done. I use `venv` for creating virtual environments. For demonstration, here I'm creating a virtual environment that uses python3.7.

* Install `python3-venv` for creating virtual environment
  ```bash
  sudo apt install python3.7-venv
  ```

* Create virtual environment named `venv` in the project folder

   ```bash
   python3.7 -m venv venv
   ```
* Activate `venv`

   ```bash
   source venv/bin/activate
   ```



## Switch Between Different Environments

To create another environment with a different python version, you have to:

* Install the desired version of python following the procedures stated above
* Install `python3-venv` specific for your python version
* Activate and deactivate the desired virtual environment


## Package Management

* For local development, I use [pip](https://pip.pypa.io/en/stable/)
* For production application and libraries [poetry](https://python-poetry.org/) is preferred
