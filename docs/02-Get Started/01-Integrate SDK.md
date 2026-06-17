# Integrate the SDK

The UNL Navigation SDK for iOS is distributed as a Swift Package hosted on GitHub, making it easy to integrate directly into your Xcode projects using Swift Package Manager (SPM).

## Adding the SDK to Your Project[​](#adding-the-sdk-to-your-project "Direct link to Adding the SDK to Your Project")

To integrate the UNL Navigation SDK into your iOS application:

1. **Open your project in Xcode**

2. **Add the *UnlNavigationSdk* to the project and prepare device builds**

2. **Add the xcframework**
   - Drag `UnlNavigationSdk-<Environment>.xcframework` into your Xcode project.

   - Enable **Copy items if needed** and select your **app target**.

   - Open **General > Frameworks, Libraries, and Embedded Content**.

   - Set **Embed and Sign** for the xcframework.

   - In Swift: `import UnlNavigationSdk`

3. **Build using `Cmd+B`**

   > ⚠️ **WARNING:** If you see **No such module UnlNavigationSdk**, the framework is not linked or not embedded correctly.

### **Device and Release Builds (Run Script)[](#device-and-release-builds "Device and Release Builds (Run Script)")**
   > 📝 **NOTE:** If you are testing in the simulator only, this step can be skipped.

   - Select your **app target -> Build Phases**
   - Click **+ -> New Run Script Phase**
   - Name the phase **Prepare UNL SDK for Device**
   - Drag theis phase **below Embed Frameworks** (Order matters)
   - Uncheck **Based on dependency analysis**
   - Set **User Script Sandboxing** to **No** in **Build Settings**
   - Paste this script:

```swift
set -eu
UNL_XCFRAMEWORK=""

for candidate in "${SRCROOT}"/*.xcframework "${SRCROOT}"/*/*.xcframework "${SRCROOT}"/*/*/*.xcframework; do
    [ -e "${candidate}" ] || continue
    case "$(basename "${candidate}")" in
       UnlNavigationSdk*.xcframework | UnlNavigationSdk.xcframework) ;;
       *) continue ;;
    esac
    if [ -x "${candidate}/unl_embed_fixup" ]; then
       UNL_XCFRAMEWORK="${candidate}"
       break
    fi
done

if [ -z "${UNL_XCFRAMEWORK}" ]; then
    echo "error: UnlNavigationSdk xcframework with unl_embed_fixup not found under ${SRCROOT}" >&2
    exit 1
fi

exec "${UNL_XCFRAMEWORK}/unl_embed_fixup"
```

   > 📝 **NOTE:** 
   >
   > Do not copy `prepare_unl_sdk_for_device.sh` or other UNL maintainer scripts into your app. The fixup tool `unl_embed_fixup` is bundled at the root of the xcframework you receive from UNL.
   >
   > Older SDK drops without `unl_embed_fixup` in the xcframework require an updated xcframework from UNL.

### **Initialize once**
   * Replace "<token>" with your actual Service Key.

```swift
let result = await UnlNavigationSdkService.shared.initialize(
   token: "<token>",
   language: Locale.current.identifier
)
guard result.isSuccess else {/* handle failure */ }

```
Optionally, token can be verified before init:

```swift
let status = await UnlNavigationSdkService.shared.verify(token: "<token>")

```

### **Verify the build log**
   * After an iphoneos build or archive, search the log for:

> `Preparing UnlNavigationSdk for device...` 
>
> `Published UnlMapEngine.framework.dSYM for archive.`
>
> `UnlNavigationSdk device preparation complete.`

<br/>

> 🚨 **DANGER:** 
>
> If the first and last lines are missing, the script did not run (wrong phase order, sandboxing is still enabled, or xcframework path not found).

### **Verify the app bundle**

   * In the built **.app -> Frameworks:** 
      - `UnlNavigationSdk.framework`
      - `UnlMapEngine.framework` (Private map engine runtime, prepared by `unl_embed_fixup`)

> 📝 **NOTE:** 
>
> There **should not** be any nested `UnlNavigationSdk.framework/Frameworks` folder in the final app.

### Why this step is required

> `UnlNavigationSdk` ships with a **private bundled runtime** inside the framework. With **Embed and Sign**, Xcode signs the outer framework but on device often does not correctly re-sign the nested runtime. iOS may then refuse to load the SDK.
>
> The `unl_embed_fixup` tool runs after embedding and:
1. Moves the private runtime to the correct location in your app bundle.
2. Updates load paths inside UnlNavigationSdk.
3. Re-signs components with your app signing identity.

<br/>

> 📝 **NOTE:** 
>
> Link only `UnlNavigationSdk.xcframework`. **Do not** add any other map or navigation xcframework to your app target.
