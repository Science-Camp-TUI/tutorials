# Code Snippets

Code taken fully or in parts from the [BirdNet Analyzer project](https://github.com/kahst/BirdNET-Analyzer){:target="_blank"} under Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International Public License.

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
    # Random noise intensity
    if amount is None:
        amount = RANDOM.uniform(0.1, 0.5)

    # Create Gaussian noise
    try:
        result_noise = RANDOM.normal(min(sig) * amount, max(sig) * amount, shape)
    except:
        result_noise = np.zeros(shape)

    return result_noise.astype("float32")
```

### main function

```python

def main():
    # load the sample chunks from file
    chunks = get_raw_audio_chunks_from_file(str(TEST_FILE_PATH))

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