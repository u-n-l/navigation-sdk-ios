# Adjust map view

The UNL Navigation SDK for iOS lets you control viewport framing, camera orientation, zoom, perspective, and coordinate transforms through `MapViewController` and `MapViewPreferencesContext`.

> 📝 **INFO**
>
> Most code snippets in this guide use UIKit syntax, but the same APIs are available in SwiftUI via the `proxy.mapViewController` provided by `MapReader`. Some are also available via modifiers or `MapBase` initializers for quick view updates, as seen in [Get started with maps](../04-Maps/01-Get%20Started%20with%20Maps.md).

## Get viewport metrics[​](#get-viewport-metrics "Direct link to Get viewport metrics")

The viewport is returned in screen pixels and is relative to the map view's parent.

```swift
let viewport = mapViewController.getViewport()
let scale = mapViewController.getScaleFactor()
let ppi = mapViewController.getPpiFactor()

print("viewport=\(viewport) scale=\(scale) ppi=\(ppi)")

```

> 📝 **INFO**
>
> Use pixel values when providing screen positions to map APIs such as centering with a custom point or cursor selection.

## Center the map[​](#center-the-map "Direct link to Center the map")

Use the centering APIs based on the target object.

### Center on coordinates[​](#center-on-coordinates "Direct link to Center on coordinates")

```swift
let location = CoordinatesObject.coordinates(withLatitude: 48.8566, longitude: 2.3522)
mapViewController.center(onCoordinates: location, zoomLevel: 68, animationDuration: 900)

```

### Center on a geographic area[​](#center-on-a-geographic-area "Direct link to Center on a geographic area")

```swift
let center = CoordinatesObject.coordinates(withLatitude: 44.93343, longitude: 25.09946)

let area = RectangleGeographicAreaObject(location: center, horizontalRadius: 3000, verticalRadius: 2000)

mapViewController.center(onArea: area, zoomLevel: -1, animationDuration: 700)

```

### Center on an area with padding[​](#center-on-an-area-with-padding "Direct link to Center on an area with padding")

Use edge area insets to reserve screen space, then center on the area.

```swift
let center = CoordinatesObject.coordinates(withLatitude: 44.93343, longitude: 25.09946)
let area = RectangleGeographicAreaObject(location: center, horizontalRadius: 3000, verticalRadius: 2000)

mapViewController.setEdgeAreaInsets(UIEdgeInsets(top: 150, left: 100, bottom: 150, right: 100))
mapViewController.center(onArea: area, zoomLevel: -1, animationDuration: 700)

// Optional: reset when you no longer need extra padding.
mapViewController.setEdgeAreaInsets(.zero)

```

### Center on routes and route parts[​](#center-on-routes-and-route-parts "Direct link to Center on routes and route parts")

```swift
mapViewController.center(onRoutes: [mainRoute, alternativeRoute], displayMode: .full, animationDuration: 900)

let screenRect = CGRect(x: 40, y: 180, width: 320, height: 420)
mapViewController.center(onRoute: mainRoute, startDist: 0, endDist: 12000, rectangle: screenRect, animationDuration: 700)

```

## Convert screen and WGS coordinates[​](#convert-screen-and-wgs-coordinates "Direct link to Convert screen and WGS coordinates")

Convert a tapped screen point to WGS and back:

```swift
let point = CGPoint(x: 200, y: 320)
let wgs = mapViewController.transformScreenToWgs(point)
let screenPoint = mapViewController.transformWgsToScreen(wgs)

print("wgs=\(wgs.latitude),\(wgs.longitude) screen=\(screenPoint)")

```

You can also convert a screen rectangle to geographic areas using `transformScreenRectToWgs(_:)`.

## Adjust zoom, angle, rotation, and perspective[​](#adjust-zoom-angle-rotation-and-perspective "Direct link to Adjust zoom, angle, rotation, and perspective")

* UIKit
* SwiftUI

```swift
// UIKit

mapViewController.setZoomLevel(65, animationDuration: 400)
mapViewController.setViewAngle(45)
mapViewController.setTiltAngle(20)

mapViewController.setPerspective(.view3D, animationDuration: 600) { success in
    print("3D perspective set: \(success)")
}

let preferences = mapViewController.getPreferences()
preferences.setRotationAngle(35)

```

```swift
// SwiftUI

MapReader { proxy in
    MapBase()
        .onAppear {
            guard let mapViewController = proxy.mapViewController else {
                return
            }
            let preferences = mapViewController.getPreferences()

            mapViewController.setZoomLevel(65, animationDuration: 400)
            mapViewController.setViewAngle(45)
            mapViewController.setTiltAngle(20)

            mapViewController.setPerspective(.view3D, animationDuration: 600) { success in
                print("3D perspective set: \(success)")
            }

            preferences.setRotationAngle(35)
        }
}

```

Here are some videos demonstrating the rotation and tilt angle changes.

**Tilt angle & view angle**
<video controls src="../assets/videos/ios_tilt_animation-be84b7e83d4db807ee83301e24c97cb3.mp4" title="Title"></video>

**Rotation angle**
<video controls src="../assets/videos/ios_rotation_animation-41fab7d04e3519a0f8919c724e5aca1b.mp4" title="Title"></video>

<br />

> 🚨 **DANGER**
>
> Do not confuse zoom level with map style detail quality. Zoom controls camera distance, while map detail quality is controlled separately with `setMapDetailsQualityLevel(_:)`.

## Store and restore camera state[​](#store-and-restore-camera-state "Direct link to Store and restore camera state")

Use `MapCameraObject` when you need to persist camera position and orientation.

```swift
if let camera = mapViewController.getMapCamera(),
   let savedState = camera.saveCameraState() {
    UserDefaults.standard.set(savedState, forKey: "savedMapCamera")
}

if let camera = mapViewController.getMapCamera(),
   let data = UserDefaults.standard.data(forKey: "savedMapCamera") {
    _ = camera.restoreCameraState(data)
}

```

## Customize follow position camera[​](#customize-follow-position-camera "Direct link to Customize follow position camera")

```swift
let preferences = mapViewController.getPreferences()
let follow = preferences.getFollowPositionPreferences()

follow.setCameraFocus(CGPoint(x: 0.5, y: 0.5))
follow.setZoomLevel(70, animationDuration: 0)
follow.setMapRotationMode(.positionHeading, angle: 0, objectFollowMap: true)

// Keep the changes made by touch gestures while following the position, instead of resetting to the
// default zoom and angle after exiting the following mode
follow.setTouchHandlerModifyPersistent(true)

```


