# Create your first application

Follow this tutorial to build an iOS app with an interactive map.

> 📝 **INFO** What you need
>
>* Xcode installed
>* Service key from UNL - via UNL Platform

## Step 1: Create a new project[​](#step-1-create-a-new-project "Direct link to Step 1: Create a new project")

1. Open Xcode and select **File > New > Project**
2. Choose **App** under the iOS tab
3. Set your product name (e.g. `HelloMap`), interface to **Storyboard** or **SwiftUI**, and language to **Swift**
4. Click **Create**

> 📝 **INFO** 
>
> Using an existing project?
> Skip to Step 2 and add the SDK to your existing app.

## Step 2: Install the SDK[​](#step-2-install-the-sdk "Direct link to Step 2: Install the SDK")

1. In Xcode, go to **File > Swift Packages > Add Package Dependency**
2. Paste the repository URL: **{TODO}**
3. Select the latest stable version and click **Add Package**

> ⚠️ **WARNING**
>
> Complete the full [SDK integration steps](../02-Get%20Started/01-Integrate%20SDK.md) if you haven't already.

## Step 3: Write the code[​](#step-3-write-the-code "Direct link to Step 3: Write the code")

> 🚨 **DANGER**
>
>The `projectApiToken` global variable is used here for simplicity. If you are going to commit your code to version control, make sure to store your API token securely.

* UIKit
* SwiftUI

Open `AppDelegate.swift` and add this to the code:

```swift
// UIKit

import UIKit
import GEMKit

let projectApiToken = "YOUR_API_TOKEN_HERE"

class MapExceptionsHandler: NSObject, GEMSdkExceptions {
    func onSdkActivationDetails(_ reason: ActivationAboutToExpireType, remainingTime: Int) {
        print("Activation expiring: \(remainingTime)s remaining")
    }
}

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    private let exceptionsHandler = MapExceptionsHandler()

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        let parameters = GEMSdkParameters(exceptions: exceptionsHandler)
        parameters.activationToken = projectApiToken

        GEMSdk.shared().initSdk(with: parameters) { error in
            if error == .kNoError {
                print("SDK initialized successfully")
            } else {
                print("SDK init failed: \(error.rawValue)")
            }
        }

        return true
    }
}

```

Then open `ViewController.swift` and replace everything with this:

```swift
// UIKit

import UIKit
import GEMKit

class ViewController: UIViewController {
    private var mapViewController: MapViewController?

    override func viewDidLoad() {
        super.viewDidLoad()

        let controller = MapViewController()
        controller.view.backgroundColor = .systemBackground
        mapViewController = controller

        addChild(controller)
        view.addSubview(controller.view)
        controller.didMove(toParent: self)

        controller.view.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            controller.view.topAnchor.constraint(equalTo: view.topAnchor),
            controller.view.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            controller.view.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            controller.view.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        ])
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        mapViewController?.startRender()
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        mapViewController?.stopRender()
    }

    deinit {
        mapViewController?.destroy()
    }
}

```

Open `HelloMapApp.swift` (or your main App file) and replace everything with this:

```swift
//SwiftUI

import SwiftUI
import GEMKit

let projectApiToken = "YOUR_API_TOKEN_HERE"

class MapExceptionsHandler: NSObject, GEMSdkExceptions {
    func onSdkActivationDetails(_ reason: ActivationAboutToExpireType, remainingTime: Int) {
        print("Activation expiring: \(remainingTime)s remaining")
    }
}

@main
struct HelloMapApp: App {
    private let exceptionsHandler = MapExceptionsHandler()

    init() {
        let parameters = GEMSdkParameters(exceptions: exceptionsHandler)
        parameters.activationToken = projectApiToken

        GEMSdk.shared().initSdk(with: parameters) { error in
            if error == .kNoError {
                print("SDK initialized successfully")
            } else {
                print("SDK init failed: \(error.rawValue)")
            }
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

```

Then open `ContentView.swift` and replace everything with this:

```swift
// SwiftUI

import SwiftUI
import GEMKit

struct ContentView: View {
    var body: some View {
        MapReader { proxy in
            MapBase()
                .ignoresSafeArea()
        }
    }
}

```

Paste your Service Key in place of `YOUR_API_TOKEN_HERE`, in between the quotes.

### Understanding the code

* **`GEMSdkParameters`** — Configuration object for SDK initialization; requires a `GEMSdkExceptions` handler
* **`activationToken`** — Your ServiceKey for SDK authentication and auto-activation
* **`initSdk(with:completionHandler:)`** — Initializes the SDK; the completion handler fires when initialization finishes
* **`MapViewController`** — UIKit view controller that displays the interactive map
* **`MapBase`** / **`MapReader`** — SwiftUI views for displaying and interacting with the map
* **`destroy()`** — Frees map resources when the view controller is deallocated

## Step 4: Run your app[​](#step-4-run-your-app "Direct link to Step 4: Run your app")

1. Select a simulator or connected device in Xcode
2. Press **Cmd + R** to build and run

> 📝 **INFO**
>
> When you initialize the SDK with a valid API key, it also performs an automatic activation. This allows better flexibility for licensing.

## What's next?[​](#whats-next "Direct link to What's next?")

Explore what you can do with the map:

* [Draw shapes](/docs/ios/examples/maps-3dscene/shapes.md) — Add polygons, lines, and circles 
* [Change map styles](/docs/ios/examples/maps-3dscene/map-style.md) — Switch between light, dark, and custom themes 
* [Search for places](/docs/ios/examples/places-search.md) — Find addresses and points of interest
* [Add navigation](/docs/ios/examples/routing-navigation.md) — Calculate routes and get turn-by-turn directions 


## Troubleshooting[​](#troubleshooting "Direct link to Troubleshooting")

### SDK initialization returns an error

Check the `SDKErrorCode` returned by `initSdk(with:completionHandler:)`:

* `.kNoError` — Success
* `.kActivation` — Activation required; verify your API token
* Other codes — Check your network connection and token validity

### How to check if my Service Key is valid

Add this code to verify your token:

```swift
GEMSdk.shared().verifySdk(projectApiToken) { status in
    switch status {
    case .valid:
        print("✓ Service key is valid")
    case .expired:
        print("✗ Service key expired")
    case .accessDenied:
        print("✗ Access denied")
    default:
        print("✗ Verification failed: \(status.rawValue)")
    }
}

```

### Map shows a watermark

This means your Service key is missing or invalid.

**Without a valid Service key:**

* ❌ Map downloads won't work
* ❌ Map updates disabled
* ⚠️ Watermark appears
* ⚠️ Limited features

**Fix:** Ensure you set a valid `activationToken` on `GEMSdkParameters` before calling `initSdk(with:completionHandler:)`.
