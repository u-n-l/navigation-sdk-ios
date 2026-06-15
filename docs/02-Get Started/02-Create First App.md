# Create your first application

Follow this tutorial to build an iOS app with an interactive map.

> 📝 **INFO** What you need
>
>* Xcode installed
>* iOS 17.0 or higher
>* Service key from UNL - via UNL Platform

> 📝 **INFO** Simulator vs Physical device
>* **Simulator** - Usually works with Embed and Sign only
>* **Physical iPhone / TestFlight / App Store** - Complete Step 2b below {TODO - add link}.
>
> ⚠️ **WARNING**
>
> Do not copy separate shell scripts into your app repo.

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

1. `UnlNavigationSdk` is distributed as a prebuilt `XCFramework`, **not** a Swift Package.
2. Your UNL SDK package should contain one file:
`UnlNavigationSdk-<Environment>.xcframework`
    - The xcframework includes `unl_embed_fixup` at its root for device builds. **Do not** copy other UNL maintainer scripts into your app project.

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
import UnlNavigationSdk

let projectApiToken = "YOUR_API_TOKEN_HERE"

@main
class AppDelegate: UIResponder, UIApplicationDelegate, UnlNavigationSdkDelegate {

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        UnlNavigationSdkService.shared.delegate = self

        Task {
            let result = await UnlNavigationSdkService.shared.initialize(
                token: projectApiToken,
                language: Locale.current.identifier
            )

            if !result.isSuccess {
                #if DEBUG
                print("UnlNavigationSdk init failed: \(result)")
                #endif
            }
        }

        return true
    }

    func sdkServiceAuthorizationKeyRejected(_ service: UnlNavigationSdkService) {
        print("Authorization key rejected")
    }

    func sdkServiceAuthorizationKeyUpdated(_ service: UnlNavigationSdkService) {
        print("Authorization key updated")
    }
}

```

Then open `ViewController.swift` and replace everything with this:

```swift
// UIKit

import UIKit
import UnlNavigationSdk

class ViewController: UIViewController {
    private var mapController: UnlMapViewController?

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        Task { @MainActor in
            await showMapWhenReady()
        }
    }

    @MainActor
    private func showMapWhenReady() async {
        if !UnlNavigationSdkService.shared.isInitialized {
            let result = await UnlNavigationSdkService.shared.initialize(token: projectApiToken)
            guard result.isSuccess else {
                print("Initialization failed: \(result)")
                return
            }
        }

        let map = UnlNavigationSdkService.shared.makeMapController(
            configuration: UnlMapConfiguration(
                initialCoordinate: UnlCoordinatesObject(latitude: 52.5200, longitude: 13.4050),
                initialZoomLevel: 12,
                showsCompass: true,
                showsLogo: true,
                startsRendering: true
            )
        )
        mapController = map

        addChild(map)
        view.addSubview(map.view)
        map.view.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            map.view.topAnchor.constraint(equalTo: view.topAnchor),
            map.view.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            map.view.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            map.view.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        ])
        map.didMove(toParent: self)
    }

    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        guard isMovingFromParent || isBeingDismissed else { return }
        mapController?.destroyMap()
        mapController = nil
    }
}

```
> 📝 **NOTE:** 
>
> Do not call destroyMap() from deinit. It is MainActor-isolated; use viewDidDisappear when the screen is removed.

Open `HelloMapApp.swift` (or your main App file) and replace everything with this:

```swift
//SwiftUI

import SwiftUI
import UnlNavigationSdk

let projectApiToken = "YOUR_API_TOKEN_HERE"

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?

    func sceneDidBecomeActive(_ scene: UIScene) {
        let screen = (scene as? UIWindowScene)?.screen ?? .main
        UnlNavigationSdkService.shared.appDidBecomeActive(on: screen)
    }

    func sceneDidEnterBackground(_ scene: UIScene) {
        UnlNavigationSdkService.shared.appDidEnterBackground()
    }
}


@main
struct HelloMapApp: App {
    @State private var isSDKReady = false

    init() {
        Task {
            let result = await UnlNavigationSdkService.shared.initialize(token: projectApiToken)
            await MainActor.run {
                isSDKReady = result.isSuccess
                if !result.isSuccess {
                    #if DEBUG
                    print("UnlNavigationSdk init failed: \(result)")
                    #endif
                }
            }
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView(isSDKReady: isSDKReady)
        }
    }
}

```

Then open `ContentView.swift` and replace everything with this:

```swift
// SwiftUI

import SwiftUI
import UnlNavigationSdk

struct ContentView: View {
    let isSDKReady: Bool

    var body: some View {
        Group {
            if isSDKReady {
                UnlNavigationSdkService.shared.makeSwiftUIMap(
                    configuration: UnlMapConfiguration(
                        initialCoordinate: UnlCoordinatesObject(latitude: 52.5200, longitude: 13.4050),
                        initialZoomLevel: 12
                    )
                )
                .ignoresSafeArea()
            } else {
                ProgressView("Starting map...")
            }
        }
    }
}

```

Paste your Service Key in place of `YOUR_API_TOKEN_HERE`, in between the quotes.

### Understanding the code

* **`UnlNavigationSdkService.shared`** — Singleton entry point
* **`initialize(token:language:)`** — Initializes the UNL SDK before maps
* **`UnlNavigationSdkDelegate`** — Optional license key callbacks
* **`makeMapController(configuration:)`** — UIKit map as UnlMapViewController
* **`makeSwiftUIMap(configuration:)`** — SwiftUI map view
* **`UnlMapConfiguration`** - Initial center, zoom, and display options
* **`destroyMap()`** — Frees map resources when the view controller is deallocated

## Step 4: Run your app[​](#step-4-run-your-app "Direct link to Step 4: Run your app")

1. Select a simulator or connected device in Xcode
2. Press **Cmd + R** to build and run

> 📝 **INFO**
>
> When you initialize the SDK with a valid API key, it also performs an automatic activation. This allows better flexibility for licensing.
