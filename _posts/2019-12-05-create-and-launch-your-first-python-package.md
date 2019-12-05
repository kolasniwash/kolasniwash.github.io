---
title: "Create and launch your first python package "
date: 2019-12-05T23:21:30-01:00
---

Starting my journey into data science one thing I took for granted was the packages we use. Numpy, Pandas, and Sklearn were just a given to everyday learning.

It was only later that I learned how to move beyond basics (and sometimes beyond the advance) you’d need to modify these packages, and depending on the application, develop entirely new functionalities.

In this post I want to share with how you can make your own package and introduce a [small package I made](https://github.com/nicholasjhana/ml-energy-utils) to help with preparing sequence data for energy forecasting ML applications. The idea of writing your own package might sound like some mammoth undertaking that only developers would do. And if you asked me 3 months ago I’d have thought the same. The reality is it's an excelent way to gather up and reuse functions.

Knowing how to make your own package is highly relevant in data science as you begin to work in teams, and with models going into production. Consider all the times you wrote the same or similar code to help you do a fairly generic task in your workflow? Or when your college asks you to share how you solved a problem? By turning your code into a package these situations become easy.

## Basic contents of a python package
Packages come in many flavours yet the core content is always the following files. We’ll explain each in detail.
- README.md
- License.txt
- setup.py
- ```__init__.py```

The ```README.md``` file is where you explain what the package is, how it works, where to find the source code, and how others can contribute. Like a readme for a GitHub repo we want the README.md to share relevant information that helps others (and sometimes ourselves) understand what is contained in the package.

The ```License.txt``` file is a that describes how the package contents my be used. There are some standard Open Source licenses used including [GPLv3](https://www.gnu.org/licenses/gpl-3.0.html), [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0), [BSD](https://opensource.org/licenses/BSD-3-Clause), and [MIT License](https://mit-license.org/). This [great post](https://www.freecodecamp.org/news/how-open-source-licenses-work-and-how-to-add-them-to-your-projects-34310c3cf94/) goes into details on each license type and where its used.  

The ```setup.py``` contains all the information about how to build your package. It calls ```setuptools``` the package build utility for python, and passes information such as name, version, and the code files to include. Later we’ll see an example of exactly what goes inside.

```__init.py__``` is the file that imports the directory as a package. This file can be left blank. However we’ll use it to allow us to import our functions or classes directly from the root package. 

### Structure of package
Below you'll find the structure of the package contents. The package-name folder contains the files mentioned above plus the scripts with the actual code contents. There is one extra file we haven't talked about, the ```setup.cfg```. Fornow we won't worry about it, just know that it's used when uploading your file to PyPi.

```bash
-containing-folder
    - setup.cfg
    - package-name
        -README.md
        -License.txt
        - setup.py
        - __ini__.py
        - function-1.py
        - function-2.py
        - ...
```

## How to make a package locally
In this section we’ll go through the steps to make a package locally using a simple package I created to help with energy forecasting projects I have been working on. The package is ml-energy-utils and can be found on GitHub and PyPi.

1 Before launching into making a package you’ll need some code you want to turn into a package. This could be a set of utility functions, models, or classes. Place all the ```.py``` scripts you want in the package in a directory. Name this directory whatever you’d like the package to be called.

2 Inside the new package directory add the ```__init__.py``` file. Here we want to add the references to where our functions/classes are located. Let's look at an example from the new package.

```python 
from .make_samples import split_sequences
from .preprocessing import transform_to_windows, plot_hour, make_shifted_features, rename_cols, trim_length
from .make_datasets import read_entsoe_data, format_entsoe_load_forecast_data, combine_data
```

Why add these? They tell the package at setup where to find the different functions. Our package will work if we don’t add them but we’d have to add an extra line when we want to import functions for use in python, iPython, Jupyter, etc.

```python
from ml_energy_utils.make_samples import split_sequences
```

When we add the references we can source the packge's functions with a simpler import statement.

```python
from make_samples import split_sequences
```

3 Create the ‘’’setup.py’’’ file and add details about the package. This is where we define the name of our package, the version etc. Make sure the name is the same as the folder contining the package. 

```python
from setuptools import setup

setup(
    name=“ml_energy_utils”,  #the name of your package
    version=“0.1”,  #current version
    author=“Nicholas Jhana”, #your name
    author_email=“nicholas@”, #your contact details
    description=“Simple set of utilities for working with ENTSOE energy data“, #short description of the package
    url="https://github.com/nicholasjhana/ml-energy-utils”,  #where to find the source code
    packages=setuptools.find_packages(), #lists all the files within the package
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent"],  #metadata about the package for pip
    python_requires='>=3.6',  #set the package requirements. In this case python 3.6 or higher
)
```

4 Navigate to the folder containing your package folder. In a virtual environment run the following pip command ```pip install -U .``` This will update any packages in the current folder.

5 Open python, iPython, or Jupiter notebook and check your package works. If it does you’ll be able to run ```python from ml-energy-utils import split_sequences``` and be able to call the function.

## Continuing with your package
If you’ve gotten this far you know how to make a simple package for your own use. The next step is to make your package available on a service such as PyPi. PyPi is where Pip looks for a package when you call ```pip install```. It’s pretty cool to think that anyone could download and use a package you’ve made with such a ubiquitous command. 

If you want to learn more about making packages and uploading them to PyPi check out [the full tutorial](https://packaging.python.org/tutorials/packaging-projects/) from the python.org website.
