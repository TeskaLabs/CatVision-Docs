# Adding a CatVision.io SDK into an Android application

In this section we will create an application called `Example` in a directory `./example-project` and it will be assumed that we created a Product `Example` at [https://catvision.io](https://catvision.io) where we also created an API Key and we know the **API Key ID** that will be referred to as `[API_KEY_ID]`.

So let us go ahead and create a new **Android Project** called `Example Project` with one application module called `Example`.

> **The minimum Android API level is 21 or later**. Otherwise CatVision Screen capture won't work.

## Dependency

Download the [**CatVision SDK**](http://get.catvision.io/CatVision_Android_v1705-release.aar) and put it in `./example-project/example/aars`

```
$ cd /path/to/example-project/example
$ mkdir aars 
$ cd aars
$ wget http://get.catvision.io/CatVision_Android_v1705-release.aar
```

In the application's `build.gradle` add a `flatDir` repository and set up dependencies:

```
repositories {
    jcenter()
    flatDir {
        dirs 'aars'
    }
}
dependencies {
    ...
    compile(name: "cvio-v1705-rc.1-release", ext:"aar")
}
```

You may wnat to **Sync project** and **Clean project**. It is needed in order to reload paths in Android Studio after you update dependencies or other Gradle file settings.

## Initialize

We need to create an `Application` object where we will initialize `CatVision` in the application object's `onCreate()` lifecycle method.

```
import com.teskalabs.cvio.CatVision;

public class ExampleApp extends Application
{
    @Override
    public void onCreate()
    {
        super.onCreate();
        CatVision.initialize(this);
    }
}
```

In order to get this to work we need to tell the `AndroidManifest.xml` we are implementing a custom Application object and that the use of `android.permission.INTERNET` is requested. The manifest also needs to contain our `[API_KEY_ID]` so that the application gets attached to your Product at [catvision.io](https://catvision.io).

```
<uses-permission android:name="android.permission.INTERNET"/>
<application
    ...
    android:name=".ExampleApp">

    <meta-data android:name="cvio.api_key_id" android:value="[API_KEY_ID]" />
    ...

</application>
```

Wherever we will need to call CatVision methods, we can simply get an instance of CatVision like this:

```
CatVision catvision = CatVision.getInstance();
```

So let us get an instance of `CatVision` after our Main Activity is created

```
public class MainActivity extends AppCompatActivity {
    private static final String TAG = MainActivity.class.getName();
    private CatVision catvision;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        catvision = CatVision.getInstance();
    }
    ...
}
```

## Application Instance Identification - _Client Handle_

**Client Handle** is a unique application instance identificator. You need to identify your running application instance so that you can specify which instance to connect to with CatVision Display component in your web application.

Whenever you are ready to create an identification for the running application instance you must do so with the following call.

```
catvision.setClientHandle(CatVision.DEFAULT_CLIENT_HANDLE);
```

A _Handle_ called **Client Tag** \(looks like `[GMYDQMRQGIYGCMBS]`\) is automatically created for you and after you call `setClientHandle` you can find new record in your Product's clients list at [catvision.io](https://catvision.io). However you are encouraged to generate and specify your own client handle.

To tell CatVision that except for the default handle you use your own client handle simply replace the default client handle with your own:

```
String customClientHandle = generateHandle();
catvision.setClientHandle(customClientHanlde);
```

> See the [TeskaLabs Identity Management docs](https://www.teskalabs.com/docs/intro/identity-management) for more information regarding indentification of an application instance

## Capture the screen

We are going to implement start and stop capture buttons in the options menu. Add the following code to the Main Activity class. It will ensure that the start button is only present when CatVision screen capture isn't already active - and vice versa with the stop capture option.

```
private final int menuItemStartCaptureId = 1;
private final int menuItemStopCaptureId = 2;

@Override
public boolean onPrepareOptionsMenu(Menu menu)
{
    menu.clear();
    int menuGroup1Id = 1;
    if (catvision.isStarted()) {
        menu.add(menuGroup1Id, menuItemStopCaptureId, 1, "Stop capture");
    } else {
        menu.add(menuGroup1Id, menuItemStartCaptureId, 1, "Start capture");
    }
    return super.onPrepareOptionsMenu(menu);
}
```

Now we implement click listeners and we will request start of CatVision screen capture when start button is clicked. First add `private int CATVISION_REQUEST_CODE = 100;` to the Main Activity class so that we can reuse that. Then add the following:

```
@Override
public boolean onOptionsItemSelected(MenuItem item)
{
    switch (item.getItemId()) {
        case menuItemStartCaptureId:
            catvision.requestStart(this, CATVISION_REQUEST_CODE);
             return true;

        case menuItemStopCaptureId:
            catvision.stop();
            return true;
        default:
            return super.onOptionsItemSelected(item);
        }
    }
}
```

Now when start button is clicked, Android will ask user to confirm that screen can be captured. We have to wait for the result of the user's decision in `onActivityResult`.

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == CATVISION_REQUEST_CODE) {
        catvision.onActivityResult(this, resultCode, data);
    }
}
```

The screen is now captured and available for connection via CatVision Display component.
