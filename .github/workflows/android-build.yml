name: Build FAYMAH Android APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:
  build:
    name: Build FAYMAH Android APK
    runs-on: ubuntu-latest

    steps:

      # 1. Checkout repository
      - name: Checkout repository
        uses: actions/checkout@v5

      # 2. Set up Java 17
      # Gradle cache is intentionally NOT enabled here because
      # the Android Gradle project is inside the source ZIP.
      - name: Set up Java 17
        uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: '17'

      # 3. Set up Android SDK
      - name: Set up Android SDK
        uses: android-actions/setup-android@v3

      # 4. Accept Android SDK licences
      - name: Accept Android SDK licences
        run: |
          yes | sdkmanager --licenses || true

      # 5. Install Android SDK packages
      - name: Install Android SDK packages
        run: |
          sdkmanager "platform-tools"
          sdkmanager "platforms;android-35"
          sdkmanager "build-tools;35.0.0"

      # 6. Inspect repository
      - name: Inspect repository
        run: |
          echo "Repository contents:"
          ls -la

          echo ""
          echo "Android/Gradle files currently in repository:"
          find . -maxdepth 5 \( \
            -name "settings.gradle" -o \
            -name "settings.gradle.kts" -o \
            -name "build.gradle" -o \
            -name "build.gradle.kts" -o \
            -name "gradlew" \
          \) -print

      # 7. Check FAYMAH source ZIP
      - name: Check FAYMAH source ZIP
        run: |
          if [ ! -f "FaymahAI_v1_2_LiveBackend.zip" ]; then
            echo "ERROR: FaymahAI_v1_2_LiveBackend.zip was not found."
            echo "Files in repository:"
            ls -lah
            exit 1
          fi

          echo "FAYMAH source ZIP found:"
          ls -lh FaymahAI_v1_2_LiveBackend.zip

      # 8. Extract FAYMAH source
      - name: Extract FAYMAH source
        run: |
          rm -rf app-source
          mkdir -p app-source

          unzip -o FaymahAI_v1_2_LiveBackend.zip -d app-source

          echo ""
          echo "Extracted FAYMAH files:"
          find app-source -maxdepth 5 -type f | head -200

      # 9. Locate Android project
      - name: Locate Android project
        id: android_project
        shell: bash
        run: |
          PROJECT_DIR=""

          # Check whether the ZIP itself contains the project at its root
          if [ -f "app-source/gradlew" ]; then
            PROJECT_DIR="app-source"

          else
            # Search for Gradle wrapper
            FOUND=$(find app-source \
              -type f \
              -name "gradlew" \
              -print \
              -quit)

            if [ -n "$FOUND" ]; then
              PROJECT_DIR="$(dirname "$FOUND")"
            else
              # If no Gradle wrapper exists, search for Gradle settings
              FOUND=$(find app-source \
                -type f \
                \( \
                  -name "settings.gradle" -o \
                  -name "settings.gradle.kts" \
                \) \
                -print \
                -quit)

              if [ -n "$FOUND" ]; then
                PROJECT_DIR="$(dirname "$FOUND")"
              fi
            fi
          fi

          if [ -z "$PROJECT_DIR" ]; then
            echo "ERROR: Could not find Gradle Android project."

            echo ""
            echo "Searching for Gradle project files:"
            find app-source \
              -type f \
              \( \
                -name "settings.gradle" -o \
                -name "settings.gradle.kts" -o \
                -name "build.gradle" -o \
                -name "build.gradle.kts" -o \
                -name "gradlew" \
              \) \
              -print

            exit 1
          fi

          echo ""
          echo "Android project found at:"
          echo "$PROJECT_DIR"

          echo "project_dir=$PROJECT_DIR" >> "$GITHUB_OUTPUT"

      # 10. Prepare Gradle
      - name: Prepare Gradle
        working-directory: ${{ steps.android_project.outputs.project_dir }}
        shell: bash
        run: |
          if [ -f "./gradlew" ]; then
            echo "Gradle wrapper found."

            chmod +x ./gradlew

            ./gradlew --version

          else
            echo "Gradle wrapper not found."
            echo "Installing system Gradle..."

            sudo apt-get update
            sudo apt-get install -y gradle

            gradle --version
          fi

      # 11. Build FAYMAH APK
      - name: Build FAYMAH APK
        working-directory: ${{ steps.android_project.outputs.project_dir }}
        shell: bash
        run: |
          echo "Starting FAYMAH Android APK build..."

          if [ -f "./gradlew" ]; then
            ./gradlew assembleDebug \
              --no-daemon \
              --stacktrace
          else
            gradle assembleDebug \
              --no-daemon \
              --stacktrace
          fi

      # 12. Find generated APK
      - name: Find APK
        shell: bash
        run: |
          echo "Searching for generated APK files..."

          find app-source \
            -type f \
            -name "*.apk" \
            -print

          APK_COUNT=$(find app-source -type f -name "*.apk" | wc -l)

          if [ "$APK_COUNT" -eq 0 ]; then
            echo ""
            echo "ERROR: No APK was generated."
            exit 1
          fi

          echo ""
          echo "APK successfully generated."

      # 13. Upload APK artifact
      - name: Upload FAYMAH APK
        uses: actions/upload-artifact@v4
        with:
          name: FAYMAH-Android-APK
          path: |
            app-source/**/*.apk
          if-no-files-found: error
          retention-days: 30
