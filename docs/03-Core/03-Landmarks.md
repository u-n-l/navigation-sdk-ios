# Landmarks

A **landmark** is a rich point-of-interest entity represented by `LandmarkObject`. It combines location, metadata, media, categories, and store membership. Landmarks represent significant, categorized locations with rich metadata and structured context about a place.

## Landmark structure[​](#landmark-structure "Direct link to Landmark structure")

### Identity and store linkage[​](#identity-and-store-linkage "Direct link to Identity and store linkage")

| Method                         | Description                                       |
| ------------------------------ | ------------------------------------------------- |
| `getLandmarkIdentifier()`      | Returns the landmark ID; `-1` when not in a store |
| `getLandmarkStoreIdentifier()` | Returns the parent store ID                       |
| `detachFromStore()`            | Removes store linkage for local-only handling     |

### Geographic details[​](#geographic-details "Direct link to Geographic details")

A landmark's position is defined by its centroid coordinates. The contour area provides the full boundary when available.

| Method                          | Description                                              |
| ------------------------------- | -------------------------------------------------------- |
| `getCoordinates()`              | Returns the centroid `CoordinatesObject`                 |
| `setCoordinates(_:)`            | Sets the centroid coordinates                            |
| `getContourGeograficArea()`     | Returns contour bounding area (`nil` when not available) |
| `isContourGeograficAreaEmpty()` | Checks if the contour area is absent                     |

Calculate the distance between two landmarks using their coordinates:

```swift
let distanceInMeters = landmarkA.getCoordinates().getDistance(landmarkB.getCoordinates())

```

See the [Base entities](../03-Core/01-Base%20Entities.md) guide for more on `CoordinatesObject`.

### Descriptive information[​](#descriptive-information "Direct link to Descriptive information")

Names adapt to the SDK language setting for localization.

| Method                                                    | Description                         |
| --------------------------------------------------------- | ----------------------------------- |
| `getLandmarkName()` / `setLandmarkName(_:)`               | Landmark name                       |
| `getLandmarkDescription()` / `setLandmarkDescription(_:)` | Landmark description                |
| `getAuthor()` / `setAuthor(_:)`                           | Author string                       |
| `getProviderId()` / `setProviderId(_:)`                   | Provider identifier                 |
| `getTimeStamp()` / `setTimeStamp(_:)`                     | Insertion or modification timestamp |

### Categories[​](#categories "Direct link to Categories")

A landmark can belong to multiple categories simultaneously.

| Method            | Description                                             |
| ----------------- | ------------------------------------------------------- |
| `getCategories()` | Returns `[LandmarkCategoryObject]`, empty array if none |

### Media and images[​](#media-and-images "Direct link to Media and images")

| Method                                  | Description                                           |
| --------------------------------------- | ----------------------------------------------------- |
| `getImage()` / `setImage(_:)`           | Primary `ImageObject`                                 |
| `getExtraImage()` / `setExtraImage(_:)` | Secondary `ImageObject`                               |
| `setImageData(_:format:)`               | Set image from raw `Data` and `ImageFormat`           |
| `getLandmarkImage(_:)`                  | Renders and returns a `UIImage` at the given `CGSize` |
| `getLandmarkImage(_:scale:ppi:)`        | Renders `UIImage` with explicit screen scale and PPI  |
| `setLandmarkImage(_:)`                  | Sets the landmark image from a `UIImage` (PNG format) |

> 📝 **INFO**
>
> `getLandmarkImage(_:)` caches the rendered result after the first call. Call `resetCacheImage` when your landmark image source changes.

### Extra info and contact data[​](#extra-info-and-contact-data "Direct link to Extra info and contact data")

| Method                                    | Description                                              |
| ----------------------------------------- | -------------------------------------------------------- |
| `getExtraInfo()` / `setExtraInfo(_:)`     | Extra metadata as `[String]`                             |
| `findExtraInfo(_:)`                       | Looks up a value in extra info by key string             |
| `getContactInfo()` / `setContactInfo(_:)` | `ContactInfoObject` with phone numbers, emails, and URLs |

> 🚨 **DANGER**
>
> If you retrieve a `ContactInfoObject` or extra info from a landmark, modify it, and want to persist the change, you must call the corresponding setter to update the landmark's stored value.

> 🚨 **DANGER**
>
> The extra info array also stores data for geographic area, contour geographic area, and Wikipedia information. Modifying `extraInfo` without preserving existing entries may cause data loss.

### Address fields[​](#address-fields "Direct link to Address fields")

Use `getAddressFieldNameWithType(_:)` with `AddressSearchFieldType` to retrieve structured address components:

| Field                                                                  | Description                                          |
| ---------------------------------------------------------------------- | ---------------------------------------------------- |
| `.streetName`                                                          | Street or road name                                  |
| `.streetNumber`                                                        | Building or house number                             |
| `.postalCode`                                                          | ZIP or postal code                                   |
| `.settlement`                                                          | Settlement name                                      |
| `.city`                                                                | City or town name                                    |
| `.county`                                                              | County (administrative level between state and city) |
| `.district`                                                            | Municipal district                                   |
| `.state`                                                               | State or province                                    |
| `.stateCode`                                                           | State abbreviation                                   |
| `.country`                                                             | Country name                                         |
| `.countryCode`                                                         | ISO 3166-1 alpha-3 country code                      |
| `.extension`                                                           | Address extension, e.g. flat number                  |
| `.buildingFloor` / `.buildingName` / `.buildingRoom` / `.buildingZone` | Building-level fields                                |
| `.crossing1` / `.crossing2`                                            | Intersecting street names                            |
| `.segmentName`                                                         | Road segment name                                    |

### Overlay and geofence linkage[​](#overlay-and-geofence-linkage "Direct link to Overlay and geofence linkage")

| Method                       | Description                                                          |
| ---------------------------- | -------------------------------------------------------------------- |
| `getOverlayItem()`           | Returns the associated `OverlayItemObject` if from an overlay search |
| `getGeofenceProximityArea()` | Returns `GeofenceProximityAreaObject` if from a geofence search      |

## Create landmarks[​](#create-landmarks "Direct link to Create landmarks")

Create a landmark in memory using the factory method, then set optional metadata:

```swift
let location = CoordinatesObject.coordinates(withLatitude: 48.858844, longitude: 2.294351)
let landmark = LandmarkObject.landmark(withName: "Eiffel Tower", location: location)

landmark.setLandmarkDescription("Iconic iron lattice tower in Paris.")
landmark.setAuthor("My App")

```

> 🚨 **DANGER**
>
> Creating a landmark does not display it automatically. Add it to a map workflow (highlights, landmark store, routing, etc.) to use it.

## Landmark categories[​](#landmark-categories "Direct link to Landmark categories")

Categories are represented by `LandmarkCategoryObject`. A landmark may belong to multiple categories simultaneously.

### Category structure[​](#category-structure "Direct link to Category structure")

| Method                         | Description                                                   |
| ------------------------------ | ------------------------------------------------------------- |
| `init(name:)`                  | Creates a new category with the given name                    |
| `getIdentifier()`              | Returns the category ID                                       |
| `getName()` / `setName(_:)`    | Gets or sets the category name                                |
| `getImage(_:)`                 | Renders the category image as `UIImage` at the given `CGSize` |
| `getImage(_:scale:ppi:)`       | Renders `UIImage` with explicit scale and PPI                 |
| `setImage(_:)`                 | Sets the category image from a `UIImage`                      |
| `getLandmarkStoreIdentifier()` | Returns the owning store ID (`0` if not in a store)           |

### Predefined generic categories[​](#predefined-generic-categories "Direct link to Predefined generic categories")

Access predefined map categories via `GenericCategoriesContext` using `GenericCategoryType`:

| `GenericCategoryType`           | ID   | Description                           |
| ------------------------------- | ---- | ------------------------------------- |
| `.gasStation`                   | 1000 | Fuel stations                         |
| `.parking`                      | 1001 | Parking lots and garages              |
| `.foodAndDrink`                 | 1002 | Restaurants, cafes, bars              |
| `.accommodation`                | 1003 | Hotels, hostels, lodging              |
| `.medicalServices`              | 1004 | Hospitals, clinics, pharmacies        |
| `.shopping`                     | 1005 | Retail stores and markets             |
| `.carServices`                  | 1006 | Auto repair and vehicle services      |
| `.publicTransport`              | 1007 | Transit stops and stations            |
| `.wikipedia`                    | 1008 | POIs with Wikipedia information       |
| `.education`                    | 1009 | Schools and universities              |
| `.entertainment`                | 1010 | Cinemas, theaters, amusement parks    |
| `.publicServices`               | 1011 | Post offices and civic buildings      |
| `.geographicalArea`             | 1012 | Geographic zones and regions          |
| `.business`                     | 1013 | Offices and corporate buildings       |
| `.sightseeing`                  | 1014 | Tourist attractions                   |
| `.religiousPlaces`              | 1015 | Churches, mosques, temples            |
| `.roadside`                     | 1016 | Roadside amenities and rest areas     |
| `.sports`                       | 1017 | Stadiums and fitness facilities       |
| `.uncategorised`                | 1018 | Landmarks with no specific category   |
| `.hydrants`                     | 1019 | Water hydrant locations               |
| `.emergencyServicesSupport`     | 1020 | Emergency service facilities          |
| `.civilEmergencyInfrastructure` | 1021 | Emergency preparedness infrastructure |
| `.chargingStation`              | 1022 | EV charging stations                  |
| `.bicycleChargingStation`       | 1023 | E-bike charging stations              |
| `.bicycleParking`               | 1024 | Bicycle parking areas                 |

Access generic categories using `GenericCategoriesContext`:

```swift
let ctx = GenericCategoriesContext()

// All predefined categories
let all = ctx.getCategories()

// A specific category by type
if let parking = ctx.getCategory(.parking) {
    let name = parking.getName()
    let icon = parking.getImage(CGSize(width: 32, height: 32))
}

// POI subcategories for a generic category
let parkingPois = ctx.getPoiCategories(Int32(GenericCategoryType.parking.rawValue))

if let parkingSubcategory = parkingPois.first {
    
    // get parent POI generic category for a subcategory
    let genericCategory = ctx.getGenericCategory(parkingSubcategory.getIdentifier())
}

```

### Category hierarchy[​](#category-hierarchy "Direct link to Category hierarchy")

Each generic category can contain multiple POI subcategories. For example, *Parking* contains *Park and Ride*, *Parking Garage*, *Parking Lot*, *RV Park*, *Truck Parking*, *Truck Stop*, and *Parking meter*.

Use `getPoiCategories(_:)` to retrieve subcategories, and `getGenericCategory(_:)` to find the parent of a POI subcategory.

> 🚨 **DANGER**
>
> **Important distinction:**
>
> * `getCategory(_:)` — Returns a `LandmarkCategoryObject` by `GenericCategoryType`
> * `getGenericCategory(_:)` — Returns the parent generic category of a POI subcategory

## Landmark stores[​](#landmark-stores "Direct link to Landmark stores")

Landmark stores are persistent collections of landmarks backed by SQLite on the device. Each store has a unique `name` and integer `id`.

> 🚨 **DANGER**
>
> Landmark coordinate precision is limited by floating-point representation, which may produce positional inaccuracies of a few centimeters to meters.

### Manage landmark stores[​](#manage-landmark-stores "Direct link to Manage landmark stores")

`LandmarkStoreContextService` manages store registration, retrieval, and removal.

#### Register a store[​](#register-a-store "Direct link to Register a store")

```swift
let service = LandmarkStoreContextService()
let storeId = service.registerLandmarkStoreContext("MyLandmarkStore", path: "/path/to/store.db")

```

The name must be unique; if a store with that name already exists at the path, the existing store ID is returned.

> 🚨 **DANGER**
>
> Stores persist across app sessions. Registering a name that already exists returns that existing store, which may already contain landmarks and categories.

#### Get a store by ID or name[​](#get-a-store-by-id-or-name "Direct link to Get a store by ID or name")

```swift
let storeById   = service.getLandmarkStoreContext(withIdentifier: storeId)
let storeByName = service.getLandmarkStoreContext(withName: "MyLandmarkStore")

```

#### Get all stores[​](#get-all-stores "Direct link to Get all stores")

```swift
let allStores = service.getStores()

```

#### Remove a store[​](#remove-a-store "Direct link to Remove a store")

```swift
let result = service.removeLandmarkStoreContext(storeId)

```

#### Predefined map stores[​](#predefined-map-stores "Direct link to Predefined map stores")

The SDK provides three built-in read-only stores:

```swift
let poisStoreId    = service.getMapPoisLandmarkStoreId()
let addressStoreId = service.getMapAddressLandmarkStoreId()
let citiesStoreId  = service.getMapCitiesLandmarkStoreId()

```

Use these IDs to determine whether a landmark originated from map data.

> 🚨 **DANGER**
>
> Do not modify predefined stores. They are used for landmark category visibility, origin checks, and search/alarm filtering.

### Store operations[​](#store-operations "Direct link to Store operations")

`LandmarkStoreContext` provides the following operations:

| Method                                                                    | Description                                                                                                               |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `getId()`                                                                 | Returns the store ID                                                                                                      |
| `getName()`                                                               | Returns the store name                                                                                                    |
| `getType()`                                                               | Returns `LandmarkStoreType` (`.none`, `.default`, `.mapAddress`, `.mapPoi`, `.mapCity`, `.mapHighwayExit`, `.mapCountry`) |
| `getFilePath()`                                                           | Returns the on-disk SQLite file path                                                                                      |
| `addCategory(_:)`                                                         | Adds a new category; must have a name                                                                                     |
| `updateCategory(_:)`                                                      | Updates an existing category (must belong to this store)                                                                  |
| `removeCategory(_:)`                                                      | Removes category; its landmarks become uncategorized                                                                      |
| `removeCategoryWithAllContent(_:)`                                        | Removes category and all its landmarks                                                                                    |
| `getCategoryById(_:)`                                                     | Returns `LandmarkCategoryObject?` by ID                                                                                   |
| `getCategories()`                                                         | Returns all categories                                                                                                    |
| `addLandmark(_:)`                                                         | Adds landmark as uncategorized                                                                                            |
| `addLandmark(_:toCategoryId:)`                                            | Adds landmark to a specific category                                                                                      |
| `updateLandmark(_:)`                                                      | Updates landmark info (does not affect its category)                                                                      |
| `removeLandmark(_:)`                                                      | Removes a landmark                                                                                                        |
| `removeLandmark(_:fromCategoryId:)`                                       | Removes landmark from a category                                                                                          |
| `removeAllLandmarks()`                                                    | Removes all landmarks in the store                                                                                        |
| `removeAllLandmarksFromCategoryId(_:)`                                    | Removes all landmarks from a category                                                                                     |
| `getLandmark(_:)`                                                         | Returns `LandmarkObject?` by landmark ID                                                                                  |
| `getLandmarkCount()`                                                      | Returns total landmark count                                                                                              |
| `getLandmarks()`                                                          | Returns all landmarks                                                                                                     |
| `getLandmarkCount(_:)`                                                    | Returns landmark count for a category ID                                                                                  |
| `getLandmarks(_:)`                                                        | Returns landmarks in a category                                                                                           |
| `getLandmarksWithRectangleGeographicArea(_:)`                             | Spatial query by `RectangleGeographicAreaObject`                                                                          |
| `getLandmarksWithGeographicArea(_:)`                                      | Spatial query by any `GeographicAreaObject`                                                                               |
| `importLandmarks(_:format:completionHandler:)`                            | Async import from file                                                                                                    |
| `importLandmarks(_:format:progressHandler:completionHandler:)`            | Async import with progress reporting                                                                                      |
| `importLandmarks(_:format:categoryId:progressHandler:completionHandler:)` | Async import directly into a category                                                                                     |
| `cancelImportLandmarks()`                                                 | Cancels an in-progress import                                                                                             |

### Import landmarks[​](#import-landmarks "Direct link to Import landmarks")

Import from a local file in KML or GeoJSON format:

```swift
let store = service.getLandmarkStoreContext(withName: "MyLandmarkStore")

store.importLandmarks(
    "/path/to/landmarks.kml",
    format: .kml,
    progressHandler: { progress in
        print("Import progress: \(Int(progress * 100))%")
    },
    completionHandler: { errorCode in
        if errorCode == .success {
            // Landmarks added as uncategorized
        }
    }
)

```

To import into a specific category:

```swift
store.importLandmarks(
    "/path/to/landmarks.geojson",
    format: .geoJson,
    categoryId: myCategory.getIdentifier(),
    progressHandler: { _ in },
    completionHandler: { errorCode in }
)

```

> ⚠️ **WARNING**
>
> The `categoryId` must already exist in the target store. Call `addCategory(_:)` first if needed.

## Landmark browse sessions[​](#landmark-browse-sessions "Direct link to Landmark browse sessions")

Use `LandmarkBrowseSessionContext` for efficient paged browsing of large stores.

### Create a browse session[​](#create-a-browse-session "Direct link to Create a browse session")

```swift
let store = service.getLandmarkStoreContext(withIdentifier: storeId)

let settings = LandmarkBrowseSessionSettingsObject()
settings.orderBy = .name
settings.descendingOrder = false
settings.nameFilter = "cafe"

let session = LandmarkBrowseSessionContext(storeId: store.getId(), settings: settings)

```

> 🚨 **DANGER**
>
> Only landmarks present in the store at the time of session creation are included in the session.

### Browse session settings[​](#browse-session-settings "Direct link to Browse session settings")

`LandmarkBrowseSessionSettingsObject` uses the following properties:

| Property           | Type                  | Default                          | Description                                                             |
| ------------------ | --------------------- | -------------------------------- | ----------------------------------------------------------------------- |
| `descendingOrder`  | `Bool`                | `false`                          | Sort direction; `false` = ascending                                     |
| `orderBy`          | `LandmarkObjectOrder` | `.name`                          | Sort criteria: `.name`, `.date`, or `.distance`                         |
| `nameFilter`       | `String?`             | `nil`                            | Filters to landmarks whose name contains this substring                 |
| `categoryIdFilter` | `NSNumber?` (int)     | `-2` (`CategoryIdFilterInvalid`) | Filter by category ID; `-2` = all categories, `-1` = uncategorized only |
| `coordinates`      | `CoordinatesObject?`  | `nil`                            | Reference point used when `orderBy == .distance`                        |

### Browse session operations[​](#browse-session-operations "Direct link to Browse session operations")

| Method                    | Description                                                 |
| ------------------------- | ----------------------------------------------------------- |
| `getId()`                 | Returns the session ID                                      |
| `getLandmarkStoreId()`    | Returns the owning store ID                                 |
| `getLandmarkCount()`      | Total number of landmarks in the session                    |
| `getLandmarksFrom(_:to:)` | Returns landmarks between the given indices (exclusive end) |
| `getLandmarkPos(_:)`      | Returns 0-based index of a landmark by its ID               |
| `getSettings()`           | Returns the current session settings                        |

Retrieve a paginated page of landmarks:

```swift
let total = session.getLandmarkCount()
let page  = session.getLandmarksFrom(0, to: min(20, total))

for landmark in page {
    print(landmark.getLandmarkName())
}

```

## Common uses[​](#common-uses "Direct link to Common uses")

* POI display and details panels
* Search results and route waypoints
* Category-filtered map content
* Persistent user-defined locations

***

Landmarks are the SDK's most complete place model and are the recommended entity when you need searchable, routable, and metadata-rich map objects.
