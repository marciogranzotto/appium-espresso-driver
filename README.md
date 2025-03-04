# Appium Espresso Driver

[![Build Status](https://dev.azure.com/AppiumCI/Appium%20CI/_apis/build/status/appium.appium-espresso-driver?branchName=master)](https://dev.azure.com/AppiumCI/Appium%20CI/_build/latest?definitionId=3&branchName=master)

[![NPM version](http://img.shields.io/npm/v/appium-espresso-driver.svg)](https://npmjs.org/package/appium-espresso-driver)
[![Downloads](http://img.shields.io/npm/dm/appium-espresso-driver.svg)](https://npmjs.org/package/appium-espresso-driver)

[![Release](https://github.com/appium/appium-espresso-driver/actions/workflows/publish.js.yml/badge.svg)](https://github.com/appium/appium-espresso-driver/actions/workflows/publish.js.yml)

Appium's Espresso Driver is a test automation server for Android that uses [Espresso](https://developer.android.com/training/testing/espresso/) as the underlying test technology. The Espresso Driver is a part of the Appium framework. The driver operates in scope of [W3C WebDriver protocol](https://www.w3.org/TR/webdriver/) with several custom extensions to cover operating-system specific scenarios.

The Espresso package consists of two main parts:
- The driver part (written in Node.js) ensures the communication between the Espresso server and Appium. Also includes several handlers that directly use ADB and/or other system tools without a need to talk to the server.
- The server part (written in Kotlin with some parts of Java), which is running on the device under test and transforms REST API calls into low-level Espresso commands.

> **Note**
>
> Since version 2.0.0 Espresso driver has dropped the support of Appium 1, and is only compatible to Appium 2. Use the `appium driver install espresso`
> command to add it to your Appium 2 dist.


## Comparison with UiAutomator2

The key difference between [UiAutomator2 Driver](https://github.com/appium/appium-uiautomator2-driver) and Espresso Driver is that UiAutomator2 is a black-box testing framework, and Espresso is a "grey-box" testing framework. The Espresso Driver itself is black-box (no internals of the code are exposed to the tester), but the Espresso framework itself has access to the internals of Android applications. This distinction has a few notable benefits. It can find elements that aren't rendered on the screen, it can identify elements by the Android View Tag and it makes use of [IdlingResource](https://developer.android.com/reference/android/support/test/espresso/IdlingResource) which blocks the framework from running commands until the UI thread is free. There is limited support to automate out of app areas using the mobile command [uiautomator](https://github.com/appium/appium-espresso-driver/blob/b2b0883ab088a131a47d88f6aeddd8ff5882087d/lib/commands/general.js#L188)


## Requirements

On top of standard Appium requirements Espresso driver also expects the following prerequisites:

- Windows, Linux and macOS are supported as hosts
- [Android SDK Platform tools](https://developer.android.com/studio/releases/platform-tools) must be installed. [Android Studio IDE](https://developer.android.com/studio) also provides a convenient UI to install and manage the tools.
- ANDROID_HOME or ANDROID_SDK_ROOT [environment variable](https://developer.android.com/studio/command-line/variables) must be set
- Java JDK must be installed and [JAVA_HOME](https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux) environment variable must be set. Java major version must be 11 or newer.
- [Emulator](https://developer.android.com/studio/run/managing-avds) platform image must be installed if you plan to run your tests on it. [Android Studio IDE](https://developer.android.com/studio) also provides a convenient UI to install and manage emulators.
- Real Android devices must have [USB debugging enabled](https://developer.android.com/studio/debug/dev-options) and should be visible as `online` in `adb devices -l` output.
- The minimum version of Android API must be 5.0 (API level 21) (6.0 is recommended as version 5 has some known compatibility issues).
- [Gradle](https://gradle.org/) must be installed in order to build Espresso server.
- Both the server package and the application under test must be signed with the same digital signature. Appium does sign them automatically upon session creation, so this could only be an issue if one wants to test an application, which is already installed on the device (using `noReset=true` capability).
- The package under test must not have mangled class names (e.g. [Proguard](https://developer.android.com/studio/build/shrink-code) must not be enabled for it)


## Scripts

- `appium driver run espresso print-espresso-path` prints the path to the Appium Espresso server root. You can modify the gradle file directly if [Espresso Build Config](#espresso-build-config) was not sufficient.
- `appium driver run espresso build-espresso` builds the espresso server since driver version 2.18.0. It helps building the espresso server outside of the Appium process. Available environment variables are below:
  - `SHOW_GRADLE_LOG` configures if the command shows the gradle task logs. `true` or `1`  sets it as enabled, but others set it as disabled. Defaults to disabled.
  - `TEST_APP_PACKAGE` configures the target application to build the espresso server for.
  - `ESPRESSO_BUILD_CONFIG` is an absolute path to the [Espresso Build Config](#espresso-build-config) as JSON format file.
    - e.g. `SHOW_GRADLE_LOG=true TEST_APP_PACKAGE=your.test.pkg ESPRESSO_BUILD_CONFIG=/path/to/the/config.json appium driver run build-espresso`

## Capabilities

### General

Capability Name | Description
--- | ---
platformName | Could be set to `android`. Appium itself is not strict about this capability value if `automationName` is provided, so feel free to assign it to any supported platform name if this is needed, for example, to make Selenium Grid working.
appium:automationName | Must always be set to `espresso`. Values of `automationName` are compared case-insensitively.
appium:deviceName | The name of the device under test (actually, it is not used to select a device under test). Consider setting `udid` for real devices and `avd` for emulators instead
appium:platformVersion | The platform version of an emulator or a real device. This capability is used for device autodetection if `udid` is not provided
appium:udid | UDID of the device to be tested. Could ve retrieved from `adb devices -l` output. If unset then the driver will try to use the first connected device. Always set this capability if you run parallel tests.
appium:noReset | Prevents the device to be reset before the session startup if set to `true`. This means that the application under test is not going to be terminated neither its data cleaned. `false` by default
appium:fullReset | Being set to `true` always enforces the application under test to be fully uninstalled before starting a new session. `false` by default
appium:printPageSourceOnFindFailure | Enforces the server to dump the actual XML page source into the log if any error happens. `false` by default.

### Driver/Server

Capability Name | Description
--- | ---
appium:systemPort | The number of the port the Espresso server is listening on. By default the first free port from 8300..8399 range is selected. It is recommended to set this value if you are running [parallel tests](docs/parallel-tests.md) on the same machine.
appium:skipServerInstallation | Skip the Espresso Server component installation on the device under test and all the related checks if set to `true`. This could help to speed up the session startup if you know for sure the correct server version is installed on the device. In case the server is not installed or an incorrect version of it is installed then you may get an unexpected error later. `false` by default
appium:espressoServerLaunchTimeout | The maximum number of milliseconds to wait util Espresso server is listening on the device. `45000` ms by default
appium:forceEspressoRebuild | Whether to always enforce Espresso server rebuild (`true`). By default Espresso caches the already built server apk and only rebuilds it when it is necessary, because rebuilding process needs extra time. `false` by default
appium:espressoBuildConfig | Either the full path to build config JSON on the server file system or the JSON content itself serialized to a string. This config allows to customize several important properties of Espresso server. Refer to [Espresso Build Config](#espresso-build-config) for more information on how to properly construct such config.
appium:showGradleLog | Whether to include Gradle log to the regular server logs while building Espresso server. `false` by default.

### App

Capability Name | Description
--- | ---
appium:app | Full path to the application to be tested (the app must be located on the same machine where the server is running). The `.apk` application extension is supported. Since driver version 2.1.0 `.aab` files are supported as well (they get converted to `.apk` format automatically if [bundletool.jar](https://github.com/google/bundletool/releases) could be found in your PATH). For older driver versions `.aab` files need to be [converted](https://stackoverflow.com/questions/53040047/generate-apk-file-from-aab-file-android-app-bundle) manually to `.apk` format using [bundletool](https://developer.android.com/studio/command-line/bundletool) first. Could also be an URL to a remote location. If neither of the `app` or `appPackage` capabilities are provided then the driver will fail to start a session. Also, if `app` capability is not provided it is expected that the app under test is already installed on the device under test and `noReset` is equal to `true`.
appium:appPackage | Application package identifier to be started. If not provided then Espresso will try to detect it automatically from the package provided by the `app` capability. Read [How To Troubleshoot Activities Startup](docs/activity-startup.md) for more details
appium:appActivity | Main application activity identifier. If not provided then Espresso will try to detect it automatically from the package provided by the `app` capability. Read [How To Troubleshoot Activities Startup](docs/activity-startup.md) for more details
appium:appWaitActivity | Identifier of the first activity that the application invokes. If not provided then equals to `appium:appActivity`. Read [How To Troubleshoot Activities Startup](docs/activity-startup.md) for more details
appium:appWaitPackage | Identifier of the first package that is invoked first. If not provided then equals to `appium:appPackage`. Read [How To Troubleshoot Activities Startup](docs/activity-startup.md) for more details
appium:appWaitDuration | Maximum amount of milliseconds to wait until the application under test is started (e. g. an activity returns the control to the caller). `20000` ms by default. Read [How To Troubleshoot Activities Startup](docs/activity-startup.md) for more details
appium:intentOptions | The mapping of custom options for the intent that is going to be passed to the main app activity. Check [Intent Options](#intent-options) for more details.
appium:activityOptions | The mapping of custom options for the main app activity that is going to be started. Check [Activity Options](#activity-options) for more details.
appium:androidInstallTimeout | Maximum amount of milliseconds to wait until the application under test is installed. `90000` ms by default
appium:autoGrantPermissions | Whether to grant all the requested application permissions automatically when a test starts(`true`). The targetSdkVersion in the application manifest must be greater or equal to 23 and the Android version on the device under test must be greater or equal to Android 6 (API level 23) to grant permissions. Applications whose targetSdkVersion is lower than or equal to 22 must be reisntalled to grant permissions, for example, by setting the `appium:fullReset` capability as `true` for Android 6+ devices. `false` by default
appium:otherApps | Allows to set one or more comma-separated paths to Android packages that are going to be installed along with the main application under test. This might be useful if the tested app has dependencies
appium:uninstallOtherPackages | Allows to set one or more comma-separated package identifiers to be uninstalled from the device before a test starts
appium:allowTestPackages | If set to `true` then it would be possible to use packages built with the test flag for the automated testing (literally adds `-t` flag to the `adb install` command). `false` by default
appium:remoteAppsCacheLimit | Sets the maximum amount of application packages to be cached on the device under test. This is needed for devices that don't support streamed installs (Android 7 and below), because ADB must push app packages to the device first in order to install them, which takes some time. Setting this capability to zero disables apps caching. `10` by default.
appium:enforceAppInstall | If set to `true` then the application under test is always reinstalled even if a newer version of it already exists on the device under test. `false` by default

### App Localization

Capability Name | Description
--- | ---
appium:localeScript | Canonical name of the locale to be set for the app under test, for example `Hans` in `zh-Hans-CN`. See https://developer.android.com/reference/java/util/Locale.html for more details.
appium:language | Name of the language to extract application strings for. Strings are extracted for the current system language by default. Also sets the language for the app under test. See https://developer.android.com/reference/java/util/Locale.html for more details. Example: en, ja
appium:locale | Sets the locale for the app under test. See https://developer.android.com/reference/java/util/Locale.html for more details. Example: EN, JA
appium:appLocale | Sets the locale for the app under test. The main difference between this option and the above ones is that this option only changes the locale for the application under test and does not affect other parts of the system. Also, it only uses public APIs for its purpose. See https://github.com/libyal/libfwnt/wiki/Language-Code-identifiers to get the list of available language abbreviations. Example: `{"language": "zh", "country": "CN", "variant": "Hans"}`

### ADB

Capability Name | Description
--- | ---
appium:adbPort | Number of the port where ADB is running. `5037` by default
appium:remoteAdbHost | Address of the host where ADB is running (the value of `-H` ADB command line option). Unset by default
appium:adbExecTimeout | Maximum number of milliseconds to wait until single ADB command is executed. `20000` ms by default
appium:clearDeviceLogsOnStart | If set to `true` then Espresso deletes all the existing logs in the device buffer before starting a new test
appium:buildToolsVersion | The version of Android build tools to use. By default Espresso driver uses the most recent version of build tools installed on the machine, but sometimes it might be necessary to give it a hint (let say if there is a known bug in the most recent tools version). Example: `28.0.3`
appium:skipLogcatCapture | Being set to `true` disables automatic logcat output collection during the test run. `false` by default
appium:suppressKillServer | Being set to `true` prevents the driver from ever killing the ADB server explicitly. Could be useful if ADB is connected wirelessly. `false` by default
appium:ignoreHiddenApiPolicyError | Being set to `true` ignores a failure while changing hidden API access policies. Could be useful on some devices, where access to these policies has been locked by its vendor. `false` by default.
appium:mockLocationApp | Sets the package identifier of the app, which is used as a system mock location provider since Appium 1.18.0+. This capability has no effect on emulators. If the value is set to `null` or an empty string, then Appium will skip the mocked location provider setup procedure. Defaults to Appium Setting package identifier (`io.appium.settings`).
appium:logcatFormat | The log print format, where `format` is one of: `brief` `process` `tag` `thread` `raw` `time` `threadtime` `long`. `threadtime` is the default value.
appium:logcatFilterSpecs | Series of `tag[:priority]` where `tag` is a log component tag (or * for all) and priority is: `V    Verbose`, `D    Debug`, `I    Info`, `W    Warn`, `E    Error`, `F    Fatal`, `S    Silent (supress all output)`. '*' means '*:d' and `tag` by itself means `tag:v`. If not specified on the commandline, filterspec is set from ANDROID_LOG_TAGS. If no filterspec is found, filter defaults to '*:I'.
appium:allowDelayAdb | Being set to `false` prevents emulator to use `-delay-adb` feature to detect its startup. See https://github.com/appium/appium/issues/14773 for more details.

### Emulator (Android Virtual Device)

Capability Name | Description
--- | ---
appium:avd | The name of Android emulator to run the test on. The names of currently installed emulators could be listed using `avdmanager list avd` command. If the emulator with the given name is not running then it is going to be started before a test
appium:avdLaunchTimeout | Maximum number of milliseconds to wait until Android Emulator is started. `60000` ms by default
appium:avdReadyTimeout | Maximum number of milliseconds to wait until Android Emulator is fully booted and is ready for usage. `60000` ms by default
appium:avdArgs | Either a string or an array of emulator [command line arguments](https://developer.android.com/studio/run/emulator-commandline).
appium:avdEnv | Mapping of emulator [environment variables](https://developer.android.com/studio/command-line/variables).
appium:networkSpeed | Sets the desired network speed limit for the emulator. It is only applied if the emulator is not running before the test starts. See emulator [command line arguments](https://developer.android.com/studio/run/emulator-commandline) description for more details.
appium:gpsEnabled | Sets whether to enable (`true`) or disable (`false`) GPS service in the Emulator. Unset by default, which means to not change the current value
appium:isHeadless | If set to `true` then emulator starts in headless mode (e.g. no UI is shown). It is only applied if the emulator is not running before the test starts. `false` by default.

### App Signing

Capability Name | Description
--- | ---
appium:useKeystore | Whether to use a custom [keystore](https://developer.android.com/studio/publish/app-signing#certificates-keystores) to sign the app under test. `false` by default, which means apps are always signed with the default Appium debug certificate (unless canceled by `noSign` capability). This capability is used in combination with `keystorePath`, `keystorePassword`, `keyAlias` and `keyPassword` capabilities.
appium:keystorePath | The full path to the keystore file on the server filesystem. This capability is used in combination with `useKeystore`, `keystorePath`, `keystorePassword`, `keyAlias` and `keyPassword` capabilities. Unset by default
appium:keystorePassword | The password to the keystore file provided in `keystorePath` capability. This capability is used in combination with `useKeystore`, `keystorePath`, `keystorePassword`, `keyAlias` and `keyPassword` capabilities. Unset by default
appium:keyAlias | The alias of the key in the keystore file provided in `keystorePath` capability. This capability is used in combination with `useKeystore`, `keystorePath`, `keystorePassword`, `keyAlias` and `keyPassword` capabilities. Unset by default
appium:keyPassword | The password of the key in the keystore file provided in `keystorePath` capability. This capability is used in combination with `useKeystore`, `keystorePath`, `keystorePassword`, `keyAlias` and `keyPassword` capabilities. Unset by default
appium:noSign | Set it to `true` in order to skip application signing. By default all apps are always signed with the default Appium debug signature. This capability cancels all the signing checks and makes the driver to use the application package as is. This capability does not affect `.apks` packages as these are expected to be already signed. Make sure that the server package is signed with the same signature as the application under test before disabling this capability.

### Device Locking

Capability Name | Description
--- | ---
appium:skipUnlock | Whether to skip the check for lock screen presence (`true`). By default Espresso driver tries to detect if the device's screen is locked before starting the test and to unlock that (which sometimes might be unstable). Note, that this operation takes some time, so it is highly recommended to set this capability to `false` and disable screen locking on devices under test.
appium:unlockType | Set one of the possible types of Android lock screens to unlock. Read the [Unlock tutorial](https://github.com/appium/appium-android-driver/blob/master/docs/UNLOCK.md) for more details.
appium:unlockKey | Allows to set an unlock key. Read the [Unlock tutorial](https://github.com/appium/appium-android-driver/blob/master/docs/UNLOCK.md) for more details.
appium:unlockStrategy | Either 'locksettings' (default) or 'uiautomator'. Setting it to 'uiautomator' will enforce the driver to avoid using special ADB shortcuts in order to speed up the unlock procedure.
appium:unlockSuccessTimeout | Maximum number of milliseconds to wait until the device is unlocked. `2000` ms by default

### Web Context

Capability Name | Description
--- | ---
appium:autoWebview | If set to `true` then Espresso driver will try to switch to the first available web view after the session is started. `false` by default.
appium:webviewDevtoolsPort | The local port number to use for devtools communication. By default the first free port from 10900..11000 range is selected. Consider setting the custom value if you are running parallel tests.
appium:ensureWebviewsHavePages | Whether to skip web views that have no pages from being shown in `getContexts` output. The driver uses devtools connection to retrieve the information about existing pages. `true` by default since Appium 1.19.0, `false` if lower than 1.19.0.
appium:enableWebviewDetailsCollection | Whether to retrieve extended web views information using devtools protocol. Enabling this capability helps to detect the necessary chromedriver version more precisely. `true` by default since Appium 1.22.0, `false` if lower than 1.22.0.
appium:chromedriverPort | The port number to use for Chromedriver communication. Any free port number is selected by default if unset.
appium:chromedriverPorts | Array of possible port numbers to assign for Chromedriver communication. If none of the port in this array is free then an error is thrown.
appium:chromedriverArgs | Array of chromedriver [command line arguments](http://www.assertselenium.com/java/list-of-chrome-driver-command-line-arguments/). Note, that not all command line arguments that are available for the desktop browser are also available for the mobile one.
appium:chromedriverExecutable | Full path to the chromedriver executable on the server file system.
appium:chromedriverExecutableDir | Full path to the folder where chromedriver executables are located. This folder is used then to store the downloaded chromedriver executables if automatic download is enabled. Read [Automatic Chromedriver Discovery article](docs/hybrid-mode.md#automatic-discovery-of-compatible-chromedriver) for more details.
appium:chromedriverChromeMappingFile | Full path to the chromedrivers mapping file. This file is used to statically map webview/browser versions to the chromedriver versions that are capable of automating them. Read [Automatic Chromedriver Discovery article](docs/hybrid-mode.md#automatic-discovery-of-compatible-chromedriver) for more details.
appium:chromedriverUseSystemExecutable | Set it to `true` in order to enforce the usage of chromedriver, which gets downloaded by Appium automatically upon installation. This driver might not be compatible with the destination browser or a web view. `false` by default.
appium:chromedriverDisableBuildCheck | Being set to `true` disables the compatibility validation between the current chromedriver and the destination browser/web view. Use it with care.
appium:autoWebviewTimeout | Set the maximum number of milliseconds to wait until a web view is available if `autoWebview` capability is set to `true`. `2000` ms by default
appium:recreateChromeDriverSessions | If this capability is set to `true` then chromedriver session is always going to be killed and then recreated instead of just suspending it on context switching. `false` by default
appium:nativeWebScreenshot | Whether to use screenshoting endpoint provided by Espresso framework (`true`) rather than the one provided by chromedriver (`false`, the default value). Use it when you experience issues with the latter.
appium:extractChromeAndroidPackageFromContextName | If set to `true`, tell chromedriver to attach to the android package we have associated with the context name, rather than the package of the application under test. `false` by default.
appium:showChromedriverLog | If set to `true` then all the output from chromedriver binary will be forwarded to the Appium server log. `false` by default.
pageLoadStrategy | One of the available page load strategies. See https://www.w3.org/TR/webdriver/#capabilities
appium:chromeOptions | A mapping, that allows to customize chromedriver options. See https://chromedriver.chromium.org/capabilities for the list of available entries.

### Other

Capability Name | Description
--- | ---
appium:disableSuppressAccessibilityService | Being set to `true` tells the instrumentation process to not suppress accessibility services during the automated test. This might be useful if your automated test needs these services. `false` by default
appium:disableWindowAnimation | To avoid flakiness google [recommends](https://developer.android.com/training/testing/espresso/setup#set-up-environment) to disable the window animation of the Android device under test when running espresso test. `true` by default


## Settings API

Espresso driver supports Appium [Settings API](https://appium.io/docs/en/advanced-concepts/settings/).
Along with the common settings the following driver-specific settings are currently available:

Name | Type | Description
--- | --- | ---
driver | 'compose' or 'espresso' | The name of the subdriver to use for elements interactions. The default value is `espresso`. Switching the value to `compose` enables interactions with [Jetpack Compose](https://developer.android.com/jetpack/compose)-based application user interfaces. Read [Jetpack Compose Support](#jetpack-compose-support) for more details.


## Jetpack Compose Support

[Jetpack Compose](https://developer.android.com/jetpack/compose) is Android’s modern toolkit for building native UI. Espresso driver supports basic interactions with Compose-based applications since version *1.46.0*.

[Appium UiAutomator2 driver](https://github.com/appium/appium-uiautomator2-driver/) allows to interact with Jetpack Compose elements via the accessibility layer by providing [testTag](https://developer.android.com/reference/kotlin/androidx/compose/ui/platform/package-summary#(androidx.compose.ui.Modifier).testTag(kotlin.String)) modifier attribute or the displayed text, but this Espresso driver allows you to access Jetpack Compose elements directly.


### Interaction With Compose Elements

Espresso driver has the concept of subdrivers. This works quite similarly to the concept of contexts, while contexts are used to switch between native and web, and subdrivers are still
under the native one. Each subdriver operates its own elements cache, so it is not be possible
to mix Espresso and Compose elements.

In order to change between subdrivers use the [driver](#settings-api) setting. Setting its value to `compose` modifies driver behavior in the way it interacts with Compose elements rather that with classic Android views. It is possible to switch between `espresso` and `compose` modes at any point of time. When `compose` mode is active the the following webdriver commands behave differently (as of driver version *1.50.0*):
- findElement(s): Element finding commands only support Compose-based locators. Read [Compose Elements Location](#compose-elements-location) for more details.
- getPageSource: The returned page source is retrieved from Compose and all elements there contain [Compose-specific](#compose-element-attributes) attributes.
- click, isDisplayed, isEnabled, clear, getText, sendKeys, getElementRect, getValue, isSelected: These commands should properly support compose elements.
- getAttribute: Accepts and returns Compose-specific element attributes. See [Compose Element Attributes](#compose-element-attributes) for the full list of supported Compose element attributes.
- getElementScreenshot: Fetches a screenshot of the given Compose element. Available since driver version *2.14.0*
- `mobile: swipe`: Performs swipe gesture on the given element in the given direction.
The `swiper` argument is not supported in Compose mode. Available since driver version *2.15.0*

Calling other driver element-specific APIs not listed above would most likely throw an exception as Compose and Espresso elements are being stored in completely separated internal caches and must not be mixed.

You could also check end-to-end tests for more examples on how to setup test capabilities and
on the Compose usage in general:
- https://github.com/appium/appium-espresso-driver/blob/master/test/functional/commands/jetpack-componse-element-values-e2e-specs.js
- https://github.com/appium/appium-espresso-driver/blob/master/test/functional/commands/jetpack-compose-attributes-e2e-specs.js
- https://github.com/appium/appium-espresso-driver/blob/master/test/functional/commands/jetpack-compose-e2e-specs.js
- https://github.com/appium/appium-espresso-driver/blob/master/test/functional/commands/jetpack-compose-e2e-specs.js


## Espresso Build Config

Espresso server is in tight connection with the application under test. That is why it is important that the server uses the same versions of common dependencies and there are no conflicts. Espresso driver allows to configure several build options via `espressoBuildConfig` capability. The configuration JSON supports the following entries:

### toolsVersions

This entry allows to explicitly set the versions of different server components. The following map entries are supported:

Name | Description | Example
--- | --- | ---
gradle | The Gradle version to use for Espresso server building. | '6.3'
androidGradlePlugin | The Gradle plugin version to use for Espresso server building. By default the version from the [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/build.gradle.kts) is used | '4.1.1'
compileSdk | Android SDK version to compile the server for. By default the version from the app [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/build.gradle.kts) is used | 28
buildTools | Target Android build tools version to compile the server with. By default the version from the app [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/build.gradle.kts) is used | '28.0.3'
minSdk | Minimum Android SDK version to compile the server for. By default the version from the app [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/build.gradle.kts) is used | 18
targetSdk | Target Android SDK version to compile the server for. By default the version from the app [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/build.gradle.kts) is used | 28
kotlin | Kotlin version to compile the server for. By default the version from the [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/build.gradle.kts) is used | '1.3.72'
composeVersion | The version for the Jetpack Compose dependencies to use for Espresso server building. By default the version from the [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/build.gradle.kts) is used | '1.1.1'
espressoVersion | The version for the Espresso dependencies to use for Espresso server building. By default the version from the [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/build.gradle.kts) is used | '3.5.0'
sourceCompatibility | The minimum version of JVM the project sources are compatible with. The default value is `VERSION_1_8` | VERSION_12
targetCompatibility | The target version of JVM the project sources are compatible with. The default value is `VERSION_1_8` | VERSION_12
jvmTarget | Target version of the generated JVM bytecode as a string. The default value is `1_8` | `1_10`
annotationVersion | The target version of `androidx.annotation:annotation` pakage. By default the version from the app [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/build.gradle.kts) is used | '1.2.0'

### additionalAppDependencies

The value of this entry must be a non empty array of dependent module names with their versions. The scripts adds all these items as `implementation` lines of `dependencies` category in the app [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/build.gradle.kts) script. Example: `["xerces.xercesImpl:2.8.0", "xerces.xmlParserAPIs:2.6.2"]`

### additionalAndroidTestDependencies

The value of this entry must be a non empty array of dependent module names with their versions. The scripts adds all these items as `androidTestImplementation` lines of `dependencies` category in the app [build.gradle.kts](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/build.gradle.kts) script. Example: `["xerces.xercesImpl:2.8.0", "xerces.xmlParserAPIs:2.6.2"]`

### Full JSON Example

```json
{
  "toolsVersions": {
    "androidGradlePlugin": "4.0.0"
  },
  "additionalAndroidTestDependencies": ["xerces.xercesImpl:2.8.0", "xerces.xmlParserAPIs:2.6.2"]
}
```


## Intent Options

By default Espresso creates the following intent to start the app activity:

```json
{
  "action": "ACTION_MAIN",
  "flags": "ACTIVITY_NEW_TASK",
  "className": "<fullyQualifiedAppActivity>"
}
```

Although, it is possible to fully customize these options by providing the `intentOptions` capability. Read [Intent documentation](https://developer.android.com/reference/android/content/Intent) for more details on this topic. The value of this capability is expected to be a map with the following entries:

Name | Type | Description | Example
--- | --- | --- | ---
action | string | An action name. Application-specific actions should be prefixed with the vendor's package name. | ACTION_MAIN
data | string | Intent data URI | content://contacts/people/1
type | string | Intent MIME type | image/png
categories | string | One or more comma-separated Intent categories | android.intent.category.APP_CONTACTS
component | string | Component name with package name prefix to create an explicit intent | com.example.app/.ExampleActivity
intFlags | string | Single string value, which represents intent flags set encoded into an integer. Could also be provided in hexadecimal format. Check [setFlags method documentation](https://developer.android.com/reference/android/content/Intent.html#setFlags(int)) for more details. | 0x0F
flags | Comma-separated string of intent flag names | 'FLAG_GRANT_READ_URI_PERMISSION, ACTIVITY_CLEAR_TASK' (the 'FLAG_' prefix could be omitted)
className | The name of a class inside of the application package that will be used as the component for this Intent | com.example.app.MainActivity
e or es | `Map<string, string>` | Intent string parameters | {'foo': 'bar'}
esn | `Array<string>` | Intent null parameters | ['foo', 'bar']
ez | `Map<string, boolean>` | Intent boolean parameters | {'foo': true, 'bar': false}
ei | `Map<string, int>` | Intent integer parameters | {'foo': 1, 'bar': 2}
el | `Map<string, long>` | Intent long integer parameters | {'foo': 1L, 'bar': 2L}
ef | `Map<string, float>` | Intent float parameters | {'foo': 1.ff, 'bar': 2.2f}
eu | `Map<string, string>` | Intent URI-data parameters | {'foo': 'content://contacts/people/1'}
ecn | `Map<string, string>` | Intent component name parameters | {'foo': 'com.example.app/.ExampleActivity'}
esa | `Map<string, List<string>>` | Intent string array parameters | {'foo': ['bar1','bar2','bar3','bar4']}
eia | `Map<string, string>` | Intent integer array parameters | {'foo': '1,2,3,4'}
ela | `Map<string, string>` | Intent long array parameters | {'foo': '1L,2L,3L,4L'}
efa | `Map<string, string>` | Intent float array parameters | {'foo': '1.1,2.2,3.2,4.4'}


## Activity Options

Espresso driver allows to customize several activity startup options using `activityOptions` capability. The capability value is expected to be a map with the following entries:

Name | Type | Description | Example
--- | --- | --- | ---
launchDisplayId | string or int | Display id which you want to assign to launch the main app activity on. This might be useful if the device under test supports multiple displays | 1


## Espresso Element Attributes

Espresso driver supports the following element attributes in `espresso` subdriver:

Name | Description | Example
--- | --- | ---
checkable | Whether the element is checkable or not | 'true'
checked | Whether the element is checked. Always `false` if the element is not checkable | 'false'
class | The full name of the element's class. Could be `null` for some elements | 'android.view.View'
clickable | Whether the element could be clicked | 'false'
content-desc | The content-description attribute of the accessible element | 'foo'
enabled | Whether the element could be clicked | 'true'
focusable | Whether the element could be focused | 'true'
focused | Whether the element could is focused. Always `false` if the element is not focusable | 'false'
long-clickable | Whether the element accepts long clicks | 'false'
package | Identifier of the package the element belongs to | 'com.mycompany'
password | Whether the element is a password input field | 'true'
resource-id | Element's resource identifier. Could be `null` | 'com.mycompany:id/resId'
scrollable | Whether the element is scrollable | 'true'
selected | Whether the element is selected | 'false'
text | The element's text. It never equals to `null` | 'my text'
hint | The element's hint. Could be `null` | 'my hint text'
bounds | The element's visible frame (`[left, top][right, bottom]`) | `[0,0][100,100]`
no-multiline-buttons | Whether the element's view hierarchy does not contain multiline buttons | 'true'
no-overlaps | Whether element's descendant objects assignable to TextView or ImageView do not overlap each other | 'true'
no-ellipsized-text | Whether the element's view hierarchy does not contain ellipsized or cut off text views | 'false'
visible | Whether the element is visible to the user | 'true'
view-tag | The tag value assigned to the element. Could be `null` | 'my tag'


## Compose Element Attributes

Espresso driver supports the following element attributes in `compose` subdriver:

Name | Description | Example
--- | --- | ---
bounds | The element's visible frame (`[left, top][right, bottom]`) | `[0,0][100,100]`
checked | Whether the element is checked. Always `false` if the element is not checkable | 'false'
class | The full name of the element's class. Could be `ComposeNode` for some elements | 'ComposeNode'
clickable | Whether the element could be clicked | 'false'
content-desc | The content-description attribute of the accessible element | 'foo'
enabled | Whether the element could be clicked | 'true'
focused | Whether the element could is focused. Always `false` if the element is not focusable | 'false'
index | Element's index in the tree hierarchy | `0`
password | Whether the element is a password input field | 'true'
resource-id | Element's resource identifier. Could be `null` | 'com.mycompany:id/resId'
scrollable | Whether the element is scrollable | 'true'
selected | Whether the element is selected | 'false'
text | The element's text | 'my text'
view-tag | The [testTag](https://developer.android.com/reference/kotlin/androidx/compose/ui/platform/package-summary#(androidx.compose.ui.Modifier).testTag(kotlin.String)) element's value. Could be `null` | 'my tag'


## Espresso Elements Location

Espresso driver supports the following location strategies in `espresso` subdriver:

Name | Description | Speed Ranking | Example
--- | --- | --- | ---
id | This strategy is mapped to the native Espresso `withId` [matcher](https://developer.android.com/reference/androidx/test/espresso/matcher/ViewMatchers#withId(org.hamcrest.Matcher%3Cjava.lang.Integer%3E)) (exact match of element's resource id). Package identifier prefix is added automatically if unset and is equal to the identifier of the current application under test. | `⭐⭐⭐⭐⭐` | 'com.mycompany:id/resourceId'
accessibility id | This strategy is mapped to the native Espresso `withContentDescription` [matcher](https://developer.android.com/reference/androidx/test/espresso/matcher/ViewMatchers#withcontentdescription_1) (exact match of element's content description). | `⭐⭐⭐⭐⭐` | 'my description'
class name | This strategy is mapped to the native Espresso `withClassName` [matcher](https://developer.android.com/reference/androidx/test/espresso/matcher/ViewMatchers#withClassName(org.hamcrest.Matcher%3Cjava.lang.String%3E)) (exact match of element's class name). | `⭐⭐⭐⭐⭐` | 'android.view.View'
text | This strategy is mapped to the native Espresso `withText` [matcher](https://developer.android.com/reference/androidx/test/espresso/matcher/ViewMatchers#withText(org.hamcrest.Matcher%3Cjava.lang.String%3E)) (exact match of element's text). | `⭐⭐⭐⭐⭐` | 'my text'
`-android viewtag` or `tag name` | This strategy is mapped to the native Espresso `withTagValue` [matcher](https://developer.android.com/reference/androidx/test/espresso/matcher/ViewMatchers#withtagvalue) (exact match of element's tag value). | `⭐⭐⭐⭐⭐` | 'my tag'
-android datamatcher | This strategy allows to create Espresso [data interaction](https://developer.android.com/reference/android/support/test/espresso/DataInteraction) selectors which can quickly and reliably scroll to the necessary elements. Read [Espresso DataMatcher Selector](docs/espresso-datamatcher-selector.md) to know more on how to construct these locators. Also check the [Unlocking New Testing Capabilities with Espresso Driver by Daniel Graham](https://www.youtube.com/watch?v=gU9EEUV5n9U) presentation video from Appium Conf 2019. | `⭐⭐⭐⭐` | {"name": "hasEntry", "args": ["title", "WebView3"]}
-android viewmatcher | This strategy allows constructing of Espresso [view matchers](https://developer.android.com/reference/androidx/test/espresso/matcher/ViewMatchers) based on the given JSON representation of them. The representation is expected to contain the following fields: `name`: _mandatory_ matcher function name; `args`: _optional_ matcher function arguments, each argument could also be a function; `class`: _optional_ full qualified class name of the corresponding matcher (if not provided then [org.hamcrest.Matchers](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html) one is used), `scope` (since Espresso driver 2.11.0): _optional_ JSON representation of a [RootMatchers](https://developer.android.com/reference/androidx/test/espresso/matcher/RootMatchers) method. Check [unit tests](https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/src/test/java/io/appium/espressoserver/test/model/HamcrestMatcherTest.kt) for more examples. | `⭐⭐⭐⭐` | {"name": "withText", "args": [{"name": "containsString", "args": "getExternalStoragePublicDirectory", "class": "org.hamcrest.Matchers"}], "class": "androidx.test.espresso.matcher.ViewMatchers"}
xpath | For elements lookup Xpath strategy the driver uses the same XML tree that is generated by page source API. Only Xpath 1.0 is supported. | `⭐⭐⭐` | By.xpath("//android.view.View[@text=\"Regular\" and @checkable=\"true\"]")


## Compose Elements Location

Espresso driver supports the following location strategies in `compose` subdriver:

Name | Description | Speed Ranking | Example
--- | --- | --- | ---
accessibility id | This strategy is mapped to the native Espresso `hasContentDescription` [matcher](https://developer.android.com/reference/kotlin/androidx/compose/ui/test/package-summary#hasContentDescription(kotlin.String,kotlin.Boolean,kotlin.Boolean)) (exact match of element's content description). | `⭐⭐⭐⭐⭐` | 'my description'
`-android viewtag` or `tag name` | This strategy is mapped to the native Compose `hasTestTag` [matcher](https://developer.android.com/reference/kotlin/androidx/compose/ui/test/package-summary#hasTestTag(kotlin.String)) (exact match of element's tag value). | `⭐⭐⭐⭐⭐` | 'my tag'
`text` or `link text` | This strategy is mapped to the native Compose `hasText` [matcher](https://developer.android.com/reference/kotlin/androidx/compose/ui/test/package-summary#hasText(kotlin.String,kotlin.Boolean,kotlin.Boolean)) (exact match of element's text). | `⭐⭐⭐⭐⭐` | 'my text'
xpath | For elements lookup Xpath strategy the driver uses the same XML tree that is generated by page source API. Only Xpath 1.0 is supported. | `⭐⭐⭐` | By.xpath("//ComposeNode[@text=\"Regular\" and @selected=\"true\"]")


## Platform-Specific Extensions

Beside of standard W3C APIs the driver provides the following custom command extensions to execute platform specific scenarios:

### mobile: shell

Executes the given shell command on the device under test via ADB connection. This extension exposes a potential security risk and thus is only enabled when explicitly activated by the `adb_shell` server command line feature specifier

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
command | string | yes | Shell command name to execute, for example `echo` or `rm` | echo
args | `Array<string>` | no | Array of command arguments | `['-f', '/sdcard/myfile.txt']`
timeout | number | no | Command timeout in milliseconds. If the command blocks for longer than this timeout then an exception is going to be thrown. The default timeout is `20000` ms | 100000
includeStderr | boolean | no | Whether to include stderr stream into the returned result. `false` by default | true

#### Returns

Depending on the `includeStderr` value this API could either return a string, which is equal to the `stdout` stream content of the given command or a dictionary whose elements are `stdout` and `stderr` and values are contents of the corresponding outgoing streams. If the command exits with a non-zero return code then an exception is going to be thrown. The exception message will be equal to the command stderr.

### mobile: execEmuConsoleCommand

Executes a command through emulator telnet console interface and returns its output.
The `emulator_console` server feature must be enabled in order to use this method.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
command | string | yes | The actual command to execute. See [Android Emulator Console Guide](https://developer.android.com/studio/run/emulator-console) for more details on available commands | help-verbose
execTimeout | number | no | Timeout used to wait for a server reply to the given command in milliseconds. `60000` ms by default | 100000
connTimeout | boolean | no | Console connection timeout in milliseconds. `5000` ms by default | 10000
initTimeout | boolean | no | Telnet console initialization timeout in milliseconds (the time between the connection happens and the command prompt). `5000` ms by default | 10000

#### Returns

The actual command output. An error is thrown if command execution fails.

### mobile: performEditorAction

Performs IME action on the _currently focused_ edit element.

Very often Android developers use [onEditorAction](https://developer.android.com/reference/android/widget/TextView.OnEditorActionListener.html#onEditorAction(android.widget.TextView,%20int,%20android.view.KeyEvent)) callback with `actionId` argument to implement actions handling, for example, when `Search` or `Done` button is pressed on the on-screen keyboard. This mobile extension is supposed to emulate the invokation of such callback on the focused element.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
action | string | yes | The name or an integer code of the editor action to be executed. The following action names are supported: `normal, unspecified, none, go, search, send, next, done, previous`. Read [EditorInfo](https://developer.android.com/reference/android/view/inputmethod/EditorInfo) for more details on this topic. | search

#### Examples

```java
// Java
driver.executeScript("mobile: performEditorAction", ImmutableMap.of("action", "Go"));
```

```python
# Python
driver.execute_script('mobile: performEditorAction', {'action': 'previous'})
```

### mobile: changePermissions

Changes package permissions in runtime.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
permissions | string or `Array<string>` | yes | The full name of the permission to be changed or a list of permissions. Mandatory argument. | `['android.permission.ACCESS_FINE_LOCATION', 'android.permission.BROADCAST_SMS']`
appPackage | string | no | The application package to set change permissions on. Defaults to the package name under test | com.mycompany.myapp
action | string | no | Either `grant` (the default action) or `revoke` | grant

### mobile: getPermissions

Gets runtime permissions list for the given application package.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
type | string | no | One of possible permission types to get. Can be one of: `denied`, `granted` or `requested` (the default value). | granted
appPackage | string | no | The application package to get permissions from. Defaults to the package name under test | com.mycompany.myapp

#### Returns

Array of strings, where each string is a permission name. the array could be empty.

### mobile: startScreenStreaming

Starts device screen broadcast by creating MJPEG server. Multiple calls to this method have no effect unless the previous streaming session is stopped. This method only works if the `adb_screen_streaming` feature is enabled on the server side. It is also required that [GStreamer](https://gstreamer.freedesktop.org/) with `gst-plugins-base`, `gst-plugins-good` and `gst-plugins-bad` packages is installed and available in PATH on the server machine.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
width | number | no | The scaled width of the device's screen. If unset then the script will assign it to the actual screen width measured in pixels. | 768
height | number | no | The scaled height of the device's screen. If unset then the script will assign it to the actual screen height measured in pixels. | 1024
bitRate | number | no | The video bit rate for the video, in bits per second. The default value is 4000000 (4 Mb/s). You can increase the bit rate to improve video quality, but doing so results in larger movie files. | 1024000
host | string | no | The IP address/host name to start the MJPEG server on. You can set it to `0.0.0.0` to trigger the broadcast on all available network interfaces. `127.0.0.1` by default | 0.0.0.0
pathname | string | no | The HTTP request path the MJPEG server should be available on. If unset then any pathname on the given `host`/`port` combination will work. Note that the value should always start with a single slash: `/` | /myserver
tcpPort | number | no | The port number to start the internal TCP MJPEG broadcast on. This type of broadcast always starts on the loopback interface (`127.0.0.1`). `8094` by default | 5024
port | number | no | The port number to start the MJPEG server on. `8093` by default | 5023
quality | number | no | The quality value for the streamed JPEG images. This number should be in range [1, 100], where 100 is the best quality. `70` by default | 80
considerRotation | boolean | no | If set to `true` then GStreamer pipeline will increase the dimensions of the resulting images to properly fit images in both landscape and portrait orientations. Set it to `true` if the device rotation is not going to be the same during the broadcasting session. `false` by default | false
logPipelineDetails | boolean | no | Whether to log GStreamer pipeline events into the standard log output. Might be useful for debugging purposes. `false` by default | true

### mobile: stopScreenStreaming

Stop the previously started screen streaming. If no screen streaming server has been started then nothing is done.

### mobile: deviceInfo

Retrieves the information about the device under test, like the device model, serial number, network connectivity info, etc.

#### Returns

The extension returns a dictionary whose entries are the device properties. Check https://github.com/appium/appium-espresso-driver/blob/master/espresso-server/app/src/androidTest/java/io/appium/espressoserver/lib/handlers/GetDeviceInfo.kt to get the full list of returned keys and their corresponding values.

### mobile: swipe

Perform swipe action. Invokes Espresso [swipe action](https://developer.android.com/reference/android/support/test/espresso/action/Swipe) under the hood.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
element | string | yes | The UDID of the element to perform the swipe on. | 123456-7890-3453-24234243
direction | string | no | Swipe direction. Either this argument or `swiper` must be provided, but not both. The following values are supported: `up`, `down`, `left`, `right` | down
swiper | string | no | Swipe speed. Either this argument or `direction` must be provided, but not both. Either `FAST` (Swipes quickly between the co-ordinates) or `SLOW` (Swipes deliberately slowly between the co-ordinates, to aid in visual debugging) | SLOW
startCoordinates | string | no | The starting coordinates for the action. The following values are supported: `TOP_LEFT`, `TOP_CENTER`, `TOP_RIGHT`, `CENTER_LEFT`, `CENTER`, `CENTER_RIGHT`, `BOTTOM_LEFT`, `BOTTOM_CENTER` (the default value), `BOTTOM_RIGHT`, `VISIBLE_CENTER` | CENTER_LEFT
endCoordinates | string | no | The ending coordinates for the action. The following values are supported: `TOP_LEFT`, `TOP_CENTER` (the default value), `TOP_RIGHT`, `CENTER_LEFT`, `CENTER`, `CENTER_RIGHT`, `BOTTOM_LEFT`, `BOTTOM_CENTER`, `BOTTOM_RIGHT`, `VISIBLE_CENTER` | TOP_LEFT
precisionDescriber | string | no | Defines the actual swipe precision. The following values are supported: `PINPOINT` (1px), `FINGER` (average width of the index finger is 16 – 20 mm), `THUMB` (average width of an adult thumb is 25 mm or 1 inch, the default value) | FINGER

### mobile: isToastVisible

Checks whether a toast notification with the given text is currently visible.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
text | string | yes | The actual toast test or a part of it | 'toast text'
isRegexp | boolean | no | Whether the `text` value should be parsed as a regular expression (`true`) or as a raw text (`false`, the default value) | false

#### Returns

Either `true` or `false`

### mobile: openDrawer

Opens the DrawerLayout drawer with the gravity. This method blocks until the drawer is fully open. No operation if the drawer is already open.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
element | string | yes | UDID of the element to perform the action on. | 123456-7890-3453-24234243
gravity | int | no | See [GravityCompat](https://developer.android.com/reference/kotlin/androidx/core/view/GravityCompat) and [Gravity](https://developer.android.com/reference/android/view/Gravity) classes documentation | `0x00800000 <bitwise_or> 0x00000003`

### mobile: closeDrawer

Closes the DrawerLayout drawer with the gravity. This method blocks until the drawer is fully closed. No operation if the drawer is already closed.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
element | string | yes | UDID of the element to perform the action on. | 123456-7890-3453-24234243
gravity | int | no | See [GravityCompat](https://developer.android.com/reference/kotlin/androidx/core/view/GravityCompat) and [Gravity](https://developer.android.com/reference/android/view/Gravity) classes documentation | `0x00800000 <bitwise_or> 0x00000005`

### mobile: scrollToPage

Perform scrolling to the given page. Invokes one of the [ViewPagerActions](https://developer.android.com/reference/androidx/test/espresso/contrib/ViewPagerActions) under the hood. Which action is invoked depends on the given arguments.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
element | string | yes | UDID of the element to perform the action on. | 123456-7890-3453-24234243
scrollTo | string | no if `scrollToPage` is provided | Shifts ViewPager to the given page. Supported values are: `first`, `last`, `left`, `right` | last
scrollToPage | int | no if `scrollTo` is provided | Moves ViewPager to a specific page number (numbering starts from zero). | 1
smoothScroll | boolean | no | Whether to perform smooth (but slower) scrolling (`true`). The default value is `false` | true

### mobile: navigateTo

Invokes [navigateTo](https://developer.android.com/reference/androidx/test/espresso/contrib/NavigationViewActions#navigateto) action under the hood.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
element | string | yes | UDID of the element to perform the action on. View constraints: View must be a child of a DrawerLayout; View must be of type NavigationView; View must be visible on screen; View must be displayed on screen | 123456-7890-3453-24234243
menuItemId | int | yes | The resource id of the destination menu item | 123

### mobile: clickAction

Perform [general click action](https://developer.android.com/reference/androidx/test/espresso/action/GeneralClickAction).

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
element | string | yes | The UDID of the element to perform the click on. | 123456-7890-3453-24234243
tapper | string | no | Tapper type. Supported types are: `SINGLE` (the default value), `LONG`, `DOUBLE` | `LONG`
coordinatesProvider | string | no | The coordinates for the action. The following values are supported: `TOP_LEFT`, `TOP_CENTER`, `TOP_RIGHT`, `CENTER_LEFT`, `CENTER`, `CENTER_RIGHT`, `BOTTOM_LEFT`, `BOTTOM_CENTER`, `BOTTOM_RIGHT`, `VISIBLE_CENTER` (the default value) | CENTER_LEFT
precisionDescriber | string | no | Defines the actual click precision. The following values are supported: `PINPOINT` (1px), `FINGER` (average width of the index finger is 16 – 20 mm, the default value), `THUMB` (average width of an adult thumb is 25 mm or 1 inch) | PINPOINT
inputDevice | int | no | Input device identifier, `0` by default | 1
buttonState | int | no | Button state id, `0` by default | 1

### mobile: getContexts

Retrieves a webviews mapping based on CDP endpoints

#### Returns

The following json demonstrates the example of WebviewsMapping object.
Note that `description` in `page` can be an empty string most likely when it comes to Mobile Chrome)

```json
 {
   "proc": "@webview_devtools_remote_22138",
   "webview": "WEBVIEW_22138",
   "info": {
     "Android-Package": "io.appium.settings",
     "Browser": "Chrome/74.0.3729.185",
     "Protocol-Version": "1.3",
     "User-Agent": "Mozilla/5.0 (Linux; Android 10; Android SDK built for x86 Build/QSR1.190920.001; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/74.0.3729.185 Mobile Safari/537.36",
     "V8-Version": "7.4.288.28",
     "WebKit-Version": "537.36 (@22955682f94ce09336197bfb8dffea991fa32f0d)",
     "webSocketDebuggerUrl": "ws://127.0.0.1:10900/devtools/browser"
   },
   "pages": [
     {
       "description": "{\"attached\":true,\"empty\":false,\"height\":1458,\"screenX\":0,\"screenY\":336,\"visible\":true,\"width\":1080}",
       "devtoolsFrontendUrl": "http://chrome-devtools-frontend.appspot.com/serve_rev/@22955682f94ce09336197bfb8dffea991fa32f0d/inspector.html?ws=127.0.0.1:10900/devtools/page/27325CC50B600D31B233F45E09487B1F",
       "id": "27325CC50B600D31B233F45E09487B1F",
       "title": "Releases · appium/appium · GitHub",
       "type": "page",
       "url": "https://github.com/appium/appium/releases",
       "webSocketDebuggerUrl": "ws://127.0.0.1:10900/devtools/page/27325CC50B600D31B233F45E09487B1F"
     }
   ],
   "webviewName": "WEBVIEW_com.io.appium.setting"
 }
```

### mobile: getNotifications

Retrieves Android notifications via Appium Settings helper. Appium Settings app itself must be *manually* granted to access notifications under device Settings in order to make this feature working. Appium Settings helper keeps all the active notifications plus notifications that appeared while it was running in the internal buffer, but no more than 100 items altogether. Newly appeared notifications are always added to the head of the notifications array. The `isRemoved` flag is set to `true` for notifications that have been removed.
See https://developer.android.com/reference/android/service/notification/StatusBarNotification and https://developer.android.com/reference/android/app/Notification.html for more information on available notification properties and their values.

#### Returns

The example output is:
```json
{
   "statusBarNotifications":[
     {
       "isGroup":false,
       "packageName":"io.appium.settings",
       "isClearable":false,
       "isOngoing":true,
       "id":1,
       "tag":null,
       "notification":{
         "title":null,
         "bigTitle":"Appium Settings",
         "text":null,
         "bigText":"Keep this service running, so Appium for Android can properly interact with several system APIs",
         "tickerText":null,
         "subText":null,
         "infoText":null,
         "template":"android.app.Notification$BigTextStyle"
       },
       "userHandle":0,
       "groupKey":"0|io.appium.settings|1|null|10133",
       "overrideGroupKey":null,
       "postTime":1576853518850,
       "key":"0|io.appium.settings|1|null|10133",
       "isRemoved":false
     }
   ]
}
```

### mobile: listSms

Retrieves the list of the most recent SMS properties list via Appium Settings helper. Messages are sorted by date in descending order.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
max | number | no | The maximum count of recent messages to retrieve. `100` by default | 10

#### Returns

The example output is:
```json
 {
   "items":[
     {
       "id":"2",
       "address":"+123456789",
       "person":null,
       "date":"1581936422203",
       "read":"0",
       "status":"-1",
       "type":"1",
       "subject":null,
       "body":"\"text message2\"",
       "serviceCenter":null
     },
     {
       "id":"1",
       "address":"+123456789",
       "person":null,
       "date":"1581936382740",
       "read":"0",
       "status":"-1",
       "type":"1",
       "subject":null,
       "body":"\"text message\"",
       "serviceCenter":null
     }
   ],
   "total":2
 }
```

### mobile: sensorSet

Emulate changing of sensor values on the connected emulator.
This extension does not work on real devices.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
sensorType | string | yes | The set of all supported sensor types could be found in [adb-emu-commands.js](https://github.com/appium/appium-adb/blob/master/lib/tools/adb-emu-commands.js) (look for *SENSORS* object values). Check the output of `sensor status` command in the [emulator console](https://developer.android.com/studio/run/emulator-console) to see more details on the available sensor types | light
value | string | yes | Check the output of `sensor get <sensorType>` command in the [emulator console](https://developer.android.com/studio/run/emulator-console) to see the acceptable value format for the given sensor type | 50

### mobile: refreshGpsCache

Sends a request to refresh the GPS cache on the device under test.
By default the location tracking is configured for
[low battery consumption](https://github.com/appium/io.appium.settings/blob/master/app/src/main/java/io/appium/settings/LocationTracker.java),
so you might need to call this extension periodically to get the updated geo
location if the actual (or mocked) device location is changed too frequently.
The feature only works if the device under test has Google Play Services installed.
In case the vanilla
[LocationManager](https://developer.android.com/reference/android/location/LocationManager)
is used the device API level must be at version 30 (Android R) or higher.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
timeoutMs | number | no | The maximum number of milliseconds to block until GPS cache is refreshed. If the API call does not receive a confirmation about successful cache refresh within this timeout then an error is thrown. Providing zero or a negative value to it skips waiting completely and does not check for any errors. 20000 ms by default. | 60000

### mobile: pullFile

Pulls a remote file from the device.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
remotePath | string | yes | The full path to the remote file or a specially formatted path, which points to an item inside an app bundle, for example `@my.app.id/my/path`. It is mandatory for the app bundle to have [debugging enabled](https://developer.android.com/studio/debug) in order to use the latter remotePath format. If the file with the given name does not exist then an exception will be thrown. | /sdcard/foo.bar

#### Returned Result

Base64-encoded string, which represents the content of the remote file.

### mobile: pushFile

Pushes a local file to the device.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
remotePath | string | yes | The path on the device to where the payload should be written. The value format is similar to the one used in [pullFile](#mobile-pullfile) extension. If the file with the same name already exists then it will be silently overridden. | /sdcard/foo.bar
payload | string | yes | Base64-encoded content of the file to be pushed. | QXBwaXVt

### mobile: pullFolder

Pulls a remote folder from the device.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
remotePath | string | yes | Same as for [pullFile](#mobile-pullfile) extension, but should be pointing to a remote folder | /sdcard/yolo/

#### Returned Result

Base64-encoded string, which represents the zipped content of the remote folder.

### mobile: deleteFile

Deletes a file on the remote device.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
remotePath | string | yes | The full path to the remote file or a file inside an application bundle | `/sdcard/myfile.txt` or `@my.app.id/path/in/bundle`

### mobile: isAppInstalled

Verify whether an application is installed on the device under test.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
appId | string | yes | The identifier of the application package to be checked | `my.app.id`

#### Returned Result

True or false

### mobile: queryAppState

Queries the current state of the app.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
appId | string | yes | The identifier of the application package to be checked | `my.app.id`

#### Returned Result

The following numbers could returned:
- The app is not installed: `0`
- The app is installed and is not running: `1`
- The app is running in background: `3`
- The app is running in foreground: `4`

### mobile: activateApp

Activates the given application or launches it if necessary.
The action literally simulates
clicking the corresponding application icon on the dashboard.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
appId | string | yes | The identifier of the application package to be activated | `my.app.id`

### mobile: removeApp

Remove the corresponding application if is installed.
The call is ignored if the app is not installed.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
appId | string | yes | The identifier of the application package to be removed | `my.app.id`

#### Returned Result

True is the app has been found on the device and successfully removed. Otherwise false.

### mobile: terminateApp

Terminates the app and waits until the app is terminated up to the given timeout
by checking the app state to ensure if the app process is actually stopped.

The app state check can be skipped if the given timeout is lower or equal to zero since Espresso driver 2.13.0.
The skip helps when you want to terminate the app process but do not want to check the process existence
because the app under test may, for example, restart it automatically.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
appId | string | yes | The identifier of the application package to be terminated | `my.app.id`
timeout | number | no | The count of milliseconds to wait until the app is terminated. 500ms by default. | 1500, 0

#### Returned Result

True if the app has been successfully terminated.

### mobile: installApp

Installs the given application package to the device under test.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
appPath | string | yes | The local .apk(s) path on the server filesystem or a remote url. | `/app/path.apk`
timeout | number | no | The count of milliseconds to wait until the app is installed.. 6000ms by default. | 120000
allowTestPackages | boolean | no | Set to true in order to allow test packages installation. false by default | true
useSdcard | boolean | no | Set to true to install the app on sdcard instead of the device memory. false by default | true
grantPermissions | boolean | no | Set to true in order to grant all the permissions requested in the application's manifest automatically after the installation is completed under Android 6+. The targetSdkVersion in the application manifest must be greater or equal to 23 and the Android version on the device under test must be greater or equal to Android 6 (API level 23) to grant permissions. Applications whose targetSdkVersion is lower than or equal to 22 must be reisntalled to grant permissions for Android 6+ devices. false by default | true
replace | boolean | no | Set it to false if you don't want the application to be upgraded/reinstalled if it is already present on the device, but throw an error instead. true by default | false

### mobile: clearApp

Deletes all data associated with a package. Calls `adb shell pm clear` under the hood.
The app should be accessible, should not be running,
and should exist on the device under test for this extension to work properly.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
appId | string | yes | The identifier of the application package to be cleared | `my.app.id`

#### Returned Result

Stdout of the corresponding adb command.

### mobile: startActivity

Starts the given activity with intent options, activity options and locale. Activity could only be executed in scope of the current app package.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
appActivity | string | yes | Applicaiton acitivity identifier to start | `com.myapp.myacitivty`
locale | object | no | Sets the locale for the app under test. It only uses public APIs for its purpose. See https://github.com/libyal/libfwnt/wiki/Language-Code-identifiers to get the list of available language abbreviations. | `{"language": "zh", "country": "CN", "variant": "Hans"}`
optionalIntentArguments | object | no | The mapping of custom options for the intent that is going to be passed to the main app activity. Check [Intent Options](#intent-options) for more details. | `{ 'flags': 'ACTIVITY_NEW_TASK', 'action': '<intent_action>', 'className': '<fullyQualifiedAppActivity>', 'es': {'foo': 'bar'} }`
optionalActivityArguments | object | no | The mapping of custom options for the main app activity that is going to be started. Check [Activity Options](#activity-options) for more details. | `{ 'launchDisplayId': 1 }`

### mobile: startService

Starts the given service intent.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
intent | string | yes | The name of the service intent to start. Only services in the app's under test scope could be started. | `com.some.package.name/.YourServiceSubClassName`
user | number or string | no | The user ID for which the service is started. The `current` user id is used by default | 1006
foreground | boolean | no | Set it to `true` if your service must be started as foreground service. | false

### mobile: stopService

Stops the given service intent.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
intent | string | yes | The name of the service intent to stop. Only services in the app's under test scope could be stopped. | `com.some.package.name/.YourServiceSubClassName`
user | number or string | no | The user ID for which the service is started. The `current` user id is used by default | 1006

### mobile: broadcast

Send a broadcast Intent. Invokes `am broadcast` command under the hood.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
intent | string | no | The full name of the intent to broadcast | `com.some.package.name/.YourIntentClassName`
user | number or string | no | Specify which user to send to; if not specified then send to all users. Possible values are `all`/`current`/`<numeric user id>` | current
receiverPermission | string | no | Require receiver to hold the given permission | android.permission.READ_PROFILE
allowBackgroundActivityStarts | boolean | no | The receiver may start activities even if in the background if set to `true` | false
action | string | no | Action name. The actual value for the Activity Manager's `-a` argument. | android.intent.action.MAIN
uri | string | no | Unified resource identifier. The actual value for the Activity Manager's `-d` argument. | https://appium.io
mimeType | string | no | Mime type. The actual value for the Activity Manager's `-t` argument. | application/json
identifier | string | no | Optional identifier. The actual value for the Activity Manager's `-i` argument. | my_identifier
categories | string or Array&lt;string&gt; | no | One or more category names. The actual value(s) for the Activity Manager's `-c` argument. | android.intent.category.LAUNCHER
component | string | no | Component name. The actual value for the Activity Manager's `-n` argument. | com.myapp/com.myapp.SplashActivity
package | string | no | Package name. The actual value for the Activity Manager's `-p` argument. | com.myapp
extras | Array&lt;Array&lt;string&gt;&gt; | no | Optional intent arguments. Must be represented as an array of arrays, where each subarray item contains two (only in case it no value is required for the given type) or three string items: value type, key (variable name) and the value itself. Supported value types are: `s`: string. Value must be a valid string; `sn`: null. Value is ignored for this type; `z`: boolean. Value must be either `true` or `false`; `i`: integer. Value must be a valid 4-byte integer number; `l`: long. Value must be a valid 8-byte long number; `f`: float: Value must be a valid float number; `u`: uri. Value must be a valid uniform resource identifier string; `cn`: component name. Value must be a valid component name string; `ia`: Integer[]. Value must be a string of comma-separated integers; `ial`: List&lt;Integer&gt;. Value must be a string of comma-separated integers; `la`: Long[]. Value must be a string of comma-separated long numbers; `lal`: List&lt;Long&gt;. Value must be a string of comma-separated long numbers; `fa`: Float[]. Value must be a string of comma-separated float numbers; `fal`: List&lt;Float&gt;. Value must be a string of comma-separated float numbers; `sa`: String[]. Value must be comma-separated strings. To embed a comma into a string escape it using "\,"; `sal`: List&lt;String&gt;. Value must be comma-separated strings. To embed a comma into a string, escape it using "\," | [['s', 'varName1', 'My String1'], ['s', 'varName2', 'My String2'], ['ia', 'arrName', '1,2,3,4']]
flags | string | no | Intent startup-specific flags as a hexadecimal string. Check [Intent documentation](https://developer.android.com/reference/android/content/Intent.html) for the list of available flag values (constants starting with `FLAG_ACTIVITY_`). Flag values could be merged using the logical 'or' operation. | 0x10200000 is the combination of two flags: 0x10000000 `FLAG_ACTIVITY_NEW_TASK` `|` 0x00200000 `FLAG_ACTIVITY_RESET_TASK_IF_NEEDED`

#### Returned Result

The actual stdout of the downstream `am` command.

### mobile: getDeviceTime

Retrieves the current device's timestamp.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
format | string | no | The set of format specifiers. Read https://momentjs.com/docs/ to get the full list of supported datetime format specifiers. The default format is `YYYY-MM-DDTHH:mm:ssZ`, which complies to ISO-8601 | YYYY-MM-DDTHH:mm:ssZ

#### Returns

The device timestamp string formatted according to the given specifiers

### mobile: setDate

Set the given date for a picker control. Invokes https://developer.android.com/reference/androidx/test/espresso/contrib/PickerActions#setdate under the hood.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
year | int | yes | The year to set | 2020
monthOfYear | int | yes | The number of the month to set | 3
dayOfMonth | int | yes | The number of the day to set | 20

### mobile: setTime

Set the given time for a picker control. Invokes https://developer.android.com/reference/androidx/test/espresso/contrib/PickerActions#settime under the hood.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
hours | int | yes | Hour to set in range 0..23 | 14
minutes | int | yes | Minute to set in range 0..59 | 15

### mobile: flashElement

Highlights the given element in the UI by adding flashing to it

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
element | string | yes | UDID of the element to perform the action on. | 123456-7890-3453-24234243
durationMillis | int | no | Duration of single flashing sequence, 30 ms by default | 50
repeatCount | int | no | Count of repeats, 15 times by default | 10

### mobile: dismissAutofill

Dismisses [autofill](https://developer.android.com/guide/topics/text/autofill) picker if it is visible on the screen.

### mobile: backdoor

Gives a possibility to invoke methods from your application under test.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
elementId | string | yes if `target` is set to `element` | UDID of the element to perform the action on. | 123456-7890-3453-24234243
target | string | yes | Select a target for the backdoor mathod execution: `activity`, `application`, `element` | activity
methods | `Array<Map>` | yes | Methods chain to execute | See [Backdoor Extension Usage](#backdoor-extension-usage)

#### Returns

The result of the last method in the chain

### mobile: uiautomator

Allows to execute a limited set of [UiAutomator](https://developer.android.com/training/testing/ui-automator) commands to allow out-of-app interactions with accessibility elements.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
strategy | string | yes | UiAutomator element location strategy. The following strategies are supported: "clazz", "res", "text", "textContains", "textEndsWith", "textStartsWith", "desc", "descContains", "descEndsWith", "descStartsWith", "pkg" | desc
locator | string | yes | Valid UiObject2 locator value for the given strategy | 'my description'
action | string | yes | The action name to perform on the found element. The following actions are supported: "click", "longClick", "getText", "getContentDescription", "getClassName",  "getResourceName", "getVisibleBounds", "getVisibleCenter", "getApplicationPackage", "getChildCount", "clear", "isCheckable", "isChecked", "isClickable", "isEnabled", "isFocusable", "isFocused", "isLongClickable", "isScrollable", "isSelected" | isEnabled
index | int | no | If the given locator matches multiple elements then only the element located by this index will be selected for the interaction, otherwise the method will be applied to all found elements.  Indexing starts from zero | 1

#### Returns

The result of the selected action applied to found elements. If index is provided then the array will only contain one item. If the index is greater than the count of found elements then an exception will be thrown.

### mobile: uiautomatorPageSource

Allows to retrieve accessibility elements hierarchy tree with [UiAutomator](https://developer.android.com/training/testing/ui-automator) framework. The extension calls [dumpWindowHierarchy](https://developer.android.com/reference/androidx/test/uiautomator/UiDevice#dumpwindowhierarchy_1) under the hood.

#### Returns

The UI accessibility hierarchy represented as XML document.

### mobile: webAtoms

Allows to run a chain of [Espresso web atoms](https://developer.android.com/training/testing/espresso/web) on a web view element.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
webviewEl | string | yes | The UDID of the destination web view element | 123456-7890-3453-24234243
forceJavascriptEnabled | boolean | yes | If webview disables javascript then webatoms won't work. Setting this argument to `true` enforces javascript to be enabled. | true
methodChain | `Array<Map>` | yes | Chain of atoms to execute. Each item in the chain must have the following properties: `name` (must be one of [WebInteraction](https://developer.android.com/reference/androidx/test/espresso/web/sugar/Web.WebInteraction) action names) and `atom`. `atom` is a map with two entries: `name` (must be one of [DriverAtoms](https://developer.android.com/reference/androidx/test/espresso/web/webdriver/DriverAtoms) method names) and `args` (must be an array of the corresponding method values). | `[{"name": "methodName", "atom": {"name": "atomName", "args": ["arg1", "arg2", ...]}}, ...]`

#### Returns

Chain items are executed sequentially and the next item is executed on the result of the previous item. The final result is returned to the caller.

### mobile: registerIdlingResources

Registers one or more [idling resources](https://developer.android.com/training/testing/espresso/idling-resource).

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
classNames | string | yes | Comma-separated list of idling resources class names. Each name must be a full-qualified java class name. Each class in the app source must implement a singleton pattern and have a static `getInstance()` method returning the class instance, which implements `androidx.test.espresso.IdlingResource` interface. Read [Integrate Espresso Idling Resources in your app to build flexible UI tests](https://android.jlelse.eu/integrate-espresso-idling-resources-in-your-app-to-build-flexible-ui-tests-c779e24f5057) for more details on how to design and use idling resources concept in Espresso. | `io.appium.espressoserver.lib.MyIdlingResource`

### mobile: unregisterIdlingResources

Unregisters one or more [idling resources](https://developer.android.com/training/testing/espresso/idling-resource).

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
classNames | string | yes | Comma-separated list of idling resources class names. Each name must be a full-qualified java class name. Each class in the app source must implement a singleton pattern and have a static `getInstance()` method returning the class instance, which implements `androidx.test.espresso.IdlingResource` interface. Read [Integrate Espresso Idling Resources in your app to build flexible UI tests](https://android.jlelse.eu/integrate-espresso-idling-resources-in-your-app-to-build-flexible-ui-tests-c779e24f5057) for more details on how to design and use idling resources concept in Espresso. | `io.appium.espressoserver.lib.MyIdlingResource`

### mobile: listIdlingResources

Lists all the previously registered [idling resources](https://developer.android.com/training/testing/espresso/idling-resource).

#### Returns

List of fully qualified class names of currently registered idling resources or an empty list if no resources have been registered yet.

### mobile: waitForUIThread

- Wait for the UI thread to become idle, in other words, wait for the APP to become [idle](https://developer.android.com/reference/androidx/test/espresso/UiController#loopMainThreadUntilIdle()).
- Use case: On compose and native combination screens, it's possible for the Espresso API to block the UI thread, which can cause the app to freeze. To resolve this issue, it's recommended to explicitly call the `mobile:waitForUIThread` API, which can help to unfreeze the UI thread.


### mobile: unlock

Unlocks the device if it is locked. Noop if the device's screen is not locked.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
key | string | yes | The unlock key. See the documentation on [appium:unlockKey](#device-locking) capability for more details | 12345
type | string | yes | The unlock type. See the documentation on [appium:unlockType](#device-locking) capability for more details | password
strategy | string | no | Unlock strategy. See the documentation on [appium:unlockStrategy](#device-locking) capability for more details | uiautomator
timeoutMs | number | no | Unlock timeout. See the documentation on [appium:unlockSuccessTimeout](#device-locking) capability for more details | 5000

### mobile: startMediaProjectionRecording

Starts a new recording of the device activity using [Media Projection](https://developer.android.com/reference/android/media/projection/MediaProjection) API. This API is available since Android 10 (API level 29) and allows to record device screen and audio in high quality. Video and audio encoding is done by Android itself.
The recording is done by [Appium Settings helper](https://github.com/appium/io.appium.settings#internal-audio--video-recording).

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
resolution | string | no | The resolution of the resulting video, which usually equals to Full HD 1920x1080 on most phones, however you could change it to one of the following supported resolutions: "1920x1080", "1280x720", "720x480", "320x240", "176x144" | 1280x720
maxDurationSec | number | no | The maximum number of seconds allowed for the recording to run. 900 seconds by default (15 minutes) | 300
priority | string | no | Recording thread priority is set to maximum (`high`) by default. However if you face performance drops during testing with recording enabled, you could reduce the recording priority to `normal` or `low`. | low
filename | string | no | You can type recording video file name as you want, but recording currently supports only "mp4" format so your filename must end with ".mp4". An invalid file name will fail to start the recording. If not provided then the current timestamp will be used as file name. | screen.mp4

#### Returned Result

`true` if a new recording has successfully started. `false` if another recording is currently running.

### mobile: isMediaProjectionRecordingRunning

Check if a media projection recording is currently running

#### Returned Result

`true` if a recording is running.

### mobile: stopMediaProjectionRecording

Stops a recording and retrieves the recently recorded media. If no recording has been started before then an error is thrown. If the recording has been already finished before this API has been called then the most recent recorded media is returned.

#### Arguments

Name | Type | Required | Description | Example
--- | --- | --- | --- | ---
remotePath | string | no | The path to the remote location, where the resulting video should be uploaded. The following protocols are supported: http/https, ftp. Null or empty string value (the default setting) means the content of resulting file should be encoded as Base64 and passed as the endpoont response value. An exception will be thrown if the generated media file is too big to fit into the available process memory. | https://myserver.com/upload
user | string | no | The name of the user for the remote authentication. | admin
pass | string | no | The password for the remote authentication. | pa$$w0rd
method | string | no | The http multipart upload method name. The 'PUT' one is used by default. | POST
headers | Map&lt;string, string&gt; | no | Additional headers mapping for multipart http(s) uploads | {'Agent': '007'}
fileFieldName | string | no | The name of the form field, where the file content BLOB should be stored for http(s) uploads. `file` by default | blob
formFields | Map&lt;string, string&gt; or Array&lt;Pair&gt; | no | Additional form fields for multipart http(s) uploads. | {'name': 'yolo.mp4'}

#### Returned Result

Base64-encoded content of the recorded media file if `remotePath` argument is falsy or an empty string.

### mobile: hideKeyboard

Tries to hide the on-screen keyboard. Throws an exception if the keyboard cannot be hidden.
Does nothing if the keyboard is already hidden.

#### Returned Result

`true` if the keyboard was successfully hidden or `false` if it was already invisible.

### mobile: isKeyboardShown

Checks if the system on-screen keyboard is visible.

#### Returned Result

`true` if the keyboard is visible


## Backdoor Extension Usage

Espresso driver allows to directly invoke a method from your application under test using `mobile: backdoor` extension. If `target` is set to `application` then methods will be invoked on the application class. If target is set to `activity` then methods will be invoked on the current application activity. If target is set to `element` then methods will be invoked on the selected view element. Only 'public' methods can be invoked ('open' modifier is necessary in Kotlin). The following primitive types are supported for method arguments: "int", "boolean", "byte", "short", "long", "float", "char". Object wrappers over primitive types with fully qualified names "java.lang.*" are also supported: "java.lang.CharSequence", "java.lang.String", "java.lang.Integer", "java.lang.Float", "java.lang.Double", "java.lang.Boolean", "java.lang.Long", "java.lang.Short", "java.lang.Character", etc.

For example, in the following arguments map

```json
{
   "target": "activity",
   "methods":
   [
     {
       "name": "someMethod",
     },
     {
       "name": "anotherMethod",
       "args": [
         {"value": "foo", "type": "java.lang.CharSequence"},
         {"value": 1, "type": "int"}
       ]
     }
   ]
}
```

the `anotherMethod` will be called on the object returned by `someMethod`, which has no arguments and which was executed on the current activity instance. Also `anotherMethod` accepts to arguments of type `java.lang.CharSequence` and `int`. The result of `anotherMethod` will be serialized and returned to the client.


## Parallel Tests

It is possible to execute tests in parallel using Espresso driver.
Appium allows to do this on per-process (multiple server processes running on different ports managing single session)
or per-request basis (single server process managing multiple sessions, more preferable, uses less resources and ensures better control over running sessions). Check [Parallel Android Tests](docs/parallel-tests.md) article for
more details.

> **Note**
> If you are _not_ going to run your tests in parallel then consider enabling the `--session-override` Appium server argument. It forces the server to close all pending sessions before a new one could be opened,
> which allows you to avoid possible issues with such sessions silently running/expiring in the background.


## Troubleshooting

* If you observe Espresso server crash on startup and various exceptions about missing class/method in the logcat output then consider updating [appium:espressoBuildConfig](#espresso-build-config) capability with module versions that match your application under test. This might require some experimentation, as different apps have different module requirements. Check, for example, [issue #812](https://github.com/appium/appium-espresso-driver/issues/812)
* If you experince issues with application activities being not found or not starting then consider checking [How To Troubleshoot Activities Startup](docs/activity-startup.md) article.
* Espresso requires the debug APK and app-under-test APK (AUT) to have the same signature. It automatically signs the AUT with the `io.appium.espressoserver.test` signature. This may be problematic if you're using an outdated Android SDK tools and/or an outdated Java version.
* If there are problems starting a session, set the capability `forceEspressoRebuild` to `true` and retry. This will force rebuilding of Espresso Server. If the following session startup is successfull, set it back to `false`, so the session startup performance is back to normal.
* If you experience session startup failures due to exceptions similar to `Resources$NotFoundException` then try to adjust your ProGuard rules:
  ```
  -dontwarn com.google.android.material.**
  -keep class com.google.android.material.** { *; }

  -dontwarn androidx.**
  -keep class androidx.** { *; }
  -keep interface androidx.** { *; }

  -dontwarn android.support.v4.**
  -keep class android.support.v4.** { *; }

  -dontwarn android.support.v7.**
  -keep class android.support.v7.** { *; }
  ```
  Please read [#449](https://github.com/appium/appium-espresso-driver/issues/449#issuecomment-537833139) for more details on this topic.


## Contributing

### Contents of Repo

* `espresso-server/`: Android Java code that gets built into a test apk. The test apk runs a NanoHTTP server that implements the WebDriver protocol.
* `lib/`: NodeJS code that constitutes the Appium driver, which is responsible for handling capabilities and starting up the Espresso instrumentation context. Once the Espresso server is up, this code is responsible for proxying user requests to it.

### Running

* To build the Espresso server _and_ the NodeJS code, run `npm run build`
* To just build the Espresso server, run `npm run build:server` or `cd espresso-server && ./gradlew clean assembleDebug assembleAndroidTest`. The server can also be built from Android Studio.
    * To build the espresso server for a custom target package `./gradlew -PappiumTargetPackage=io.appium.android.apis assembleAndroidTest`
* To just build NodeJS code, run `npm run build:node`

### Tests

* Espresso server unit tests are located at `io.appium.espressoserver.test` and can be run in Android Studio
* NodeJS unit tests are run with `npm run test`
* End-to-end tests are run with `npm run e2e-test` (remember to run `npm run build` before running this command so that it has up-to-date Espresso Server and NodeJS code)
