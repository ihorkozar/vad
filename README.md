# VAD
VAD is a Flutter library for Voice Activity Detection (VAD) across **iOS** , **Android** , and **Web**  platforms. This package allows applications to start and stop VAD-based listening and handle various VAD events seamlessly.
Under the hood, the VAD Package uses `dart:js_interop` for Web to run [VAD JavaScript library](https://github.com/ricky0123/vad) and [onnxruntime](https://github.com/gtbluesky/onnxruntime_flutter) for iOS and Android utilizing onnxruntime library with full-feature parity with the JavaScript library.
The package provides a simple API to start and stop VAD listening, configure VAD parameters, and handle VAD events such as speech start, speech end, errors, and misfires.

## Table of Contents
- [Live Demo](#live-demo)
- [Features](#features)

- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
    - [Web](#web)

    - [iOS](#ios)

    - [Android](#android)

- [Installation](#installation)

- [Usage](#usage)
  - [Example](#example)
    - [Explanation of the Example](#explanation-of-the-example)

- [VADHandler API](#vadhandler-api)
  - [Methods](#methods)
    - [`create`](#--create)

    - [`startListening`](#--startlistening)

    - [`stopListening`](#--stoplistening)

    - [`dispose`](#--dispose)

  - [Event Streams](#event-streams)
    - [`onSpeechEnd`](#--onspeechend)

    - [`onSpeechStart`](#--onspeechstart)

    - [`onVADMisfire`](#--onvadmisfire)

    - [`onError`](#--onerror)

- [Permissions](#permissions)
  - [iOS](#ios-1)

  - [Android](#android-1)

  - [Web](#web-1)

- [Cleaning Up](#cleaning-up)

- [Tested Platforms](#tested-platforms)

- [License](#license)

## Live Demo
Check out the [VAD Package Example App](https://keyur2maru.github.io/vad/) to see the VAD Package in action on the Web platform.

## Features

- **Cross-Platform Support:**  Works seamlessly on iOS, Android, and Web.

- **Event Streams:**  Listen to events such as speech start, speech end, errors, and misfires.

- **Configurable Parameters:**  Customize VAD behavior with various parameters.

- **Easy Integration:**  Simple setup and usage within Flutter applications.

## Getting Started

### Prerequisites

Before integrating the VAD Package into your Flutter application, ensure that you have the necessary configurations for each target platform.

#### Web
To use VAD on the web, include the following scripts in your `web/index.html` file to load the necessary VAD libraries:

```html
<!-- VAD Dependencies -->
<script src="https://cdn.jsdelivr.net/npm/onnxruntime-web/dist/ort.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@ricky0123/vad-web@0.0.19/dist/bundle.min.js"></script>
<script src="https://cdn.jsdelivr.net/gh/keyur2maru/vad_dart@latest/dist/vad_web.js"></script>
```

#### iOS
For iOS, you need to configure microphone permissions and other settings in your `Info.plist` file.
1. **Add Microphone Usage Description:** Open `ios/Runner/Info.plist` and add the following entries to request microphone access:

```xml
<key>NSMicrophoneUsageDescription</key>
<string>This app needs access to the microphone for Voice Activity Detection.</string>
```

2. **Configure Build Settings:** Ensure that your `Podfile` includes the necessary build settings for microphone permissions:

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
    target.build_configurations.each do |config|
      config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= [
        '$(inherited)',
        'PERMISSION_MICROPHONE=1',
      ]
    end
  end
end
```

#### Android
For Android, configure the required permissions and build settings in your `AndroidManifest.xml` and `build.gradle` files.
1. **Add Permissions:** Open `android/app/src/main/AndroidManifest.xml` and add the following permissions:

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
```

2. **Configure Build Settings:** Open `android/app/build.gradle` and add the following settings:
```gradle
android {
    compileSdkVersion 34
    ...
}
```


## Installation
Add the VAD Package to your `pubspec.yaml` dependencies:
Note: `audioplayers` is used for audio playback in the example below. You can replace it with any other audio player package of your choice.

```yaml
dependencies:
  flutter:
    sdk: flutter
  vad: ^0.0.4
  permission_handler: ^11.3.1
  audioplayers: ^6.1.0
```
Then, run `flutter pub get` to fetch the packages.
## Usage

### Example

Below is a simple example demonstrating how to integrate and use the VAD Package in a Flutter application.
For a more detailed example, check out the [VAD Example App](https://github.com/keyur2maru/vad/tree/master/example) in the GitHub repository.


```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:vad/vad.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text("VAD Example")),
        body: const MyHomePage(),
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final _vadHandler = VadHandler.create(isDebug: true);
  bool isListening = false;
  final List<String> receivedEvents = [];

  @override
  void initState() {
    super.initState();
    _setupVadHandler();
  }

  void _setupVadHandler() {
    _vadHandler.onSpeechStart.listen((_) {
      debugPrint('Speech detected.');
      setState(() {
        receivedEvents.add('Speech detected.');
      });
    });

    _vadHandler.onSpeechEnd.listen((List<double> samples) {
      debugPrint('Speech ended, first 10 samples: ${samples.take(10).toList()}');
      setState(() {
        receivedEvents.add('Speech ended, first 10 samples: ${samples.take(10).toList()}');
      });
    });

    _vadHandler.onVADMisfire.listen((_) {
      debugPrint('VAD misfire detected.');
      setState(() {
        receivedEvents.add('VAD misfire detected.');
      });
    });

    _vadHandler.onError.listen((String message) {
      debugPrint('Error: $message');
      setState(() {
        receivedEvents.add('Error: $message');
      });
    });
  }

  @override
  void dispose() {
    _vadHandler.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          ElevatedButton.icon(
            onPressed: () async {
              setState(() {
                if (isListening) {
                  _vadHandler.stopListening();
                } else {
                  _vadHandler.startListening();
                }
                isListening = !isListening;
              });
            },
            icon: Icon(isListening ? Icons.stop : Icons.mic),
            label: Text(isListening ? "Stop Listening" : "Start Listening"),
            style: ElevatedButton.styleFrom(
              minimumSize: const Size(double.infinity, 48),
            ),
          ),
          const SizedBox(height: 8),
          TextButton.icon(
            onPressed: () async {
              final status = await Permission.microphone.request();
              debugPrint("Microphone permission status: $status");
            },
            icon: const Icon(Icons.settings_voice),
            label: const Text("Request Microphone Permission"),
            style: TextButton.styleFrom(
              minimumSize: const Size(double.infinity, 48),
            ),
          ),
          const SizedBox(height: 16),
          Expanded(
            child: ListView.builder(
              itemCount: receivedEvents.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(receivedEvents[index]),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```
#### Explanation of the Example
1. **Initialization:**
- Initializes the `VadHandler` with debugging enabled.

- Sets up listeners for various VAD events (`onSpeechStart`, `onSpeechEnd`, `onError`, `onVADMisfire`).

2. **Permissions:**
- Requests microphone permission when the "Request Microphone Permission" button is pressed.

3. **Listening Controls:**
- Toggles listening on and off with the "Start Listening"/"Stop Listening" button.

- Configures the audio player to mix with other audio sources on iOS.

4. **Event Handling:**
- Displays received events in a list view.

- Updates the UI based on the received events.

## VadHandler API

### Methods

#### - **`create`**
  Creates a new instance of the `VadHandler` with optional debugging enabled with the `isDebug` parameter and optional configurable model path with the `modelPath` parameter but it's only applicable for the iOS and Android platforms. It has no effect on the Web platform.

#### - **`startListening`**
  Starts the VAD with configurable parameters.
  Note: The sample rate is fixed at 16kHz, which means one frame is equal to 1536 samples or 96ms.

```dart
void startListening({
  double positiveSpeechThreshold = 0.5,
  double negativeSpeechThreshold = 0.35,
  int preSpeechPadFrames = 1,
  int redemptionFrames = 8,
  int frameSamples = 1536,
  int minSpeechFrames = 3,
  bool submitUserSpeechOnPause = false,
});
```

#### - **`stopListening`**
  Stops the VAD session.


```dart
void stopListening();
```

#### - **`dispose`**
  Disposes the VADHandler and closes all streams.


```dart
void dispose();
```

### Event Streams

#### - **`onSpeechEnd`**
  Triggered when speech ends with optional audio data.


```dart
Stream<List<double>> get onSpeechEnd;
```

#### - **`onSpeechStart`**
  Triggered when speech starts.


```dart
Stream<void> get onSpeechStart;
```

#### - **`onVADMisfire`**
  Triggered when the VAD misfires.


```dart
Stream<void> get onVADMisfire;
```

#### - **`onError`**
  Triggered when an error occurs.


```dart
Stream<String> get onError;
```

## Permissions

Proper handling of microphone permissions is crucial for the VAD Package to function correctly on all platforms.

### iOS

- **Configuration:** Ensure that `NSMicrophoneUsageDescription` is added to your `Info.plist` with a descriptive message explaining why the app requires microphone access.

- **Runtime Permission:** Request microphone permission at runtime using the `permission_handler` package.

### Android

- **Configuration:** Add the `RECORD_AUDIO`, `MODIFY_AUDIO_SETTINGS`, and `INTERNET` permissions to your `AndroidManifest.xml`.

- **Runtime Permission:** Request microphone permission at runtime using the `permission_handler` package.

### Web

- **Browser Permissions:**
  Microphone access is managed by the browser. Users will be prompted to grant microphone access when the VAD starts listening.

## Cleaning Up 
To prevent memory leaks and ensure that all resources are properly released, always call the `dispose` method on the `VadHandler` instance when it's no longer needed.

```dart
vadHandler.dispose();
```

## Tested Platforms
The VAD Package has been tested on the following platforms:

- **iOS:**  Tested on iPhone 15 Pro Max running iOS 18.1.
- **Android:**  Tested on Lenovo Tab M8 Running Android 10.
- **Web:**  Tested on Chrome Mac/Windows/Android/iOS, Safari Mac/iOS.

## Contributing
Contributions are welcome! Please feel free to submit a pull request or open an issue if you encounter any problems or have suggestions for improvements.

## Acknowledgements
Special thanks to [Ricky0123](https://github.com/ricky0123) for creating the [VAD JavaScript library](https://github.com/ricky0123/vad), [gtbluesky](https://github.com/gtbluesky) for building the [onnxruntime package](https://github.com/gtbluesky/onnxruntime_flutter) and Silero Team for the [VAD model](https://github.com/snakers4/silero-vad) used in the library.


## License
This project is licensed under the [MIT License](https://opensource.org/license/mit) . See the [LICENSE](https://github.com/keyur2maru/vad/blob/master/LICENSE)  file for details.

---

For any issues or contributions, please visit the [GitHub repository](https://github.com/keyur2maru/vad) .