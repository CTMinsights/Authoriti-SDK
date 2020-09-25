## Setup

### Android

1. Specify the permissions in the App manifest file:

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />

<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
```

The SDK requires access to camera to handle driver's license validation; and Internet access to talk to Authoriti servers.

2. Add the Authoriti SDK dependency in **build.gradle**

The SDK is hosted in a private Amazon S3 Maven repository.

The root `build.gradle` should have the following entry

```
allprojects {
    repositories {
        google()
        jcenter()
        maven {
            url "s3://authoriti-sdk.s3.amazonaws.com"
            credentials(AwsCredentials) {
                accessKey "ACCESS-KEY"
                secretKey "SECRET-KEY"
            }
        }
    }
}

```

Instead of adding the keys directly, it is recommended that the keys are added to the file `local.properties`

submodule `build.gradle` should have the following entry

```
dependencies {
    implementation 'com.authoritisdk:AuthoritiSDK:0.1'
}
```

To include the Driver's License Validation module, add `implementation 'com.authoriti:dlv:0.1'` to the dependencies.

Syncing the project either via Terminal or the Android Studio IDE should let S3 be used as an artifact repository.

### iOS

Using Cocoapods add the following podfile:

```
pod 'AuthoritiSDK', '~> 0.0.4'
```

To add Driver's License Validation functionality, add the following dependency to the podfile:

```
pod 'AuthoritiDLV', '~> 0.0.1'
```
