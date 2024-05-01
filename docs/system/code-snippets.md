# Code Snippets

Here are some code snippets from the system tutorial.  The full code can be found in the the [birdnet-mini repository on GitHub](https://github.com/Science-Camp-TUI/birdnet-mini){:target="_blank"}.


## File Reader - run method

```python
    def run(self):
        for file in self._files:
            file_ts, file_lat, file_lon = self._metadata.get_timestamp_lat_lon(str(Path(file).stem))
            if self._interrupted:
                break
            if file_ts is None or file_lat is None or file_lon is None:
                print(f"[filereader] Skipping file {file} - no metadata found")
                continue
            print(f"[filereader] Loading file {file}")
            # since we know that our audio is 5min max - load it at once
            sig, rate = open_audio_file(file, self._sample_rate)
            chunks = split_signal(sig, rate, self._chunk_size, self._overlap, self._min_len)
            for chunk_idx, chunk in enumerate(chunks):
                if self._interrupted:
                    break
                chunk_ts = file_ts + datetime.timedelta(seconds=chunk_idx * self._chunk_size)
                time.sleep(2)  # wait 2 seconds between chunks since they represent 3 seconds of realtime audio
                self._sample_queue.put((chunk, chunk_ts, file_lat, file_lon))
        if self._interrupted:
            print("[filereader] Interrupted")
        else:
            print("[filereader] Finished reading all files")

```

# BirdClassifier - run method

```python
    def run(self):
        while not self._interrupted:
            try:
                chunk, timestamp, lat, long = self._sample_queue.get(timeout=0.1)
            except queue.Empty:
                continue
            prediction = self._model.predict(chunk)
            class_id = prediction[0][0]
            class_label = prediction[0][1]
            confidence = prediction[0][2]
            print(f"[classifier] Top prediction {class_label}({class_id}), with confidence: {confidence:.02f}")
            data = {
                "class_id": class_id,
                "confidence": confidence,
                "lat": lat,
                "lon": long,
                "timestamp": timestamp
            }
            self._result_queue.put(data)
```

# SerialSender - run method


```python
    def run(self):
        while not self._interrupted:
            try:
                data_dict = self._result_queue.get(timeout=0.1)
            except queue.Empty:
                continue

            data_simple = [data_dict['class_id'], data_dict['confidence'],
                           data_dict['lat'], data_dict['lon'], data_dict['timestamp'].timestamp()]

            print(f"[serial] sending result data : {data_simple}")

            binary_data_struct = struct.pack('Hfffd', *data_simple)

            if len(binary_data_struct) != FIXED_MESSAGE_LENGTH:
                print(f"[serial] Error: Message size does not match: {len(binary_data_struct)} <> {FIXED_MESSAGE_LENGTH} bytes")
            else:
                self._serial.write(binary_data_struct)
            time.sleep(self.send_interval)

        self._serial.close()
```

# main function

```python
SCRIPT_DIR = Path(__file__).resolve().parent
AUDIO_FOLDER_PATH = Path("k:") / "science-camp" / "Data"
METADATA_FILE_PATH = Path("k:") / "science-camp" / "SMM11597_Summary.txt"
MODEL_FILE_PATH = SCRIPT_DIR / "models" / "BirdNET_GLOBAL_6K_V2.4_Model_FP32.tflite"
LABELS_FILE_PATH = SCRIPT_DIR / "models" / "BirdNET_GLOBAL_6K_V2.4_Labels.txt"


def main():

    sample_queue = Queue()
    result_queue = Queue()

    with open(LABELS_FILE_PATH, "r") as f:
        labels = f.read().splitlines()

    model = Model(MODEL_FILE_PATH, labels)

    # Create the file_reader, classifier, and serial_sender objects
    file_reader = FileReader(AUDIO_FOLDER_PATH, METADATA_FILE_PATH, 48000, sample_queue)
    classifier = BirdClassifier(model, sample_queue, result_queue)
    serial_sender = SerialSender(result_queue, "COM4")

    # Start the threads
    serial_sender.start()
    classifier.start()
    file_reader.start()

    # enter main loop
    try:
        while True:
            time.sleep(0.2)
    except KeyboardInterrupt:
        print("[main ] Externally interrupted. Stopping threads.")
        file_reader.interrupt()
        classifier.interrupt()
        serial_sender.interrupt()

    file_reader.join()
    classifier.join()
    serial_sender.join()

```
