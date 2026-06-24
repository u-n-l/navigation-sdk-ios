# Projections

Learn how to create projection objects and convert between coordinate systems using `ProjectionContext`.

## Supported projection types[​](#supported-projection-types "Direct link to Supported projection types")

* `WGS84` - World Geodetic System 1984
* `GK` - Gauss-Kruger
* `UTM` - Universal Transverse Mercator
* `LAM` - Lambert
* `BNG` - British National Grid
* `MGRS` - Military Grid Reference System


## WGS84 projection[​](#wgs84-projection "Direct link to WGS84 projection")

The **WGS84** projection is a widely used geodetic datum that serves as the foundation for GPS and other mapping systems.

Create a `WGS84` projection using coordinates:

```swift
let wgs = ProjectionWGS84Object(
    coordinates: CoordinatesObject.coordinates(withLatitude: 51.5074, longitude: -0.1278)
)

```

Access and modify coordinates using the getter and setter:

```swift
if let coords = wgs.getCoordinates() {
    print("latitude=\(coords.getLatitude()) longitude=\(coords.getLongitude())")
}

wgs.setCoordinates(
    CoordinatesObject.coordinates(withLatitude: 10.0, longitude: 10.0)
)

```

## GK projection[​](#gk-projection "Direct link to GK projection")

The **Gauss-Kruger** projection is a cylindrical map projection commonly used for large-scale mapping in regions with a north-south orientation. It divides the Earth into zones, each with its own coordinate system.

Create a `Gauss-Kruger` projection:

```swift
let gk = ProjectionGKObject(x: 6325113.72, y: 5082540.66, zone: 1)

```

Access and modify values using getter and setter methods:

```swift
let easting = gk.getEasting()
let northing = gk.getNorthing()
let zone = gk.getZone()

print("easting=\(easting) northing=\(northing) zone=\(zone)")

gk.setX(1, setY: 1, zone: 2)

```

> 🚨 **DANGER**
>
> The `Gauss-Kruger` projection is currently supported only for countries that use the **Bessel ellipsoid**. Converting to and from `Gauss-Kruger` projection for other countries will result in an `SDKErrorCode` error.

## BNG projection[​](#bng-projection "Direct link to BNG projection")

The **BNG** (British National Grid) projection is a coordinate system used in Great Britain for mapping and navigation. It provides a grid reference system for precise location identification.

Create a `BNG` projection using easting and northing:

```swift
let bng = ProjectionBNGObject(easting: 500000, northing: 4649776)

```

Or create from a grid reference string:

```swift
let bng2 = ProjectionBNGObject(gridReference: "TL56")

```

Access and modify values:

```swift
let easting = bng.getEasting()
let northing = bng.getNorthing()
let gridRef = bng.getGridReference()

print("easting=\(easting) northing=\(northing) grid=\(gridRef)")

bng.setEasting(1, northing: 1)
bng.setGridReference("SJ23")

```

## MGRS projection[​](#mgrs-projection "Direct link to MGRS projection")

The **MGRS** (Military Grid Reference System) projection is a coordinate system used by the military for precise location identification. It combines the UTM and UPS coordinate systems.

Create a `MGRS` projection:

```swift
let mgrs = ProjectionMGRSObject(easting: 99316, northing: 10163, zone: "30U", letters: "XC")

```

Access and modify values:

```swift
let easting = mgrs.getEasting()
let northing = mgrs.getNorthing()
let zone = mgrs.getZone()
let sq100k = mgrs.getSq100kIdentifier()

print("easting=\(easting) northing=\(northing) zone=\(zone) sq100k=\(sq100k)")

mgrs.setEasting(1, northing: 1, zone: "B", letters: "AB")

```


## LAM projection[​](#lam-projection "Direct link to LAM projection")

The **LAM** (Lambert) projection is a conic map projection commonly used for large-scale mapping in regions with an east-west orientation.

Create a `LAM` projection:

```swift
let lam = ProjectionLAMObject(x: 6325113.72, y: 5082540.66)

```

Access and modify coordinates:

```swift
let x = lam.getX()
let y = lam.getY()

print("x=\(x) y=\(y)")

lam.setX(1, setY: 1)

```

## UTM projection[​](#utm-projection "Direct link to UTM projection")

The **UTM** (Universal Transverse Mercator) projection is a global map projection that divides the world into a series of zones, each with its own coordinate system.

Create a `UTM` projection:

```swift
let utm = ProjectionUTMObject(x: 6325113.72, y: 5082540.66, zone: 1, hemisphere: .north)

```

Access and modify values:

```swift
let x = utm.getX()
let y = utm.getY()
let zone = utm.getZone()
let hemisphere = utm.getHemisphere()

print("x=\(x) y=\(y) zone=\(zone) hemisphere=\(hemisphere)")

utm.setX(1, setY: 1, zone: 2, hemisphere: .south)

```

## Convert between projections[​](#convert-between-projections "Direct link to Convert between projections")

`ProjectionContext.convert(_:to:completionHandler:)` converts from one projection object to another.

```swift
let context = ProjectionContext()
let from = ProjectionWGS84Object(
    coordinates: CoordinatesObject.coordinates(withLatitude: 51.5074, longitude: -0.1278)
)
let to = ProjectionMGRSObject(easting: 0, northing: 0, zone: "", letters: "")

let code = context.convert(from, to: to) { result in
    print("conversion result = \(result)")
    print("zone=\(to.getZone()) easting=\(to.getEasting()) northing=\(to.getNorthing())")
}

print("operation started with code = \(code)")

```

> 🚨 **DANGER**
>
> `ProjectionContext.convert` works with `ProjectionW3WObject` only if the object has a **valid** token that can be obtained from [what3words.com](https://developer.what3words.com/public-api). If the token is not set or invalid, the conversion will fail and return `SDKErrorCodeKNotSupported`.

