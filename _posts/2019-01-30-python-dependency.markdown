# Python dependency Management

In order to have some background about the topic, in the current structure of the project which is only one repository consists of all the subprojects, we faced difficulty on sharing and managing the packages dependencies blueprints and satisfying all the needs for virtual environments in the same place. It is worth mentioning that the provided solution here through the whole process should not be considered as the only way to create and manage system level configurations which that simply implies we can use any tools for managing our python interpreters and virtual environments as we want (e.g. Anaconda, Pipenv, venv or ...) but at the last step which we are dealing with the dependencies on packages and using pip, we need to have a common format for our requirement files in such a way that can be shared and seamlessly work on any platform.

Moreover, there are so many good threads on the internet about tools comparison which proved me that this problem is more subjective on the project and actual needs. Although, I decided not to deep dive into the comparison details instead I preferred to share a few links here:

 * https://jakevdp.github.io/blog/2016/08/25/conda-myths-and-misconceptions/
 * https://stackoverflow.com/questions/20994716/what-is-the-difference-between-pip-and-conda
 * https://stackoverflow.com/questions/29950300/what-is-the-relationship-between-virtualenv-and-pyenv
 * https://stackoverflow.com/questions/34398676/does-conda-replace-the-need-for-virtualenv

#### How to-do

1.  **Python interpreter**: For having multiple versions of python in the same system I found **[pyenv](https://github.com/pyenv/pyenv)** really handy and easy to use by having no dependency to the python itself since pyenv uses pure shell script and can be install as a standalone application system-wide. You can find instruction for installing and using it either within past comments or in project github page. 

The beauty of the pyenv is you can install various kind of python builds like pure python by getting source from python.org and compiling it local or getting python builds from anaconda or even special builds like python from Continuum analytics! 

**NOTE**: If you currently have Anaconda installed in your machine then you can consider installing pyenv optional.

```bash
# installing pyenv on mac
$ brew update
$ brew install pyenv

# first thing is installing python versions which you want
# for getting a list of all available versions run the 
# following command:
$ pyenv install --list

# Then install the desired versions:
# for the sake of having examples
# here we install 3 versions
$ pyenv install 3.6.5
$ pyenv install 3.7.0
$ pyenv install 2.7

# Then we need to activate the python version we want 
# either in project directory scope (local) 
# or system wide (global).
# For more visit https://github.com/pyenv/pyenv/blob/master/COMMANDS.md

# init and active desired python on current session
$ pyenv init 
# follow th instruction and alter add the command to your terminal profile
# Load the desired python version
$ pyenv shell 3.6.5
$ python --version # check the python version
Python 3.6.5
```

2. **Python Virtualenvs**: This step can be handled as simple as using standard library like venv on python3 or virtualenv on previous python versions. But here since we are using pyenv for managing python versions we can install a plugin for it in order to manage virtual environments in the same command as before. 

**NOTE**: If you use Anaconda, you can skip this step altogether since the whole process is handling by conda env management and you don't need any extra hassle.


```bash
# installing pyenv-virtualenv
$ brew install pyenv-virtualenv

# init the plugin in current session
$ pyenv virtualenv-init

# create your first virtualenv with desired python version
$ pyenv virtualenv 3.6.5 test_virtualenv

# In order to activate or deactive the virtual env
$ pyenv activate <name>
$ pyenv deactivate
``` 

3. **Dependency management**: This step is the most important one, here we define our requirements hierarchy and the way they can be used in different virtual environments.

The idea is, since in the same codebase (repository) we maintain different projects, packages and modules, it's quite impossible to have only one central requirements file and consider that as input for bootstrapping our virtual environments. The current requirements.txt file in the project is more like a dump from all the packages which are used throughout the whole repository and it can't be used for syncing our venvs due to incompatibility issues. 

The other factor which we need to consider is we might installed and using different tools as our system/python-package management (e.g. Anaconda, native python packages, aws bundles...) and on top of that also we might have different packages to be installed in our working stations like IDE compatibility python packages which probably we don't want them to be shared with others. Last but not least, we probably will have a setup on on production stages which is vary from development or testing. Besides, having different stages for automate deployments should be considered as a crucial need for making a seamless deployment cycle.

To address all above problems, one approach would be having separate standalone virtual environments for certain parts of the project which could cause incompatibility on packages dependencies. Moreover, we can benefit from some level of abstraction by defining requirements in multiple chunks which can be all driven from each other in an explicit hierarchy of stages. 

The solution for managing multiple virtual environments is mostly covered in previous parts of this comment and can be handled either ways but regarding the hierarchy of the requirements we can use a pip package called **pip-tools** which efficiently compare all the packages sub-dependencies and compiles a list of all the required packages respectively in such a deterministic way which guarantee we will have the exact same environment setup after deployment.

In order to split requirements down into multiple chunks, defining the hierarchy is the initial step:

![Screenshot_2019-01-25_at_15.04.04](/uploads/0bb3aba6907f52630338a5203a998d69/Screenshot_2019-01-25_at_15.04.04.png) 

As you already found in above scheme, we can have some sort of top down hierarchy which any of upper level requirement chunk can be load into the lower levels. Notice that, none of the middle stages are not compulsory and can be omitted for constructing the lower level requirements.   

Also, pip-tools provides an abstraction interface on defining requirements file and basically it expects us only define main packages and all the sub-packages will be automatically linked and dumped into output requirements.txt files. A simple instructions of .in and .txt files definition in pip-tools you can check out the following link. https://koed00.github.io/managed-environments-with-piptools/

But in order to have a more realistic example consider the following snippets:

```bash
# imagine for make an fresh venv without any package installed
# go to the project directory
$ cd project_dir

# install the pip-tools via pip
# ATTENTION: DO NOT UPDATE PIP TO ITS LATEST VERSION.
$ pip install pip-tools

# lets define the our requirements chunks
$ mkdir requirements
$ cd requirements

# As top level requirement file lets create main.in first
# (all the packages name in the following command are just arbitrary 
# can be literally anything)
# here we just see the content of the file which we created

# main.in content:
awscli
boto3

# dev.in content:
-r main.in
pandas
numpy

# data_simulation.in content:
-r dev.in
geopandas
turicreate
rtree

# mark-local.in content:
-r data_simulation.in 
pep98
pylint


# Running the pip-compile on the last file of the chain 
# to compile and output .txt pip requirements file
$ pip-compile mark-local.in  # it could take awhile

# Run pip-sync to actually install the package
$ pip-sync mark-local.txt  # consider the txt extension
```

###### A few notes:
 * Consider that the outcome of the pip-compile will be an `*.txt` file with same name as input file.
 * Do not upgrade the pip to version 19.0 or above since it has some unresolved bugs.
 * pip-compile file can be run on any file in the hierarchy not only the last in the chain.
 * You might ran into maximum pip iteration threshold if you have many nested .in files which can be resolve by increasing that threshold system-wide (google it).
 * For adding or removing any package, just remove it from the `*.in` file and run pip-compile to regenerate the `*.txt` file and at end pip-sync to reflect on venv. That automatically take care of all sub-dependency and remove them all too.
 * Since we usually don't pin the packages in `*.in` files (except when we have to), in order to update or upgrade packages we need to run pip-compile on regular basis. (for more on pinning package read [this article](https://nvie.com/posts/pin-your-packages/)).
 * Both `*.in` and `*.txt`  files should be tracked with version control system (git in our case).
