# Getting started with Search

The UNL Navigation SDK for iOS provides flexible search functionality for finding locations using text queries, coordinates, categories, landmark stores, and overlays:

* **Text search**
    - Perform searches using a text query and geographic coordinates to prioritize results within a specific area
* **Search preferences** 
    - Customize search behavior using fuzzy results, address inclusion, offline-only mode, and distance limits
* **Category-based search** 
    - Filter search results by predefined categories such as fuel stations, parking, or entertainment
* **Proximity search** 
    - Retrieve nearby landmarks without specifying a text query

> ⚠️ **WARNING**
>
> These are simplified code snippets. To make sure the `SearchContext` object works correctly, it must be retained in memory for the duration of the search request. If the `SearchContext` instance is deallocated before the request completes, the completion handler will not be called. Check the examples linked at the end of this page for reference implementations.

## Text search[​](#text-search "Direct link to Text search")

Search by providing text and coordinates. The coordinates act as a location hint so the SDK can prioritize results near the specified area.

```swift
let searchContext = SearchContext()
searchContext.setMaxMatches(40)
searchContext.setAllowFuzzyResults(true)

let location = CoordinatesObject.coordinates(withLatitude: 45.0, longitude: 10.0)

searchContext.search(withQuery: "Paris", location: location) { results in
    if results.isEmpty {
        print("No results")
    } else {
        print("Results count: \(results.count)")
    }
}

```

> 📝 **INFO**
>
> `SearchContext` completion handlers return `[LandmarkObject]` only. If you need to stop an in-flight request before launching another search, call `cancelSearch()`.

You can also restrict the search area to a rectangle or radius using `setLocationHint(_:)`:

```swift
let hint = RectangleGeographicAreaObject(
    location: location,
    horizontalRadius: 2_000,
    verticalRadius: 2_000
)
searchContext.setLocationHint(hint)

```

## Specify preferences[​](#specify-preferences "Direct link to Specify preferences")

Before searching, configure `SearchContext` to control the search behavior.

| Method                               | Description                                                                           |
| ------------------------------------ | ------------------------------------------------------------------------------------- |
| `setMaxMatches(_:)`                  | Sets the maximum number of results returned by the request.                           |
| `setAllowFuzzyResults(_:)`           | Enables approximate matching for partially matching queries.                          |
| `setExactMatch(_:)`                  | Restricts results to exact text matches only.                                         |
| `setSearchAddresses(_:)`             | Includes address results, including roads.                                            |
| `setSearchMapPOIs(_:)`               | Includes map POIs in the result set.                                                  |
| `setSearchOnlyOnboard(_:)`           | Restricts the request to onboard data only.                                           |
| `setThresholdDistance(_:)`           | Sets the maximum lookup distance in meters for nearby searches and reverse geocoding. |
| `setEstimateMissingHouseNumbers(_:)` | Enables house-number interpolation for address search results.                        |
| `setEasyAccessOnlyResults(_:)`       | Restricts results to locations considered easily accessible.                          |
| `setSearchGeofences(_:)`             | Includes geofence search targets in the request.                                      |
| `setLocationHint(_:)`                | Restricts the search to a specific `RectangleGeographicAreaObject`.                   |

### Search by category[​](#search-by-category "Direct link to Search by category")

Filter search results using predefined categories from `GenericCategoriesContext`.

The following example limits results to the **Food & Drink** category.

```swift
let searchContext = SearchContext()
searchContext.setSearchAddresses(false)
searchContext.setSearchMapPOIs(false)

let genericCategories = GenericCategoriesContext()

if let foodAndDrink = genericCategories.getCategory(.foodAndDrink) {
    _ = searchContext.setCategory(foodAndDrink)
}

let location = CoordinatesObject.coordinates(withLatitude: 45.0, longitude: 10.0)

searchContext.searchAround(withLocation: location) { results in
    print("Category results: \(results.count)")
}

```

> 💡 **TIP**
>
> Set `setSearchAddresses(false)` and `setSearchMapPOIs(false)` when you want only category-filtered results.

You can retrieve the full predefined category list with `GenericCategoriesContext().getCategories()`.

### Search on custom landmarks[​](#search-on-custom-landmarks "Direct link to Search on custom landmarks")

By default, `SearchContext` searches the built-in map stores. To search custom landmarks, create or open a `LandmarkStoreContext`, add landmarks to it, and enable that store in the search store collection.

```swift
let customStore = LandmarkStoreContext(name: "searchable-landmarks")

let landmark1 = LandmarkObject.landmark(
    withName: "My Custom Landmark 1",
    location: CoordinatesObject.coordinates(withLatitude: 25.0, longitude: 30.0)
)

let landmark2 = LandmarkObject.landmark(
    withName: "My Custom Landmark 2",
    location: CoordinatesObject.coordinates(withLatitude: 25.005, longitude: 30.005)
)

_ = customStore.addLandmark(landmark1)
_ = customStore.addLandmark(landmark2)

let searchContext = SearchContext()
searchContext.setSearchAddresses(false)
searchContext.setSearchMapPOIs(false)

if let stores = searchContext.getLandmarkStoreCollection() {
    _ = stores.addAllStoreCategories(customStore.getId())
}

let location = CoordinatesObject.coordinates(withLatitude: 25.003, longitude: 30.003)

searchContext.search(withQuery: "My Custom Landmark", location: location) { results in
    print("Custom landmark results: \(results.count)")
}

```

> 🚨 **DANGER**
>
> `LandmarkStoreContext(name:)` persists its data in the app sandbox. Reusing the same store name can reopen an existing store that already contains landmarks and categories. See the [Landmarks guide](../03-Core/03-Landmarks.md#manage-landmark-stores) for details.

> 💡 **TIP**
>
> Set `setSearchAddresses(false)` and `setSearchMapPOIs(false)` when you want only custom-landmark results.

### Search on overlays[​](#search-on-overlays "Direct link to Search on overlays")

Perform searches on overlays by adding overlay identifiers to the search overlay collection.

The example below searches within the built-in safety overlay.

```swift
let searchContext = SearchContext()
searchContext.setSearchAddresses(false)
searchContext.setSearchMapPOIs(false)

if let overlays = searchContext.geOverlayMutableCollection() {
    overlays.add(Int32(CommonOverlayIdentifier.publicTransport.rawValue))
}

let location = CoordinatesObject.coordinates(withLatitude: 48.76930, longitude: 2.34483)

searchContext.search(withQuery: "Louis", location: location) { results in
    if let landmark = results.first,
       let overlayItem = landmark.getOverlayItem() {
        print("Overlay item: \(overlayItem.getName())")
    }
}

```

To convert a returned `LandmarkObject` to an overlay-backed result, use `getOverlayItem()`.

> 💡 **TIP**
>
> Set `setSearchAddresses(false)` and `setSearchMapPOIs(false)` when you want only overlay results.

> 🚨 **DANGER**
>
> Overlay search requires a map style that contains the target overlay. If the overlay is not available in the current style, the request may return no results.

## Search for location[​](#search-for-location "Direct link to Search for location")

Without specifying text, nearby landmarks are returned around the reference location, limited by `setMaxMatches(_:)`.

```swift
let searchContext = SearchContext()
searchContext.setMaxMatches(40)
searchContext.setAllowFuzzyResults(true)

let location = CoordinatesObject.coordinates(withLatitude: 45.0, longitude: 10.0)

searchContext.searchAround(withLocation: location) { results in
    if results.isEmpty {
        print("No results")
    } else {
        print("Nearby results: \(results.count)")
    }
}

```

## Search in a certain area[​](#search-in-a-certain-area "Direct link to Search in a certain area")

Use `searchAround(withLocation:)` together with `setLocationHint(_:)` to find landmarks within a specific rectangular area, ordered by distance from the reference coordinates. Provide a `RectangleGeographicAreaObject` defined by its top-left and bottom-right corners, and set the reference coordinates to a point inside that area.

```swift
let topLeft = CoordinatesObject.coordinates(withLatitude: 67.69866, longitude: 24.81115)
let bottomRight = CoordinatesObject.coordinates(withLatitude: 67.58326, longitude: 25.36093)
let referencePoint = CoordinatesObject.coordinates(withLatitude: 67.63826, longitude: 24.94154)

let area = RectangleGeographicAreaObject(topLeftLocation: topLeft, bottomRightLocation: bottomRight)

let searchContext = SearchContext()
searchContext.setLocationHint(area)

searchContext.searchAround(withLocation: referencePoint) { results in
    if results.isEmpty {
        print("No results")
    } else {
        print("Area results: \(results.count)")
    }
}

```

To additionally filter results by name, use `search(withQuery:location:)` instead of `searchAround(withLocation:)` and pass the text query alongside the reference coordinates:

```swift
searchContext.setLocationHint(area)
searchContext.search(withQuery: "cafe", location: referencePoint) { results in
    print("Filtered area results: \(results.count)")
}

```

> 🚨 **DANGER**
>
> The reference coordinates must be located **within** the `RectangleGeographicAreaObject` provided to `setLocationHint(_:)`. If the reference point is outside the area, the search will return empty results.

## Show results on the map[​](#show-results-on-the-map "Direct link to Show results on the map")

In most use cases, landmarks returned by a search are already rendered on the map as part of the loaded map data. If the search was performed on custom landmark stores, refer to the [display landmarks](../04-Maps/05-Display%20Map%20Items/01-Display%20Landmarks.md#add-custom-landmarks) guide for adding them to the map view.

To center the map on a landmark returned from search, retrieve its coordinates with `getCoordinates()` and pass them to `center(onCoordinates:zoomLevel:animationDuration:)` on your `MapViewController`. See the [adjust map view](../04-Maps/02-Adjust%20Map%20View.md) guide for full centering options.

```swift
let searchContext = SearchContext()
let location = CoordinatesObject.coordinates(withLatitude: 48.8566, longitude: 2.3522)

searchContext.search(withQuery: "Eiffel Tower", location: location) { results in
    guard let landmark = results.first else { return }
    let coords = landmark.getCoordinates()
    // mapViewController is your MapViewController instance
    mapViewController.center(onCoordinates: coords, zoomLevel: 15, animationDuration: 0.5)
}

```

## Change the language of results[​](#change-the-language-of-results "Direct link to Change the language of results")

The language of search results and category names is controlled by the SDK-wide language setting, set via `GEMSdk.shared().setLanguage(_:)`. By default the SDK uses the device's preferred language and region.

The language value uses ISO 639-3 (e.g. `"eng"` for generic English, `"fra"` for French) or ISO 639-1 (e.g. `"en"`, `"fr"`) format, optionally combined with an ISO 3166 region code (e.g. `"eng-USA"`, `"en-US"`).

```swift
GEMSdk.shared().setLanguage("eng-USA")

```

All subsequent search requests will return results in the specified language where the data is available.
