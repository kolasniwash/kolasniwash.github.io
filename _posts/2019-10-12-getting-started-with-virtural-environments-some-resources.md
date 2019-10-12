---
title: "Getting Started With Virtural Environments: Some Resources"
date: 2019-10-12T10:38:30-01:00
---

One of the first things we learn when starting into a data or software project is to setup a virtual environment. Why set up a virtual environment? A virtual environment allows us to install packages (i.e. numpy, pandas, tensorflow etc…) in a controlled space. That way we know exactly the version of each package installed. Any requirements for that specific version are included inside the environment.

This is beneficial when working with multiple projects that use different packages, and have different version requirements. We can change our environment and some what magically everything we need for that project will be there.

In this post we talk about setting up virtual environments and point to excellent resources. In order we will:
1. How to setup a virtual environment natively
2. How to setup a virtual environment with anaconda
3. Installing packages with pip and conda
4. How to trouble shoot a virtual environment

# How to setup a virtual environment from scratch.

To start a virtual environment you will use the python command venv (native to python3). This creates a directory where all the environment variables are stored.

### SETUP A VENV IN WITH PYTHON

1. Setup the environment
```shell
  python3 -m venv my-new-environment
```

2. Start the new environment
```shell
  source my-new-envrionment/bin/activate 
```

3. Install packages - typically you will want to use pip to do your installs, but can also use other installers
```shell
  pip install package-name package=x.x
```
    2. If we add the package=x.x we can define the specific version of the package we install

4. Create a setup file that contains all the packages you’ve installed - produces a file that looks like (example)
```shell
  pip freeze > requirements.txt
```
5. Delete a virtual environment
```shell
  rm -r my-new-environment
```

These are all the core steps you’ll need to create and manage a venv virtual environment. 

The downside to this type of virtual environment is that it does not manage packages. So for example if you install a newer version of tensorflow and it has additional required packages, you’ll have to install them manually before tensorflow will work again.

To learn more about working directly with venv an excellent reference can be found in the article: [A guide to python’s Virtual Environment] (https://towardsdatascience.com/virtual-environments-104c62d48c54). 


# How to setup a virtual environment with ananconda

Conda is a package manager for the Anaconda distribution. Unlike venv conda will manage dependencies within each virtual environments. This means when you install the new tensorflow, conda will take care of installing all the updates and additional packages required to get tensorflow running.

### Setup a virtual environment In CONDA in 3 steps

1. Create a new environment
```shell
  conda create -n my-new-envionment
```
2. See what environments are already setup
```shell
  conda info —envs
```

3. Install packages - you will often want to reference the anaconda website (links) when installing (more later)
```shell
  conda install package-name package=x.x
```

 2. Again adding package=x.x we define the version
4. Remove an environment
 ```shell
  conda remove — my-new-envrionment —all
```

The downside to using conda is that the newest distributions of packages are not always immediately available via anaconda. So if you find yourself in that situation you can install additional packages using pip from within a conda environment. However that package will not benefit from condo’s package management options. 


Fore a detailed explanation of setting up a conda virtual environment the articule [Getting started with python environments (using conda)] (https://towardsdatascience.com/getting-started-with-python-environments-using-conda-32e9f2779307) is a great reference.

Using anaconda has an additional benefit - installing and management via the terminal is optional. Using the desktop installation you can easily manage all your packages via the anaconda user interface. The article [How to setup your python environment for machine learning with anaconda](https://machinelearningmastery.com/setup-python-environment-machine-learning-deep-learning-anaconda/) is a nice tutorial on getting started with this.



# Packages to get started with

Some of the most used packages in the data science stack are:
numpy, pandas, matplotlib, seaborne, jupiter notebook, pytorch, tensorflow, keras, etc.

Working with Jupiter Notebook and Conda

Opening jupyter notebook from inside a conda or venv virtual environment can be don by calling jupyter notebook from the command line. When it launches you’ll want to check that you receive the ‘kernel ready’ message and that it says Python 3 on the top right menu bar.

(Image example)

If you find this isn’t happening, check that ipython is installed by calling the conda list - - v command. This command will list all the current packages and their versions installed in the environment.

(Image example)

Sometimes when you open jupyter notebook you find that it doesn’t connect to the iPython kernel, or that a specific package won’t load despite it showing up when you call conda list - - v.

A quick workaround is to reset the iPython kernel your jupyter notebook is connecting to. This can be done by closing down the jupyter instance, and from inside the virtual environment calling the command:

```shell
  python -m ipykernel install - -user
```
Once run, open jupyter notebook again and check you have a connection and that the package is now working.
