# Recorder

The Recorder module manages sensor data recording with configurable parameters through `RecorderConfigurationObject`. Key features include:

* **Customizable storage options** - Define log directories, manage disk space, and specify recording duration
* **Data type selection** - Specify which data types (video, audio, or sensor data) to record
* **Video and audio options** - Set video resolution, enable/disable audio recording, and manage chunk durations. Record sensor data only, sensor data + video, sensor data + audio, or all three

Control the recording lifecycle:

* Start, stop, pause, and resume recordings
* Automatic restarts for continuous recording with chunked durations

The Recorder supports various transportation modes (car, pedestrian, bike), enabling detailed analysis and classification based on context. Set disk space limits to prevent overwhelming device storage - logs are automatically managed based on retention thresholds.

## Create recorder configuration[​](#create-recorder-configuration "Direct link to Create recorder configuration")

Create a recorder configuration with a logs folder and a data source context.

```swift
let source = DataSourceContext()
let config = RecorderConfigurationObject(folderPath: tracksPath, context: source)

config.setDataType(.position)
config.setMinDurationSeconds(10)
config.setChunkDurationSeconds(config.kInfiniteRecording)
config.setContinuousRecording(false)

```

`RecorderConfigurationObject` controls what is recorded, how long chunks are, and how storage limits are applied.

### Common configuration options[​](#common-configuration-options "Direct link to Common configuration options")

| API                             | Description                                                       |
| ------------------------------- | ----------------------------------------------------------------- |
| `setDataType(_:)`               | Adds a data type to be recorded (for example position, camera).   |
| `setMinDurationSeconds(_:)`     | Minimum duration required for saving a recording.                 |
| `setChunkDurationSeconds(_:)`   | Chunk length in seconds (`kInfiniteRecording` disables chunking). |
| `setContinuousRecording(_:)`    | Restarts recording automatically at chunk boundaries.             |
| `setEnableAudio(_:)`            | Enables audio recording (when permitted).                         |
| `setVideoQuality(_:)`           | Sets camera recording resolution (`ResolutionFormat`).            |
| `setMaxDiskSpaceUsed(_:)`       | Caps storage used by all logs.                                    |
| `setKeepMinSeconds(_:)`         | Minimum retained recording duration before cleanup is allowed.    |
| `setDeleteOlderThanKeepMin(_:)` | Deletes older logs once retention threshold is exceeded.          |
| `setTransportMode(_:)`          | Sets transport mode metadata for the recording.                   |

You can also enable audio:

```swift
config.setEnableAudio(true)

```

Set video quality with one of the `ResolutionFormat` values (for example `ResolutionFormatHD_720p`) when recording camera frames.

> ⚠️ **WARNING**
>
> If the recording duration is shorter than the configured `setMinDurationSeconds(_:)`, the `stopRecording()` method does not save the recording and returns `SDKErrorCodeRecordedLogTooShort`.
>
> The `SDKErrorCodeRecordedLogTooShort` error may also occur if an insufficient number of positions were emitted, even when the duration between `startRecording()` and `stopRecording()` exceeds `setMinDurationSeconds(_:)`. To test recording functionality, create a custom external `DataSourceContext` and push custom positions. Refer to the [custom positioning guide](../05-Positioning%20&%20Sensors/04-Custom%20Positioning.md) for details.
>
> The `SDKErrorCodeKGeneral` result might be returned if the application has been sent to the background without the required configuration. See the Recording lifecycle section below for information about proper recording management.
>
> If `setChunkDurationSeconds(_:)` is set too high, it may cause `SDKErrorCodeKNoDiskSpace` since the SDK determines how much space is required for the entire chunk.
>
> Ensure that the `DataType` values passed via `setDataType(_:)` are supported by the target platform. Using unsupported data types may cause the `startRecording()` method to return `SDKErrorCodeKInvalidInput`.

### Resolution formats[​](#resolution-formats "Direct link to Resolution formats")

| Enum value                     | Resolution  |
| ------------------------------ | ----------- |
| `ResolutionFormatSD_480p`      | 640 × 480   |
| `ResolutionFormatHD_720p`      | 1280 × 720  |
| `ResolutionFormatFullHD_1080p` | 1920 × 1080 |
| `ResolutionFormatWQHD_1440p`   | 2560 × 1440 |
| `ResolutionFormatUHD_4K_2160p` | 3840 × 2160 |
| `ResolutionFormatUHD_8K_4320p` | 7680 × 4320 |

## Recorder permissions (iOS)[​](#recorder-permissions-ios "Direct link to Recorder permissions (iOS)")

Add the required usage descriptions in `Info.plist` depending on what you record:

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>Location is needed for recording location-tagged logs.</string>

<key>NSCameraUsageDescription</key>
<string>Camera is needed for video recording.</string>

<key>NSMicrophoneUsageDescription</key>
<string>Microphone is needed for audio recording.</string>

```

Location is required for position-based recording. Add camera/microphone keys when recording camera or audio streams.

## Start and stop recording[​](#start-and-stop-recording "Direct link to Start and stop recording")

```swift
let recorder = RecorderContext(configuration: config)

let startCode = recorder.startRecording()
print("start = \(startCode)")

// ...

let stopCode = recorder.stopRecording()
print("stop = \(stopCode)")

```

`startRecording()` and `stopRecording()` return `SDKErrorCode`, which should always be checked in production flows.

## Pause and resume[​](#pause-and-resume "Direct link to Pause and resume")

```swift
_ = recorder.pauseRecording()
_ = recorder.resumeRecording()

```

Pause/resume is useful when the session should remain open but data capture is temporarily suspended.

## Recording lifecycle[​](#recording-lifecycle "Direct link to Recording lifecycle")

Typical lifecycle:

1. Create `RecorderConfigurationObject`.
2. Create `RecorderContext`.
3. Start recording.
4. Optionally pause/resume or toggle audio.
5. Stop recording and inspect resulting path / status.

Recorder states are exposed through `RecorderStatus`:

| Status                                           | Meaning                                         |
| ------------------------------------------------ | ----------------------------------------------- |
| `RecorderStatusStopped`                          | Recorder is idle.                               |
| `RecorderStatusStarting`                         | Start is in progress.                           |
| `RecorderStatusRecording`                        | Recording is active.                            |
| `RecorderStatusPausing` / `RecorderStatusPaused` | Pause transition / paused state.                |
| `RecorderStatusResuming`                         | Resume transition in progress.                  |
| `RecorderStatusStopping`                         | Stop is in progress.                            |
| `RecorderStatusRestarting`                       | Restart due to configuration or chunk behavior. |

## Track status via delegate[​](#track-status-via-delegate "Direct link to Track status via delegate")

```swift
final class RecorderHandler: NSObject, RecorderContextDelegate {
    func recorderContext(_ recorderContext: RecorderContext, recordingStatusChanged status: RecorderStatus) {
        print("status=\(status)")
    }

    func recorderContext(_ recorderContext: RecorderContext, recordingCompleted path: String, code: SDKErrorCode) {
        print("saved at \(path), code=\(code)")
    }
}

let handler = RecorderHandler()
recorder.delegate = handler

```

Use the delegate to drive UI state, progress labels, and post-recording workflows.

## Record audio and camera streams[​](#record-audio-and-camera-streams "Direct link to Record audio and camera streams")

Enable optional streams through configuration and recorder controls:

```swift
config.setDataType(.camera)
config.setVideoQuality(.hd_720p)
config.setEnableAudio(true)

let recorder = RecorderContext(configuration: config)

_ = recorder.startRecording()
recorder.startAudioRecording()

// ...

recorder.stopAudioRecording()
_ = recorder.stopRecording()

```

## Recorder bookmarks and metadata[​](#recorder-bookmarks-and-metadata "Direct link to Recorder bookmarks and metadata")

The SDK uses the proprietary `.gm` format for recordings and provides APIs to inspect, convert, and enrich recorded logs.

Use `LogBookmarksContext` for log management tasks:

* **Export and import logs** - Convert logs to and from formats such as GPX, NMEA, KML, GeoJSON, CSV, FIT, and TCX
* **Log metadata access** - Read timestamps, coordinates, routes, transport mode, and size
* **Custom metadata and files** - Attach or read additional binary metadata and activity-related files

```swift
let bookmarks = LogBookmarksContext(folderPath: tracksPath)
let logs = bookmarks.getLogs()

```

### Export logs[​](#export-logs "Direct link to Export logs")

```swift
let bookmarks = LogBookmarksContext(folderPath: yourPath)
let logs = bookmarks.getLogs()

if let logPath = logs.last {
    let code = bookmarks.exportLog(
        logPath,
        to: .gpx,
        exportedFileName: "My_File_Name",
        positionDistance: 0
    )
    print("export = \(code)")
}

```

The resulting file is `My_File_Name.gpx`. If `exportedFileName` is empty, the SDK uses the original log name.

> ⚠️ **WARNING**
>
> Exporting a `.gm` file to other formats can lead to data loss, depending on which sensor types are supported by the target format.

> 💡 **TIP**
>
> The exported file is saved in the same directory as the original log file.

### Import logs[​](#import-logs "Direct link to Import logs")

Import logs from standard formats such as `gpx`, `nmea`, or `kml` into SDK log format:

```swift
let bookmarks = LogBookmarksContext(folderPath: tracksPath)
let code = bookmarks.importLog("/path/to/file.gpx", importedFileName: "My_File_Name")
print("import = \(code)")

```

### Access metadata[​](#access-metadata "Direct link to Access metadata")

For iOS, metadata is accessed through `LogBookmarksContext` methods for a given log path:

```swift
let bookmarks = LogBookmarksContext(folderPath: tracksPath)
let logs = bookmarks.getLogs()

if let logPath = logs.last, bookmarks.isMetadataAvailable(logPath) {
    let start = bookmarks.getStartTimestamp(logPath)
    let end = bookmarks.getEndTimestamp(logPath)
    let duration = bookmarks.getDuration(logPath)
    let startPosition = bookmarks.getStartPosition(logPath)
    let endPosition = bookmarks.getEndPosition(logPath)
    let route = bookmarks.getRoute(logPath)
    let preciseRoute = bookmarks.getPreciseRoute(logPath)
    let transportMode = bookmarks.getTransportMode(logPath)
    let isProtected = bookmarks.isProtected(logPath)
    let logSize = bookmarks.getLogSize(logPath)
    let metrics = bookmarks.getLogMetrics(logPath)

    print("start=\(start) end=\(end) duration=\(duration)")
    print("distance=\(metrics.distanceMeters) elevation=\(metrics.elevationGainMeters) avgSpeed=\(metrics.avgSpeedMps)")
    print("transport=\(transportMode) protected=\(isProtected) size=\(logSize)")
    print("startPos=\(String(describing: startPosition)) endPos=\(String(describing: endPosition))")
    print("routePoints=\(route.count) preciseRoutePoints=\(preciseRoute.count)")
}

```

> ⚠️ **WARNING**
>
> Check `isMetadataAvailable(_:)` before reading metadata. Metadata access can fail for missing files or invalid log content.

The metadata exposed by `LogBookmarksContext` includes:

* `getStartPosition(_:)` / `getEndPosition(_:)` - Geographic coordinates for the start and end of the log
* `getRoute(_:)` - Reduced route polyline
* `getPreciseRoute(_:)` - Full recorded coordinate sequence
* `getTransportMode(_:)` - `RecordingTransportMode` used while recording
* `getStartTimestamp(_:)` / `getEndTimestamp(_:)` - Timestamp of first/last recorded sensor data
* `getDuration(_:)` / `getActiveDuration(_:)` - Total and active recording durations
* `isProtected(_:)` - Whether the log is protected from automatic cleanup
* `getLogSize(_:)` - Log size in bytes
* `isDataTypeAvailable(_:filePath:)` - Whether a specific `DataType` exists in the log
* `getSoundMarks(_:)` - Recorded sound marks metadata
* `getLogMetrics(_:)` - `LogBookmarksMetrics` values (`distanceMeters`, `elevationGainMeters`, `avgSpeedMps`)

### Custom user metadata[​](#custom-user-metadata "Direct link to Custom user metadata")

Add custom metadata while recording with `RecorderContext`, then read it from `LogBookmarksContext`:

```swift
let config = RecorderConfigurationObject(folderPath: tracksPath, context: source)
config.setDataType(.position)

let recorder = RecorderContext(configuration: config)
_ = recorder.startRecording()

let text = "Hello world"
if let textData = text.data(using: .utf8) {
    recorder.addUserMetadata("textData", buffer: textData)
}

_ = recorder.stopRecording()

let bookmarks = LogBookmarksContext(folderPath: tracksPath)
let logs = bookmarks.getLogs()
if let logPath = logs.last,
   let payload = bookmarks.getUserMetadata(logPath, keyString: "textData"),
   let decoded = String(data: payload, encoding: .utf8) {
    print(decoded)
}

```

## Log Activity Files[​](#log-activity-files "Direct link to Log Activity Files")

Represented by `LogActivityFileObject` attachments managed through `LogBookmarksContext`.

### Attributes[​](#attributes "Direct link to Attributes")

| Attribute        | Description                                                 |
| ---------------- | ----------------------------------------------------------- |
| `activityFileID` | Unique identifier of the activity file.                     |
| `activityID`     | Identifier of the related activity.                         |
| `fileType`       | Type of attached file (`photo`, `video`, `audio`, `route`). |
| `fileData`       | Binary payload of the attached file.                        |
| `offsetMillis`   | Timestamp offset in milliseconds within the activity.       |
| `coordinates`    | Coordinates where the file was captured.                    |

### LogMetrics[​](#logmetrics "Direct link to LogMetrics")

Use `LogBookmarksMetrics` (retrieved with `getLogMetrics(_:)`) for log statistics:

| Attribute             | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `distanceMeters`      | Total distance covered during the log, measured in meters.   |
| `elevationGainMeters` | Total elevation gained during the log, measured in meters.   |
| `avgSpeedMps`         | Average speed during the log, measured in meters per second. |

### Setting the activity record[​](#setting-the-activity-record "Direct link to Setting the activity record")

Attach activity-related files to a log using `addActivityFiles(_:filePath:)`:

```swift
let bookmarks = LogBookmarksContext(folderPath: tracksPath)
let logs = bookmarks.getLogs()

if let logPath = logs.last {
    let activityFile = LogActivityFileObject()
    activityFile.fileType = .photo
    activityFile.offsetMillis = 15_000
    activityFile.coordinates = bookmarks.getStartPosition(logPath)
    activityFile.fileData = Data() // Replace with image/audio/video/route bytes

    bookmarks.addActivityFiles([activityFile], filePath: logPath)
}

```

> ⚠️ **WARNING**
>
> Attach activity files to an existing log file path. Ensure recording is completed and the target log exists before adding files.

### Get the activity record[​](#get-the-activity-record "Direct link to Get the activity record")

Retrieve attached files from a log:

```swift
let bookmarks = LogBookmarksContext(folderPath: tracksPath)
let logs = bookmarks.getLogs()

if let logPath = logs.last {
    let activityFiles = bookmarks.getActivityFiles(logPath)
    print("activity files count = \(activityFiles.count)")
}

```
