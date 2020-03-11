---
title: Python Virtual Environment Workflow for Sanity
toc: false
layout: post
description: Keeping python's virtual environment madness in check
categories: [python]
---

There are multiple ways of installing Python, creating and switching between different virtual environments. Also, Python's package manager hyperspace is a mess. So, things can quickly get out of hands while dealing with projects that require quick environment switching across multiple versions of Python. I use Debian linux in my primary development environment and this is how I keep the option explosion in check:

## Installing Python 

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

## Creating Virtual Environment

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
* Deactivate `venv`
   ```bash
   deactivate
   ```


## Switching Between Different Environments

To create another environment with a different python version, you have to:

* Install the desired version of python following the procedures stated [above.](#installing-python)
* Install `python3-venv` specific for your python version, like if you are using python3.8, 
  you should run: 
  ```bash
  sudo apt install python3.8-venv
  ```
* Create multiple environments with multiple versions and name them distinctively. i.e. `venv3.7`, `venv3.8` etc. Follow the instructions [above](#creating-virtual-environment).
* Activate and deactivate the desired virtual environment 


## Package Management

* For local development, I use [pip](https://pip.pypa.io/en/stable/).
* For production application and libraries [poetry](https://python-poetry.org/) is preferred.
