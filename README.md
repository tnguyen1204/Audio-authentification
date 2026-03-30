# 🎵 Audio Identification

> A Python learning project on audio signal processing — implementation of an audio fingerprinting algorithm from scratch.

⚠️ **Work In Progress**

---

## 📖 Description

This project implements step by step an audio fingerprinting algorithm capable of identifying a song from a short excerpt. The goal is educational: understand how an audio signal can be transformed into a unique fingerprint, robust to noise, and comparable against a database.

---

## ⚙️ Installation

### Prerequisites
- Python 3.8+
- [Anaconda](https://www.anaconda.com/) recommended

### Install dependencies

```bash
pip install librosa numpy matplotlib scipy seaborn
```

Or from Anaconda Prompt:

```bash
conda install librosa numpy matplotlib scipy seaborn
```

---

## 🧠 Algorithm

### 1. Audio loading
The MP3 file is loaded with `librosa` at 44,100 Hz to capture all audible frequencies (0–22,050 Hz).

### 2. Spectrogram
A Short-Time Fourier Transform (STFT) is computed to get a time/frequency representation of the signal, converted to decibels.

### 3. Peak detection (constellation map)
Local maxima are detected in the spectrogram — the most energetic points — spread across 4 frequency bands:
- 0 – 500 Hz
- 500 – 2,000 Hz
- 2,000 – 5,000 Hz
- 5,000 – 11,000 Hz

### 4. Pairing
Each anchor peak is associated with its N nearest neighbors in time, forming pairs. The time delta between the two peaks is kept.

### 5. Hashing
Each pair is converted into an MD5 hash from:
```
key = (f1, f2, delta_t)
hash = MD5(key)
```
The absolute time `t1` of the anchor peak is stored alongside the hash for alignment during recognition.

---

## 💡 Usage examples

### Load an audio file
```python
import librosa
y, sr = librosa.load("song.mp3", sr=44100)
```

### Generate the spectrogram
```python
D = librosa.stft(y)
S_db = librosa.amplitude_to_db(np.abs(D), ref=np.max)
```

### Detect peaks
```python
peaks = get_peaks(S_db, neighborhood_size=20)
```

### Generate pairs and hashes
```python
pairs = get_pairs(peaks, n_neighbors=3)
hashes = get_hashes(peaks, pairs)
```

---

## 📐 Technical notes

### Pairing — choosing N neighbors

The `n_neighbors` parameter controls how many neighbors each anchor peak is paired with.

- **High N** → more pairs → denser fingerprint → **more precise** matching
- **Low N** → fewer pairs → lighter fingerprint → **more resilient to noise**

This is a tradeoff: in a noisy environment (e.g. a crowded café), some peaks will be missed. A lower N means each pair is more likely to be fully detected. A higher N gives more redundancy in a clean environment but risks partial matches in noisy conditions.

A value of **N = 3 to 5** is generally a good balance.

---

### Sample rate & the 512 division

When working with the STFT output matrix, the columns are not in seconds — they are **frame indices**. Each frame corresponds to a hop of **512 samples** (the default `hop_length` in librosa).

To convert a frame index to seconds:

```
time (seconds) = frame_index × hop_length / sample_rate
               = frame_index × 512 / 44100
```

Similarly, frequency bins (rows) are converted to Hz:

```
frequency (Hz) = bin_index × sample_rate / n_fft
               = bin_index × 44100 / 2048
```

These conversions are essential to correctly overlay peaks on the spectrogram, which is displayed in seconds and Hz.

---

### FFT vs STFT — what's the difference?

A standard **FFT** (Fast Fourier Transform) transforms the entire signal at once into the frequency domain. You get a single spectrum showing which frequencies are present — but **no information about when** they occur.

A **STFT** (Short-Time Fourier Transform) splits the signal into small overlapping windows and applies an FFT to each one. The result is a 2D matrix: **time × frequency**, which is what we display as a spectrogram.

In this project we use the STFT because we need to know **both** when and at what frequency peaks occur — that temporal information is what makes the fingerprint unique and matchable.

---

## 🗺️ Roadmap

- [x] Audio loading
- [x] Spectrogram
- [x] Peak detection per frequency band
- [x] Peak pairing
- [x] MD5 hashing
- [ ] Hash storage in database
- [ ] Recognition of an unknown excerpt
- [ ] User interface

---

## 🛠️ Tech stack

- `librosa` — audio processing
- `numpy` — matrix computation
- `matplotlib` — visualization
- `scipy` — local maxima detection
- `hashlib` — MD5 hashing (Python built-in)
