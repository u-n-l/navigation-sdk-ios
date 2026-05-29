# Public Transit Stops

This API provides access to public transport data including agencies, routes, stops, and trips. Fetch and explore real-time public transportation information from selected positions on the map.

> 💡 **TIP**
>
> The public transport data structure follows the [General Transit Feed Specification (GTFS)](https://gtfs.org/documentation/schedule/reference/) and offers access to a subset of GTFS fields and entities.

**Key features:**

* Query public transport overlays by screen position
* Retrieve information about transport agencies, stops, routes, and trips
* Access real-time data including delays and cancellations
* View metadata about accessibility, bike allowances, and platform details

**How it works:**

* Set a cursor position on the map using `setCursorPosition(_:)` on `MapViewController`
* Retrieve overlay items at that position using `getCursorSelectionOverlayItems()`
* Filter items by overlay UID to isolate public transit stops
* Fetch extended stop data asynchronously using `getPreviewExtendedDataWithCompletionHandler(_:)` on each `OverlayItemObject`
* Parse the returned `SearchableParameterListObject` using `findPublicTransportListParameterType(_:)` with the appropriate `PublicTransportParameterType`

## Query Public Transit Stops[​](#query-public-transit-stops "Direct link to Query Public Transit Stops")

Implement the `mapViewController(_:onLongTouchPoint:)` delegate method on `MapViewControllerDelegate`. When the user long-presses the map, set the cursor to that position and retrieve public transit overlay items.

```swift
func mapViewController(_ mapViewController: MapViewController, onLongTouch point: CGPoint) {

    mapViewController.setCursorPosition(point)

    let overlays = mapViewController.getCursorSelectionOverlayItems()
    let publicTransportId = Int(CommonOverlayIdentifier.publicTransport.rawValue)

    for item in overlays where item.getOverlayUid() == publicTransportId {

        guard item.hasPreviewExtendedData() else { continue }

        item.getPreviewExtendedData { parametersList in
            // agencies
            let agencies = parametersList.findPublicTransportListParameterType(.agencies)
            // stops (routes without heading set)
            let stops = parametersList.findPublicTransportListParameterType(.stops)
            // trips (route, agency, stop times, real-time info)
            let trips = parametersList.findPublicTransportListParameterType(.trips)
            // example of parsing stops
            for stop in stops {

                if let stopId   = stop[PublicTransportStopParameterType.id.rawValue]   as? NSValue,
                   let stopName = stop[PublicTransportStopParameterType.name.rawValue] as? NSValue {
                    print("Stop id: \(stopId) name: \(stopName)")
                }

                if let routes = stop[NSNumber(value: PublicTransportStopParameterType.routes.rawValue)] as? [[NSNumber: NSValue]] {
                    for route in routes {
                        
                        if let value = route[NSNumber(value: PublicTransportRouteParameterType.shortName.rawValue)] {
                            
                            if let data = value.nonretainedObjectValue as? NSData,
                               let name = String(data: data as Data, encoding: .utf8) {
                                print("  Route short name: \(name)")
                            }
                        }
                    }
                }
            }
        }
    }
}

```

> 🚨 **DANGER**
>
> All returned times are local times encoded as Unix timestamps (seconds since 1970-01-01T00:00:00). Use `TimezoneContext` to convert them to other time zones.

> 🚨 **DANGER**
>
> Two types of public transit stops exist on the map:
>
> * `OverlayItemObject` stops selected via `getCursorSelectionOverlayItems()` — provide extensive extended data and display with a blue icon (default style)
> * `LandmarkObject` stops selected via `getCursorSelectionLandmarks()` — provide limited details and display with a gray icon (default style)

## Agencies[​](#agencies "Direct link to Agencies")

Agencies are returned as an array of `[NSNumber: NSValue]` dictionaries. Use `PublicTransportAgencyParameterType` enum values as keys.

| Key                                      | Type               | Description                     |
| ---------------------------------------- | ------------------ | ------------------------------- |
| `PublicTransportAgencyParameterTypeId`   | `NSValue` (Int)    | Agency ID                       |
| `PublicTransportAgencyParameterTypeName` | `NSValue` (String) | Full name of the transit agency |
| `PublicTransportAgencyParameterTypeUrl`  | `NSValue` (String) | URL of the transit agency       |

## Public Transport Routes[​](#public-transport-routes "Direct link to Public Transport Routes")

Route info is embedded within stop and trip dictionaries. Use `PublicTransportRouteParameterType` enum values as keys.

| Key                                           | Type               | Description                                              |
| --------------------------------------------- | ------------------ | -------------------------------------------------------- |
| `PublicTransportRouteParameterTypeId`         | `NSValue` (Int)    | Route ID                                                 |
| `PublicTransportRouteParameterTypeShortName`  | `NSValue` (String) | Short name (e.g., "32", "100X")                          |
| `PublicTransportRouteParameterTypeLongName`   | `NSValue` (String) | Full name, often including destination                   |
| `PublicTransportRouteParameterTypeType`       | `NSValue` (Int)    | Transport type — see `PublicTransportRouteTransportType` |
| `PublicTransportRouteParameterTypeRouteColor` | `NSValue` (String) | Route color as a hex string                              |
| `PublicTransportRouteParameterTypeTextColor`  | `NSValue` (String) | Text color for use against route color                   |
| `PublicTransportRouteParameterTypeHeading`    | `NSValue` (String) | Optional heading information                             |

> 🚨 **DANGER**
>
> Route info embedded in stop and trip dictionaries provides information about public transit routes available at a specific stop. `PTRouteObject` represents a computed public transit route between multiple waypoints.
>
> See [Compute Public Transit Routes](./07-Routing/03-Advanced%20Features.md#compute-public-transit-routes) for computing routes.

### Route transport types[​](#route-transport-types "Direct link to Route transport types")

The `PublicTransportRouteTransportType` enum represents the type of public transport route:

| Enum value                                        | Description                        |
| ------------------------------------------------- | ---------------------------------- |
| `PublicTransportRouteTransportTypeBus`            | Bus or trolleybus                  |
| `PublicTransportRouteTransportTypeUnderground`    | Subway or metro                    |
| `PublicTransportRouteTransportTypeRailway`        | Intercity or long-distance rail    |
| `PublicTransportRouteTransportTypeTram`           | Tram, streetcar, or light rail     |
| `PublicTransportRouteTransportTypeWaterTransport` | Ferry or other water-based transit |
| `PublicTransportRouteTransportTypeMisc`           | Other types of public transport    |

## Stops[​](#stops "Direct link to Stops")

Stops are returned as an array of `[NSNumber: NSValue]` dictionaries. Use `PublicTransportStopParameterType` enum values as keys.

| Key                                         | Type                     | Description                                                         |
| ------------------------------------------- | ------------------------ | ------------------------------------------------------------------- |
| `PublicTransportStopParameterTypeId`        | `NSValue` (Int)          | Stop ID                                                             |
| `PublicTransportStopParameterTypeName`      | `NSValue` (String)       | Name of the stop as it appears on timetables and signage            |
| `PublicTransportStopParameterTypeIsStation` | `NSValue` (Bool)         | Whether this location is a station containing one or more platforms |
| `PublicTransportStopParameterTypeRoutes`    | `NSArray` of route dicts | Routes serving this stop                                            |

## Stop Times[​](#stop-times "Direct link to Stop Times")

Stop times are embedded within trip dictionaries under the `PublicTransportTripParameterTypeStopTimes` key. Each stop time is a `[NSNumber: NSValue]` dictionary. Use `PublicTransportStopTimeParameterType` enum values as keys.

| Key                                                 | Type               | Description                                                |
| --------------------------------------------------- | ------------------ | ---------------------------------------------------------- |
| `PublicTransportStopTimeParameterTypeStopName`      | `NSValue` (String) | Name of the serviced stop                                  |
| `PublicTransportStopTimeParameterTypeDepartureTime` | `NSValue` (Int)    | Departure time (seconds since 1970-01-01T00:00:00)         |
| `PublicTransportStopTimeParameterTypeHasRealtime`   | `NSValue` (Bool)   | Whether data is provided in real-time                      |
| `PublicTransportStopTimeParameterTypeDelay`         | `NSValue` (Int)    | Delay in seconds. Not meaningful if `hasRealtime` is false |
| `PublicTransportStopTimeParameterTypeIsBefore`      | `NSValue` (Bool)   | Whether the stop time is before the current time           |
| `PublicTransportStopTimeParameterTypeLatitude`      | `NSValue` (Double) | Stop latitude                                              |
| `PublicTransportStopTimeParameterTypeLongitude`     | `NSValue` (Double) | Stop longitude                                             |

## Trips[​](#trips "Direct link to Trips")

Trips are returned as an array of `[NSNumber: NSValue]` dictionaries. Use `PublicTransportTripParameterType` enum values as keys.

| Key                                                    | Type                         | Description                                                              |
| ------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------------------ |
| `PublicTransportRouteParameterTypeId`                  | `NSValue` (Int)              | Route ID                                                                 |
| `PublicTransportRouteParameterTypeShortName`           | `NSValue` (String)           | Route short name                                                         |
| `PublicTransportRouteParameterTypeLongName`            | `NSValue` (String)           | Route long name                                                          |
| `PublicTransportRouteParameterTypeType`                | `NSValue` (Int)              | Route transport type                                                     |
| `PublicTransportRouteParameterTypeHeading`             | `NSValue` (String)           | Optional heading                                                         |
| `PublicTransportTripParameterTypeHasRealtime`          | `NSValue` (Bool)             | Whether real-time data is available                                      |
| `PublicTransportTripParameterTypeDepartureTime`        | `NSValue` (Int)              | Departure time from first stop (seconds since 1970-01-01T00:00:00)       |
| `PublicTransportTripParameterTypeTripDate`             | `NSValue` (Int)              | Date of the trip (seconds since 1970-01-01T00:00:00)                     |
| `PublicTransportTripParameterTypeTripDelayMinutes`     | `NSValue` (Int)              | Delay in minutes. Not meaningful if `hasRealtime` is false               |
| `PublicTransportTripParameterTypeStopPlatformCode`     | `NSValue` (String)           | Platform code                                                            |
| `PublicTransportTripParameterTypeIndex`                | `NSValue` (Int)              | Trip index                                                               |
| `PublicTransportTripParameterTypeStopIndex`            | `NSValue` (Int)              | Stop index                                                               |
| `PublicTransportTripParameterTypeIsCancelled`          | `NSValue` (Bool)             | Whether the trip is cancelled                                            |
| `PublicTransportTripParameterTypeWheelchairAccessible` | `NSValue` (Int)              | Wheelchair accessibility — see `PublicTransportWheelchairAccessibleType` |
| `PublicTransportTripParameterTypeBikesAllowed`         | `NSValue` (Int)              | Bikes allowed — see `PublicTransportBikesAllowedType`                    |
| `PublicTransportTripParameterTypeAgencyId`             | `NSValue` (Int)              | ID of the associated agency                                              |
| `PublicTransportTripParameterTypeStopTimes`            | `NSArray` of stop time dicts | Details of stop times in the trip                                        |

### Wheelchair accessibility values[​](#wheelchair-accessibility-values "Direct link to Wheelchair accessibility values")

| Value                                           | Description                                       |
| ----------------------------------------------- | ------------------------------------------------- |
| `PublicTransportWheelchairAccessibleTypeNoInfo` | No accessibility information                      |
| `PublicTransportWheelchairAccessibleTypeYes`    | At least one wheelchair rider can be accommodated |
| `PublicTransportWheelchairAccessibleTypeNo`     | No wheelchair riders can be accommodated          |

### Bikes allowed values[​](#bikes-allowed-values "Direct link to Bikes allowed values")

| Value                                   | Description                              |
| --------------------------------------- | ---------------------------------------- |
| `PublicTransportBikesAllowedTypeNoInfo` | No bike information                      |
| `PublicTransportBikesAllowedTypeYes`    | At least one bicycle can be accommodated |
| `PublicTransportBikesAllowedTypeNo`     | No bicycles are allowed                  |

> 🚨 **DANGER**
>
> Route info in stop and trip dictionaries represents the public-facing service riders recognize (e.g., "Bus 42"). A trip is a single scheduled journey along that route at a specific time with its own stop times. The route is the line identity; trips are individual vehicle runs throughout the day.

