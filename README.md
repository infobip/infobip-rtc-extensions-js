## Introduction

Infobip RTC extensions is a JavaScript library which provides extended functionality to Infobip RTC SDK.

Currently available functionalities are:

- audio filter implementations
- video filter implementations

Here you will find an overview, and a quick guide on how to include and use these extensions in your application.
There is also in-depth reference documentation available.

## Prerequisites

Infobip RTC Extensions requires ES6.

## Getting the library

There are a few ways in which you can get our library.
We publish it as an NPM package and as a standalone JS file hosted on a CDN.

If you want to add it as an NPM dependency, run the following:

```bash
npm install infobip-rtc-extensions --save
```

After which you would use it in your project like this:

```javascript
let infobipRtcExtensions = require('infobip-rtc-extensions');
```

or as ES6 import:

```javascript
import {VirtualBackgroundVideoFilter} from "infobip-rtc-extensions"
```

You can include our distribution file in your JavaScript from our CDN:

```html
<script src="//rtc.cdn.infobip.com/0.0.10/infobip.rtc.extensions.js"></script>
```

The latest tag is also available:

```html
<script src="//rtc.cdn.infobip.com/latest/infobip.rtc.extensions.js"></script>
```

## Audio filters

## Video filters

The Infobip RTC supports user-defined video filters which manipulate outgoing video streams while in call. This library
provides a feature-rich implementation of commonly used video filters which allow easy configuration.

Currently available implementations are:

- VirtualBackgroundVideoFilter

### VirtualBackgroundVideoFilter

This filter allows you to alter your background.

Supported video filter modes are:

- Virtual background
  ([`VideoFilterMode.VIRTUAL_BACKGROUND`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VideoFilterMode#virtual-background))
  -Set a custom image to be drawn behind you as your background
- Background blur
  ([`VideoFilterMode.BACKGROUND_BLUR`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VideoFilterMode#background-blur))
  -Blur your background
- None ([`VideoFilterMode.NONE`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VideoFilterMode#none)) - No
  video filtering, just pass video frames as-is. Prefer this over destroying the video filter to avoid repeatedly
  reallocating video filter resources, which can cause visible hiccups.

To be able to use it, first you need to create an instance
of [`VirtualBackgroundVideoFilter`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VirtualBackgroundVideoFilter)
object. The constructor accepts optional
[`VirtualBackgroundVideoFilterOptions`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VirtualBackgroundVideoFilterOptions)
to be set.

```javascript
const options = {
    mode: VideoFilterMode.VIRTUAL_BACKGROUND,
    image: sourceImage // can be an instance of ImageBitmap, ImageData, HTMLImageElement, …
};

const videoFilter = new VirtualBackgroundVideoFilter(options);
```

For best performance, avoid reallocating new video filter instances just to change the current mode. Pass the new
options to the existing video filter instance:

```javascript
const options = {
    mode: VideoFilterMode.NONE
};

await videoFilter.setOptions(options);
``` 

### Applying the video filter

After you created the video filter, you can apply it to the
existing [`ApplicationCall`](https://github.com/infobip/infobip-rtc-js/wiki/ApplicationCall) using
the [`setVideoFilter`](https://github.com/infobip/infobip-rtc-js/wiki/ApplicationCall#set-video-filter) method:

```javascript
let videoFilter = createVideoFilterImplementation();
await applicationCall.setVideoFilter(videoFilter);
```

Another way to set the filter is through
the [`VideoOptions`](https://github.com/infobip/infobip-rtc-js/wiki/VideoOptions)
in [`CallOptions`](https://github.com/infobip/infobip-rtc-js/wiki/CallOptions) object when
making [`ApplicationCall`](https://github.com/infobip/infobip-rtc-js/wiki/ApplicationCall) via
the [`callApplication`](https://github.com/infobip/infobip-rtc-js/wiki/InfobipRTC#call-application) method:

```javascript
let infobipRTC = new InfobipRTC('2e29c3a0-730a-4526-93ce-cda44556dab5', {debug: true});
infobipRTC.connect();

let callOptions = CallOptions.builder().setVideo(true).setVideoFilter(() => createVideoFilterImplementation()).build()
let applicationCall = infobipRTC.callApplication('45g2gql9ay4a2blu55uk1628', callOptions);
```
