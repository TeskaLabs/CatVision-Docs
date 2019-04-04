---
layout: default
title: CatVision documentation
---

# Adding a CatVision.io SDK into an Android application

In this section we describe how to integrate a CatVision.io SDK into an Android application so that an operator can access it remotely.

## Prerequisities

* A source code of the Android application
* Android Studio
* _CatVision.io API Key ID_ \(see [Catvision.io API Key]({{site.url}}/get-started/api-key.html)\)

_Remark about Android version requirement:_ Android provides the screen capture functionality starting from API level 21 respective Android 5.0. CatVision.io SDK for Android can be integrated into applications targeting older Android versions, but screen sharing will not be functional.

## Add the CatVision.io SDK

Insert a following line into `dependencies` section of your Android application `build.gradle` and press 'Sync Now'. Android Studio will automatically download an _CatVision.io SDK_ from [JCenter](https://bintray.com/teskalabs/CatVision.io/catvision-io-sdk-android) and integrate it into the application source code.

```
compile 'com.teskalabs.cvio:catvision-io-sdk-android:+'
```

![Add CatVision.io SDK dependency via Android Studio]({{site.url}}/assets/images/cvio_android_studio_dependencies.png)

The CatVision.io SDK is now added to your Android application.

## Add the CatVision.io API Key ID

_CatVision.io API Key ID_ has to be added into the _App Manifest_ \(or `AndroidManifest.xml`\) of the Android app so that the application authenticates properly to [catvision.io](https://app.catvision.io). See [Catvision.io API Key]({{site.url}}/get-started/api-key.html) for more information of how to get obtain _CatVision.io API Key ID_ if you don't have one.

Open `AndroidManifest.xml` file of your Android app and add a `<meta-data android:name="cvio.api_key_id" ...>` line into `<application>` tag:

```xml
<application ...>

...
<meta-data android:name="cvio.api_key_id" android:value="[CatVision.io API Key ID]" />
...

</application>
```

![CatVision.io API Key ID is added to AndroidManifest.xml]({{site.url}}/assets/images/cvio_android_studio_manifest.png)


## Start a screen sharing

The application needs to implement start and stop actions of CatVision.io screen sharing. In this example we are going to implement start and stop buttons in the options menu.

Add the following code to the Activity class:

```java
...
import com.teskalabs.cvio.CatVision;
...


public class MyActivity extends Activity {

    private CatVision catvision;

    private final int menuItemStartScreenShareId = 1101;
    private final int menuItemStopScreenShareId = 1102;
    private final int CATVISION_REQUEST_CODE = 1103;

    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        ...
        // Obtain a CatVision.io SDK reference
        catvision = CatVision.getInstance(this);
        ...

    }

    ...

    @Override
    public boolean onPrepareOptionsMenu(Menu menu)
    {
        menu.clear();
        int menuGroup1Id = 1100;

        if (!catvision.isStarted()) {
            menu.add(menuGroup1Id, menuItemStartScreenShareId, 1, "Share screen");
        } else {
            menu.add(menuGroup1Id, menuItemStopScreenShareId, 1, "Stop sharing");
        }

        return super.onPrepareOptionsMenu(menu);
    }

    ...

    @Override
    public boolean onOptionsItemSelected(MenuItem item)
    {
        ...

        if (item.getItemId() == menuItemStartScreenShareId) {
            catvision.requestStart(this, menuItemStopScreenShareId);
            return true;
        }

        else if (item.getItemId() == menuItemStopScreenShareId) {
            catvision.stop();
            return true;
        }

        ...

        return super.onOptionsItemSelected(item);
    }

    ...

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == CATVISION_REQUEST_CODE) {
            catvision.onActivityResult(this, resultCode, data);
            return;
        }

        super.onActivityResult(requestCode, resultCode, data);
    }
```

Screen sharing function is ready. You can compile the application now and start it.

When screen sharing is started in the mobile apps, you should see the connected client at [app.catvision.io](https://app.catvision.io):

![Android Application connected to CatVision.io]({{site.url}}/assets/images/cvio_android_emulator_share.png)

