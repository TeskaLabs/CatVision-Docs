# CatVision.io Architecture

<div style="text-align: center">
	<a href="assets/Catvision.svg"><img src="assets/Catvision.svg"/></a>
</div>

1. Your device equipped with [**SeaCat Android SDK**](https://s3.amazonaws.com/resources.seacat.mobi/releases/SeaCatClient_Android_v1611-rc-2-release.aar) and  [**CatVision SDK**](https://s3.amazonaws.com/resources.seacat.mobi/releases/tlra-v1611-rc-2-release.aar) connects to the **SeaCat Gateway**
2. The backend of your web application uses an **API key** to request a time-limited **Auth Token** from **CatVision.io API**, which is then passed to the [**CatVision Display**](https://github.com/TeskaLabs/CatVision-Display) component.
3. **Websocket Proxy** authenticates websocket connection from the **CatVision Display** and establishes a connection to the device via **SeaCat Gateway.**
