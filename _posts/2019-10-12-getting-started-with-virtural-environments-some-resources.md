---
title: "Virtual Environments Quick Start: Setup, Resources, and Background"
date: 2019-10-12T10:38:30-01:00
---

One of the first things we learn when starting into a data or software project is to setup a virtual environment. Why set up a virtual environment? A virtual environment allows us to install packages (i.e. numpy, pandas, tensorflow etc…) in a controlled space. That way we know the exact the version of each package installed and we contain any dependencies between packages. 

This is beneficial when working with multiple projects that use different packages, and have different version requirements. We can create an environment for each project and by doing so ensure that our code will aways find what it needs. 

In this post we set up virtual environments natively and with conda. Later we discuss some basic packages to get started, and simple environment debugging.

# How to setup a virtual environment from scratch

To start a virtual environment you will use the python command venv (native to python3). This creates a directory where all the environment variables are stored.

### Set up a venv with python

1. Setup the environment
We setup the environment calling the ```venv``` command followed by the name of the environment. The ```-m``` option indicates we will pass a module name. In the example the module name is _my-new-environment_. We usually want to name the environment something related to our project.
```shell
  python3 -m venv my-new-environment
```

2. Start the new environment
We start the new environment by calling the ```source``` command. ```bin/activate/``` contains the activation script within the virtual environments directory. 
```shell
  source my-new-envrionment/bin/activate 
```

3. Installing packages
When managing your virtual environments directly you will use a package installer. Pip is a common installer for python. If your machine does not already have pip installed you can run the [instructions in the python documentation](https://pip.pypa.io/en/stable/installing/#id9).

Pip instals packages by calling the command below. We can specify the package version by calling the ```package=x.x``` and replace x.x with the respective version.
```shell
  pip install package-name package=x.x
  pip install pandas package=0.25.0
```

4. Create a setup file that contains all the packages you’ve installed
This will produce a text file that we can use to recreate the same environment.

```shell
  pip freeze > requirements.txt
```

We can inspect the file we created with nano, pico, or any other text editor i.e. ```nano ./requirements.txt```

5. Exiting and deleting a virtual environment
We can exit a virtual environment by calling ```deactivate```. If we find that the we no longer use the environment, or are unable to configure dependencies we will want to removing it. Here the ```-r``` option indicates recursive and will delete all the elements within the environment’s file tree. 

```shell
  rm -r my-new-environment
```

These are all the core steps you’ll need to create and manage a venv virtual environment. 

The downside to this type of virtual environment is that it does not manage packages. So for example if you install a newer version of tensorflow and it has additional required packages, you’ll have to install them manually before tensorflow will work again.

To learn more about working directly with venv an excellent reference can be found in the article: [A guide to python’s Virtual Environment](https://towardsdatascience.com/virtual-environments-104c62d48c54). 


# How to setup a virtual environment with Ananconda

Conda is a package manager for the Anaconda distribution. Unlike the venv method, conda will manage dependencies within each virtual environments. This means when you install a new version of tensorflow, conda will automatically take care of installing all the updates and additional packages required.

### Setup a virtual environment with Conda

1. Create a new environment
Like we saw with the python method, conda has its own command ```create``` that will initialize a virtual environment. The ```-n``` option indicates we will pass a name, in this case ```my-new-environment```.

```shell
  conda create -n my-new-envionment
```

2. Starting a conda environment
We start our conda environment by calling ```activate``` and the environment name.

```shell
  conda activate my-environment
```


3. Installing packages with conda
We can install packages in the same way with conda. Again the x.x is the version of the package and is optional.


```shell
  conda install package-name package=x.x
```

One difference between using the python method and conda is that we will often want to check the anaconda documentation for the updated install command. Here's an [example with tensorflow](https://anaconda.org/conda-forge/tensorflow) where we are specifying the distribution.

```shell
  conda install -c conda-forge tensorflow
```

4. Creating a setup file in conda
We create and export a setup file for our conda environment with ```export```. Using the > pipes the output to the file we choose. Note how conda environment files use the .yml extension. More details on [conda environment files](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#create-env-file-manually).

```shell
  conda my-environment export > requirements.yml
```

5. Deactivating and cleaning up environments
We deactivate an environment with ```conda deactivate```. If we find we have many environments we can generate a list of all environments with ```conda info —envs```. 

When we decide an environment is no longer needed we can remove it.

 ```shell
  conda remove —n my-new-envrionment —all
```

We can also remove a single package within an environment in the same way. 

 ```shell
  conda remove —n my-new-envrionment package-to-uninstall
```

Using conda is great for worry free package management. However the downside is that the newest distributions of packages are not always immediately available via anaconda. In order to use an unsupported package version with conda we can install with pip. 

**Warning** installing with pip inside a conda environment works, but packages installed with pip will **not** be managed by conda. Doing this often results in mixed dependencies within the environment. If we want to use an unsupported package, it's better to use a standard python environment.

Fore a detailed explanation of setting up a conda virtual environment the articule [Getting started with python environments (using conda)] (https://towardsdatascience.com/getting-started-with-python-environments-using-conda-32e9f2779307) is a great reference.

One last thing. Using anaconda has an additional benefit - installing and management via the terminal is optional. Using the desktop installation you can easily manage all your packages via the anaconda user interface. The article [How to setup your python environment for machine learning with anaconda](https://machinelearningmastery.com/setup-python-environment-machine-learning-deep-learning-anaconda/) is a nice tutorial on getting started with this.


# Packages to get started with

We've looked at setting up virtual environments. Now we'll look at installing some basic packages to get started.

Some of the most used packages in the data science stack are: numpy, pandas, scipy, statsmodels, matplotlib, seaborne, jupiter notebook, pytorch, tensorflow, keras

Try creating an environment for yourself and practice installing and uninstalling some packages.

# Working with Jupiter Notebook and Conda

Opening jupyter notebook from inside a conda or venv virtual environment is be done by calling ```jupyter notebook``` from the command line. When it launches you’ll want to check that you receive the ‘kernel ready’ message and that it says Python 3 on the top right menu bar.

{% raw %}
<img src="http://nicholasjhana.github.io/assets/images/jupyter-kernel-ok.png" alt="" class="full">
{% endraw %}

If you find this isn’t happening, check that ipython is installed by calling the conda list - - v command. This command will list all the current packages and their versions installed in the environment.

{% raw %}
<img src="http://nicholasjhana.github.io/assets/images/jupyter-kernel-stuck.png" alt="" class="full">
{% endraw %}

Sometimes when you open jupyter notebook you find that it doesn’t connect to the iPython kernel, or that a specific package won’t load despite it showing up when you call ```conda list -- v```.

A quick workaround is to reset the iPython kernel your jupyter notebook is connecting to. This can be done by closing down the jupyter instance, and from inside the virtual environment calling the command:

```shell
  python -m ipykernel install --user
```
Once run, open jupyter notebook again and check you have a connection and that the package is now working.
