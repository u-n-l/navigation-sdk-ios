# Display route instructions

Learn how to focus the map on route instructions and show maneuver arrows during route visualization.

## Center on route instructions[​](#center-on-route-instructions "Direct link to Center on route instructions")

Get instructions from route segments, then center on the desired instruction. A simplified example is shown below. For more details on obtaining a route's instructions see the [Get the route segments and instructions](../../07-Routing/01-Get%20Started%20with%20Routing.md#retrieve-route-instructions)
section.

```swift
if let firstSegment = route.getSegments().first {
    if let instruction = firstSegment.getInstructions().first {
        mapViewController.center(onRouteInstruction: instruction, zoomLevel: 75, animationDuration: 700)
    }
}

```

## Remove instruction[​](#remove-instruction "Direct link to Remove instruction")

The route instruction arrow is automatically cleared when a new route instruction is centered on or when the route is cleared. If you want to clear the instruction manually, center on an empty instruction object, like this:

```swift
mapViewController.center(onRouteInstruction: RouteInstructionObject.init(), zoomLevel: -1, animationDuration: 0)

```
