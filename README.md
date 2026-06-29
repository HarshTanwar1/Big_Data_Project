<div align="center">

# 🛰️ Real-Time IoT Attack Detection via Streamed Ensemble Learning

A Big Data Engineering project — detecting cyber-attacks on IoT devices in real time using an online, incrementally-trained machine learning pipeline

<br>

![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-Ensemble-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)
![TensorFlow](https://img.shields.io/badge/TensorFlow-Keras-FF6F00?style=for-the-badge&logo=tensorflow&logoColor=white)
![WebSockets](https://img.shields.io/badge/WebSockets-Streaming-010101?style=for-the-badge&logo=socketdotio&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-Data-150458?style=for-the-badge&logo=pandas&logoColor=white)

</div>

<br>

## 📌 Overview

Most ML pipelines train **once** on a static dataset. This project does the opposite: it builds a custom **client–server streaming pipeline** that pushes IoT network records **row-by-row over a WebSocket**, then trains an **ensemble model incrementally** on each incoming chunk — continuously improving as new data arrives, the way a real intrusion-detection system would.

The pipeline classifies **cyber-attacks on IoT devices** (e.g. DDoS, injection, backdoor, scanning) in near real time, using the [TON_IoT datasets](https://research.unsw.edu.au/projects/toniot-datasets) from UNSW.

The repo ships **two complete experiments**:

| Experiment                       | Description                                                                                                                                                             |
| :------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🔌 IoT_Modbus                    | Streaming + ensemble pipeline on a **single device type** (Modbus industrial-protocol data, ~287K records). Includes a multithreaded client and live evaluation plots.  |
| 🌐 All_IoT_Devices_Combined      | Same pipeline on a **heterogeneous dataset** built by cleaning and merging all 7 TON_IoT device datasets — the model must classify attacks across mixed device sources. |

<br>

## ✨ Key Highlights

- ⚡ **Custom real-time streaming engine** — a from-scratch WebSocket server/client pipeline with per-row acknowledgements for back-pressure.
- 🔄 **Online / incremental learning** — trains per chunk and folds each new model into a running ensemble instead of retraining from scratch.
- 📦 **Adaptive chunk sizing** — automatically grows a chunk when it can't be trained (e.g. only one class present yet), solving a real streaming-data pitfall.
- 🧵 **Multithreaded client** — decouples network I/O from model training so streaming never blocks on a training step.
- 🗳️ **7-model voting ensemble** — LR · SVM · RF · LDA · KNN · CART · Naive-Bayes, plus standalone **LSTM & GRU** deep-learning models.
- 🧹 **End-to-end data engineering** — cleans, deduplicates, encodes, and merges 7 heterogeneous datasets into one trainable source.
- 📊 **Live metrics & visualization** — per-chunk accuracy, precision, recall, F1 and training time, tracked and plotted with Plotly.

<br>

## 🛠️ Tech Stack

| Layer                       | Technologies                                                                         |
| :-------------------------- | :----------------------------------------------------------------------------------- |
| **Language / Env**          | Python 3, Jupyter Notebook                                                           |
| **Streaming & Concurrency** | `websockets`, `asyncio`, `threading`                                                 |
| **Data**                    | `pandas`, `numpy`                                                                    |
| **Classical ML**            | scikit-learn — LR, SVM, RF, LDA, KNN, CART, Gaussian Naive-Bayes, `VotingClassifier` |
| **Deep Learning**           | TensorFlow / Keras — LSTM, GRU                                                       |
| **Preprocessing**           | `LabelEncoder`, `StandardScaler`, `train_test_split`                                 |
| **Persistence**             | `pickle`                                                                             |
| **Visualization**           | `plotly`, `matplotlib`                                                               |

<br>

## ⚙️ Features & Functionality

- **Streaming server** ([`server.ipynb`](IoT_Modbus/server/server.ipynb)) — reads a CSV and streams it one row at a time over `ws://localhost:8765` with a configurable delay, waiting for an acknowledgement after every row.
- **Incremental client** ([`client.ipynb`](IoT_Modbus/client/client.ipynb)) — buffers rows into chunks, trains an ensemble per chunk, and merges each new chunk's model into the running ensemble (online learning).
- **Adaptive chunking** — starts at `INITIAL_CHUNK_SIZE`, switches to `FINAL_CHUNK_SIZE`, and grows by `CHUNK_SIZE_INCREMENT_FACTOR` when a chunk can't be trained.
- **Multithreaded variant** ([`client_multithread.ipynb`](IoT_Modbus/client/client_multithread.ipynb)) — uses a background thread + `threading.Event` so receiving and training run concurrently.
- **Heterogeneous data pipeline** — [`data_preprocessing.ipynb`](All_IoT_Devices_Combined/data_preparation/data_preprocessing.ipynb) cleans each device dataset (drops NaNs, groups duplicates by mode) and [`data_merging.ipynb`](All_IoT_Devices_Combined/data_preparation/data_merging.ipynb) merges all devices.
- **Run logging** — `stream.log` (streaming events) and `ensemble.log` (training events) capture each run.

<br>

## 🚀 Getting Started

### 1. Install dependencies

```bash
pip install pandas numpy scikit-learn tensorflow websockets plotly matplotlib jupyter
```

### 2. Prepare data & train the base models

- Place your IoT CSV in the `data/` folder (a `Test_*` CSV is a trimmed copy for quick streaming demos).
- Encode string columns to numeric with `LabelEncoder` in each `models/notebooks/*` notebook and in [`client.ipynb`](IoT_Modbus/client/client.ipynb).
- For LR & SVM, set the `iloc` upper limit to the row where the second class label first appears.
- Run every notebook in `models/notebooks/` to train and pickle the individual models into `models/h5s/`.

### 3. Run the stream

- Run  server/server.ipynb   →  starts streaming on ws://localhost:8765
- Run  client/client.ipynb   →  connects, chunks, and trains incrementally (or [`client/client_multithread.ipynb`](IoT_Modbus/client/client_multithread.ipynb))

Watch `client/stream.log` & `client/ensemble.log`, then open `client/Graphs.ipynb` to visualize per-chunk metrics.

> For the combined experiment, first run [`data_preprocessing.ipynb`](All_IoT_Devices_Combined/data_preparation/data_preprocessing.ipynb) → [`data_merging.ipynb`](All_IoT_Devices_Combined/data_preparation/data_merging.ipynb)

<br>

## 🎓 What I Learned

- **Streaming architecture** — building a producer/consumer pipeline over WebSockets with `asyncio`, including back-pressure via per-row acknowledgements.
- **Online vs. batch learning** — keeping a model improving as new chunks arrive instead of retraining from scratch.
- **Streaming-data pitfalls** — why fixed chunk sizes break (a chunk may contain only one class) and how adaptive sizing fixes it.
- **Ensemble methods** — combining diverse classifiers via voting, and the gap between classical ML and deep models (LSTM/GRU) on tabular IoT data.
- **Concurrency** — moving from a single `asyncio` loop to a multithreaded client so training doesn't block the stream.
- **Real-world data wrangling** — cleaning, deduplicating, encoding, and merging heterogeneous datasets with inconsistent schemas.

<br>

## 🔭 Future Improvements

- **Production streaming stack** — swap the hand-rolled server for Apache Kafka / Spark Streaming (fault tolerance, partitioning, horizontal scaling).
- **True incremental learners** — adopt `partial_fit` models (`SGDClassifier`, `river`) and add concept-drift detection.
- **Productionize** — extract logic from notebooks into reusable modules with a CLI, config files, and unit tests.
- **Stronger evaluation** — handle class imbalance with stratified sampling, weighted metrics, and confusion matrices.

<br>

---

<div align="center">

⭐ _If you found this project helpful or interesting, consider giving it a star!_ ⭐

</div>
