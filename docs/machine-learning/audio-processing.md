# Audio Processing

## Introduction

The goal of this lesson is to

* load an audio file 
* divide the file into pieces of a specific length (chunks)
* plot the signal of a chunk
* save the signal to a file and listen to it 

In order for you to get a feeling of how audio data is represented in a computer read the following section.


### Digital representation of audio data[^1]

The way of a sound wave to a digital representation is as follows:

1. **Sound Capture**: Sound waves from voices, instruments, or any audio source hit the microphone.

2. **Electrical Conversion:**: Inside the microphone, there's a device that converts these sound waves into electrical signals. These signals are still analog, which means they vary continuously like the original sound waves. The microphone sends these analog signals to an ADC (Analog-Digital-Convert). 

3. **Sampling**: The ADC takes snapshots of the analog signal at regular intervals. This is called 'sampling'. Each snapshot measures the amplitude (volume) of the audio signal at that specific moment. The rate at which this happens is called `sampling rate`. A typical sampling rate is 44100 Hz which means the ADC takes 44100 measurements per second. 

4. **Quantization**: Each sampled amplitude value is then converted into a digital number. The range of numbers that can be used is determined by the 'bit depth'. Higher bit depth means the audio can be more accurately represented. A typical bit depth ist 16 bit. That means that 2^16=65536 different values can be stored for each sample. 

5. **Binary Representation**: The numbers are converted into binary code, which is a series of 0s and 1s, because that's the language computers understand. A wave file contains all the samples at their specific bit depth. 

6. **Compression and Encoding (Optional)**: Sometimes the raw audio data is compressed to save space. Famous compression methods are MP3 (MPEG-1 Layer 3) or AAC (Advanced Audio Coding).

7. **Storage**: Finally, these binary numbers are stored in a file on the computer. If it's a WAV file, it also includes a header with information about the audio like its sample rate, bit depth, and channels (mono or stereo).


You'll find a more detailed introduction [here](https://woodandfirestudio.com/en/sample-rate-bit-depth/){:target="_blank"}[^2]

[^1]: The section was partly produced by GPT-4 Turbo and manually modified afterwards. 

[^2]: Link leads to external resources. Neither TU Ilmenau no any other party involved in this tutorial are responsible for the content linked. 

## Lesson Steps

the file [test_1min.wav](./files/test_1min.wav){:target="_blank"} contains a one minute recording of bird sounds. 

1. Download the file [test_1min.wav](./files/test_1min.wav){:target="_blank"} and store it in your project folder under a new folder you name `testdata`:

    ```
    └───birdnet-mini
        ├───birdnet_mini
        │       main.py
        │
        └───testdata
                test_1min.wav
    ```

2. Use the [`load`](https://librosa.org/doc/latest/generated/librosa.load.html#librosa.load){:target="_blank"}-function  from the Python package `librosa` to load the file in python. 