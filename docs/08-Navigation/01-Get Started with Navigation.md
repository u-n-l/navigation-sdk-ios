# Get started with Navigation

This guide shows you how to implement turn-by-turn navigation in your iOS app using `NavigationContext` and `NavigationContextDelegate`.

>📝 **INFO**
>
> **What you need**
>
> * A computed route (non-navigable routes like range routes are not supported)
> * Proper location permissions for real GPS navigation
> * Map data downloaded for offline functionality

**Key features:**

* **Turn-by-Turn Directions** - Detailed route instructions based on current location
* **Live Guidance** - Text and voice instructions via `SoundContext` integration
* **Warning Alerts** - Speed limits, traffic reports, and route events
* **Offline Support** - Works offline with pre-downloaded map data

The navigation system tracks your device location, speed, and heading, matching them against the route to generate accurate guidance. Instructions update dynamically as you progress.

When you deviate from the route, the system notifies you and offers recalculation options. It can also adjust routes based on real-time traffic for faster alternatives.

You can test navigation features using the built-in location simulator during development.

## How navigation works[​](#how-navigation-works "Direct link to How navigation works")

The SDK offers two navigation methods:

* **Navigation** - Uses position data from `GEMSdk` position services to guide users along the route
* **Simulation** - Simulates navigation instructions without real position data for testing

Navigation mode uses the SDK's position service with:

* **Real GPS Data** - Requires location permissions on iOS. See [Get started with positioning](../05-Positioning%20&%20Sensors/02-Get%20Started%20with%20Positioning.md)
* **Custom Position Data** - Configure a custom data source for position updates. No permissions required. See [Custom positioning](../05-Positioning%20&%20Sensors/04-Custom%20Positioning.md)

> ⚠️ **WARNING**
>
> Only one navigation or simulation can be active at a time, regardless of map count.

> ⚠️ **WARNING**
>
> These are simplified code snippets, make sure the `MapViewController` object is properly set up as shown in previous guides.

## Start navigation[​](#start-navigation "Direct link to Start navigation")

Once you have a computed route, start navigation using `navigateWithRoute(_:completionHandler:)` on a `NavigationContext` instance. Implement `NavigationContextDelegate` to receive navigation events:

```swift
// Retain these as properties to keep navigation alive
var navigationContext: NavigationContext?
var mapView: MapViewController?

func startNavigation(route: RouteObject) {
    let preferences = RoutePreferencesObject()
    navigationContext = NavigationContext(preferences: preferences)
    navigationContext?.delegate = self
    navigationContext?.navigate(withRoute: route) { [weak self] success in
        if success {
            self?.mapView?.presentRoutes([route], withTraffic: nil, showSummary: false, animationDuration: 0)
            self?.mapView?.startFollowingPosition(withAnimationDuration: 0,zoomLevel: -1) { _ in }
        } else {
            print("Navigation failed to start")
        }
    }
}

// MARK: - NavigationContextDelegate
func navigationContext(_ navigationContext: NavigationContext,
                       navigationStartedForRoute route: RouteObject) {
    print("Navigation started")
}

func navigationContext(_ navigationContext: NavigationContext,
                       navigationInstructionUpdatedForRoute route: RouteObject,
                       updatedEvents events: Int32) {
    guard let instruction = navigationContext.getNavigationInstruction() else { return }

    if events & Int32(NavigationInstructionUpdateEvent.nextTurnUpdated.rawValue) > 0 {
        let text = instruction.getNextTurnInstruction()
        print("Turn: \(text)")
    }
    if events & Int32(NavigationInstructionUpdateEvent.nextTurnImageUpdated.rawValue) > 0 {
        let image = instruction.getNextTurnImage(CGSize(width: 48, height: 48))
        // update turn image in UI
    }
    if events & Int32(NavigationInstructionUpdateEvent.laneInfoUpdated.rawValue) > 0 {
        // update lane guidance UI
    }
}

func navigationContext(_ navigationContext: NavigationContext,
                       route: RouteObject,
                       navigationDestinationReached waypoint: LandmarkObject) {
    print("Destination reached: \(waypoint.getLandmarkName())")
}

func navigationContext(_ navigationContext: NavigationContext,
                       route: RouteObject?,
                       navigationError code: Int) {
    print("Navigation error: \(code)")
}

```

> 📝 **INFO**
>
> `NavigationContext` must be kept alive for the duration of navigation. Store it as a property on your view controller or data manager. Deallocating it will stop navigation.

The `NavigationContextDelegate` delivers the following navigation events:

| Delegate method                                                                         | Significance                                                                                                                                                           |
| --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `navigationContext(_:navigationStartedForRoute:)`                                       | Called when navigation begins, signaling the start of route guidance.                                                                                                  |
| `navigationContext(_:navigationInstructionUpdatedForRoute:updatedEvents:)`              | Triggered when a new navigation instruction is available. The `updatedEvents` bitmask indicates what changed using `NavigationInstructionUpdateEvent` flags.           |
| `navigationContext(_:route:navigationStatusChanged:)`                                   | Called when the navigation status changes. See `NavigationStatus` values below.                                                                                        |
| `navigationContext(_:route:navigationWaypointReached:)`                                 | Invoked when a waypoint in the route is reached, including details of the waypoint.                                                                                    |
| `navigationContext(_:route:navigationDestinationReached:)`                              | Called upon reaching the final destination, with information about the destination landmark.                                                                           |
| `navigationContext(_:navigationRouteUpdated:)`                                          | Fired when the current route is updated, providing the new route details.                                                                                              |
| `navigationContext(_:route:navigationError:)`                                           | Called when a navigation error occurs. See error codes below.                                                                                                          |
| `navigationContext(_:onBetterRouteDetected:branchingCoords:travelTime:delay:timeGain:)` | Triggered when a better alternative route is detected. See the [Better route detection guide](../08-Navigation/03-Better%20Route%20Detection.md) for more details. |
| `navigationContext(_:onBetterRouteInvalidated:)`                                        | Indicates that a previously suggested better route is no longer valid.                                                                                                 |
| `navigationContext(_:onSkipNextIntermediateDestinationDetected:)`                       | Indicates the user is moving away from the next intermediate waypoint. Consider calling `skipNextIntermediateDestination()` on the navigation context.                 |
| `navigationContext(_:canPlayNavigationSoundForRoute:)`                                  | Returns whether the SDK is allowed to play a navigation sound. Return `true` to allow built-in audio playback.                                                         |
| `navigationContext(_:route:navigationSound:)`                                           | Provides a `SoundObject` when a sound needs to be played. See [Add voice guidance](../08-Navigation/02-Add%20Voice%20Guidance.md).                                     |

**`NavigationStatus` values:**

| Value                                  | Meaning                                                       |
| -------------------------------------- | ------------------------------------------------------------- |
| `NavigationStatusRunning`              | Normal running state.                                         |
| `NavigationStatusWaitingRoute`         | Paused, waiting for route update (re-routing enabled).        |
| `NavigationStatusWaitingGPS`           | Paused, waiting for GPS location to recover.                  |
| `NavigationStatusWaitingReturnToRoute` | Paused, waiting to return to the route (re-routing disabled). |

**`NavigationInstructionUpdateEvent` flags:**

| Flag                                                   | Meaning                            |
| ------------------------------------------------------ | ---------------------------------- |
| `NavigationInstructionUpdateEventNextTurnUpdated`      | The next turn instruction changed. |
| `NavigationInstructionUpdateEventNextTurnImageUpdated` | The turn arrow image changed.      |
| `NavigationInstructionUpdateEventLaneInfoUpdated`      | Lane guidance information changed. |

> 💡 **TIP**
>
> Present the route and start following position inside the navigation/simulation completion handler to ensure the map updates only after the session has started successfully. See [Show your location on the map](../05-Positioning%20&%20Sensors/03-Show%20Location%20on%20Map.md) for camera customization options.

Display the route on the map for better navigation clarity. Turn-by-turn navigation arrows disappear once passed. Learn more in [Display routes](../04-Maps/05-Display%20Map%20Items/04-Display%20Routes.md).

The traveled portion of the route changes color using the `traveledInnerColor` parameter of `RouteRenderSettings`.

## Start simulation[​](#start-simulation "Direct link to Start simulation")

Start a simulation using `simulateWithRoute(_:speedMultiplier:completionHandler:)`. The delegate methods work identically to real navigation:

```swift
func startSimulation(route: RouteObject) {
    let preferences = RoutePreferencesObject()
    navigationContext = NavigationContext(preferences: preferences)
    navigationContext?.delegate = self

    navigationContext?.simulate(withRoute: route, speedMultiplier: 2.0) { [weak self] success in
        if success {
            self?.mapView?.presentRoutes([route], withTraffic: nil, showSummary: false, animationDuration: 0)
            self?.mapView?.startFollowingPositionWithAnimationDuration(0, zoomLevel: -1) { _ in }
        } else {
            print("Simulation failed to start")
        }
    }
}

```

The `speedMultiplier` sets simulation speed (default is 1.0, matching the maximum speed limit for each road segment). Use `getSimulationMinSpeedMultiplier()` and `getSimulationMaxSpeedMultiplier()` to query the allowed range.

## Read navigation progress[​](#read-navigation-progress "Direct link to Read navigation progress")

At any point during navigation or simulation, read the current instruction via `getNavigationInstruction()` on the `NavigationContext`:

```swift
if let instruction = navigationContext?.getNavigationInstruction() {
    // Next turn
    let turnText = instruction.getNextTurnInstruction()
    let turnImage = instruction.getNextTurnImage(CGSize(width: 48, height: 48))

    // Distances and times
    let toNextTurn = instruction.getTimeDistanceToNextTurn()
    let remaining = instruction.getRemainingTravelTimeDistance()

    // Current street
    let streetName = instruction.getCurrentStreetName()
    let speedLimit = instruction.getCurrentStreetSpeedLimit() // m/s
}

```

Retrieve ETA and remaining distance as formatted strings directly from `NavigationContext`:

```swift
let eta = navigationContext?.getEstimateTimeOfArrivalFormatted() ?? ""
let etaUnit = navigationContext?.getEstimateTimeOfArrivalUnitFormatted() ?? ""
let timeLeft = navigationContext?.getRemainingTravelTimeFormatted() ?? ""
let distLeft = navigationContext?.getRemainingTravelDistanceFormatted() ?? ""

```

## Skip intermediate destination[​](#skip-intermediate-destination "Direct link to Skip intermediate destination")

When `onSkipNextIntermediateDestinationDetected` fires, call `skipNextIntermediateDestination()` to drop the current waypoint and continue to the next one:

```swift
func navigationContext(_ navigationContext: NavigationContext,
                       onSkipNextIntermediateDestinationDetected state: Bool) {
    if state {
        navigationContext.skipNextIntermediateDestination()
    }
}

```

> 📝 **INFO**
>
> If there are no more intermediate waypoints on the route, `skipNextIntermediateDestination()` returns `.kNotFound`.

## Stop navigation or simulation[​](#stop-navigation-or-simulation "Direct link to Stop navigation or simulation")

Call `cancelNavigateRoute()` or `cancelSimulateRoute()` to stop an active session:

```swift
// Stop navigation
navigationContext?.cancelNavigateRoute()

// Stop simulation
navigationContext?.cancelSimulateRoute()

```

> 📝 **INFO**
>
> After stopping simulation, the position service reverts to the previous data source if one exists.

## Run navigation in background[​](#run-navigation-in-background "Direct link to Run navigation in background")

To use navigation while your app is in the background, additional setup is required. See the [Background location guide](../05-Positioning%20&%20Sensors/07-Background%20Location.md) for configuration instructions.

