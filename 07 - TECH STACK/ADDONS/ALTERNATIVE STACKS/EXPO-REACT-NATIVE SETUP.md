# EXPO-REACT-NATIVE SETUP [ ALTERNATIVE STACKS ]

------------------------------------------------------------------------

Expo is the production framework for React Native. It gives you a mobile
app (iOS + Android) with file-based routing via Expo Router — the same
mental model as the Next.js App Router — plus over-the-air updates and a
managed native build pipeline.

------------------------------------------------------------------------

## STEP 1 : Create The App

```bash
pnpm create expo-app@latest mobile --template default
cd mobile
pnpm dlx expo install expo-router react-native-safe-area-context
```

The default template ships with Expo Router, TypeScript, and a tab
layout already wired.

------------------------------------------------------------------------

## STEP 2 : Understand File-Based Routing

```text
   app/
   ├── _layout.tsx        → root stack (like layout.tsx)
   ├── index.tsx          → route "/"
   ├── (tabs)/
   │   ├── _layout.tsx    → tab navigator
   │   ├── home.tsx       → /home
   │   └── profile.tsx    → /profile
   └── [id].tsx           → dynamic route /:id

   Same convention as Next.js App Router, rendered natively.
```

------------------------------------------------------------------------

## STEP 3 : Write A Screen

```tsx
// app/(tabs)/home.tsx
import { View, Text, Pressable } from "react-native";
import { useRouter } from "expo-router";

export default function Home() {
  const router = useRouter();
  return (
    <View style={{ flex: 1, alignItems: "center", justifyContent: "center" }}>
      <Text style={{ fontSize: 20 }}>Home</Text>
      <Pressable onPress={() => router.push("/profile")}>
        <Text>Go to profile</Text>
      </Pressable>
    </View>
  );
}
```

------------------------------------------------------------------------

## STEP 4 : Run On A Device

```bash
pnpm dlx expo start          # QR code → Expo Go app
pnpm dlx expo start --ios     # iOS simulator (macOS only)
pnpm dlx expo start --android # Android emulator
```

Windows note: the iOS simulator requires macOS. On Windows, use an
Android emulator (Android Studio) or scan the QR with Expo Go on a
physical iPhone/Android device.

------------------------------------------------------------------------

## STEP 5 : Fetch From Your Next.js API

```tsx
import { useEffect, useState } from "react";

export function useHealth() {
  const [ok, setOk] = useState(false);
  useEffect(() => {
    // use LAN IP, not localhost — the device is a separate host
    fetch("http://192.168.1.10:3000/api/health")
      .then((r) => r.json())
      .then((d) => setOk(d.status === "ok"));
  }, []);
  return ok;
}
```

`localhost` on a phone points at the phone itself; always target your
machine's LAN IP or a tunnel URL.

------------------------------------------------------------------------

## STEP 6 : Build For Stores

```bash
pnpm dlx eas-cli login
pnpm dlx eas build --platform all      # cloud native build
pnpm dlx eas submit --platform ios     # upload to App Store Connect
pnpm dlx eas update                    # OTA JS update, no rebuild
```

EAS builds run in the cloud, so store-ready iOS binaries are possible
even from Windows.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use Expo Router for App-Router-style file-based navigation
✓ Install native deps with expo install to match SDK versions
✓ Target LAN IP or a tunnel for API calls, never localhost
✓ Ship JS-only changes via eas update OTA, skip store review
✓ Build in the cloud with EAS to get iOS builds on Windows
✓ Keep secrets in EAS secrets, not in the JS bundle
✓ Test on a real device early — simulators hide perf issues
✓ Share types with the web app through a monorepo package
✓ Pin the Expo SDK version and upgrade deliberately
```
