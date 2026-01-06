# heroic-acapella-ai
AI backend for generating heroic multi-voice Pashto nasheeds from a single a-capella vocal.
main.py
Perfect — APK it is. I’ll generate a clean, ready‑to‑paste GitHub Actions workflow that builds and signs a release APK, assuming your Android module is the standard:

`
android/app
`

If your module is different, just tell me and I’ll adjust it.

Below is the final APK‑only workflow, optimized, simplified, and production‑ready.

---

🚀 android-build.yml — Signed APK Build (Ready to Copy)

`yaml
name: Android APK Build

on:
  push:
    branches: [ main, master ]
  workflow_dispatch:

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
          java-version: '17'

      - name: Cache Gradle
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('/.gradle', '/gradle-wrapper.properties') }}

      - name: Decode keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > release.keystore

      - name: Build Release APK
        working-directory: android
        run: ./gradlew clean assembleRelease --no-daemon

      - name: Sign APK
        run: |
          unsigned_apk=$(find android/app/build/outputs/apk/release -name "*release-unsigned.apk" -print -quit)
          aligned_apk=app-release-aligned.apk
          signed_apk=app-release-signed.apk

          zipalign -v -p 4 "$unsignedapk" "$alignedapk"

          apksigner sign \
            --ks release.keystore \
            --ks-pass pass:${{ secrets.KEYSTORE_PASSWORD }} \
            --ks-key-alias "${{ secrets.KEY_ALIAS }}" \
            --key-pass pass:${{ secrets.KEY_PASSWORD }} \
            --out "$signed_apk" \
            "$aligned_apk"

          apksigner verify --print-certs "$signed_apk"

      - name: Upload APK Artifact
        uses: actions/upload-artifact@v4
        with:
          name: release-apk
          path: app-release-signed.apk
`

---

🔐 Required Secrets (GitHub → Settings → Secrets → Actions)

| Secret Name | Value |
|------------|--------|
| KEYSTORE_BASE64 | Base64 of your .jks file |
| KEYSTORE_PASSWORD | Keystore password |
| KEY_ALIAS | Alias inside keystore |
| KEY_PASSWORD | Key password |

---

🧩 Add this to android/app/build.gradle

`gradle
android {
    signingConfigs {
        release {
            storeFile file("release.keystore")
            storePassword System.getenv("KEYSTORE_PASSWORD")
            keyAlias System.getenv("KEY_ALIAS")
            keyPassword System.getenv("KEY_PASSWORD")
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled false
            shrinkResources false
        }
    }
}
`

---

📝 Add to gradle.properties (local only — DO NOT COMMIT)

`
KEYSTORE_PASSWORD=yourpassword
KEY_ALIAS=youralias
KEY_PASSWORD=yourkeypassword
`

---

If you want, I can now generate:

✅ A version of this workflow that also builds debug APKs  
✅ A version that builds both APK + AAB  
✅ A version that automatically creates GitHub Releases and attaches the APK  

Just tell me what you want next.
