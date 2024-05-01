# Starter Tutorial

## Introduction 

In this lesson you will 

* Create the basic structure of your Python project you are going to work on the next days
* Start your Python editor / IDE (PyCharm) and load your project
* Create a basic "Hello World" program and run it


If you never used Python before it's recommended that you get familiar with the basic language features. Don't be afraid. It's a fairly simple computer programming language and good to learn. It will be fun!

Some links to get started:

* [Video Deutsch 10 min, Absolute Beginner, online editor](https://www.youtube.com/watch?v=9mmVa6O-hzQ){:target="_blank"}[^1]

* [Video Deutsch 60 min, Beginner, uses PyCharm](https://www.youtube.com/watch?v=362fjQdpFlc){:target="_blank"}[^1]

* [Crashkurs Deutsch, 24 Videos Playlist, uses PyCharm](https://www.youtube.com/watch?v=oxXAb8IikHM&list=PL_pqkvxZ6ho3u8PJAsUU-rOAQ74D0TqZB){:target="_blank"}[^1]

* [Official reference tutorial, English](https://docs.python.org/3.11/tutorial/index.html){:target="_blank"}[^1]


[^1]: Link leads to external resources. Neither TU Ilmenau no any other party involved in this tutorial are responsible for the content linked. 

## Lesson Steps

### Create Python project structure

1. Create a new project folder for your Songbird classification project: `birdnet-mini`
2. Inside the folder create another folder called `birdnet_mini` (Notice the underscore!) this will be your Python package name.
3. Inside the folder `birdnet_mini` create an empty file `__init__.py` and a file with the name `main.py` and a file . This will be the main entry point to your program. Your project folder should now look like this:

    ```
    └───birdnet-mini
        └───birdnet_mini
                main.py
                __init__.py
    ```

### Load your project into PyCharm and setup your project

Now you are ready to start!

1. Open the PyCharm IDE (Integrated Development Environment) by clicking its icon:

    ![PyCharm Icon](pictures/pycharm_symbol.png){: style="width:50px"}

2. Select Open from the file menu

    ![PyCharm Icon](pictures/pycharm_open.png){: style="width:250px"}

3. Select your `birdnet-mini` project folder

So your project is loaded but there is (probably) no Python interpreter associated with it. 

We've already installed a recent Python version for you on your workshop computer. We did not simply use the [standard installer](https://www.python.org/downloads/) which would be fine if you just do small projects and tutorials. If you start working on larger projects that use a lot of different Python packages you probably want to switch between different Python version and package (versions). This is called an environment. 

A system that allows you to manage and easily switch different python environments is [Miniconda](https://docs.anaconda.com/free/miniconda/index.html) which is already install at your computer. We even created an environment for you that you can use for this workshop. PyCharm support Conda environments. 

4. Select `File->Settings` form the PyCharm Menu

5. Select the `Python Interpreter` under the `Project` section and then select `Add Interpreter->Add Local Interpreter` on the top right:

    ![PyCharm Icon](pictures/pycharm_interpreter.png){: style="width:750px"}

6. In the following dialogue select `Conda Environment` on the left side and tick `Use existing environment`. From the Dropdown menu select `birdnet-minimal-dev`

7. Confirm all Dialogues by clicking `OK`multiple times. 

From now on any code that you are going to run will run using this environment.

Now you are all set to start.

### Create your first program and run it

You will now write your first typical "Hello World" program. 


1. From the Project Explorer on the left side, open the file `main.py` by double clicking it. 

2. Now the task is up to you. Create a function called `main` in that file that prints "Hello World" and then returns 0

3. Define the main entry point into your program using the (partly weird line):

    ```python
    if __name__ == "__main__":
    ```

    and call your `main` function from there. 

4. Run the program by right-clicking `main.py` and select `Run 'main'`.
