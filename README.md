# Guitar Learner - Real-Time Guitar Chord Recognition & Practice Tool

A computer vision and audio project that helps beginners practice guitar chords using their webcam and microphone.

---

## What is this project?

When you're trying to learn guitar by yourself, it's difficult to know if your fingers are on the right frets or if your chord sounds clean. It's also annoying to constantly stop playing to click around on a computer or phone because both of your hands are busy holding the guitar.

We built this project to solve that. It gives you instant visual and audio feedback while you play:
1. **Camera (Computer Vision):** Watches your fretting hand and checks your finger positions.
2. **Microphone (Audio):** Listens to your strum and matches the frequencies against real guitar recordings from Kaggle.
3. **Song & Progression Matcher:** Tracks the chords you play in real time, tells you which famous songs use that progression, and suggests what chord to play next.

---

## Supported Chords

The system currently recognizes 6 chords:
* **C Major** (Open chord)
* **D Major** (Open triangle shape)
* **E Major** (Open full chord)
* **F Major** (1st fret barre with middle finger on the 2nd fret)
* **G minor** (3rd fret barre)
* **Gm Barre** (Full 6-string 3rd fret barre)

---

## How It Works

### 1. Hand Detection (Camera)
If you train a neural network directly on raw camera photos, it easily gets confused by different skin tones, shadows, or messy room backgrounds.

To fix this, we use Google's **MediaPipe** library. It detects 21 landmark points on your hand (joints, fingertips, wrist). We then draw these points as pure white skeleton lines on a black canvas and crop tightly around the hand. That clean black-and-white skeleton ($200 \times 200$) is what our CNN looks at. Because of this, it works consistently no matter what your room lighting or skin tone is.

### 2. Strum & Sound Matching (Microphone)
When you strum your guitar, the microphone captures the sound. Using Fast Fourier Transform (FFT) and Chromagram feature extraction, the program breaks down the sound into the 12 musical notes ($C, C\#, D, \dots, B$).

We downloaded 48 real Fender acoustic and electric guitar chord recordings from Kaggle (`audio_dataset/`). When you play, the mic extracts the frequencies and finds the closest matching chord from the Kaggle dataset. There's also a live visual equalizer on the screen with bouncing bars and a decibel meter so you can see it actively listening.

### 3. Song Progressions & Next-Chord Prediction
We have a dataset of 20+ songs (like The Beatles, Bob Dylan, Pink Floyd, Adele) stored in `kaggle_guitar_chords_dataset.json`. 

As you play chords, the program remembers the sequence (for example: `C Major -> G minor -> F Major`). It matches your sequence against the songs and shows you what you're playing (e.g. *100% match with "Let It Be"*). It also uses a bi-gram transition model to suggest what chord usually sounds best next.

### 4. Combining Camera + Sound
If your fingers are partly hidden, the audio helps confirm the chord. If your room is a bit noisy, the camera confirms it. When both the camera and sound agree, the program locks in with high confidence (up to 99%).

---

## Accuracy & Model Details

* **Dataset:** 450 balanced hand skeleton images (exactly 75 images per chord).
* **Model:** A custom 4-layer Convolutional Neural Network (CNN) built in Keras/TensorFlow.
* **Validation Accuracy:** **98.89%** (1.11% error rate).
* **Overall Dataset Accuracy:** **97.11%** (440 out of 450 images classified correctly).
* **Model Size:** 40.7 MB (`guitar_learner.h5`), making it lightweight and fast enough to run at 30 FPS on normal laptop CPUs without needing an expensive GPU.

---

## Requirements & Setup

### Requirements
* Python 3.11
* A regular laptop webcam and microphone

### Installation
Open your terminal in the project folder and run:

```bash
pip install tensorflow==2.15.0 keras==2.15.0 opencv-python mediapipe pyaudio librosa soundfile sounddevice pywin32 SpeechRecognition requests huggingface_hub
```

---

## How to Run

### Main Application (Camera + Sound + Song Matching)
This is the main script to run:
```bash
python GuitarMultimodalKaggleAudioCV.py
```

**Controls while running:**
* **Keys `1` to `5`:** If you don't have a guitar with you right now, press keys 1, 2, 3, 4, or 5 to play real Fender guitar audio samples through your laptop speakers to test the sound detection live.
* **Key `c`:** Clears your played chord sequence history.
* **Key `q`:** Quits the program.

### Other Useful Scripts
* **Camera only (Visual baseline):**
  ```bash
  python GuitarLearner.py
  ```
* **Song matcher focus:**
  ```bash
  python GuitarSongMatcher.py
  ```
* **Retrain the model:**
  ```bash
  python Trainer.py
  ```

---

## Project Structure

```text
├── GuitarMultimodalKaggleAudioCV.py  # Main program (Camera + Kaggle Sound + Song matcher + Live EQ)
├── GuitarSongMatcher.py              # Song progression matcher & N-gram chord predictor
├── GuitarLearner.py                  # Original camera-only chord detection
├── GuitarCoachVoice.py               # Voice-controlled practice coach
├── Trainer.py                        # Script used to train the CNN model
├── guitar_learner.h5                 # Trained model weights (40.7 MB, 97.11% accuracy)
│
├── audio_dataset/                    # Downloaded Kaggle guitar audio recordings (WAV format)
│   ├── C/                            # Real Fender C chord audio files
│   ├── D/                            # Real Fender D chord audio files
│   ├── E/                            # Real Fender E chord audio files
│   ├── F/                            # Real Fender F chord audio files
│   └── Gm/                           # Real Fender Gm chord audio files
│
├── chords/                           # 450 hand skeleton training images (75 per chord)
│   ├── 0/ to 5/                      # Subfolders for chords 0 to 5
│
├── kaggle_audio_features.npz         # Extracted audio features from the Kaggle WAV files
├── kaggle_guitar_chords_dataset.json # 20+ famous songs with chords and lyrics
└── README.md                         # This file
```
