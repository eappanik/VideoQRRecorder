name: Android APK Build
on:
  workflow_dispatch:
  push:
    branches: [ main, master ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: Set up Android SDK
        uses: android-actions/setup-android@v3
        with:
          packages: |
            platform-tools
            platforms;android-34
            build-tools;34.0.0

      - name: Create local.properties
        run: echo "sdk.dir=$ANDROID_SDK_ROOT" > local.properties

      - name: Install Gradle and generate wrapper
        run: |
          curl -s "https://get.sdkman.io" | bash
          source "$HOME/.sdkman/bin/sdkman-init.sh"
          sdk install gradle 8.7
          gradle --version
          gradle wrapper --gradle-version 8.7

      - name: Build debug APK
        run: ./gradlew assembleDebug

      - name: Upload APK artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-debug.apk
          path: app/build/outputs/apk/debug/app-debug.apk
