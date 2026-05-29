# Navigation Instructions

The UNL Navigation SDK for iOS provides real-time navigation guidance through `NavigationInstructionObject`. It exposes current street and road details, speed-limit context, remaining time and distance, lane guidance, and upcoming maneuvers during active navigation or simulation.

> 📝 **INFO**
>
> Distinguish between `NavigationInstructionObject` and `RouteInstructionObject`.
>
> * `NavigationInstructionObject`: live, continuously updated turn-by-turn guidance during navigation or simulation
> * `RouteInstructionObject`: route-plan guidance available immediately after route calculation

## Get navigation instructions[​](#get-navigation-instructions "Direct link to Get navigation instructions")

You do not instantiate navigation instructions directly. The SDK provides them only while navigation or simulation is active.

Use `NavigationContext` in two common ways:

* **Pull current state** with `getNavigationInstruction()`
* **React to updates** through `NavigationContextDelegate` and `navigationInstructionUpdatedForRoute:updatedEvents:`

> ⚠️ **WARNING**
>
> Call `getNavigationInstruction()` only when navigation or simulation is running.

### Read the current instruction[​](#read-the-current-instruction "Direct link to Read the current instruction")

```swift
if let instruction = navigationContext.getNavigationInstruction() {
	print(instruction.getNextTurnInstruction())

	if let remaining = instruction.getRemainingTravelTimeDistance() {
		print("Remaining distance: \(remaining.getTotalDistance()) m")
		print("Remaining time: \(remaining.getTotalTime()) s")
	}
}

```

### React to updates from `NavigationContext`[​](#react-to-updates-from-navigationcontext "Direct link to react-to-updates-from-navigationcontext")

```swift
final class NavigationObserver: NSObject, NavigationContextDelegate {
	func navigationContext(
		_ navigationContext: NavigationContext,
		navigationInstructionUpdatedForRoute route: RouteObject,
		updatedEvents events: Int32
	) {
		guard let instruction = navigationContext.getNavigationInstruction() else { return }

		print("Updated events bitmask: \(events)")
		print("Next turn: \(instruction.getNextTurnInstructionFormatted())")
	}
}

```

### Navigation instruction update events[​](#navigation-instruction-update-events "Direct link to Navigation instruction update events")

`updatedEvents` is a bitmask that can contain:

| Value                                                  | Description                |
| ------------------------------------------------------ | -------------------------- |
| `NavigationInstructionUpdateEventNextTurnUpdated`      | Next maneuver data changed |
| `NavigationInstructionUpdateEventNextTurnImageUpdated` | Next-turn image changed    |
| `NavigationInstructionUpdateEventLaneInfoUpdated`      | Lane guidance changed      |

## Understand the structure[​](#understand-the-structure "Direct link to Understand the structure")

| Method                                           | Return type           | Description                                              |
| ------------------------------------------------ | --------------------- | -------------------------------------------------------- |
| `getNavigationStatus()`                          | `NavigationStatus`    | Current navigation or simulation state                   |
| `getCurrentStreetName()`                         | `String`              | Current street name                                      |
| `getCurrentStreetSpeedLimit()`                   | `Double`              | Current street speed limit in m/s (`0` when unavailable) |
| `getCurrentStreetSpeedLimitFormatted()`          | `String`              | Localized speed-limit text                               |
| `getDriveSide()`                                 | `DriveSideType`       | Current drive side                                       |
| `hasNextTurnInfo()`                              | `Bool`                | Whether next-turn information is available               |
| `getNextTurnInstruction()`                       | `String`              | Next maneuver text                                       |
| `getNextTurnDetails()`                           | `TurnDetailsObject?`  | Full next-turn details                                   |
| `hasNextNextTurnInfo()`                          | `Bool`                | Whether next-next-turn information is available          |
| `getNextNextTurnInstruction()`                   | `String`              | Next-next maneuver text                                  |
| `getNextNextTurnDetails()`                       | `TurnDetailsObject?`  | Full next-next-turn details                              |
| `getTimeDistanceToNextTurn()`                    | `TimeDistanceObject?` | Time and distance to next turn                           |
| `getTimeDistanceToNextNextTurn()`                | `TimeDistanceObject?` | Time and distance to next-next turn                      |
| `getRemainingTravelTimeDistance()`               | `TimeDistanceObject?` | Remaining route metrics                                  |
| `getRemainingTravelTimeDistanceToNextWaypoint()` | `TimeDistanceObject?` | Remaining metrics to next waypoint                       |
| `getTraveledTimeDistance()`                      | `TimeDistanceObject?` | Traveled time and distance                               |
| `getSegmentIndex()` / `getInstructionIndex()`    | `Int32`               | Current segment and instruction indices                  |
| `getCurrentPosition()`                           | `PositionObject?`     | Current position snapshot                                |

## Turn and maneuver details[​](#turn-and-maneuver-details "Direct link to Turn and maneuver details")

### Next turn[​](#next-turn "Direct link to Next turn")

Use the next-turn APIs to populate maneuver UI with text, images, and detailed turn metadata:

```swift
if instruction.hasNextTurnInfo() {
	let nextTurnText = instruction.getNextTurnInstruction()
	let formattedText = instruction.getNextTurnInstructionFormatted()
	let turnDetails = instruction.getNextTurnDetails()
	let turnImage = instruction.getNextTurnImage(CGSize(width: 96, height: 96))

	print(nextTurnText)
	print(formattedText)
	print(turnDetails?.getRoundaboutExitNumber() ?? -1)
	print(turnImage != nil)
}

```

You can also render customized turn images with the overloads that accept active and inactive colors.

### Next-next turn[​](#next-next-turn "Direct link to Next-next turn")

Use the same pattern for the turn after the next one:

```swift
if instruction.hasNextNextTurnInfo() {
	let nextNextTurnText = instruction.getNextNextTurnInstruction()
	let nextNextDetails = instruction.getNextNextTurnDetails()
	let nextNextImage = instruction.getNextNextTurnImage(CGSize(width: 96, height: 96))

	print(nextNextTurnText)
	print(nextNextDetails?.getRoundaboutExitNumber() ?? -1)
	print(nextNextImage != nil)
}

```

> 📝 **INFO**
>
> `hasNextNextTurnInfo()` is typically `false` when the next instruction is the destination.

## Street and road information[​](#street-and-road-information "Direct link to Street and road information")

Use these APIs to populate current, next, and next-next street panels.

```swift
let currentStreetName = instruction.getCurrentStreetName()
let currentCountryCode = instruction.getCurrentCountryCodeISO()
let driveSide = instruction.getDriveSide()

print(currentStreetName)
print(currentCountryCode)
print(driveSide)

if instruction.hasCurrentRoadInfo() {
	for road in instruction.getCurrentRoadInformation() {
		print(road.getRoadName())
	}
}

let currentRoadCodeImage = instruction.getCurrentRoadCodeImage(CGSize(width: 120, height: 36))
let nextStreetName = instruction.getNextStreetName()
let nextCountryCode = instruction.getNextCountryCodeISO()

if instruction.hasNextRoadInfo() {
	let nextRoadCodeImage = instruction.getNextRoadCodeImage(CGSize(width: 120, height: 36))
	print(nextStreetName)
	print(nextCountryCode)
	print(nextRoadCodeImage != nil)
}

if instruction.hasNextNextRoadInfo() {
	let nextNextStreetName = instruction.getNextNextStreetName()
	let nextNextRoadCodeImage = instruction.getNextNextRoadCodeImage(CGSize(width: 120, height: 36))
	print(nextNextStreetName)
	print(nextNextRoadCodeImage != nil)
}

```

> 📝 **INFO**
>
> Some roads may not have a street name. In that case, `getCurrentStreetName()` returns an empty string.

## Signpost guidance[​](#signpost-guidance "Direct link to Signpost guidance")

When signpost data is available, you can show both text and a rendered sign image:

```swift
if instruction.hasSignpostInfo() {
	let signpostText = instruction.getSignpostInstruction()
	let signpostDetails = instruction.getSignpostDetails()
	let signpostImage = instruction.getSignpostImage(
		CGSize(width: 320, height: 96),
		border: 2,
		roundCorners: true,
		rows: 2
	)

	print(signpostText)
	print(signpostDetails?.getItems().count ?? 0)
	print(signpostImage != nil)
}

```

## Get speed limit information[​](#get-speed-limit-information "Direct link to Get speed limit information")

`NavigationInstructionObject` exposes the current street speed limit and formatted text.

```swift
let speedLimitMS = instruction.getCurrentStreetSpeedLimit()
let speedLimitText = instruction.getCurrentStreetSpeedLimitFormatted()

if speedLimitMS == 0 {
	print("Speed limit unavailable")
} else {
	print("Speed limit: \(speedLimitText)")
}

```

## Display lane guidance[​](#display-lane-guidance "Direct link to Display lane guidance")

Lane guidance is rendered directly as `UIImage` in iOS:

```swift
if instruction.hasLaneInfo() {
	let laneImage = instruction.getLaneImage(
		CGSize(width: 400, height: 120),
		backgroundColor: .black,
		activeColor: .white,
		inactiveColor: .lightGray
	)

	print(laneImage != nil)
}

```

![Example of lane guidance image](../assets/images/ios_core_lanes-5dd44558ff5fa590b9dd0afe7d5a9c79.png "Example of lane guidance image")

**Example of lane guidance image**

## Return-to-route support[​](#return-to-route-support "Direct link to Return-to-route support")

When navigation status is `NavigationStatusWaitingReturnToRoute`, use the return-to-route APIs to guide the user back to the navigation path.

```swift
if instruction.getNavigationStatus() == .waitingReturnToRoute {
	let returnPosition = instruction.getReturnToRoutePosition()
	let returnImage = instruction.getReturnToRouteImage(
		CGSize(width: 96, height: 96),
		colorActiveInner: .systemBlue,
		colorActiveOuter: .white,
		colorInactiveInner: .lightGray,
		colorInactiveOuter: .darkGray
	)

	print("\(returnPosition.latitude), \(returnPosition.longitude)")
	print(returnImage != nil)
}

```

## Change instruction language[​](#change-instruction-language "Direct link to Change instruction language")

Navigation instruction texts follow the language configured in the SDK. The same language setting affects the strings returned by methods such as `getNextTurnInstruction()`, `getSignpostInstruction()`, and `getCurrentStreetSpeedLimitFormatted()`.

***

Use `NavigationInstructionObject` as the live UI model for maneuver text, images, street and road info, signposts, lane guidance, and progress metrics during navigation.
