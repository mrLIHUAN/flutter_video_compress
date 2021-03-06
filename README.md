<!-- Copyright (c) 2019 Amami Ruri -->

<img align="right" src="https://raw.githubusercontent.com/TenkaiRuri/flutter_video_compress/master/doc/images/logo.svg?sanitize=true" height="180px" style="pointer-events: none;cursor: default;">

# flutter_video_compress

Generate a new path by compressed video, Choose to keep the source video or delete it by a parameter. Get video thumbnail from a video path and provide video information. Easy to deal with compressed video. Considering reduce application size is not using FFmpeg in IOS.

<p align="left">
  <a href="https://pub.dartlang.org/packages/flutter_video_compress"><img alt="pub version" src="https://img.shields.io/pub/v/flutter_video_compress.svg"></a>
  <img alt="license" src="https://img.shields.io/github/license/TenkaiRuri/flutter_video_compress.svg">
  <img alt="android min Sdk Version" src="https://img.shields.io/badge/android-16-success.svg?logo=android">
  <img alt="ios min target" src="https://img.shields.io/badge/ios-8-lightgrey.svg?logo=apple">
</p>

<div align="center">
  <img height="500px" alt="flutter compress video" style="pointer-events: none;cursor: default;" src="https://github.com/TenkaiRuri/flutter_video_compress/raw/master/doc/images/preview.gif"/>
</div>

## languages
[English](https://github.com/TenkaiRuri/flutter_video_compress#flutter_video_compress) [简体中文](https://github.com/TenkaiRuri/flutter_video_compress/blob/master/doc/chinese.md#flutter_video_compress) [日本語](https://github.com/TenkaiRuri/flutter_video_compress/blob/master/doc/japanese.md#flutter_video_compress) 

## Usage

**Installing**
add [flutter_video_compress](https://pub.dartlang.org/packages/flutter_video_compress) as a dependency in your pubspec.yaml file.
```yaml
dependencies:
  flutter_video_compress: ^0.3.x
```

**Create an instance**
```dart
final _flutterVideoCompress = FlutterVideoCompress();
```

**Get thumbnail from video path**
```dart
final uint8list = await _flutterVideoCompress.getThumbnail(
  file.path,
  quality: 50, // default(100)
  position: -1 // default(-1)
);
```

**Get thumbnail file from video path**
```dart
final thumbnailFile = await _flutterVideoCompress.getThumbnailWithFile(
  file.path,
  quality: 50, // default(100)
  position: -1 // default(-1)
);
```

**Convert video to a gif**
```dart
final file = await _flutterVideoCompress.convertVideoToGif(
  videoFile.path,
  startTime: 0, // default(0)
  duration: 5, // default(-1)
  // endTime: -1 // default(-1)
);
debugPrint(file.path);
```

**Get media information**
> only support video now.

```dart
final info = await _flutterVideoCompress.getMediaInfo(file.path);
debugPrint(info.toJson().toString());
```

**Compression Video**
> Compatible with IOS, Android and Web after compression.

```dart
final info = await _flutterVideoCompress.compressVideo(
  file.path,
  quality: VideoQuality.DefaultQuality, // default(VideoQuality.DefaultQuality)
  deleteOrigin: false, // default(false)
);
debugPrint(info.toJson().toString());
```

**Check Compressing state**
```dart
_flutterVideoCompress.isCompressing
```

**Stop compression**
> Will print InterruptedException in android, but not affect to use.

```dart
await _flutterVideoCompress.cancelCompression()
```

**delete all cache files**
> Delete all files generated by this will delete all files located at 'flutter_video_compress', you shoule ought to know what are you doing.

```dart
await _flutterVideoCompress.deleteAllCache()
```

**Subscribe the compression progress steam**
```dart
class ... extends State<MyApp> {
  Subscription _subscription;

  @override
  void initState() {
    super.initState();
    _subscription =
        _flutterVideoCompress.compressProgress$.subscribe((progress) {
      debugPrint('progress: $progress');
    });
  }

  @override
  void dispose() {
    super.dispose();
    _subscription.unsubscribe();
  }
}
```

## Methods
|Functions|Parameters|Description|Returns|
|--|--|--|--|
|getThumbnail|String `path`[video path], int `quality`(1-100)[thumbnail quality], int `position`[Get a thumbnail from video position]|get thumbnail from video `path`|`Future<Uint8List>`|
|getThumbnailWithFile|String `path`[video path], int `quality`(1-100)[thumbnail quality], int `position`[Get a thumbnail from video position]|get thumbnail file from video `path`|`Future<File>`|
|convertVideoToGif|String `path`[video path], int `startTime`(from 0 start)[convert video to gif start time], int `endTime`[convert video to gif end time], int `duration`[convert video to gif duration from start time]|convert video to gif from video `path`|`Future<File>`|
|getMediaInfo|String `path`[video path]|get media information from video `path`|`Future<MediaInfo>`|
|compressVideo|String `path`[video path], VideoQuality `quality`[compressed video quality], bool `deleteOrigin`[delete the origin video], int `startTime`[compression video start time], int `duration`[compression video duration from start time], bool `includeAudio`[is include audio in compressed video], int `frameRate`[compressed video frame rate]|compression video at origin video `path`|`Future<MediaInfo>`|
|cancelCompression|`none`|cancel compressing|`Future<void>`|
|deleteAllCache|`none`|Delete all files generated by 'flutter_video_compress' will delete all files located at 'flutter_video_compress'|`Future<bool>`|

## Subscriptions
|Subscriptions|Description|Stream|
|--|--|--|
|compressProgress$|Subscribe the compression progress steam|double `progress`|

## Notice
If your application is significantly larger after using the plugin, you can reduce the application size in the following way:

* exclude `x86` related files (`./assets`)

* This library not use `ffprobe`, only used `ffmpeg`, but the application still has `ffprobe`, so you will need to exclude (`asssets/arm` or `assets/x86`)

add this config in `build.gradle`:
* __Don't use__ `ignoreAssetsPattern "!x86"` in debug mode, will crash on the simulator

 ```gradle
android {
  ...
  // Reduce your application size with this configuration
  aaptOptions {
      ignoreAssetsPattern "!x86:!*ffprobe"
  }
  
  buildTypes {
  ...
}
```
[look detail](https://github.com/bravobit/FFmpeg-Android/wiki/Reduce-APK-File-Size#exclude-architecture)

## Questions

If your application is not enabled `AndroidX`, you will need to add the following code to the last line of the `android/build.gradle` file.
```groovy
rootProject.allprojects {
    subprojects {
        project.configurations.all {
            resolutionStrategy.eachDependency { details ->
                if (details.requested.group == 'androidx.core' && !details.requested.name.contains('androidx')) {
                    details.useVersion "1.0.1"
                }
            }
        }
    }
}
```

If your application not support swift, you need to add the following code in `ios/Podfile`.
```ruby
target 'Runner' do
  use_frameworks! # <--- add this
  ...
end
```

[look detail](https://github.com/flutter/flutter/issues/16049#issuecomment-382629492)


If your application never used a swift plugin before, maybe you would meet the error, you need to add the following code in `ios/Podfile`.
> The 'Pods-Runner' target has transitive dependencies that include static binaries

```ruby
pre_install do |installer|
  # workaround for https://github.com/CocoaPods/CocoaPods/issues/3289
  Pod::Installer::Xcode::TargetValidator.send(:define_method, :verify_no_static_framework_transitive_dependencies) {}
end
```

**If the above method not work, you report bug of the error repository. The reason is「can't support swift」**

[look detail](https://github.com/flutter/flutter/issues/16049#issue-309580132)

if show error of `Regift`

> - `Regift` does not specify a Swift version and none of the targets (`Runner`) integrating it have the `SWIFT_VERSION` attribute set. Please contact the author or set the `SWIFT_VERSION` attribute in at least one of the targets that integrate this pod.

```ruby
pre_install do |installer|
  installer.analysis_result.specifications.each do |s|
      if s.name == 'Regift'
        s.swift_version = '4.0'
      end
  end
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'NO'
    end
  end
end
```

## Contribute

Contributions are always welcome!
<!-- Please read the [contribution guidelines](contributing.md) first. -->