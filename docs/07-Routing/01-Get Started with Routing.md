# Get started with routing

This guide explains how to calculate routes, retrieve turn-by-turn instructions, access terrain profiles, traffic events, and detailed route information using `NavigationContext` and `RoutePreferencesObject`.

Here's a quick overview of what you can do with routing:

* Calculate routes between a start point and destination.
* Include intermediate waypoints for multi-stop routes.
* Compute range routes to determine areas reachable within a specific range.
* Plan routes over predefined tracks using `PathObject`.
* Customize routes with `RoutePreferencesObject` — route type, transport mode, vehicle profiles, and avoidances.
* Retrieve turn-by-turn instructions via `RouteInstructionObject`.
* Access terrain profiles using `RouteTerrainProfileObject`.

> **WARNIING**
>
> `NavigationContext` must be retained in memory for the duration of the route calculation. If the instance is deallocated before the request completes, the completion handler will not be called. Store it as a property on your view controller or data manager.

## Calculate routes[​](#calculate-routes "Direct link to Calculate routes")

Create a `NavigationContext` with a `RoutePreferencesObject`, then call `calculateRoute(withWaypoints:statusHandler:completionHandler:)`.

```swift
let preferences = RoutePreferencesObject()

let departure = LandmarkObject.landmark(
    withName: "Paris",
    location: CoordinatesObject.coordinates(withLatitude: 48.85682, longitude: 2.34375)
)
let destination = LandmarkObject.landmark(
    withName: "Brussels",
    location: CoordinatesObject.coordinates(withLatitude: 50.84644, longitude: 4.34587)
)

let navigationContext = NavigationContext(preferences: preferences)

navigationContext.calculateRoute(withWaypoints: [departure, destination],
    statusHandler: { status in
        print("Route status: \(status)")
    },
    completionHandler: { routes, code in
        switch code {
        case .kNoError:
            print("Routes computed: \(routes.count)")
        case .kCancel:
            print("Route computation cancelled")
        default:
            print("Error: \(code)")
        }
    }
)

```

The `statusHandler` receives `RouteStatus` values as the calculation progresses:

| `RouteStatus`                          | Meaning                   |
| -------------------------------------- | ------------------------- |
| `RouteStatusCalculating`               | Computation in progress.  |
| `RouteStatusWaitingInternetConnection` | Waiting for connectivity. |
| `RouteStatusReady`                     | Route is ready.           |
| `RouteStatusError`                     | Computation failed.       |

The `SDKErrorCode` passed to the completion handler can include:

| Code                              | Significance                                                      |
| --------------------------------- | ----------------------------------------------------------------- |
| `SDKErrorCodeKNoError`            | Successfully completed.                                           |
| `SDKErrorCodeKCancel`             | Cancelled by the user.                                            |
| `SDKErrorCodeKWaypointAccess`     | One or more waypoints not accessible with current preferences.    |
| `SDKErrorCodeKConnectionRequired` | Online routing required but `allowOnlineCalculation` is disabled. |
| `SDKErrorCodeKRouteTooLong`       | Route took too long on the server.                                |
| `SDKErrorCodeKInvalidated`        | Offline map data changed during calculation.                      |
| `SDKErrorCodeKNoMemory`           | Routing engine could not allocate required memory.                |

Cancel an ongoing calculation:

```swift
navigationContext.cancelCalculateRoute()

```

When cancelled the completion handler receives `SDKErrorCodeKCancel`.

## Retrieve time and distance information[​](#retrieve-time-and-distance-information "Direct link to Retrieve time and distance information")

Use `getTimeDistance()` and `getTimeDistance(withActivePart:)` on a `RouteObject` to access the route duration and length.

```swift
if let td = route.getTimeDistance() {
    let totalDistanceM = td.getTotalDistance()
    let totalDurationS = td.getTotalTime()
    print("Distance: \(totalDistanceM) m, Duration: \(totalDurationS) s")
}

// Remaining portion only
if let remaining = route.getTimeDistance(withActivePart: true) {
    print("Remaining distance: \(remaining.getTotalDistance()) m")
}

```

Pass `false` for the full route; pass `true` (default) for the active remaining portion. Distance is in meters, time in seconds.

### Access traffic events[​](#access-traffic-events "Direct link to Access traffic events")

```swift
let trafficEvents = route.getTrafficEvents()

for event in trafficEvents {
    let from = event.getFrom() ?? "unknown"
    let to = event.getTo() ?? "unknown"
    print("From: \(from) To: \(to)")
}

```

See the [Traffic Events guide](../03-Core/09-Traffic%20Events.md) for detailed information.

## Display routes on the map[​](#display-routes-on-the-map "Direct link to Display routes on the map")

Routes are not automatically displayed after calculation. Refer to the [display routes](../04-Maps/05-Display%20Map%20Items/04-Display%20Routes.md) guide for visualization and customization options.

## Get the terrain profile[​](#get-the-terrain-profile "Direct link to Get the terrain profile")

Enable terrain profile generation by setting `setBuildTerrainProfile(true)` on `RoutePreferencesObject` before calculating the route:

```swift
let preferences = RoutePreferencesObject()
preferences.setBuildTerrainProfile(true)

```

> 🚨 **DANGER**
>
> `setBuildTerrainProfile(true)` must be set on the preferences used for `calculateRoute(withWaypoints:)` before the call. It cannot be applied retroactively to an already-computed route.

Access elevation and terrain data from the profile:

```swift
if let profile = route.getTerrainProfile() {
    let minElevation = profile.getMinElevation()
    let maxElevation = profile.getMaxElevation()
    let minElevDist = profile.getMinElevationDistance()
    let maxElevDist = profile.getMaxElevationDistance()
    let totalUp = profile.getTotalUp()
    let totalDown = profile.getTotalDown()

    // Elevation at 100 m from the start
    let elevation = profile.getElevation(100)

    for section in profile.getRoadTypeSections() {
        let roadType: RoadType = section.type
        let startDist = section.startDistanceM
    }

    for section in profile.getSurfaceSections() {
        let surface: SurfaceType = section.type
        let startDist = section.startDistanceM
    }

    for section in profile.getClimbSections() {
        let grade: ClimbGrade = section.grade
        let slope = section.slope
        let start = section.startDistanceM
        let end = section.endDistanceM
    }

    let categs: [NSNumber] = [-16, -10, -7, -4, -1, 1, 4, 7, 10, 16]
    for section in profile.getSteepSections(categs) {
        let categ = section.categ
        let startDist = section.startDistanceM
    }
}

```

**`RoadType`** values: `motorways`, `stateRoad`, `road`, `street`, `cycleway`, `path`, `singleTrack`.

**`SurfaceType`** values: `asphalt`, `paved`, `unpaved`, `unknown`.

## Retrieve route instructions[​](#retrieve-route-instructions "Direct link to Retrieve route instructions")

Access turn-by-turn `RouteInstructionObject` instances through each `RouteSegmentObject`.

Each **segment** covers the route portion between two consecutive waypoints. A route with five waypoints has four segments.

```swift
for segment in route.getSegments() {
    for instruction in segment.getInstructions() {
        let turn = instruction.getTurnInstruction()
        let followRoad = instruction.getFollowRoadInstruction()

        if let traveled = instruction.getTraveledTimeDistance() {
            let distanceM = traveled.getTotalDistance()
        }

        if let toNextTurn = instruction.getTimeDistanceToNextTurn() {
            print("To next turn: \(toNextTurn.getTotalDistance()) m")
        }

        if instruction.hasTurnInfo(), let details = instruction.getTurnDetails() {
            // details.getAbstractGeometry() for turn geometry image
        }

        if instruction.hasSignpostInfo() {
            let signpost = instruction.getSignpostInstruction()
        }

        let countryCode = instruction.getCountryCodeISO()
    }
}

```

Key `RouteInstructionObject` methods:

| Method                                           | Description                                                      |
| ------------------------------------------------ | ---------------------------------------------------------------- |
| `getTurnInstruction()`                           | Textual description of the turn, e.g. "Bear left onto A 5".      |
| `getFollowRoadInstruction()`                     | Follow-road description, e.g. "Follow A 5 for 132m".             |
| `getTraveledTimeDistance()`                      | Distance and time traveled from route start to this instruction. |
| `getRemainingTravelTimeDistance()`               | Remaining distance and time to route end.                        |
| `getRemainingTravelTimeDistanceToNextWaypoint()` | Distance and time to the next waypoint.                          |
| `getTimeDistanceToNextTurn()`                    | Distance and time to the next turn instruction.                  |
| `getTurnDetails()`                               | Full `TurnDetailsObject` for the maneuver.                       |
| `getRoadInfo()`                                  | `RoadInfoObject` array for the current road.                     |
| `hasFollowRoadInfo()`                            | Whether follow-road information is available.                    |
| `getCountryCodeISO()`                            | ISO 3166-1 alpha-3 country code at this instruction.             |

