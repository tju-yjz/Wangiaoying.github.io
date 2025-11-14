# RAG Pulse Workload Replay Project

## Overview
This project aims to replay workload traces against a single online instance using the RAG Pulse framework. It includes preprocessing of trace data, simulation of real-time requests, and collection of metrics such as Time to First Token (TTFT) and Time per Output Token (TPOT).

## Project Structure
```
RAGPulse/example
└── single_online_instance
├── ReadMe.md
├── logger.py
├── logs/
├── main.py
├── metrics/
├── metrics.py
├── online_server.py
├── preprocess_data.py
└── start.sh
```

## Files Description

 ### 1. logger.py
  - **Purpose**: Implements a custom logging class `RAGPulseLogger` for logging events and errors.
  - **Main Features**:
    - Logs messages to both console and file simultaneously.
    - Supports multiple log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL).
    - Automatically creates log directories and handles log file rotation.
  - **Functions**:
    - `__init__`: Initializes the logger with specified log level, directory, and file.
    - `log`: Logs messages with optional string formatting.
    - `debug`, `info`, `warning`, `error`, `critical`: Convenience methods for logging at different levels.
    - `exception`: Logs exceptions with traceback.
### 2. main.py
  - **Purpose**: Main script to orchestrate the workload replay process.
  - **Functions**:
    - `get_args`: Parses command-line arguments for API key, model name, trace directory, etc.
    - `main`: Initializes logger, preprocesses data, runs the online server, and collects metrics.
  - **Usage**:
  ```bash
  python main.py 
  ```

 ### 3. metrics.py
  - **Purpose**: Collects and saves metrics (TTFT and TPOT) from workload replay.
  - **Main Features**:
    - Stores metrics in memory and saves them to a JSON file upon completion.
  - **Functions**:
    - `add_metrics`: Adds TTFT and TPOT metrics to the collection.
    - `save_metrics`: Saves metrics to a JSON file in the specified directory.

### 4. online_server.py
  - **Purpose**: Simulates real-time requests to an online instance and collects response metrics.
  - **Main Features**:
    - Uses a thread to handle incoming requests and simulate real-time behavior.
    - Supports both streaming and non-streaming response modes.
  - **Functions**:
    - `__init__`: Initializes the online server with API key, model name, and logger.
    - `request_worker`: Worker thread to process incoming requests.
    - `response`: Sends requests to the online instance and calculates TTFT and TPOT.
    - `run`: Simulates real-time requests based on trace data.
    - `close`: Stops the server and saves metrics.

 ### 5. preprocess_data.py
  - **Purpose**: Preprocesses trace data and hash IDs for workload replay.
  - **Main Features**:
    - Reads trace files and hash ID files.
    - Simulates token lists for each hash ID based on token length.
  - **Functions**:
    - `simulate_token_lists`: Generates random token lists for hash IDs.
    - `simulate_token_lists_to_input_text`: Converts token lists to input text using a tokenizer.
    - `preprocess`: Main preprocessing pipeline to load traces, hash IDs, and simulate input text.

### 6. start.sh
  - **Purpose**: Starts the VLLM server with specified model and configuration.
  - **Usage**:
```
bash start.sh
```

## Environment Setup
- Create Virtual Environment:
```
conda create -n myenv python=3.12 -y
conda activate myenv
```
- Install Dependencies:
```
pip install vllm==0.6.2
conda install conda-forge::pyairports
```
# Parameter Configuration

## Main Configuration via `main.py`
&lt;details&gt;
  &lt;summary&gt;Click to expand!&lt;/summary&gt;

The `main.py` script utilizes the `argparse` library to parse command-line arguments. These arguments are essential for customizing the behavior of the workload replay process. Here's a breakdown of the available parameters:

- **--api_key**
  - *Purpose*: Specifies the OpenAI API key used for authentication.
  - *Default*: `"dummy_key"`
  - *Required*: Yes, replace with your actual API key.

- **--api_base**
  - *Purpose*: Defines the base URL of the online instance.
  - *Default*: `"http://0.0.0.0:8000/v1"`
  - *Required*: Yes, ensure this matches your online instance's URL.

- **--model_name**
  - *Purpose*: Indicates the model name to use for generation.
  - *Default*: `"Qwen2.5-14B-Instruct"`
  - *Required*: Yes, specify the model you intend to use.

- **--trace_dir**
  - *Purpose*: Points to the directory containing trace files.
  - *Default*: `"/home/TwenWorkload/trace"`
  - *Required*: Yes, update this to the directory where your trace files are stored.

- **--log_dir**
  - *Purpose*: Specifies the directory where logs will be saved.
  - *Default*: `"/home/TwenWorkload/example/single_online_instance/logs"`
  - *Required*: Yes, ensure this directory exists or modify as needed.

- **--time_scale_factor**
  - *Purpose*: A factor to speed up the workload replay.
  - *Default*: `10`
  - *Required*: No, adjust based on your needs for faster or slower replay.

- **--use_trace_num**
  - *Purpose*: The number of traces to use for workload replay, starting from the front.
  - *Default*: `5`
  - *Required*: No, modify this number based on the volume of traces you want to process.

- **--tokenizer_path**
  - *Purpose*: The path to the tokenizer.
  - *Default*: `"/home/share/models/Qwen2.5-14B-Instruct"`
  - *Required*: Yes, ensure this path points to your tokenizer's location.

- **--streaming**
  - *Purpose*: A boolean flag indicating whether to use streaming generation.
  - *Default*: `True`
  - *Required*: No, set to `False` if streaming generation is not desired.

&lt;/details&gt;

## Server Configuration via `start.sh`
&lt;details&gt;
  &lt;summary&gt;Click to expand!&lt;/summary&gt;

The `start.sh` script is used to launch the VLLM server with specific configurations that optimize performance and resource utilization. Here are the key parameters used in the script:

- **CUDA_VISIBLE_DEVICES=0**
  - *Purpose*: Restricts the server to use the first available GPU device.
  - *Required*: No, adjust the device number based on your GPU setup.

- **vllm serve /home/share/models/Qwen2.5-14B-Instruct**
  - *Purpose*: Specifies the model to serve.
  - *Required*: Yes, ensure the path to your model is correct.

- **--served-model-name Qwen2.5-14B-Instruct**
  - *Purpose*: Names the served model.
  - *Required*: Yes, match this with your model's name.

- **--port 8000**
  - *Purpose*: Sets the port number for the server.
  - *Required*: Yes, ensure this port is available and not in use by another service.

- **--host 0.0.0.0**
  - *Purpose*: Allows the server to accept connections from any IP address.
  - *Required*: Yes, use this to make the server accessible externally.

- **--gpu-memory-utilization 0.8**
  - *Purpose*: Limits GPU memory usage to 80%.
  - *Required*: No, adjust this value based on your GPU's capabilities.

- **--trust-remote-code**
  - *Purpose*: Allows execution of remote code (use with caution).
  - *Required*: No, include this only if you trust the source of the code.

- **--max-model-len 20000**
  - *Purpose*: Sets the maximum length of the model.
  - *Required*: No, adjust based on your model's requirements.

- **--enable-prefix-caching**
  - *Purpose*: Enables caching for improved performance.
  - *Required*: No, use this to enhance performance if needed.

&lt;/details&gt;

## Running the Project
- Start VLLM Server:
```
cd /path/to/single_online_instance
bash start.sh
```
- Run Workload Replay:
```
python main.py 
```
## Expected Output
The script will log events and metrics during the workload replay process. Example output:
```
【INFO】2025-11-12 05:00:18   logger.py:138  Arguments: Namespace(api_key='dummy_key', api_base='http://0.0.0.0:8000/v1', model_name='Qwen2.5-14B-Instruct', trace_dir='/home/TwenWorkload/trace', log_dir='/home/TwenWorkload/example/single_online_instance/logs', time_scale_factor=10, use_trace_num=5, tokenizer_path='/home/share/models/Qwen2.5-14B-Instruct', streaming=True)
【INFO】2025-11-12 05:00:18   logger.py:138  RAG Pulse Online Instance Logger initialized.
【INFO】2025-11-12 05:00:18   logger.py:138  Preprocessing traces and hash_ids...
【INFO】2025-11-12 05:00:19   logger.py:138  Loaded 7106 traces from 0_trace.jsonl.
【INFO】2025-11-12 05:00:19   logger.py:138  Loaded vocab size: 151643 from /home/share/models/Qwen2.5-14B-Instruct/vocab.json.
【INFO】2025-11-12 05:00:24   logger.py:138  Loaded 26924 hash_ids from ['1_sys_prompt.jsonl', '2_passages.jsonl', '3_history.jsonl', '4_user_input.jsonl', '5_web_search.jsonl'].
【INFO】2025-11-12 05:00:33   logger.py:138  Preprocessed 5 traces.
【INFO】2025-11-12 05:00:33   logger.py:138  Initializing OnlineServer...
【INFO】2025-11-12 05:00:33   logger.py:138  OnlineServer initialized.
【INFO】2025-11-12 05:00:33   logger.py:138  Starting workload replay...
【INFO】2025-11-12 05:00:33   logger.py:138  Starting simulation of real-time requests...
【INFO】2025-11-12 05:00:41   logger.py:138  Sent request at time 8.20s with input_text length 15792 and original scaling timestamp 8.2.
```
## Expected Results
- Logs: Logs will be saved in the logs/ directory.
- Metrics: Metrics (TTFT and TPOT) will be saved in the metrics/ directory in a JSON file.
- Console Output: Real-time logging of events and metrics during workload replay.

## Troubleshooting
- Ensure the VLLM server is running before executing main.py.
- Verify that all paths (e.g., model directory, trace directory) are correctly specified.
- Check logs for any errors or warnings during execution.
- Ensure that the timestamps in the trace records are in ascending order.





