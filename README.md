# Musico: Audio Visualizer

![Project Cover](res/cover.png)

> A real-time audio visualizer in C++ using SDL2 and FFTW3.

**Demo:** https://youtu.be/W0MV3Ehnk0A

## Features

- **Three modes:** play a WAV file (optionally downloaded from YouTube), record from the microphone and play it back, or visualize the microphone live.
- **Two views:** FFT spectrum bars or a time-domain waveform. Press `m` to switch.
- **Controls:** pause (freezes the current frame), stop and replay, in a resizable window.

## How it works

Each audio buffer SDL delivers (2048 samples at 44.1 kHz) is turned into one frame by [`visualizerOutput()`](src/visualize.cpp):

1. **Decode** the 16-bit PCM samples to doubles in $[-1, 1)$.
2. **Window** them with a Hann window, $w[n] = \sin^2(\pi n / N)$, to reduce spectral leakage.
3. **FFT** with FFTW: a 2048-point forward DFT, using a plan created once at startup and reused for every buffer.
4. **Magnitude** of bins $0 \dots N/2-1$, where bin $k$ is $k \cdot 44100/2048 \approx 21.5k$ Hz.
5. **Bars:** each bar takes the peak magnitude in its ~22 Hz band, log-scaled above 2. Colors come from an HSV rainbow that shifts every frame.

The waveform view skips steps 3–5 and draws the windowed samples as a line.

## Building and running

**Linux (Debian/Ubuntu):** `install.sh` installs SDL2, SDL2_image, SDL2_ttf, FFTW3, ffmpeg and youtube-dl.

```bash
sudo sh install.sh
make
./musico
```

**Windows (64-bit):** install [MinGW-w64 GCC](https://winlibs.com/) with the posix thread model (`choco install winlibs-llvm-free`), run `install.ps1` in PowerShell, then run `make`. Start the program from inside `bin64`:

```powershell
cd bin64
.\musico.exe
```

## Usage

```bash
./musico                 # start menu
./musico -f song.wav     # play a WAV file
./musico -r              # record, then play back
./musico -R              # real-time microphone input
```

| Key | Action |
|---|---|
| `Space` | start recording / pause / resume / replay |
| `q` | stop recording, or return to the menu |
| `m` | switch between bars and waveform |

## Known issues

- WAV playback assumes 16-bit stereo at 44.1 kHz, so other formats play at the wrong speed.
- Window resizing doesn't work on Windows ([libsdl-org/SDL#1059](https://github.com/libsdl-org/SDL/issues/1059)).
- Leaving the YouTube URL dialog open for a long time makes the program stop responding, because the dialog blocks the main thread.
