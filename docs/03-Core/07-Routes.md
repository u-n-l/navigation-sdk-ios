# Routes

A route represents a navigable path between two or more landmarks (waypoints), including distance, estimated time, and navigation instructions.

Compute routes in different ways:

* **Waypoint-based** - Based on 2 or more landmarks (navigable)
* **Over-track** - Based on a predefined `path` from GPX files or other sources (navigable)
* **Route ranges** - Not navigable, without segments or instructions

Navigable routes consist of segments. Each segment represents the portion between consecutive waypoints with its own route instructions.

## Create Routes[​](#create-routes "Direct link to Create Routes")

Routes cannot be instantiated directly. Compute them based on a list of landmarks. See [Get started with Routing](/docs/ios/guides/routing/get-started-routing.md) for details.

> 🚨 **DANGER**
>
> Calculating a route does not automatically display it on the map. See [Display routes](/docs/ios/guides/maps/display-map-items/display-routes.md) for instructions.

## Route types[​](#route-types "Direct link to Route types")

The iOS SDK supports standard, public transport, over-track, and EV route variants.

| Route type       | Type checks and conversion      | Class           |
| ---------------- | ------------------------------- | --------------- |
| Normal route     | default `RouteObject`           | `RouteObject`   |
| Public transport | `isPTRoute()` and `toPTRoute()` | `PTRouteObject` |
| Over-track       | `isOTRoute()` and `toOTRoute()` | `OTRouteObject` |
| Electric vehicle | `isEVRoute()` and `toEVRoute()` | `EVRouteObject` |

```swift
func inspectType(route: RouteObject) {
	if route.isPTRoute(), let ptRoute = route.toPTRoute() as? PTRouteObject {
		print("PT fare: \(ptRoute.getPTFare())")
	} else if route.isOTRoute(), let otRoute = route.toOTRoute() as? OTRouteObject {
		print("OT route has track: \(otRoute.getTrack() != nil)")
	} else if route.isEVRoute(), route.toEVRoute() is EVRouteObject {
		print("EV route detected")
	} else {
		print("Standard route")
	}
}

```

## RouteObject structure[​](#routeobject-structure "Direct link to RouteObject structure")

`RouteObject` is the shared contract for all route types.

### Core methods[​](#core-methods "Direct link to Core methods")

| Method                             | Return type                      | Description                                                     |
| ---------------------------------- | -------------------------------- | --------------------------------------------------------------- |
| `getWaypoints()`                   | `[LandmarkObject]`               | Route waypoints in order: departure, intermediates, destination |
| `getWaypoints(_:)`                 | `[LandmarkObject]`               | Waypoints filtered by `RouteWaypointsOption`                    |
| `getTimeDistance()`                | `TimeDistanceObject?`            | Total route metrics                                             |
| `getTimeDistance(withActivePart:)` | `TimeDistanceObject?`            | Metrics for active/remaining part or full route                 |
| `getGeographicArea()`              | `RectangleGeographicAreaObject?` | Bounding rectangle for the route                                |
| `getSummary()`                     | `String`                         | Human-readable summary                                          |
| `getSegments()`                    | `[RouteSegmentObject]`           | Segment list                                                    |
| `getTrafficEvents()`               | `[RouteTrafficEventObject]`      | Traffic events affecting the route                              |
| `getStatus()`                      | `RouteStatus`                    | Route lifecycle state                                           |
| `getIncursCosts()`                 | `Bool`                           | Cost-incurring flag                                             |
| `hasFerryConnections()`            | `Bool`                           | Whether ferry segments are present                              |
| `hasTollRoads()`                   | `Bool`                           | Whether toll roads are present                                  |

```swift
func summarize(route: RouteObject) {
	print("Summary: \(route.getSummary())")
	print("Waypoint count: \(route.getWaypoints().count)")

	if let td = route.getTimeDistance() {
		print("Distance (m): \(td.getTotalDistance())")
		print("Duration (s): \(td.getTotalTime())")
	}

	print("Has toll roads: \(route.hasTollRoads())")
	print("Has ferry connections: \(route.hasFerryConnections())")
}

```

### Geometry, sampling, and export[​](#geometry-sampling-and-export "Direct link to Geometry, sampling, and export")

| Method                                                | Return type                       | Description                                  |
| ----------------------------------------------------- | --------------------------------- | -------------------------------------------- |
| `getPath()`                                           | `PathObject?`                     | Full route geometry                          |
| `getPath(_:end:)`                                     | `PathObject?`                     | Geometry subset between two distances        |
| `getCoordinateOnRoute(_:)`                            | `CoordinatesObject?`              | Coordinate sampled at distance from start    |
| `getClosestSegment(_:)`                               | `Int32`                           | Closest segment index to a coordinate        |
| `getDistanceOnRoute(_:activePart:)`                   | `Int32`                           | Distance from departure to location          |
| `getTimeDistanceCoordinateOnRoute(_:)`                | `TimeDistanceCoordinatesObject?`  | Distance/time-aligned coordinate on route    |
| `getTimeDistanceCoordinates(from:end:step:stepType:)` | `[TimeDistanceCoordinatesObject]` | Distance or time-stepped route sampling      |
| `export(as:)`                                         | `Data?`                           | Exports route geometry in a path file format |
| `export(as:withCompresion:)`                          | `Data?`                           | Export with compression flag                 |
| `getDominantRoads()`                                  | `[String]`                        | Main roads covering the route                |

### Preferences and metadata[​](#preferences-and-metadata "Direct link to Preferences and metadata")

| Method                 | Return type                      | Description                                         |
| ---------------------- | -------------------------------- | --------------------------------------------------- |
| `getTerrainProfile()`  | `RouteTerrainProfileObject?`     | Elevation/slope analytics if enabled in preferences |
| `getPreferences()`     | `RoutePreferencesObject?`        | Route preferences used for computation              |
| `getExtraInfo()`       | `SearchableParameterListObject?` | User metadata bag                                   |
| `setExtraInfo(_:)`     | `Void`                           | Sets custom metadata                                |
| `isEqualWithRoute(_:)` | `Bool`                           | Equality check against another route                |

### RouteWaypointsOption[​](#routewaypointsoption "Direct link to RouteWaypointsOption")

| Value                                  | Description                                                |
| -------------------------------------- | ---------------------------------------------------------- |
| `RouteWaypointsOptionInitial`          | Initial waypoints used at first route calculation          |
| `RouteWaypointsOptionRemainingInitial` | Remaining initial waypoints (passed intermediates removed) |
| `RouteWaypointsOptionRemaining`        | Remaining waypoints including service-added entries        |

### RouteStatus[​](#routestatus "Direct link to RouteStatus")

| Value                                  | Description                    |
| -------------------------------------- | ------------------------------ |
| `RouteStatusUninitialized`             | Route object has no ready data |
| `RouteStatusCalculating`               | Calculation in progress        |
| `RouteStatusWaitingInternetConnection` | Waiting for connectivity       |
| `RouteStatusReady`                     | Route data is ready            |
| `RouteStatusError`                     | Calculation failed             |

### Sample route geometry and export[​](#sample-route-geometry-and-export "Direct link to Sample route geometry and export")

```swift
if let path = route.getPath() {
    print("Route points: \(path.getCoordinates().count)")
}
if let gpxData = route.export(as: .gpx) {
    print("Exported GPX size: \(gpxData.count) bytes")
}
if let point = route.getCoordinateOnRoute(2_000) {
    print("Coordinate at 2km: \(point.latitude), \(point.longitude)")
}
let samples = route.getTimeDistanceCoordinates(from: 0, end: 5_000, step: 500, stepType: true)
for sample in samples {
    print("distance=\(sample.getDistance()) ts=\(sample.getTimestamp())")
}

```

## RouteSegmentObject structure[​](#routesegmentobject-structure "Direct link to RouteSegmentObject structure")

A `RouteSegmentObject` represents the route portion between two consecutive waypoints.

| Method                | Return type                      | Description                                 |
| --------------------- | -------------------------------- | ------------------------------------------- |
| `getWaypoints()`      | `[LandmarkObject]`               | Segment departure and destination waypoints |
| `getTimeDistance()`   | `TimeDistanceObject?`            | Segment distance and travel time            |
| `getGeographicArea()` | `RectangleGeographicAreaObject?` | Segment bounding rectangle                  |
| `getIncursCosts()`    | `Bool`                           | Segment cost-incurring flag                 |
| `getSummary()`        | `String`                         | Segment summary text                        |
| `getInstructions()`   | `[RouteInstructionObject]`       | Segment turn instructions                   |
| `getPTInstructions()` | `[PTRouteInstructionObject]`     | PT instruction list                         |
| `isCommon()`          | `Bool`                           | Segment mode matches parent route mode      |

```swift
for (index, segment) in route.getSegments().enumerated() {
	print("Segment \(index): \(segment.getSummary())")

	if let td = segment.getTimeDistance() {
		print("  \(td.getTotalDistance()) m / \(td.getTotalTime()) s")
	}

	for instruction in segment.getInstructions() {
		print("  - \(instruction.getTurnInstruction())")
	}
}

```

## RouteInstructionObject structure[​](#routeinstructionobject-structure "Direct link to RouteInstructionObject structure")

`RouteInstructionObject` provides maneuver details, signposts, and road metadata.

| Method                                           | Return type              | Description                        |
| ------------------------------------------------ | ------------------------ | ---------------------------------- |
| `getCoordinates()`                               | `CoordinatesObject`      | Instruction location               |
| `getCountryCodeISO()`                            | `String`                 | ISO country code                   |
| `getTurnInstruction()`                           | `String`                 | Maneuver text                      |
| `getTurnDetails()`                               | `TurnDetailsObject?`     | Turn detail metadata               |
| `getTurnImage(_:)`                               | `UIImage?`               | Turn image at requested size       |
| `hasTurnInfo()`                                  | `Bool`                   | Turn info availability             |
| `getFollowRoadInstruction()`                     | `String`                 | Follow-road text                   |
| `hasFollowRoadInfo()`                            | `Bool`                   | Follow-road availability           |
| `getSignpostInstruction()`                       | `String`                 | Signpost text                      |
| `getSignpostDetails()`                           | `SignpostDetailsObject?` | Signpost detail object             |
| `hasSignpostInfo()`                              | `Bool`                   | Signpost availability              |
| `getRoadInfo()`                                  | `[RoadInfoObject]`       | Road metadata list                 |
| `getRoadInfoImage(_:)`                           | `UIImage?`               | Road info rendered image           |
| `hasRoadInfo()`                                  | `Bool`                   | Road info availability             |
| `getTimeDistanceToNextTurn()`                    | `TimeDistanceObject?`    | Remaining metrics to next maneuver |
| `getRemainingTravelTimeDistance()`               | `TimeDistanceObject?`    | Remaining metrics to destination   |
| `getRemainingTravelTimeDistanceToNextWaypoint()` | `TimeDistanceObject?`    | Remaining metrics to next waypoint |
| `getTraveledTimeDistance()`                      | `TimeDistanceObject?`    | Traveled metrics                   |
| `getExitDetails()`                               | `String`                 | Exit details text                  |
| `isExit()`                                       | `Bool`                   | Main-road exit flag                |
| `isFerry()`                                      | `Bool`                   | Ferry instruction flag             |
| `isTollRoad()`                                   | `Bool`                   | Toll-road instruction flag         |
| `isCommon()`                                     | `Bool`                   | Shares parent route transport mode |
| `isEV()`                                         | `Bool`                   | Instruction belongs to EV route    |

> 📝 **INFO**
>
> `RouteInstructionObject` is route-overview data. For live, position-dependent turn guidance during navigation, use `NavigationInstructionObject`.

```swift
if let instruction = route.getSegments().first?.getInstructions().first {
	print(instruction.getTurnInstruction())

	let turnImage = instruction.getTurnImage(CGSize(width: 64, height: 64))
	print("Turn image available: \(turnImage != nil)")

	if instruction.hasRoadInfo() {
		print("Road info count: \(instruction.getRoadInfo().count)")
	}
}

```

## Public transport route extensions[​](#public-transport-route-extensions "Direct link to Public transport route extensions")

### PTRouteObject[​](#ptrouteobject "Direct link to PTRouteObject")

`PTRouteObject` adds transit-specific details on top of `RouteObject`.

| Method                           | Return type  | Description                                 |
| -------------------------------- | ------------ | ------------------------------------------- |
| `getPTFare()`                    | `String`     | Fare text                                   |
| `getPTFrequency()`               | `Int`        | Frequency value                             |
| `getPTRespectsAllConditions()`   | `Bool`       | Whether all PT preferences are met          |
| `getCountBuyTicketInformation()` | `Int`        | Ticket info entry count                     |
| `getBuyTicketURL(_:)`            | `String`     | Ticket purchase URL for entry index         |
| `getSolutionPartIndexes(_:)`     | `[NSNumber]` | Route-solution part indexes for ticket info |

```swift
if route.isPTRoute(), let ptRoute = route.toPTRoute() as? PTRouteObject {
	print("Fare: \(ptRoute.getPTFare())")
	print("Frequency: \(ptRoute.getPTFrequency())")

	let ticketInfoCount = ptRoute.getCountBuyTicketInformation()
	for i in 0..<ticketInfoCount {
		let index = Int32(i)
		print("Ticket URL: \(ptRoute.getBuyTicketURL(index))")
		print("Affected parts: \(ptRoute.getSolutionPartIndexes(index))")
	}
}

```

### PTRouteSegmentObject[​](#ptroutesegmentobject "Direct link to PTRouteSegmentObject")

`PTRouteSegmentObject` provides schedule, agency, line, and realtime details.

| Method                                                        | Return type      | Description                                   |
| ------------------------------------------------------------- | ---------------- | --------------------------------------------- |
| `getName()`                                                   | `String`         | Segment/display name                          |
| `getShortName()`                                              | `String`         | Segment short name                            |
| `getPlatformCode()`                                           | `String`         | Platform code                                 |
| `getDepartureTime()` / `getArrivalTime()`                     | `TimeObject?`    | Raw departure/arrival times                   |
| `getDepartureTimeFormatted()` / `getArrivalTimeFormatted()`   | `String`         | Formatted time strings                        |
| `getAgencyName()`                                             | `String`         | Agency name                                   |
| `getAgencyPhone()` / `getAgencyUrl()` / `getAgencyFareUrl()`  | `String`         | Agency contact and fare URLs                  |
| `getLineFrom()` / `getLineTowards()`                          | `String`         | Transit line origin/destination               |
| `getTransitType()`                                            | `TransitType`    | Transport mode (`walk`, `bus`, `tram`, etc.)  |
| `getRealtimeStatus()`                                         | `RealtimeStatus` | Delay/on-time/unknown status                  |
| `getDepartureDelayInSeconds()` / `getArrivalDelayInSeconds()` | `Int32`          | Delay values in seconds                       |
| `getHasWheelchairSupport()` / `getHasBicycleSupport()`        | `Bool`           | Accessibility/bike support flags              |
| `getStayOnSameTransit()`                                      | `Bool`           | Stay-on-vehicle indicator                     |
| `getLineColor()` / `getLineTextColor()`                       | `UIColor`        | Line rendering colors                         |
| `getCountAlerts()`                                            | `Int32`          | Alert count for segment                       |
| `isSignificant()`                                             | `Bool`           | Indicates if segment is worth surfacing in UI |

### PTRouteInstructionObject[​](#ptrouteinstructionobject "Direct link to PTRouteInstructionObject")

| Method                                    | Return type   | Description                |
| ----------------------------------------- | ------------- | -------------------------- |
| `getName()`                               | `String`      | PT instruction label       |
| `getPlatformCode()`                       | `String`      | Platform code              |
| `getArrivalTime()` / `getDepartureTime()` | `TimeObject?` | PT arrival/departure data  |
| `getHasWheelchairSupport()`               | `Bool`        | Accessibility support flag |

## Route traffic updates via delegate[​](#route-traffic-updates-via-delegate "Direct link to Route traffic updates via delegate")

Assign `RouteObjectDelegate` when you need route-level traffic update callbacks.

| Delegate callback                           | Description                                                        |
| ------------------------------------------- | ------------------------------------------------------------------ |
| `routeObject(_:onTrafficEventsUpdated:)`    | Called when route traffic events change; includes delay difference |
| `routeObject(_:onTrafficEventsAlongRoute:)` | Called when traffic-event verification status changes              |

```swift
final class RouteObserver: NSObject, RouteObjectDelegate {
	func routeObject(_ route: RouteObject, onTrafficEventsUpdated delayDiff: Int32) {
		print("Route delay delta: \(delayDiff) s")
	}

	func routeObject(_ route: RouteObject, onTrafficEventsAlongRoute checked: Bool) {
		print("Traffic events verified: \(checked)")
	}
}

```

## Related classes[​](#related-classes "Direct link to Related classes")

### TimeDistanceObject[​](#timedistanceobject "Direct link to TimeDistanceObject")

`TimeDistanceObject` is used across route, segment, and instruction APIs.

| Method                                                            | Return type               | Description                               |
| ----------------------------------------------------------------- | ------------------------- | ----------------------------------------- |
| `getTotalTime()` / `getTotalDistance()`                           | `UInt32`                  | Total duration/distance                   |
| `getUnrestrictedTime()` / `getUnrestrictedDistance()`             | `UInt32`                  | Public-road travel metrics                |
| `getRestrictedTime()` / `getRestrictedDistance()`                 | `UInt32`                  | Restricted-road metrics                   |
| `getRestrictedTimeAtBegin()` / `getRestrictedTimeAtEnd()`         | `UInt32`                  | Restricted-time split over begin/end      |
| `getRestrictedDistanceAtBegin()` / `getRestrictedDistanceAtEnd()` | `UInt32`                  | Restricted-distance split over begin/end  |
| `getTotalDistanceMeasurement()`                                   | `Measurement<UnitLength>` | Measurement object with unit support      |
| `getTotalDistanceFormatted()` / `getTotalDistanceUnitFormatted()` | `String`                  | Localized distance value/unit             |
| `getTotalTimeFormatted()` / `getTotalTimeUnitFormatted()`         | `String`                  | Localized time value/unit                 |
| `getFormattedDistance(_:)` / `getFormattedDistanceUnit(_:)`       | `String`                  | Formatted value/unit for arbitrary meters |

### TurnDetailsObject[​](#turndetailsobject "Direct link to TurnDetailsObject")

| Method                                                                                     | Return type                                     | Description                        |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------- | ---------------------------------- |
| `getEvent()`                                                                               | `TurnType`                                      | Turn event enum                    |
| `getAbstractGeometry()`                                                                    | `AbstractGeometryObject?`                       | Vector geometry for turn           |
| `getAbstractGeometryImage()`                                                               | `AbstractGeometryImageObject?`                  | Renderable abstract geometry image |
| `getRoundaboutExitNumber()`                                                                | `Int32`                                         | Exit number or `-1` if unavailable |
| `getTurnImage(_:colorActiveInner:colorActiveOuter:colorInactiveInner:colorInactiveOuter:)` | `UIImage?`                                      | Color-customized turn image        |
| `getTurnImageId()`                                                                         | `Int32`                                         | Turn image ID                      |
| `getTurnId32()` / `getTurnId64()`                                                          | `TurnSimplifiedType32` / `TurnSimplifiedType64` | Simplified maneuver IDs            |

```swift

let scale = UIScreen.main.scale
let imgSize = CGSize.init(width: 40.0 * scale, height: 40.0 * scale)

turnDetailsObject.getTurnImage(imgSize,
		colorActiveInner: UIColor.white,
		colorActiveOuter: UIColor.black,
		colorInactiveInner: UIColor.lightGray,
		colorInactiveOuter: UIColor.lightGray)

turnDetailsObject.getTurnImage(imgSize,
		colorActiveInner: UIColor.red,
		colorActiveOuter: UIColor.green,
		colorInactiveInner: UIColor.blue,
		colorInactiveOuter: UIColor.yellow)

```

![Turn image with basic colors](../assets/images/turn_image_basic_colors.png "Turn Image with basic colors")

**Turn Image with basic colors**

![Turn image with customized colors](../assets/images/turn_image_custom_colors.png "Turn Image with customized colors")

**Turn Image with customized colors**

### SignpostDetailsObject[​](#signpostdetailsobject "Direct link to SignpostDetailsObject")

Signposts near roadways indicate intersections and directions. The SDK provides realistic image renderings with additional information.

![Signpost image](../assets/images/signpost_image-eeff2259d472a997f59b32830e683611.png "Signpost image captured during highway navigation")

**Signpost image captured during highway navigation**

| Method                                          | Return type            | Description                         |
| ----------------------------------------------- | ---------------------- | ----------------------------------- |
| `getItems()`                                    | `[SignpostItemObject]` | Signpost elements                   |
| `hasBorderColor()` / `getBorderColor()`         | `Bool` / `UIColor`     | Border color availability/value     |
| `hasTextColor()` / `getTextColor()`             | `Bool` / `UIColor`     | Text color availability/value       |
| `hasBackgroundColor()` / `getBackgroundColor()` | `Bool` / `UIColor`     | Background color availability/value |
| `getImage(_:)`                                  | `UIImage?`             | Rendered signpost image             |

### RouteTerrainProfileObject[​](#routeterrainprofileobject "Direct link to RouteTerrainProfileObject")

If route preferences enabled terrain profile generation, `getTerrainProfile()` returns this object.

| Method                                                          | Return type                | Description                            |
| --------------------------------------------------------------- | -------------------------- | -------------------------------------- |
| `getMinElevation()` / `getMaxElevation()`                       | `Float`                    | Min/max elevation                      |
| `getMinElevationDistance()` / `getMaxElevationDistance()`       | `Int32`                    | Distances for min/max elevation points |
| `getTotalUp()` / `getTotalDown()`                               | `Float`                    | Total ascent/descent                   |
| `getTotalUp(_:end:)` / `getTotalDown(_:end:)`                   | `Float`                    | Ascent/descent for route range         |
| `getElevationSamples(_:)`                                       | `[NSNumber]`               | Elevation sample list                  |
| `getElevationSamples(_:distBegin:distEnd:)`                     | `[NSNumber]`               | Elevation samples for route subsection |
| `getElevation(_:)`                                              | `Float`                    | Elevation at distance                  |
| `getClimbSections()`                                            | `[ClimbSectionObject]`     | Climb sections                         |
| `getSurfaceSections()`                                          | `[SurfaceSectionObject]`   | Surface type sections                  |
| `getRoadTypeSections()`                                         | `[RoadTypeSectionObject]`  | Road type sections                     |
| `getSteepSections(_:)`                                          | `[RoadSteepSectionObject]` | Steepness-classified sections          |
| `getElevationChartMinValueY()` / `getElevationChartMaxValueY()` | `Float`                    | Chart range helpers                    |

### RouteTrafficEventObject[​](#routetrafficeventobject "Direct link to RouteTrafficEventObject")

`RouteTrafficEventObject` extends traffic-event data with route-relative context.

| Method                                                            | Return type          | Description                                        |
| ----------------------------------------------------------------- | -------------------- | -------------------------------------------------- |
| `getDistanceToDestination()`                                      | `Int32`              | Distance from event to destination                 |
| `getFrom()` / `getTo()`                                           | `CoordinatesObject?` | Event start/end coordinates                        |
| `getFromLandmark()` / `getToLandmark()`                           | `LandmarkObject?`    | Event start/end landmarks                          |
| `hasTrafficEventOnDistance(_:)`                                   | `Bool`               | Whether event affects the remaining route distance |
| `isInsideTrafficEventOnDistance(_:)`                              | `Bool`               | Whether the point is inside event impact zone      |
| `getDistanceFormatted()` / `getDistanceUnitFormatted()`           | `String`             | Formatted event distance                           |
| `getDelayTimeFormatted()` / `getDelayTimeUnitFormatted()`         | `String`             | Formatted delay time                               |
| `getDelayDistanceFormatted()` / `getDelayDistanceUnitFormatted()` | `String`             | Formatted delay distance                           |

***

Use `RouteObject` as the base route contract, then branch to `PTRouteObject`, `OTRouteObject`, or `EVRouteObject` only when route-type-specific behavior is needed.
