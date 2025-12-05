# EMS OTG Communication Library

USB accessory communication layer extracted from the EMS7700 test app and packaged as an Android library.

## Modules
- `ems-otg-lib` — reusable comms library (`groupId=com.changyow.ems`, `artifactId=ems-otg-lib`).
- `app` — sample UI wired to the library.

## Add to your project
```gradle
// settings.gradle (or build.gradle in repositories block)
repositories {
    maven {
        url = uri("https://maven.pkg.github.com/ChangYow/EMS77xx-lib")
        credentials {
            // Use a GitHub personal access token with read:packages scope
            username = findProperty("gpr.user") ?: System.getenv("GPR_USER")
            password = findProperty("gpr.key")  ?: System.getenv("GPR_KEY")
        }
    }
    google()
    mavenCentral()
}
```
Add the dependency as usual:
```gradle
dependencies {
    implementation "com.changyow.ems:ems-otg-lib:1.0.0"
}
```

## Core API (package `com.changyow.ems.otg`)
- `UsbAccessoryManager.createInstance(UsbManager, UsbAccessory)` / `UsbAccessoryManager.instance`
- `openAccessory()`, `closeAccessory()`
- Commands:
  - `setTargetPower(power: Int)`
  - `requestDfuMode()`
  - `deviceInfo` (getter triggers request)
  - `setSerialNumber(year: Int, month: Int, sn: Int)`
  - `getSerialNumber()`
- Events (via EventBus):
  - `EmsInitializedEvent`, `EmsReadErrorOccurEvent`
  - `EmsRequestDfuModeEvent`
  - `EmsGetDeviceInfoEvent`
  - `EmsSetControlEvent`
  - `EmsGetSerialNumberEvent`
  - `EmsSetSerialNumberDoneEvent`

## Minimal usage snippet
```kotlin
class MyActivity : AppCompatActivity() {
    private var mgr: UsbAccessoryManager? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        EventBus.getDefault().register(this)
    }

    override fun onDestroy() {
        mgr?.closeAccessory()
        EventBus.getDefault().unregister(this)
        super.onDestroy()
    }

    private fun connect(usbManager: UsbManager, accessory: UsbAccessory) {
        mgr = UsbAccessoryManager.createInstance(usbManager, accessory)
        mgr?.openAccessory()
    }

    @Subscribe(threadMode = ThreadMode.MAIN)
    fun onDeviceInfo(event: EmsGetDeviceInfoEvent) { /* update UI */ }
}
```

## App Manifest notes
- Declare your accessory filter and permission intent in the app module (library ships no components):
  ```xml
  <uses-feature android:name="android.hardware.usb.accessory" />
  <application>
      <meta-data android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED"
                 android:resource="@xml/accessory_filter" />
  </application>
  ```
- Request permission and handle detaches in your app (see `app/src/main/java/com/changyow/ems7700test/MainActivity.kt`).

## Building
```bash
./gradlew :ems-otg-lib:assembleRelease   # build AAR
```

## Publishing (prepared, not executed)
Credentials/URL are read from Gradle properties or env vars:
- `EMS_MAVEN_URL` — repository URL (defaults to `build/maven-repo` if unset)
- `EMS_MAVEN_USER`, `EMS_MAVEN_PASSWORD` — optional credentials
- `EMS_MAVEN_ALLOW_INSECURE` — set to `true` to allow http

Run when ready:
```bash
./gradlew :ems-otg-lib:publish
```
This will publish the `release` AAR + sources JAR and generated POM to the configured repo.

## Versioning
Library version is set in `ems-otg-lib/build.gradle` (`version = "1.0.0"`). Update group/artifact/version as needed before publishing.
