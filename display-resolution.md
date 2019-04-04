---
layout: default
title: CatVision documentation
---

# CatVision.io Display Resolution

CatVision.io tries to automatically choose optimal screen resolution of a remote display in order to provide optimal experience and also save network bandwidth which is also very important for a fluent experience of the remote operator.

The image of a captured screen is typically downscaled by 2x or 4x because modern smartphones are using subpixel resolutions with very high DPIs. However, CatVision.io SDK can be configured to use other downscale value, including 1 or even &lt;1. It is highly recommended for performance reasons to choose integer values of downscale factor.


## Android

CatVision.io for Android uses [`DisplayMetrics.densityDpi`](https://developer.android.com/reference/android/util/DisplayMetrics.html#densityDpi) to determine a downscale factor of the remote display.

| The screen density \(densityDpi\) | Downscale factor |
| :---      | :--- |
| &lt; 280  | 1x   |
| &lt; 400  | 2x   |
| &ge; 400  | 3x   |

[Read  more about Android DPI ranges.](https://developer.android.com/guide/practices/screens_support.html#range)


## iOS

CatVision.io for iOS uses [UIScreen
 scale](https://developer.apple.com/documentation/uikit/uiscreen/1617836-scale) property directly as a downscale factor.

