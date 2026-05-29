# Core

This section covers the core runtime entities used across the Navigation SDK for iOS: geographic primitives, positions, landmarks, markers, overlays, routes, navigation instructions, traffic events, and image rendering APIs.

You can use these guides as a reference path from low-level map entities to real-time navigation data.

- [Base Entities](./01-Base%20Entities.md) -
  This page covers the fundamental building blocks of the iOS SDK: coordinates, paths, and geographic areas.
- [Positions](./02-Positions.md) - This page covers position handling in iOS using PositionObject and PositionContext.
- [Landmarks](./03-Landmarks.md) - A landmark is a rich point-of-interest entity represented by LandmarkObject.
- [Markers](./04-Markers.md) - A marker is a visual geometry represented by MarkerObject.
- [Overlays](./05-Overlays.md) - An overlay is an additional map layer with data stored on UNL servers, accessible in both online and offline modes.
- [Landmarks vs Markers vs Overlays](./06-Landmarks%20vs%20Markers%20vs%20Overlays.md) - When building map features, choose the entity that matches your data lifecycle and interaction model.
- [Routes](./07-Routes.md) - A route represents a navigable path between two or more landmarks (waypoints), including distance, estimated time, and navigation instructions.
- [Navigation Instructions](./08-Navigation%20Instructions.md) - The UNL Navigation SDK for iOS provides real-time navigation guidance through NavigationInstructionObject.
- [Traffic Events](./09-Traffic%20Events.md) - The UNL Navigation SDK for iOS provides real-time traffic information about delays, incidents, and road restrictions that can affect routing and navigation.
- [Images](./10-Images.md) - The UNL Navigation SDK for iOS uses ImageObject for plain SDK images and UIKit-native UIImage rendering for display.