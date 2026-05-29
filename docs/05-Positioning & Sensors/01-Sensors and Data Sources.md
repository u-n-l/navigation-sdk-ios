# Sensors and data sources

Learn how `DataSourceContext` provides live, playback, simulation, and external sensor streams for positioning workflows.

Use these APIs when you need to read device sensor data, replay recorded logs, simulate movement on a route, or inject your own custom samples.

## Sensor data types[​](#sensor-data-types "Direct link to Sensor data types")

`DataSourceContext` works with `DataType` values exposed by the iOS SDK. The following data types are defined in GEMKit for iOS:

| Type                        | Description                                                                    |
| --------------------------- | ------------------------------------------------------------------------------ |
| `DataType.acceleration`     | Linear acceleration data from device motion sensors.                           |
| `DataType.activity`         | Activity classification data exposed by the SDK data model.                    |
| `DataType.attitude`         | 3D device attitude/orientation data.                                           |
| `DataType.battery`          | Battery state and level information.                                           |
| `DataType.camera`           | Camera frames and related camera-feed data.                                    |
| `DataType.compass`          | Heading information from compass sensors.                                      |
| `DataType.magneticField`    | Raw magnetic-field sensor data.                                                |
| `DataType.orientation`      | Device orientation updates.                                                    |
| `DataType.position`         | Raw geographic position data.                                                  |
| `DataType.improvedPosition` | Improved or map-matched position data.                                         |
| `DataType.rotationRate`     | Rotation-rate / gyroscope-style motion data.                                   |
| `DataType.temperature`      | Device temperature information.                                                |
| `DataType.notification`     | System or SDK notification-type data events.                                   |
| `DataType.mountInformation` | Device mount/orientation metadata, useful for camera and in-vehicle scenarios. |
| `DataType.unknown`          | Fallback value for unsupported or unclassified data.                           |

For position structure details, see [Positions](../03-Core/02-Positions.md).

> 📝 **INFO**
>
> Not every `DataType` is guaranteed to be available from every source or on every device. Use `getAvailableDataTypes()` or `isDataTypeAvailable(_:)` to inspect what a specific `DataSourceContext` can actually provide at runtime.

## Working with data sources[​](#working-with-data-sources "Direct link to Working with data sources")

The main iOS classes used in this area are:

| Class                           | Role                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------ |
| `DataSourceContext`             | Main entry point for live, playback, simulation, and external data streams.    |
| `DataSourceConfigurationObject` | Configures how a source produces position-related data.                        |
| `DataSourcePlaybackContext`     | Adds playback controls such as pause, resume, seek, and speed multiplier.      |
| `DataSourceContextDelegate`     | Receives new data, interruptions, progress updates, and playing-state changes. |

## Create data sources[​](#create-data-sources "Direct link to Create data sources")

Create a source type that matches your scenario:

| Source kind      | Initializer                             | Typical use                                    |
| ---------------- | --------------------------------------- | ---------------------------------------------- |
| Live             | `DataSourceContext()`                   | Real device sensor input.                      |
| Log playback     | `DataSourceContext(filePath:)`          | Replay a previously recorded file such as GPX. |
| Route simulation | `DataSourceContext(route:)`             | Generate simulated positions along a route.    |
| External         | `DataSourceContext(externalDataTypes:)` | Push custom samples from your own system.      |

Use the following initializers:

```swift
let liveSource = DataSourceContext()
let logSource = DataSourceContext(filePath: "/path/to/track.gpx")
let simulationSource = DataSourceContext(route: route)
let externalSource = DataSourceContext(externalDataTypes: [NSNumber(value: DataType.position.rawValue)])

guard let logSource, let simulationSource, let externalSource else { return }

```

The live source initializer is non-nullable. Log, simulation, and external initializers are nullable and should be unwrapped with `guard let` or `if let`.

## Configure and control sources[​](#configure-and-control-sources "Direct link to Configure and control sources")

Use `DataSourceConfigurationObject` to tune how position data is produced, then start or stop the source as needed.

```swift
let source = DataSourceContext()
let config = DataSourceConfigurationObject()

config.setPositionAccuracy(.whenMoving)
config.setPositionDistanceFilter(0)

_ = source.setConfiguration(config, for: .position)
_ = source.start()

// ...

_ = source.stop()

```

Common operations on `DataSourceContext`:

| Method                       | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| `setConfiguration(_:for:)`   | Applies configuration for a specific `DataType`.             |
| `start()`                    | Starts the source.                                           |
| `stop()`                     | Stops the source.                                            |
| `isStopped()`                | Checks whether the source is stopped.                        |
| `getAvailableDataTypes()`    | Returns the data types currently supported by the source.    |
| `getLatestData(_:)`          | Returns the latest sample for a specific type, if available. |
| `setMockData(withPosition:)` | Applies mock position data for testing on supported sources. |
| `pushData(_:)`               | Pushes a `DataObject` into an external source.               |

### Inspect source metadata[​](#inspect-source-metadata "Direct link to Inspect source metadata")

`DataSourceContext` also exposes helper information about the current source:

| Method                 | Description                                                     |
| ---------------------- | --------------------------------------------------------------- |
| `getDataSourceType()`  | Returns whether the source is `live`, `playback`, or `unknown`. |
| `getOrigin()`          | Tells whether the source is SDK-provided or external.           |
| `getLogPath()`         | Returns the playback file path when applicable.                 |
| `getCurrentPosition()` | Returns the current playback cursor in milliseconds.            |
| `getDuration()`        | Returns the playback duration in milliseconds.                  |

### Use playback controls[​](#use-playback-controls "Direct link to Use playback controls")

For playback-capable sources, use `DataSourcePlaybackContext`. This applies to log or simulation flows, not standard live sensor sources.

```swift
let playback = DataSourcePlaybackContext(context: logSource)
playback.setSpeedMultiplier(2.0)
_ = playback.pause()
_ = playback.resume()

```

Playback-specific methods include:

| Method                   | Description                             |
| ------------------------ | --------------------------------------- |
| `pause()`                | Pauses playback.                        |
| `resume()`               | Resumes playback.                       |
| `step()`                 | Advances playback incrementally.        |
| `getState()`             | Returns the current `PlayingStatus`.    |
| `setSpeedMultiplier(_:)` | Adjusts playback speed.                 |
| `setCurrentPosition(_:)` | Seeks to a specific playback timestamp. |
| `getDuration()`          | Returns total playback duration.        |

## Listen for source events[​](#listen-for-source-events "Direct link to Listen for source events")

Use `DataSourceContextDelegate` for new data, interruptions, and status changes.

Main delegate callbacks:

| Callback                  | Description                                                               |
| ------------------------- | ------------------------------------------------------------------------- |
| `onNewData`               | Called when a new `DataObject` is available.                              |
| `onPlayingStatusChanged`  | Called when a source changes between stopped, paused, and playing states. |
| `onDataInterruptionEvent` | Called when a data stream is interrupted or restored.                     |
| `onProgressChanged`       | Reports playback progress in milliseconds.                                |
| `onDeviceMountedChanged`  | Reports changes in mount state and portrait orientation.                  |
| `onTemperatureChanged`    | Reports thermal state changes.                                            |

Example:

```swift
final class SourceListener: NSObject, DataSourceContextDelegate {
    func dataSourceContext(_ dataSourceContext: DataSourceContext, onNewData dataObject: DataObject) {
        print("new data type=\(dataObject.getType())")
    }

    func dataSourceContext(_ dataSourceContext: DataSourceContext, onPlayingStatusChanged type: DataType, status: PlayingStatus) {
        print("status for \(type) = \(status)")
    }

    // Implement other delegate methods as needed...
}

```

```swift
func setupListener() {
    let listener = SourceListener()
    let source = DataSourceContext()
    source.delegate = listener
    _ = source.startDelegateNotification(with: .position)
}

```

Call `stopDelegateNotification(with:)` or `stopDelegateNotifications()` when you no longer need updates.
