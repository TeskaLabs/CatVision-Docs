# CatVision

CatVision is a set of components that makes it possible to securely transfer screen, mouse and keyboard events to your mobile and IoT device.

<div style="text-align: center">
	<a href="assets/Catvision.svg"><img src="assets/Catvision.svg"/></a>
</div>

1. Your device equipped with [**SeaCat Android SDK**](https://s3.amazonaws.com/resources.seacat.mobi/releases/SeaCatClient_Android_v1611-rc-2-release.aar) and  [**CatVision SDK**](https://s3.amazonaws.com/resources.seacat.mobi/releases/tlra-v1611-rc-2-release.aar) connects to the **SeaCat Gateway**
2. The backend of your web application uses an **API key** to request a time-limited **Auth Token** from **CatVision.io API**, which is then passed to the [**CatVision Display**](https://github.com/TeskaLabs/CatVision-Display) component.
3. **Websocket Proxy** authenticates websocket connection from the **CatVision Display** and establishes a connection to the device via **SeaCat Gateway.**

## Quick Start

Here's a quick how-to that gets your app CatVision-enabled and your frontend set up for viewing a device's screen.

> Each chapter has a **detailed step-by-step** where you will find how to proceed "by the book". 

### 1. Create an API key at CatVision.io ([detailed step-by-step](./CatVision.io.md))

CatVision.io is a portal where you can manage your devices and API keys that are needed for your backend be able to request a time limited auth token that you need to pass to the CatVision Display (see *CatVision Display integration* below) component to be granted to establish connection.

1. **Create an account** at [https://catvision.io/register](https://catvision.io/register).
2. **Create new product**
3. **Register your application**
4. **Create an API key**

### 2. Integrate CatVision SDK into your Android app ([detailed step-by-step](./CatVisionAndroidSDK.md))

Note that this how-to is a **minimum setup** for your Android application to have `CatVision` integrated. To integrate `CatVision` the correct sustainable way follow the [detailed step-by-step](./CatVisionAndroidSDK.md)

Let us say we have an **Android Project** called `Example Project` with an **application** called `Example`

#### Dependency


Download the **CatVision SDK** and **SeaCat SDK** and put it in `example-project/example/aars`

```
$ cd /path/to/example-project/example
$ mkdir aars 
$ cd aars
$ wget https://s3.amazonaws.com/resources.seacat.mobi/releases/cvio-v1705-rc.1-release.aar
$ wget https://s3.amazonaws.com/resources.seacat.mobi/releases/SeaCatClient_Android_v1611-rc-2-release.aar
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
	compile(name: "SeaCatClient_Android_v1611-rc-2-release", ext:"aar")
}
```

> **Sync project** and **Clean project** is needed in order to reload paths in android studio after you update dependencies

#### Initialize
Initialize `CatVision` and `SeaCatClient` in your `Application` object's `onCreate()`

```
import com.teskalabs.cvio.CatVision;
import com.teskalabs.seacat.android.client.SeaCatClient;

public class ExampleApp extends Application
{
	@Override
	public void onCreate()
	{
		super.onCreate();
		CatVision.initialize();
		SeaCatClient.initialize(getApplicationContext());
	}
```

#### Application Instance Identification *(Client Handle)*

**Client Handle** is a unique application instance identificator. **SeaCat** automatically generates a **Client ID** and a *Handle* called **Client Tag** that looks like this: `[GMYDQMRQGIYGCMBS]`.

> see the [TeskaLabs Identity Management docs](https://www.teskalabs.com/docs/intro/identity-management) for more information regarding indentification of an application instance

A `[CLIENT_HANDLE]` (the **Client Tag** or a **Custom Client Handle**) is an identificator used to specify what device's screen you want to display using CatVision Display component in step 3.

Browse client handles associated with your product at [CatVision.io](https://catvision.io).

#### Custom Client Handle

To enable CatVision to identify your device with a custom *client handle*, initialize `SeaCatClient` like this:

```
SeaCatClient.initialize(getApplicationContext(), (Runnable) null);
```

Then once you have your custom identificator available run this code in order to create a custom handle and pair it to the SeaCat client ID and repeat it anytime the identification string of your device changes.

```
String customId = getCustomId();
SeaCatClient.setCSRWorker(new Runnable() {
	public void run() {
		CSR csr = new CSR();
		csr.setUniqueIdentifier(customId);
		try {
			csr.submit();
		} catch (IOException e) {
			Log.e(SeaCatInternals.L, "Exception in CSR.createDefault:", e);
		}
	}
});
```

#### Make sure SeaCat is initialized

Create `CatVision` object.

```
public class MainActivity extends AppCompatActivity {
	Handler handler;
	Runnable stateChecker;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		catvision = CatVision.CreateOrGet(this);
	}
	...
```

Use the following `stateChecker` runnable to ensure that the `SeaCatClient` is in a ready state. You can put this code at the end of the `onCreate` method of your activity.

```
handler = new Handler();
stateChecker = new Runnable()
{
	@Override
	public void run()
	{
		String state = SeaCatClient.getState();
		if ((state.charAt(3) == 'Y') && (state.charAt(4) == 'N') && (state.charAt(0) != 'f')) {
			isSeaCatClientInitialized = true;
		}
		else {
			handler.postDelayed(this, 500);
		}
	}
};
stateChecker.run();
```

#### Capture screen with CatVision

Once SeaCat Client is in a ready state you can start capturing the screen.

> SeaCatClient must be initialized at this point

```
public class MainActivity extends AppCompatActivity {
	private int CATVISION_REQUEST_CODE = 100;
	
	...

	private void onStartCaptureButtonClick()
	{
		if (isSeaCatClientInitialized) {
			catvision.requestStart(this, CATVISION_REQUEST_CODE);
		}
	}
	
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		if (requestCode == CATVISION_REQUEST_CODE) {
			catvision.onActivityResult(this, resultCode, data);
		}
	}
```

### 3. Show CatVision Display on your web

At this point you have your application equipped with the **SeaCat Android SDK** and **CatVision SDK** and you have **registered your application** at CatVision.io and **created an API key**. Now you will add the CatVision Display component to your web application front-end.

#### Get auth token from CatVision.io

The CatVision Display component needs a time limited auth token to authorize itself to the Websocket Proxy. Your **backend** will use your **secret API key** that you created before to fetch the auth token and will provide it to the web application front-end.

Here is an example of how to fetch the token from CatVision.io using popular languages. The endpoint to be called is the following: `https://catvision.io/api/authtoken?api_key=[API_KEY]` where `[API_KEY]` is the one you generated in  *step 1 - CatVision.io API Key*.

##### PHP

```php
<?php
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://catvision.io/api/authtoken?api_key=[API_KEY]');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POST, 1);

$response = curl_exec($ch);
$response_json = json_decode($response);

$auth_token = $response_json->{'auth_token'};

```

##### Python

```py
import requests, json
r = requests.post('https://ra.teskalabs.com/api/authtoken?api_key=[API_KEY]')
res = json.loads(r.text)

auth_token = res['auth_token']
```

#### Integrate CatVision Display

At this point you have your auth token and it is assumed you can render it to the response HTML.

First load the CatVision Display javascript from the TeskaLabs CDN.

```
<head>
	<script type="text/javascript" src="https://cdn.teskalabs.com/cvio.min.js"></script>
	...
```

To initialize CatVision Displays, put this code immediatlly after the body tag.

```html
<body>
	<script type="text/javascript">
   		CVIO.init({authToken: '[AUTH_TOKEN]'});
	</script>
	...
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

