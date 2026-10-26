# Android autolinking after the project moves

React Native 0.88.0's template app, unchanged: `ReproducerApp` autolinks one native library,
`react-native-safe-area-context`. The first Android build caches the `@react-native-community/cli config`
output, with absolute paths, in `android/build/generated/autolinking/autolinking.json`. After the project
folder moves, the next build still reads that cache and includes the library from the folder it moved from.

## Steps

```bash
cd ReproducerApp
yarn install
cd android
./gradlew assembleDebug        # BUILD SUCCESSFUL
./gradlew --stop
cd ../..
mv ReproducerApp ReproducerApp-moved
cd ReproducerApp-moved/android
./gradlew assembleDebug
```

## Expected

The second build links `react-native-safe-area-context` from `ReproducerApp-moved/node_modules` and succeeds.

## Seen

```text
* What went wrong:
Configuring project ':react-native-safe-area-context' without an existing directory is not allowed. The configured projectDirectory '<checkout>/ReproducerApp/node_modules/react-native-safe-area-context/android' does not exist, can't be written to or is not a directory.

BUILD FAILED
```

`android/build/generated/autolinking/autolinking.json` still names `<checkout>/ReproducerApp` in every
`root` and `sourceDir`. Copying the project instead of moving it builds, but the copy then compiles the
library from the original folder's `node_modules`, into that folder's `build` directory.

Deleting `android/build/generated/autolinking` before the second build makes it pass.
