# ema-tools
NDI Vox-EMA data collection and analysis support tools.

## Dependencies
Dependencies may be installed by running:
```
pip install numpy moviepy
```

## ema-stim
[ema-stim.py](ema-stim.py) augments a video clip with a fixation image and two blanking intervals. The output video clip contains the sequence, image, blanking, video, blanking. Additionally, a linear timecode track is embedded in the right audio channel of the output video. By recording this track, along with the Vox-EMA timecode signal and the participant speech signal, the relative timings of the stimulus presentation, kinematic response, and speech signal may be precisely captured.

The tool will report usage help if invoked with the `--help` option. By way of example, to augment the video clip `source.mov` with blanking interval 0.1 s, fixation period 0.5 s, and a timecode track containing the identifier `R2D2`, to the output file `stimulus.mp4`, run:
```
python ema-stim.py source.mov stimulus.mp4 --blanking 0.1 --characters R2D2 --fixation 0.5 --image stim/fixation-720.png
```
Note that the fixation image dimensions must match that of the source video.

The linear timecode track is encoded as specified in SMPTE ST 12-1:2014. The time address 00:00:00.00 is coincident with the first frame of the video segment and increments at frame rate. Binary group data is constant for each frame and contains the four-character identifier passed to the tool with `--characters`.

## ema-time
[ema-time.py](ema-time.py) decodes two linear timecode tracks from a stereo WAV file and reports their relative timing. Stimulus timecode must be in the left channel and EMA timecode in the right. The stimulus identifier is reported for each presentation detected in the recording, along with the onset and offset time addresses, sample indices, and corresponding EMA frame numbers.

The tool will report usage help if invoked with the `--help` option. By way of example, to analyse the timecode tracks recorded to `timecode.wav`, where the stimulus video had a frame rate of 30 fps, run:
```
python ema-time.py timecode.wav --framerate 30
```
Note that the Vox-EMA frame rate is ostensibly 800 Hz and its timecode rate 25 Hz. The EMA frame count therefore increments by 32 for each timecode frame. EMA frame numbers reported by the tool are interpolated to the closest even frame corresponding to a given stimulus time address. This provides timing resolution equivalent to one sample period at the maximum Vox-EMA sampling rate of 400 Hz.

## ema-trans
[ema-trans.py](ema-trans.py) performs rigid body registration and, optionally, bite plane transformation on Vox-EMA kinematic data.

The tool will report usage help if invoked with the `--help` option. By way of example, to perform rigid body registration and bite plane transformation on the kinematic data contained in `experiment.csv`, using bite plane recording `biteplane.csv`, run:
```
python ema-trans.py experiment.csv --bitefile biteplane.csv
```
Omit the `--bitefile` option and filename to perform rigid body registration only.

The tool exclusively makes use of reference sensor data with status `OK`. Non-reference sensor data are not filtered, so will be erroneous at output if erroneous at input. Status fields of the output file contain 1.0 if the corresponding sample is valid, 0.0 otherwise.

Processed kinematic data are written to `[inputname]-t.csv`. This file discards quaternion data and duplicate columns present in the input file. Its format is obvious from inspection.

## ema-trig
[ema-trig.py](ema-trig.py) generates a video file from a source image, the onset of which is coincident with a tone burst embedded in the left audio channel, while a timecode track is embedded in the right.

The tool will report usage help if invoked with the `--help` option. By way of example, to generate a video named `trigger.mp4` from the image `ema-trig.png`, run:
```
python ema-trig.py --ipfile stim/ema-trig.png --opfile trigger.mp4
```
The output file may be used in conjunction with a photodetector and an oscilloscope to measure audio/video synchronisation latency and variation of a playback system.
