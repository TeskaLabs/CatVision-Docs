---
layout: default
title: CatVision documentation
sidebar: catvision
---

# Adding a CatVision.io SDK into an iOS application

In this section we describe how to integrate a CatVision.io SDK into an iOS application so that an operator can access it remotely.

## Prerequisities

* A source code of the iOS application
* XCode \(9.0+ is tested\)
* _CatVision.io API Key ID_ \(see [Catvision.io API Key]({{site.url}}/catvision/get-started/api-key.html)\)

_Remark about iOS version requirement:_ Apple added a screen capturing functionality into iOS 11 \(autumn 2017\). Older versions of iOS do not support screen sharing.

_Remark about a simulator:_ As of iOS 11.2, a screen capture is not enabled by Apple in the simulator. You get only a dark blue screen instead of the remote screen image.

## Add a CatVision.io SDK

Download [CatVision.io SDK for iOS](https://s3.amazonaws.com/resources.seacat.mobi/releases/CatVisionIO.framework-17.12.105.zip) and unzip downloaded ZIP archive. The archive contains `CatVisionIO.framework`.

![Unzip downloaded CatVision.io SDK for iOS]({{site.url}}/catvision/assets/images/cvio_ios_download.png)

Open your iOS app project with XCode and go to _General_ configuration. Scroll down to "Embedded Binaries" and press "+" button.

![Add CatVision.io SDK in XCode as a Embedded Binary, step 1]({{site.url}}/catvision/assets/images/cvio_ios_xcode_1.png)

Select "Add Other ..."

![Add CatVision.io SDK in XCode as a Embedded Binary, step 2]({{site.url}}/catvision/assets/images/cvio_ios_xcode_2.png)

Select previously prepared `CatVisionIO.framework` a press "Open".

![Add CatVision.io SDK in XCode as a Embedded Binary, step 3]({{site.url}}/catvision/assets/images/cvio_ios_xcode_3.png)

Ensure that "Copy items if needed" is checked. It means that `CatVisionIO.framework` will be copied into XCode project. Finally press "Finish" to complete integration procedure. Now you may delete the downloaded ZIP archive and unzipped `CatVisionIO.framework`.

![Add CatVision.io SDK in XCode as a Embedded Binary, step 4]({{site.url}}/catvision/assets/images/cvio_ios_xcode_4.png)

The CatVision.io SDK is now added to your iOS application.

## Initialization

CatVision.io SDK has to be initialized during an application launch. It is done by adding an initialization one liner into a `didFinishLaunchingWithOptions` method of an `AppDelegate` class.

Open a source code file of your application delegate, it typically called `AppDelegate.m`. Add `#import <CatVisionIO/CatVisionIO.h>` into import section in the beginning of the source file and `[CatVision initialize];` into `didFinishLaunchingWithOptions` method.

See following example:

```objs
...
#import <CatVisionIO/CatVisionIO.h>
...

@implementation AppDelegate
    ...

    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

        ...
        [CatVision initialize];
        ...
    }

    ...
```

Now your app should look like this:

![Add CatVision.io SDK in XCode as a Embedded Binary, step 5]({{site.url}}/catvision/assets/images/cvio_ios_xcode_5.png)

_CatVision.io API Key ID_ has to be added into the _information property list file_ \(or `Info.plist` for short\) of the iOS app so that the application authenticates properly to [catvision.io](https://app.catvision.io). See [Catvision.io API Key]({{site.url}}/catvision/get-started/api-key.html) for more information of how to get obtain _CatVision.io API Key ID_ if you don't have one.

Open `Info.plist` file of your iOS app and _add row_ from a context menu. The _Key_ is `CVIOApiKeyId`, the type is a String and the value is the _CatVision.io API Key ID_.

![Add CatVision.io SDK in XCode as a Embedded Binary, step 5]({{site.url}}/catvision/assets/images/cvio_ios_xcode_6.png)

## Start a screen sharing

The application needs to implement start and stop actions of CatVision.io screen sharing. In this example we are going to implement the switch that controls a screen sharing function.

Add a switch to a storyboard, its initial _State_ is **Off**.

![Add a switch to a storyboard]({{site.url}}/catvision/assets/images/cvio_ios_xcode_7.png)

Add the following code to the ViewController header (.h):

```objc
@interface ViewController : ...

...
@property (weak, nonatomic) IBOutlet UISwitch * ScreenShareSwitch;
...

@end

```

Link `ScreenShareSwitch` outlet with a Switch in a storyboard.

Add the following code to the ViewController implementation class (.m):

```objc
...
#import <CatVisionIO/CatVisionIO.h>
...

@implementation ViewController

...
@synthesize ScreenShareSwitch;
...

...
- (IBAction)onScreenShareSwitchValueChanged:(id)sender {
    if ([ScreenShareSwitch isOn]) {
        [[CatVision sharedInstance] start];
    }
    else {
        [[CatVision sharedInstance] stop];
    }
}
...

@end

```

Link `onScreenShareSwitchValueChanged` method with a Switch _Value Changed_ event. Screen sharing is ready. You can compile the application now and run the app.

When screen sharing is started in the mobile apps, you will see the connected client at [app.catvision.io](https://app.catvision.io):

![Add CatVision.io SDK in XCode as a Embedded Binary, step 5]({{site.url}}/catvision/assets/images/cvio_ios_done.png)
