# Add voice guidance

Enhance navigation experiences with spoken instructions. This guide covers enabling the built-in `SoundContext`, managing voice settings, switching voices and languages, and integrating custom TTS playback using `AVSpeechSynthesizer`.

## What you need[​](#what-you-need "Direct link to What you need")

The UNL Navigation SDK for iOS provides two options for instruction playback:

* **Built-in solutions** - Playback using human voice recordings or device Text-to-Speech via `SoundContext`
* **External integration** - Delivery of `SoundObject` via `NavigationContextDelegate` for use with third-party audio frameworks

The built-in solution provides automatic audio session management via `AVFoundation`, ducking other playbacks (such as music) while instructions play.

## Step 1: Create and configure SoundContext[​](#step-1-create-and-configure-soundcontext "Direct link to Step 1: Create and configure SoundContext")

Create a `SoundContext` instance and store it as a property to keep it alive:

```swift
var soundContext: SoundContext?

func configureSoundContext() {
    soundContext = SoundContext()
    soundContext?.setVolume(7) // 0-10 range
}

```

Use `setVolume(_:)` and `getVolume()` to check or change the playback volume.

> ⚠️ **WARNING**
>
> Ensure a valid TTS voice is configured, volume is set to a positive value, and `canPlayNavigationSoundForRoute` returns `true` in the delegate to automatically play voice instructions.

> 💡 **TIP**
>
> By default, device TTS uses the current locale via `AVSpeechSynthesisVoice`. Call `setUseTtsWithCompletionHandler(_:)` to activate it automatically.

**Limitations:**

Customizing the timing of TTS instructions is not supported. Filtering TTS instructions based on custom logic is not available.

## Step 2: Configure the SoundContext[​](#step-2-configure-the-soundcontext "Direct link to Step 2: Configure the SoundContext")

The `SoundContext` manages voice playback with the following features:

| Method / Property                                        | Description                                                                            |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `setUseTtsWithCompletionHandler(_:)`                     | Activates TTS using the device's current locale (async)                                |
| `setUseTtsWithLanguage(_:completionHandler:)`            | Activates TTS with a specific BCP 47 language code, e.g. `"en-US"` (async)             |
| `setUseTtsWithLanguage(_:identifier:completionHandler:)` | Activates TTS with language and a specific `AVSpeechSynthesisVoice` identifier (async) |
| `setUseHumanVoiceWithIdentifier(_:completionHandler:)`   | Activates a downloaded human voice by identifier (async)                               |
| `playText(_:)`                                           | Plays a given TTS text string (computer voices only)                                   |
| `playSound(_:)`                                          | Plays a `SoundObject` delivered by the navigation delegate                             |
| `setVolume(_:)` / `getVolume()`                          | Gets or sets volume level (0-10)                                                       |
| `isPlaying`                                              | Returns whether a sound is currently playing                                           |
| `cancel()`                                               | Cancels ongoing sound playback                                                         |
| `getTtsVoiceName()`                                      | Returns the TTS voice name based on device locale                                      |
| `updateSession(withAudioCategory:)`                      | Updates the AVAudioSession category                                                    |
| `updateSession(withAudioOutput:)`                        | Updates the audio output route                                                         |

## Step 3: Select and configure voices[​](#step-3-select-and-configure-voices "Direct link to Step 3: Select and configure voices")

The SDK provides voices for each supported language. Download and activate voices to deliver navigation prompts such as turn instructions, warnings, and announcements.

### Voice types[​](#voice-types "Direct link to Voice types")

The SDK offers two types of voice guidance:

* **Human voices** - Pre-recorded human voices delivering instructions in a natural tone. Supports basic instruction types only; **does not** include road or settlement names. Activate with `setUseHumanVoiceWithIdentifier(_:completionHandler:)`.
* **Computer TTS** - Device `AVSpeechSynthesizer` providing detailed, flexible guidance. **Fully supports** street and place names. Quality depends on device capabilities. Activate with `setUseTtsWithCompletionHandler(_:)`.

> 🚨 **DANGER**
>
> ⚠️ **Do not confuse voice and language concepts**
>
> * **Language** defines **what** is said — words, phrasing, and localization
> * **Voice** defines **how** it is said — accent, tone, and gender
>
> Ensure the selected voice is compatible with the chosen language. Mismatched combinations may result in unnatural or incorrect pronunciation.

### Get available human voices[​](#get-available-human-voices "Direct link to Get available human voices")

Retrieve the available human voices list using `getLocalList()` from `HumanVoiceContext`:

```swift
let humanVoiceContext = HumanVoiceContext()
func getVoices() {
    // getOnlineList(completionHandler:) also available to check for and download additional voices
    let voices = humanVoiceContext.getLocalList()
    for item in voices {
        let name = item.getName()
        let gender = humanVoiceContext.getHumanVoiceGender(with: item)
        let nativeLanguage = humanVoiceContext.getNativeLanguage(item)
        print("\(name) — \(nativeLanguage)")
    }
}

```

`HumanVoiceGender` has three values: `HumanVoiceGenderMale`, `HumanVoiceGenderFemale`, `HumanVoiceGenderComputer`.

> 💡 **TIP**
>
> See the [Manage Content Guide](../09-Offline/02-Manage%20Content.md) for downloading, deleting, and managing voices.

### Activate a human voice[​](#activate-a-human-voice "Direct link to Activate a human voice")

Provide the voice identifier from `ContentStoreObject`:

```swift
let voiceId: Int = voices.first?.getIdentifier() ?? 0

soundContext?.setUseHumanVoiceWithIdentifier(voiceId) { success in
    if success {
        print("Human voice activated")
    }
}

```

### Activate a TTS voice by language[​](#activate-a-tts-voice-by-language "Direct link to Activate a TTS voice by language")

Activate computer voices using `setUseTtsWithLanguage(_:completionHandler:)`:

```swift
soundContext?.setUseTtsWithLanguage("en-GB") { success in
    if success {
        print("TTS voice activated for en-GB")
    }
}

```

To use a specific `AVSpeechSynthesisVoice` identifier:

```swift
let voices = AVSpeechSynthesisVoice.speechVoices()
if let voice = voices.first(where: { $0.language == "en-GB" }) {
    soundContext?.setUseTtsWithLanguage("en-GB", identifier: voice.identifier) { success in
        print("TTS voice set: \(success)")
    }
}

```

> ⚠️ **WARNING**
>
> Selecting a TTS voice in an unsupported language may cause a mismatch between spoken voice and instruction content. Exact behavior depends on device TTS capabilities.

## Step 4: Connect SoundContext to navigation[​](#step-4-connect-soundcontext-to-navigation "Direct link to Step 4: Connect SoundContext to navigation")

Implement the `NavigationContextDelegate` sound methods to route audio through `SoundContext`:

```swift
// Return true to allow the SDK's built-in sound playback
func navigationContext(_ navigationContext: NavigationContext,
                       canPlayNavigationSoundForRoute route: RouteObject) -> Bool {
    return true
}

// Play the sound object delivered by the navigation engine
func navigationContext(_ navigationContext: NavigationContext,
                       route: RouteObject,
                       navigationSound sound: SoundObject) {
    soundContext?.playSound(sound)
}

```

## Step 5: Integrate external TTS (optional)[​](#step-5-integrate-external-tts-optional "Direct link to Step 5: Integrate external TTS (optional)")

The `NavigationContextDelegate` delivers `SoundObject` instances for each maneuver. Use `AVSpeechSynthesizer` or any third-party TTS engine to speak the instruction text:

```swift
let synthesizer = AVSpeechSynthesizer()

func navigationContext(_ navigationContext: NavigationContext,
                       canPlayNavigationSoundForRoute route: RouteObject) -> Bool {
    // Return false to suppress built-in playback and handle audio yourself
    return false
}

func navigationContext(_ navigationContext: NavigationContext,
                       route: RouteObject,
                       navigationSound sound: SoundObject) {
    // Use the text from the current navigation instruction instead
    if let instruction = navigationContext.getNavigationInstruction() {
        let text = instruction.getNextTurnInstruction()
        let utterance = AVSpeechUtterance(string: text)
        utterance.voice = AVSpeechSynthesisVoice(language: "en-US")
        synthesizer.speak(utterance)
    }
}

```

> 💡 **TIP**
>
> **Disable internal playback:**
>
> Return `false` from `canPlayNavigationSoundForRoute`. Instructions still arrive via `navigationSound`, but no built-in audio plays.

## Step 6: Adjust audio output[​](#step-6-adjust-audio-output "Direct link to Step 6: Adjust audio output")

Select the audio route with `updateSession(withAudioOutput:)` on `SoundContext`:

* **`AudioOutputAutomatic`** - Uses Bluetooth A2DP when available, otherwise speaker
* **`AudioOutputSpeakerOnly`** - Forces speaker output (requires `AudioCategoryPlaybackAndRecording`)
* **`AudioOutputBluetoothAsPhoneCall`** - Routes audio as a phone call ⚠️

Update the audio session category with `updateSession(withAudioCategory:)`:

* **`AudioCategoryPlayback`** - For audio playback only
* **`AudioCategoryPlaybackAndRecording`** - For recording and playback (required for Bluetooth phone-call and speaker-only output)

```swift
soundContext?.updateSession(with: .playbackAndRecording)
soundContext?.updateSession(with: .bluetoothAsPhoneCall)

```

For Bluetooth phone-call routing, set the call start delay in milliseconds:

```swift
soundContext?.setDelay(500)

```

## Step 7: Monitor audio session interruptions[​](#step-7-monitor-audio-session-interruptions "Direct link to Step 7: Monitor audio session interruptions")

Implement `SoundContextDelegate` to react to audio session interruptions (e.g., incoming calls):

```swift
soundContext?.delegate = self

// MARK: - SoundContextDelegate

func soundContextNotifyInterruptionBegin(_ soundContext: SoundContext) {
    print("Audio session interrupted — pausing navigation audio")
}

func soundContextNotifyInterruptionEnded(_ soundContext: SoundContext) {
    print("Audio session resumed")
}

```

## Step 8: Play custom instructions[​](#step-8-play-custom-instructions "Direct link to Step 8: Play custom instructions")

Use `playText(_:)` on `SoundContext` to play custom instructions (e.g., road warnings or alerts). This uses the currently configured computer TTS voice and is **not available for human voices**.

```swift
soundContext?.playText("Traffic incident ahead. Consider an alternative route.")

```

