# Positioning & Sensors

This section covers positioning workflows in the UNL Navigation SDK for iOS: data sources, live and custom positions, follow-position behavior, recorder setup, projection conversion, and camera-feed integration.

Use these guides to start from device location updates and move toward advanced testing and playback scenarios.

- [Sensors and data sources](./01-Sensors%20and%20Data%20Sources.md) - Learn how DataSourceContext provides live, playback, simulation, and external sensor streams for positioning workflows.
- [Get started with positioning](/./02-Get%20Started%20with%20Positioning.md) - Learn how to configure permissions and start receiving live position updates in iOS.
- [Show location on map](./03-Show%20Location%20on%20Map.md) - Learn how to follow the current position, tune follow-position behavior, and customize the position tracker on the map.
- [Custom positioning](./04-Custom%20Positioning.md) - Learn how to push custom positions into an external DataSourceContext and consume them through PositionContext.
- [Recorder](./05-Recorder.md) - The Recorder module manages sensor data recording with configurable parameters through RecorderConfigurationObject. 
- [Projections](./06-Projections.md) - [Learn how to create projection objects and convert between coordinate systems using ProjectionContext.]
- [Background location](./07-Background%20Location.md) - Learn how to enable location updates while your app is in background on iOS.
- [Camera feed](./08-Camera%20Feed.md) - The SDK DataSourceContext provides access to camera frames for both live camera feeds and recorded logs.