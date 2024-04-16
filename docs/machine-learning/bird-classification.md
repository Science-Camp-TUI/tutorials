# Birdsong Classification

In this lesson you will classify bird by their songs. The instructor will give you a short introduction to the topic of neural networks and machine learning. After that you will create the minimal code to run the classification.


## Lesson Steps

### Part 1: Create a Model class

We will now implement a Python class that is managing the bid classification model[^1]

A class is an abstract data type that defines a blueprint for objects. It contains attributes and methods that are shared by all objects of this class.

[^1]: All models used in this Tutorial are based on the BirdNET model by Stefan Kahl. The model is licensed under the Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License. You can find more information about the model [here](https://www.sciencedirect.com/science/article/pii/S1574954121000273){:target="_blank"} and [on this GitHub page](https://github.com/kahst/BirdNET-Analyzer){:target="_blank"} .

1. Download the files 

    * [BirdNET_GLOBAL_6K_V2.4_Model_FP32.tflite](./files/BirdNET_GLOBAL_6K_V2.4_Model_FP32.tflite){:target="_blank"}
    * [BirdNET_GLOBAL_6K_V2.4_Labels.txt](./files/BirdNET_GLOBAL_6K_V2.4_Labels.txt){:target="_blank"} 
    
    and store it in your package folder `birdnet_mini` under a new folder you name `models`. Also create an empty `__init__.py` file in the `models` folder.

2. Create a file `model.py` in the `birdnet_mini` folder. This will contain all your model class. Your directory shall now look like this:

    ```
    └───birdnet-mini
        ├───birdnet_mini
        |       audio.py
        │       main.py
        |       model.py
        |       __init__.py
        ├───models
        │       BirdNET_GLOBAL_6K_V2.4_Labels.txt
        │       BirdNET_GLOBAL_6K_V2.4_Model_FP32.tflite
        │       __init__.py
        └───testdata
                test_1min.wav
    ```

3. Add the following code to the `model.py` file. This is the basic structure of the model class.

    ```python
    class Model:
        def __init__(self, model_path=None, num_threads=1):
            raise NotImplementedError

        def predict(self, samples):
            raise NotImplementedError
            
        @staticmethod
        def _flat_sigmoid(x, sensitivity=1.0):
            return 1 / (1.0 + np.exp(sensitivity * np.clip(x, -15, 15)))
    ```

4. The __init__ initializes the object and sets all important class members (in other languages this can be seen as a constructor).We will create the interpreter that is running the model. We use Tensorflow Lite, since this is also available on mini computers such as the RaspberryPi. Load the model file using the [tflite.Interpreter](https://www.tensorflow.org/api_docs/python/tf/lite/Interpreter){:target="_blank"} function and store the interpreter as the member variable `self._interpreter`.

5. [Allocated the tensors](https://www.tensorflow.org/api_docs/python/tf/lite/Interpreter#allocate_tensors){:target="_blank"} for the interpreter an then retrieve [input and output information](https://www.tensorflow.org/api_docs/python/tf/lite/Interpreter#get_input_details){:target="_blank"} from the model and store it in the member variables         

    * `self._input_layer_index`
    * `self._output_layer_index`

6. Once we have loaded the model in the `__init__`function we will now implement the `predict` function. This function will take a single chunk of samples as input and return the prediction of the model. The steps are:

    * convert the samples to a Numpy array of type `float32`
    * set the input tensor of the interpreter with the sample data via the [set_tensor](https://www.tensorflow.org/api_docs/python/tf/lite/Interpreter#set_tensor){:target="_blank"} function
    * invoke the interpreter with the TFLite [invoke](https://www.tensorflow.org/api_docs/python/tf/lite/Interpreter#invoke){:target="_blank"} function
    * retrieve the result tensor via the [get_tensor](https://www.tensorflow.org/api_docs/python/tf/lite/Interpreter#get_tensor){:target="_blank"} function
    * apply the sigmoid function `self._flat_sigmoid` to the result tensor and return the result (). 

7. Now it it time to test your model. In the `main.py` file import the `Model` class and create an instance of it. Load the test file and split it into chunks of 3 seconds. Then predict the class of the first chunk and print the results. 