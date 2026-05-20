# Connectivity Recovery in Teacher-Student RNNs

This project investigates whether recurrent connectivity can be recovered in recurrent neural networks (RNNs).

The project compares:

- RNNs **without biological constraints**
- RNNs **with biological constraints (Dale’s law)**

and evaluates how different student learning objectives affect the recovery of recurrent connectivity and latent dynamics.

---

## 1. Experiment Setup

### NeuroGym task

A 2-choice perceptual decision-making task is generated using NeuroGym.

### Task structure

```text
Fixation → Stimulus → Delay (0 ms) → Decision (100 ms)
```

### Input / Output representation

Observation (input to RNN hidden layer):

| Index | Meaning |
|--------|---------|
| 0 | fixation |
| 1 | stimulus A |
| 2 | stimulus B |

Action (logits computed from hidden activity `r`) has size 3 with the same encoding.

---

## 2. RNN Architectures

Two teacher RNNs are implemented.

### (A) RNN without biological constraints

- No Dale’s law
- Diagonal of recurrent connectivity matrix `W_hh` is masked to zero to prevent self-connections
- Noise (`sigma`) = 0
- Hidden recurrent bias = `False`

### (B) RNN with biological constraints (Dale’s law)

- Dale’s law applied
- Excitatory neuron proportion = 0.8
- Noise (`sigma`) = 0
- Hidden recurrent bias = `False`

The objective is to evaluate whether recurrent connectivity matrix recovery is possible under different biological assumptions.

---

## 3. Teacher Network Training

Teacher RNNs are trained on the NeuroGym task.

To avoid learning fixed temporal structures:

- fixation duration is randomly sampled
- stimulus duration is randomly sampled

from 4 different values.

Training settings:

- sequence length = `100`
- batch size = `16`

> Sequence length is **not equivalent to trial length**.

---

## 4. Sanity Check

---

## 5. Dataset Generation from Teacher Network

A synthetic dataset is generated using the trained teacher RNN with known effective recurrent connectivity.

Settings:

- `500` trials
- fixation duration = `500 ms`
- stimulus duration = `500 ms`
- batch size = `1`

These durations are intentionally different from training to test generalization and resemble real experimental settings.

---

## 6. Student Networks

Four student RNNs are trained.

### Student A — Label supervision

Learns:

```text
Teacher ground-truth labels
```

---

### Student B — Activity supervision

Learns:

```text
Teacher hidden activity r(t)
```

---

### Student C — Probability supervision

Learns:

```text
Teacher final-step behavioral probability distribution
```

The probability distribution is computed using:

```text
softmax(logits, temperature=1)
```

---

### Student D — Frozen input layer

Learns:

```text
Teacher hidden activity r(t)
```

while using the teacher network’s:

- input → hidden weights
- input → hidden bias

Only recurrent connectivity is optimized.

---

## 7. Multi-run Evaluation

The entire process is repeated:

```text
10 times
```

for each condition:

- without Dale’s law
- with Dale’s law

to evaluate robustness and variability across runs.

---

## 8. Evaluation Metrics

The following metrics are computed:

### Hidden activity recovery

- Activity MSE (`r` MSE)

### Behavioral recovery

- Accuracy

### Connectivity recovery

- Signed correlation of recurrent matrix `W_hh`

### Latent dynamics recovery

PCA is performed by projecting student activities onto teacher PCs.

- top 3 PCs used
- explain ~90% of teacher variance

---

## Repository Structure

```text
Project_7/
│
├── figures/
├── final_results/
├── multi_run_results/
│
├── multi_run.ipynb
├── RNN_dale.ipynb
├── RNN_no_dale.ipynb
│
└── README.md
```

---

## Installation

### 1. Create environment

Using conda:

```bash
conda create -n rnn_recovery python=3.10
conda activate rnn_recovery
```

---

### 2. Install required packages

```bash
pip install neurogym
pip install torch
pip install numpy
pip install scipy
pip install pandas
pip install matplotlib
pip install scikit-learn
pip install jupyter
```

---

## How to Reproduce the Experiments

Simply run:

```text
multi_run.ipynb
```

This notebook automatically:

1. Runs:

```text
RNN_no_dale.ipynb
```

and

```text
RNN_dale.ipynb
```

2. Repeats experiments multiple times

3. Computes evaluation metrics:

- hidden activity MSE
- behavioral accuracy
- recurrent weight correlation
- PCA overlap

4. Generates figures and summary statistics

Results are saved in:

```text
final_results/
```

Figures are saved in:

```text
figures/
```