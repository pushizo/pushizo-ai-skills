---
name: pushizo-sdk-setup
description: Install and initialize the Pushizo client SDK (web push, iOS, Android, Flutter, React Native/Expo, Capacitor, Cordova, .NET MAUI, Unity, React, Vue, Angular) in an app, request notification permission and identify the user. Use when the user asks to add Pushizo, push notifications or in-app messaging to a client app.
---

# Pushizo client SDK setup

Pushizo is a multi-channel messaging platform (push, email, SMS, in-app). Client SDKs register the device and
identify the user; sending happens from a server (see the `pushizo-server-api` skill).

## Rules

- The client needs only the **App ID** (public) and, for self-hosted or staging setups, the **API URL**
  (default `https://api.pushizo.com`). Never put an API key (`hs_live_…`) or SDK secret into client code,
  `.env` files shipped to the browser, or mobile app bundles.
- Ask the user for the App ID if it is not in the project; do not invent one. Read it from config or environment
  (`NEXT_PUBLIC_HS_APP_ID`, `VITE_HS_APP_ID`, build settings) rather than hard-coding when the project has a config
  pattern.
- Initialize exactly once per app launch (web: once per page load, in the browser only; never during SSR).
- Request notification permission at a meaningful moment (after a user action), not on first paint.
- Call `login(externalId)` with the app's own stable user id after sign-in and `logout()` on sign-out. If identity
  verification is enabled for the app, the server must provide an `identity_hash` or ES256 JWT (see
  `pushizo-server-api`).
- Push credentials (APNs `.p8`, FCM service account, VAPID keys) are uploaded in the Pushizo dashboard or with the
  server API, never committed to the repository.
- Match the project's language and framework; follow its existing file layout and lint rules. Respond to the user in
  their language.

## Pick the platform

| Project signal | Package | Init |
|---|---|---|
| Plain HTML / any site | script `https://api.pushizo.com/sdk-web/PushizoSDK.js` | `Pushizo.init({ appId })` via `PushizoDeferred` |
| npm web app (Vite, Webpack) | `@pushizo/web` | `Pushizo.init({ appId, serviceWorkerPath: "/PushizoSDKWorker.js" })` |
| React / Next.js | `@pushizo/react` + `@pushizo/web` | `<PushizoProvider options={…}>`, `usePushizo()` |
| Vue 3 / Nuxt | `@pushizo/vue` + `@pushizo/web` | plugin + composables |
| Angular | `@pushizo/angular` + `@pushizo/web` | provider + injectable service |
| iOS (Swift) | SPM `https://github.com/pushizo/pushizo-ios` (products `Pushizo`, `PushizoExtension`) or CocoaPods `Pushizo` | `Pushizo.initialize(appId, launchOptions:)` |
| Android (Kotlin/Java) | `com.pushizo:pushizo-android` (Maven Central) | `Pushizo.initialize(this, appId, options)` in `Application` |
| Flutter | `pushizo_flutter` (pub.dev) | `await Pushizo.initialize(appId)` |
| React Native / Expo | `@pushizo/react-native` (Expo config plugin included) | `Pushizo.initialize(appId, { apiUrl })` |
| Capacitor / Ionic | `@pushizo/capacitor` + `@pushizo/web` | `await Pushizo.initialize({ appId })` |
| Cordova | `cordova-plugin-pushizo` | `window.Pushizo.initialize(appId)` on `deviceready` |
| .NET MAUI | `Pushizo.Maui` (NuGet) | `builder.UsePushizo(appId, …)` |
| Unity | UPM `https://github.com/pushizo/pushizo-unity.git#<version>` or OpenUPM `com.pushizo.unity` | see the Unity guide |

Some framework packages may still be rolling out to their registry; each repository README under
`github.com/pushizo` shows the current status and a release-archive install command.

## Web (script tag)

```html
<script src="https://api.pushizo.com/sdk-web/PushizoSDK.js" async></script>
<script>
  window.PushizoDeferred = window.PushizoDeferred || [];
  PushizoDeferred.push(async function (Pushizo) {
    await Pushizo.init({ appId: "YOUR_APP_ID" });
  });
</script>
```

Service worker at the site root, `/PushizoSDKWorker.js`:

```js
const apiUrl = new URL(self.location.href).searchParams.get("apiUrl") || "https://api.pushizo.com";
importScripts(apiUrl.replace(/\/$/, "") + "/sdk-web/PushizoSDKWorker.js");
```

If the site already has a service worker, add `importScripts("https://api.pushizo.com/sdk-web/PushizoSDKWorker.js")`
to it and pass `serviceWorkerPath` / `serviceWorkerScope` to `init` instead of adding a second worker.

Then, after a user action:

```js
await Pushizo.Notifications.requestPermission();
await Pushizo.login("user_42");
Pushizo.User.addTag("plan", "pro");
```

## React / Next.js

```tsx
"use client";
import type { ReactNode } from "react";
import { PushizoProvider } from "@pushizo/react";

const options = { appId: process.env.NEXT_PUBLIC_HS_APP_ID ?? "", serviceWorkerPath: "/PushizoSDKWorker.js" };

export function Providers({ children }: { children: ReactNode }) {
  return <PushizoProvider options={options}>{children}</PushizoProvider>;
}
```

Use `usePushizo()` for the SDK instance and `usePushSubscription()` / `usePushizoState()` for state. In Next.js,
serve `/PushizoSDKWorker.js` with the `Service-Worker-Allowed: /` header.

## iOS

1. Add the package (`Pushizo` to the app target, `PushizoExtension` to a Notification Service Extension target).
2. Capabilities: Push Notifications and Background Modes → Remote notifications.
3. AppDelegate:

```swift
import Pushizo
import UIKit

final class AppDelegate: NSObject, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil) -> Bool {
        Pushizo.initialize("YOUR_APP_ID", launchOptions: launchOptions)
        return true
    }

    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        Pushizo.setDeviceToken(deviceToken)
    }

    func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
        Pushizo.setDeviceTokenError(error)
    }
}
```

Request permission with `Pushizo.Notifications.requestPermission { granted in … }`. In the extension, forward
`didReceive` and `serviceExtensionTimeWillExpire` to `PushizoNotificationServiceExtension(appId:apiUrl:appGroup:)`.

## Android

```kotlin
import android.app.Application
import com.pushizo.sdk.Pushizo

class App : Application() {
    override fun onCreate() {
        super.onCreate()
        Pushizo.initialize(this, "YOUR_APP_ID", Pushizo.Options(apiUrl = "https://api.pushizo.com"))
    }
}
```

Add `implementation("com.pushizo:pushizo-android:<version>")`, the `com.google.gms.google-services` plugin and
`google-services.json` (from the user's Firebase project). Register `android:name=".App"` in the manifest. On Android 13+
call `Pushizo.Notifications.requestPermission(activity)` and forward `onRequestPermissionsResult` when
`requestCode == Pushizo.Notifications.PERMISSION_REQUEST_CODE`.

## Flutter, React Native, Capacitor, Cordova

```dart
WidgetsFlutterBinding.ensureInitialized();
await Pushizo.initialize('YOUR_APP_ID');
await Pushizo.Notifications.requestPermission();
```

```tsx
import { Pushizo } from "@pushizo/react-native";
Pushizo.initialize("YOUR_APP_ID", { apiUrl: "https://api.pushizo.com" });
```

Expo: add `["@pushizo/react-native", { "mode": "development", "appId": "YOUR_APP_ID" }]` to `app.json` plugins
(`"production"` for release builds), then `npx expo prebuild --clean`.

```ts
import { Pushizo } from "@pushizo/capacitor";
await Pushizo.initialize({ appId: "YOUR_APP_ID" });
```

All mobile wrappers need the native steps: `google-services.json` for Android and Push Notifications + Background
Modes capabilities for iOS.

## Verify

1. Run the app, grant permission and confirm the subscription in the Pushizo dashboard or with
   `GET /v1/apps/{appId}/users/by/external_id/{id}` (server API).
2. Send a test to test devices only (dashboard "Test push" or the server API with `target.test_devices_only: true`).
3. Report what was changed and the remaining manual steps (credentials, capabilities) to the user.

Docs: https://docs.pushizo.com/docs/sdk
