# CatVision

CatVision is a technology that provides an easy and secure remote access to a display of your mobile application.

Users of your CatVision-equipped application can get a real-time online support without being asked what is currently on their screen. An application can be used remotely without a person to be physically present at the hardware that the application runs on. For example, this can be very useful in use with room control tablets or Android IoT devices.

## Quick Start

Here's a quick how-to that gets your app CatVision-enabled and your web application set up for viewing a mobile application's screen in three simple steps:

1. Create an API key at CatVision.io
2. Integrate CatVision SDK into your Android app
3. Integrate CatVision Display into your web application

> Each chapter has a **detailed step-by-step** where you will find how to proceed "by the book". 

### 1. Create an API key at CatVision.io ([detailed step-by-step](./CatVision.io.md))

CatVision.io is a web service where you can manage your CatVision API keys. They are needed to authorize CatVision Display component in your web application.

> Authorization process is described in the [Architecture and Security](./Architecture.md) section of the docs.

The process of obtaining an API key is following:

1. **Create an account** at [https://catvision.io/register](https://catvision.io/register).
2. **Create new product**
3. **Create an API key**

A **Product** is a namespace for your applications. Typically you will manage one application within one Product.

The API key consists of two parts. **Secret API Key** let's you obtain an auth token for the CatVision Display component. The **API Key ID** is used to board your application into the CatVision Product that you created.


### 2. Integrate CatVision SDK into your Android app ([detailed step-by-step](./CatVisionAndroidSDK.md))

In this section we will create an application called `Example` in a directory `./example-project` and it will be assumed that we created a Product `Example` at [https://catvision.io](https://catvision.io) where we also created an API Key and we know the **API Key ID** that will be referred to as `[API_KEY_ID]`.

So let us go ahead and create a new **Android Project** called `Example Project` with one application module called `Example`.

> **The minimum Android API level is 21 or later**. Otherwise CatVision Screen capture won't work.

#### Dependency


Download the **CatVision SDK** and put it in `./example-project/example/aars`

```
$ cd /path/to/example-project/example
$ mkdir aars 
$ cd aars
$ wget https://s3.amazonaws.com/resources.seacat.mobi/releases/cvio-v1705-rc.1-release.aar
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

#### Initialize

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

#### Application Instance Identification - *Client Handle*

**Client Handle** is a unique application instance identificator. You need to identify your running application instance so that you can specify which instance to connect to with CatVision Display component in your web application.

Whenever you are ready to create an identification for the running application instance you must do so with the following call.

```
catvision.setClientHandle(CatVision.DEFAULT_CLIENT_HANDLE);
```

A *Handle* called **Client Tag** (looks like `[GMYDQMRQGIYGCMBS]`) is automatically created for you and after you call `setClientHandle` you can find new record in your Product's clients list at [catvision.io](https://catvision.io). However you are encouraged to generate and specify your own client handle.

To tell CatVision that except for the default handle you use your own client handle simply replace the default client handle with your own:

```
String customClientHandle = generateHandle();
catvision.setClientHandle(customClientHanlde);
```

> See the [TeskaLabs Identity Management docs](https://www.teskalabs.com/docs/intro/identity-management) for more information regarding indentification of an application instance


#### Capture the screen

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

### 3. Integrate CatVision Display into your web application

At this point you have your application equipped with the **CatVision SDK** and you have **created an API key** at [catvision.io](https://catvision.io). Now you will add the CatVision Display component to your web application.

#### Get auth token from CatVision.io

The CatVision Display component needs a time limited auth token to authorize. Your **web application backend** will use your **Secret API key** (`SECRET_API_KEY`) to fetch the token.

Here is an example of how to fetch the token from CatVision.io using some of the popular languages. The endpoint to be called is the following: `https://catvision.io/api/authtoken?api_key=[SECRET_API_KEY]` where `[SECRET_API_KEY]` is the one you generated in  *step 1 - CatVision.io API Key*.

##### PHP

```php
<?php
$url = 'https://catvision.io/api/authtoken';
$secret_api_key = '[SECRET_API_KEY]';
$options = array(
    'http' => array(
        'header'  => "X-SC-SecretAPIKey: $secret_api_key\r\n",
        'method'  => 'POST',
        'content' => ''
    )
);
$context  = stream_context_create($options);
$result = file_get_contents($url, false, $context);
$obj = json_decode($result);
$auth_token = $obj->{'auth_token'};
```

##### Python

```py
import requests, json
r = requests.post('https://catvision.io/api/authtoken', headers={
	'X-SC-SecretAPIKey' : '[SECRET_API_KEY]'
})
res = json.loads(r.text)
auth_token = res['auth_token']
```

#### Integrate CatVision Display

First load the CatVision Display javascript from the TeskaLabs CDN.

```
<head>
	<script type="text/javascript" src="https://cdn.teskalabs.com/cvio.min.js"></script>
	...
```

For initialization, put this code immediatlly after the body tag.

```html
<script type="text/javascript">
	CVIO.init({authToken: '[AUTH_TOKEN]'});
</script>
```


You need to have an existing `<canvas id='mycanvas'>` in your DOM. Once it has loaded you can instantiate a CatVision Display and connect.

```html
<canvas id='cviocanvas'>
<script type="text/javascript">
	var cvioDisplay = new CVIODisplay({
		target:       document.getElementById('cviocanvas'),
		clientHandle: '[CLIENT_HANDLE]',
	});
	cvioDisplay.connect();
</script>
```

##### Custom Client Handle

If you want to use your own Client Handle initialize and connect to the CatVision like this:

```js
var cvioDisplay = new CVIODisplay({
	target:          document.getElementById('cviocanvas'),
	deviceHandle:    '[CLIENT_HANDLE]',
	deviceHandleKey: 'u'
});
cvioDisplay.connect();
```

The option `deviceHandleKey: 'u'` says you want to identify the device by custom client handle.

