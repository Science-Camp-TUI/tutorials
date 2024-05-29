# Code Snippets

Code taken fully or in parts from the [BirdNet Analyzer project](https://github.com/kahst/BirdNET-Analyzer){:target="_blank"} under Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International Public License.

The full code for this can be found [in this github repository](https://github.com/Science-Camp-TUI/birdnet-mini). All other code is grouped in [our github orga](https://github.com/orgs/Science-Camp-TUI/repositories).

## Audio Processing

### Load audio file with librosa

```python
def open_audio_file(path: str, sample_rate=48000, offset=0.0, duration=None):
    """Open an audio file.

    Opens an audio file with librosa and the given settings.

    Args:
        path: Path to the audio file.
        sample_rate: The sample rate at which the file should be processed.
        offset: The starting offset in seconds.
        duration: Maximum duration of the loaded content.

    Returns:
        Returns the audio time series and the sampling rate.
    """
    sig, rate = librosa.load(path, sr=sample_rate, offset=offset, duration=duration, mono=True, res_type="kaiser_fast")
    return sig, rate
```

### Save audio signal to file

```python
def open_audio_file(path: str, sample_rate=48000, offset=0.0, duration=None):
   def save_signal(sig, fname: str):
    """Saves a signal to file.

    Args:
        sig: The signal to be saved.
        fname: The file path.
    """
    sf.write(fname, sig, 48000, "PCM_16")
```

### Split audio signal into chunks

```python
def split_signal(sig, rate, seconds, overlap, min_len):
    """Split signal with overlap.

    Args:
        sig: The original signal to be split.
        rate: The sampling rate.
        seconds: The duration of a segment.
        overlap: The overlapping seconds of segments.
        min_len: Minimum length of a split.
    
    Returns:
        A list of splits.
    """
    sig_splits = []

    for i in range(0, len(sig), int((seconds - overlap) * rate)):
        split = sig[i: i + int(seconds * rate)]

        # End of signal?
        if len(split) < int(min_len * rate) and len(sig_splits) > 0:
            break

        # Signal chunk too short?
        if len(split) < int(rate * seconds):
            split = np.hstack((split, noise(split, (int(rate * seconds) - len(split)), 0.5)))

        sig_splits.append(split)

    return sig_splits
``` 

### Create noise

```python

def noise(sig, shape, amount=None):
    """Creates noise.

    Creates a noise vector with the given shape.

    Args:
        sig: The original audio signal.
        shape: Shape of the noise.
        amount: The noise intensity.

    Returns:
        An numpy array of noise with the given shape.
    """
    rs = np.random.RandomState(42)

    # Random noise intensity
    if amount is None:
        amount = rs.uniform(0.1, 0.5)

    # Create Gaussian noise
    try:
        result_noise = rs.normal(min(sig) * amount, max(sig) * amount, shape)
    except:
        result_noise = np.zeros(shape)

    return result_noise.astype("float32")
```

### main function

```python

SCRIPT_DIR = Path(__file__).resolve().parent
TEST_FILE_PATH = SCRIPT_DIR / ".." / "testdata" / "test_1min.wav"

def main():
    # load the sample chunks from file
    sig, rate = open_audio_file(str(TEST_FILE_PATH))
    chunks = split_signal(sig, rate, 3.0, 0.0, 1.0)

    # plot the samples for the first chunk
    plt.plot(chunks[0])
    plt.xlabel("Sample")
    plt.ylabel("Amplitude")

    plt.figure()
    # plot the spectrogram for the first chunk
    frequencies, times, spectrogram = signal.spectrogram(chunks[0], 48000)
    plt.pcolormesh(times, frequencies, 10 * np.log10(spectrogram), shading='auto')
    plt.ylabel('Frequency (Hz)')
    plt.xlabel('Time (s)')
    plt.colorbar(label='Power Spectral Density (dB)')
    plt.title('Spectrogram')

    plt.show()

    # save the first chunk to a file for listening inspection
    save_signal(chunks[0], "chunk.wav")
```

## Birdsong Classification


### Model Class in `model.py`

```python
import operator
import numpy as np

try:
    import tflite_runtime.interpreter as tflite
except ModuleNotFoundError:
    from tensorflow import lite as tflite


class Model:
    def __init__(self, model_path, labels, num_threads=1):

        self._interpreter = tflite.Interpreter(model_path=model_path, num_threads=num_threads)
        self._interpreter.allocate_tensors()
        input_details = self._interpreter.get_input_details()
        output_details = self._interpreter.get_output_details()
        self._output_layer_index = output_details[0]["index"]
        self._input_layer_index = input_details[0]["index"]
        self.labels = labels

    def _load_meta_model(self):
        raise NotImplementedError

    def predict(self, chunk):

        # Make sure the data is in the right format
        # the models has the batch size as the first dimension
        # we are only using one chunk at a time - we need to wrap it in list such that we get
        # dimensions [1, num_samples]
        batch_data = np.array([chunk], dtype="float32")

        # Set the input tensor
        self._interpreter.set_tensor(self._input_layer_index, np.array(batch_data, dtype="float32"))

        # run the model
        self._interpreter.invoke()

        # Retrieve the output tensor
        prediction = self._interpreter.get_tensor(self._output_layer_index)

        # Apply sigmoid function to get confidence score for each class (they do not sum up to 1 though)
        prediction = self._flat_sigmoid(np.array(prediction))

        # Check if the prediction has the right shape
        assert prediction.shape[1] == len(self.labels)

        # Assign scores to labels - we use prediction[0] because we only have one batch entry
        p_labels = zip(range(len(self.labels)), self.labels, prediction[0])

        # Sort by score in ascending order
        p_sorted = sorted(p_labels, key=operator.itemgetter(2), reverse=True)

        # return top 5 predictions
        return list(p_sorted)[:5]

    @staticmethod
    def _flat_sigmoid(x, sensitivity=-1.0):
        return 1 / (1.0 + np.exp(sensitivity * np.clip(x, -15, 15)))

```

### main function

```python

SCRIPT_DIR = Path(__file__).resolve().parent
TEST_FILE_PATH = SCRIPT_DIR / ".." / "testdata" / "test_1min.wav"
MODEL_FILE_PATH = SCRIPT_DIR / "models" / "BirdNET_GLOBAL_6K_V2.4_Model_FP32.tflite"
LABELS_FILE_PATH = SCRIPT_DIR / "models" / "BirdNET_GLOBAL_6K_V2.4_Labels.txt"


def main():
    # load the sample chunks from file
    sig, rate = open_audio_file(str(TEST_FILE_PATH))
    chunks = split_signal(sig, rate, 3.0, 0.0, 1.0)

    # load the labels
    with open(LABELS_FILE_PATH, "r") as f:
        labels = f.read().splitlines()

    # load the model
    model = Model(str(MODEL_FILE_PATH), labels)

    # predict the chunks
    prediction = model.predict(chunks[0])

    # print the predictions
    for label_idx, label, score in prediction[:5]:
        print(f"{label_idx:04d} {label}: {score}")
```
