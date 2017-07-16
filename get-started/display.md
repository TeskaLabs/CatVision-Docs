# Add CatVision.io Display into your web application

At this point you have your application equipped with the **CatVision.io SDK for Android** and you have **created an API key** at [app.catvision.io](https://app.catvision.io). Now you will add the CatVision Display component to your web application.

## Get an auth token from app.catvision.io

The CatVision Display component needs a time limited auth token to authorize. Your **web application backend** will use your **Secret API key** \(`SECRET_API_KEY`\) to fetch the token.

Here is an example of how to fetch the token from [app.catvision.io](https://app.catvision.io) API using some of the popular languages. The endpoint to be called is the following: `https://app.catvision.io/api/authtoken`. You need to add the `[SECRET_API_KEY]` in header `X-SC-SecretAPIKey`.

### PHP

```php
<?php
$url = 'https://app.catvision.io/api/authtoken';
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

### Python

```py
import requests, json
r = requests.post('https://app.catvision.io/api/authtoken', headers={
    'X-SC-SecretAPIKey' : '[SECRET_API_KEY]'
})
res = json.loads(r.text)
auth_token = res['auth_token']
```

## Integrate CatVision Display

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

### Custom Client Handle

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

