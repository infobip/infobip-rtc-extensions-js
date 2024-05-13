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

There are a few ways in which you can get our library. We publish it as an NPM package and as a standalone JS file
hosted on a CDN.

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

The Infobip RTC supports user-defined audio filters which manipulate outgoing audio streams during a call. This library
implements several audio filters which are easy to configure and use.

Filters which are currently available are:

- [`BackgroundMusicAudioFilter`](#background-music-audio-filter)
- [`NoiseSuppressionFilter`](#noise-suppression-filter)

<a name="background-music-audio-filter"></a>

### BackgroundMusicAudioFilter

`BackgroundMusicAudioFilter` allows the user to specify audio to be played alongside their outgoing stream. This audio
is heard by other participants of the call, but not by the user.

To use this filter, an instance of
[`BackgroundMusicAudioFilter`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/BackgroundMusicAudioFilter)
needs to be created. The only parameter of the class constructor is a string containing the URL of the audio file which
is to be combined with the user's outgoing stream. The audio file loops until the filter is turned off or the call is
terminated.

```javascript
const musicURL = "path/to/desired/audio.mp3";
const backgroundMusicFilter = new BackgroundMusicAudioFilter(musicURL);
```

<a name="noise-suppression-filter"></a>

### NoiseSuppressionFilter

The `NoiseSuppressionFilter` enhances speech by removing several types of background noise. This filter works in
real-time. Currently, it is focused on removing background noises commonly encountered in call centers, such as babble,
noise produced by different devices (e.g. air conditioner) and keyboard typing sounds. However, it performs well on a
wider range of noise types. It is also independent of the language spoken.

Since noise suppression uses a neural network, a deep learning model needs to be loaded into the filter. The model
consists of two files:

- `model.json` - a JSON which contains metadata describing the model
- `group1-shard1of1.bin` - a binary file containing model data

To use the noise suppression filter, an instance of the class
[`NoisesuppressionFilter`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/NoiseSuppressionFilter) needs to be
created. Its constructor accepts a single parameter - a string containing a URL of the `model.json` file. The binary
file is then automatically loaded, which is possible only if it is available at the same location as the JSON.

Model files are available at our CDN:

- `model.json` at https://d217hhmiwkst9z.cloudfront.net/noise-suppression-tfjs-model/latest/model.json
- `group1-shard1of1.bin`
  at https://d217hhmiwkst9z.cloudfront.net/noise-suppression-tfjs-model/latest/group1-shard1of1.bin

If you wish to use our CDN directly, it suffices to pass the first URL to the constructor. Another possibility is to
download both files and host them. When doing so, both files must be available at the same path (see example code).

Real-time noise suppression is a hardware-intensive process, so not all hardware will be able to sustain it. In case of
insufficient performance, the filter is automatically disabled in order to not interfere with the call.

```javascript
const modelURL = "https://www.example-cdn.com/noise-suppression-model/model.json";
// group1-shard1of1.bin must be available at https://www.example-cdn.com/noise-suppression-model/group1-shard1of1.bin
const noiseSupressionFilter = new NoiseSuppressionFilter(modelURL);
```

## Video filters

The Infobip RTC supports user-defined video filters capable of manipulating outgoing video streams during calls. The
library provides an extensive implementation of commonly used video filters, making configuration easier and enabling
seamless integration.

Currently available implementations are:

- [`VirtualBackgroundVideoFilter`](#virtual-background-video-filter)

<a name="virtual-background-video-filter"></a>

### VirtualBackgroundVideoFilter

This filter allows users to modify their background during video calls.

Supported video filter modes include:

- Virtual background
  ([`VideoFilterMode.VIRTUAL_BACKGROUND`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VideoFilterMode#virtual-background)) -
  Users can set a custom image to be displayed as their background
- Background blur
  ([`VideoFilterMode.BACKGROUND_BLUR`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VideoFilterMode#background-blur)) -
  Users can blur their background.
- None ([`VideoFilterMode.NONE`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VideoFilterMode#none)) -
  No video filtering is applied; video frames are passed through unchanged. This option is recommended over repeatedly
  reallocating video filter resources to avoid visible disruptions.

To utilize this feature, begin by creating an instance of
the [`VirtualBackgroundVideoFilter`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VirtualBackgroundVideoFilter)
object. The constructor accepts optional
[`VirtualBackgroundVideoFilterOptions`](https://github.com/infobip/infobip-rtc-extensions-js/wiki/VirtualBackgroundVideoFilterOptions)
for customization.

```javascript
const options = {
    mode: VideoFilterMode.VIRTUAL_BACKGROUND,
    image: sourceImage // can be an instance of ImageBitmap, ImageData, HTMLImageElement, …
};
const videoFilter = new VirtualBackgroundVideoFilter(options);
```

For optimal performance, it's recommended to avoid reallocating video filter instances solely for mode changes. Instead,
pass the new options directly to the existing video filter instance. This approach minimizes resource overhead and
enhances overall efficiency.

```javascript
const options = {
    mode: VideoFilterMode.NONE
};
await videoFilter.setOptions(options);
``` 

### Applying the video filter

Once you've created the video filter, you can utilize it during calls.

You can set it beforehand when initiating a
new [`ApplicationCall`](https://github.com/infobip/infobip-rtc-js/wiki/ApplicationCall)
using [`VideoOptions`](https://github.com/infobip/infobip-rtc-js/wiki/VideoOptions) object within
the [`ApplicationCallOptions`](https://github.com/infobip/infobip-rtc-js/wiki/ApplicationCallOptions) object:

```javascript
let token = obtainToken();
let infobipRTC = new InfobipRTC(token);
infobipRTC.connect();

let videoOptions = VideoOptions.builder().setVideoFilterFactory(() => videoFilter).build();
let applicationCallOptions = ApplicationCallOptions.builder().setVideo(true).setVideoOptions(videoOptions).build();
let applicationCall = infobipRTC.callApplication('45g2gql9ay4a2blu55uk1628', applicationCallOptions);
```

Alternatively, you can apply the filter to the
existing [`ApplicationCall`](https://github.com/infobip/infobip-rtc-js/wiki/ApplicationCall) using the
[`setVideoFilter`](https://github.com/infobip/infobip-rtc-js/wiki/ApplicationCall#set-video-filter) method:

```javascript
await applicationCall.setVideoFilter(videoFilter);
```