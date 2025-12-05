# EMS OTG Communication Library

USB accessory communication layer for the EMS77xx.

## Modules
- `ems-otg-lib` — reusable comms library

## Add to your project
```gradle
// settings.gradle (or build.gradle in repositories block)
repositories {
    maven {
        url = uri("https://maven.pkg.github.com/ChangYow/EMS77xx-lib")
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
- Request permission and handle detaches in your app
