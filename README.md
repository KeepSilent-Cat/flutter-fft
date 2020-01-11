# Flutter Fft

**Warning:** *Currently works only on Android! This plugin makes use of platform channels, and only the Java/Android platform channel has been implemented.*

**The plugin was developed and tested in a Pixel 2 emulator, API 29. Does not work on iOS at the moment due to the platform channel having yet to be implemented.**

**Minimum SDK version >= 24**: You can set update the minimum SDK requirements at "/android/app/build.gradle" in the line `minSdkVersion 16`.

The following needs to be added to your project's `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.INTERNET" />
```

My first (and currently only) Flutter plugin and Java project.

I was making my personal guitar tuner application on Flutter, when I realized that I couldn't find any examples of audio analysis/processing/manipulation with Flutter.

Flutter doesn't have great support for device specific hardware, such as microphone input, which obviously is the fundamental pillar for anything that deals with audio processing in real-time. 
Since Flutter has "just" started to become mainstream, there are still not many real-world projects or examples around.

Regardless, the plan I ended up coming up with was to code a platform channel for android, which is basically a way to call native code from within Flutter - i.e. Calling Java functions on Flutter.

The problem with this is that, as it calls native platform code, the "one codebase" Flutter feature is rendered useless, since I would have to code the same thing for both platforms. (Objective-C/Swift for iOS & Java/Kotlin for Android)

For this reason, at the moment I have only coded the android platform channel. An iOS platform channel is in the plans for future versions.

## How to use

As mentioned above, this plugin was purely intended for usage in my personal project, however, since I couldn't find similar implementations, I decided to upload it here, in case anyone else goes through the same.

Because of this, what you can do with the plugin is very strict: start recording and get data back from the platform channel and stop recording.

If you know how to program however, you can easily modify the code for your own needs.

There are many getters and setters for the processed data, which are discussed below.

Simple example:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_fft/flutter_fft.dart';

void main() => runApp(Application());

class Application extends StatefulWidget {
  @override
  ApplicationState createState() => ApplicationState();
}

class ApplicationState extends State<Application> {
  double frequency;
  String note;
  bool isRecording;

  FlutterFft flutterFft = new FlutterFft();

  @override
  void initState() {
    isRecording = flutterFft.getIsRecording;
    frequency = flutterFft.getFrequency;
    note = flutterFft.getNote;
    super.initState();
    _async();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: "Simple flutter fft example",
      theme: ThemeData.dark(),
      color: Colors.blue,
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              isRecording
                  ? Text(
                      "Current frequency: $note",
                      style: TextStyle(
                        fontSize: 35,
                      ),
                    )
                  : Text(
                      "None yet",
                      style: TextStyle(
                        fontSize: 35,
                      ),
                    ),
              isRecording
                  ? Text(
                      "Current frequency: ${frequency.toStringAsFixed(2)}",
                      style: TextStyle(
                        fontSize: 35,
                      ),
                    )
                  : Text(
                      "None yet",
                      style: TextStyle(
                        fontSize: 35,
                      ),
                    ),
            ],
          ),
        ),
      ),
    );
  }

  _async() async {
    print("starting...");
    await flutterFft.startRecorder();
    setState(() => isRecording = flutterFft.getIsRecording);
    flutterFft.onRecorderStateChanged.listen(
      (data) => {
        setState(
          () => {
            frequency = data[1],
            note = data[2],
          },
        ),
        flutterFft.setNote = note,
        flutterFft.setFrequency = frequency,
      },
    );
  }
}
```

## Methods

When we instantiate the plugin, a method channel variable is created, with the tag "com.slins.flutterfft/record".

This is the variable that is used to estabilish a connection between Dart and the platform channel.

**For the section below, it is assumed that the plugin was instantiated and stored to a variable called "flutterFft".**

### Three main methods

1. `flutterFft.onRecorderStateChanged`
   - Stream that listens to the recording's state.  
2. `flutterFft.startRecording()` 
   - Starts recording using the data from the plugin's **local** instance. In other words,  if you want to pass custom values, other than the default ones, you have to set it, i.e. `flutterFft.setSampleRate = 22050`, then start the recorder.  
3. `flutterFft.stopRecording()`
   - Stops recording.

### Default variables, with getters and setters

* `_isRecording = false` controller for the recorder state.
      - `flutterFft.getIsRecording`
      - `flutterFft.setIsRecording = BOOL`  

* `_subscriptionDuration = 0.25` controller for the interval between platform channel function calls.
      - `flutterFft.getSubscriptionDuration`
      - `flutterFft.setSubscriptionDuration = DOUBLE`  

3. `_numChannels = 1` controller for the number of channels that gets passed to the pitch detector.
      - `flutterFft.getNumChannels`
      - `flutterFft.setNumChannels = INT`

4. `_sampleRate = 44100` controller for the sample rate that gets passed to the pitch detector.
      - `flutterFft.getSampleRate`
      - `flutterFft.setSampleRate = INT`  

5. `_androidAudioSource` controller for the audio source. (Microphone, etc.)
      - `flutterFft.getAndroidAudioSource`
      - `flutterFft.setAndroidAudioSource = AndroidAudioSource`  

6. `_tolerance` controller for the tolerance. (How far apart can the current frequency from the target frequency in order to be considered on pitch)
      - `flutterFft.getTolerance`
      - `flutterFft.setTolerance = DOUBLE`  

7. `_frequency` controller for the frequency.
      - `flutterFft.getFrequency`
      - `flutterFft.setFrequency = DOUBLE`  

8. `_note` controller for the detected note.
      - `flutterFft.getNote`
      - `flutterFft.setNote = STRING`  

9. `_target` controller for the target frequency. (Based on the current selected tuning, calculate the nearest frequency in tune to be considered as the target, i.e: `IF currentNote == A && A.frequency.distanceToB IS SmallestTargetDistance -> _target = A.frequency.distanceToB`
      - `flutterFft.getTarget`
      - `flutterFft.setTarget = DOUBLE`  

10. `_distance` controller for the distance between the current frequency and the target frequency.
      - `flutterFft.getDistance`
      - `flutterFft.setDistance = DOUBLE` 

11. `_octave` controller for the detected octave.
      - `flutterFft.getOctave`
      - `flutterFft.setOctave = INT`  

12. `_nearestNote` controller for nearest note. (Based on the current note)
      - `flutterFft.getNearestNote`
      - `flutterFft.setNearestNote = STRING` 

13. `_nearestTarget` controller for nearest target. (Second smallest distance, as the smallest distance is already `_target`)
      - `flutterFft.getNearestTarget`
      - `flutterFft.setNearestTarget = DOUBLE` 

14. `_nearestDistance` controller for nearest distance. (Second smallest distance)
       -  `flutterFft.getNearestDistance`
       -  `flutterFft.setNearestDistance = DOUBLE`  

15. `_nearestOctave` controller for nearest octave. (Based on the "nearest" data)
       -  `flutterFft.getNearestOctave`
       -  `flutterFft.setNearestOctave = INT`  

16. `_isOnPitch` controller for the pitch
       -  `flutterFft.getIsOnPitch`
       -  `flutterFft.setIsOnPitch = BOOL`  

17. `_tuning` controller for the tuning target. Format: `["E4", "B3", "G3", "D3", "A2", "E2"]` (The detected frequency is compared to these values in order to gather the above data)
       -  `flutterFft.getTuning`
       -  `flutterFft.setTuning = List<String>`
