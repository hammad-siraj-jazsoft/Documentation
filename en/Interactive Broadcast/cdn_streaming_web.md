
---
title: Push Streams to CDN
description: 
platform: Web
updatedAt: Fri Mar 27 2020 06:11:10 GMT+0800 (CST)
---
# Push Streams to CDN
## Introduction

The process of publishing streams into the CDN (Content Delivery Network) is called CDN live streaming, where users can view the live broadcast through a web browser.

When multiple hosts are in a channel in CDN live streaming, [transcoding](https://docs.agora.io/en/AgoraPlatform/terms?platform=AllPlatforms#transcoding) is used to combine the streams of all the hosts into a single stream. Transcoding sets the audio/video profiles and the picture-in-picture layout for the stream to be pushed to the CDN.

![](https://web-cdn.agora.io/docs-files/1569414336352)


<div class="alert info">Click the <a href="https://webdemo.agora.io/agora-web-showcase/examples/Agora-Interactive-Broadcasting-Live-Streaming-Web/">online demo</a> to try this feature out.</div>

## Prerequisites

Ensure that you enable the RTMP Converter service before using this function.

1. Log in [Console](https://dashboard.agora.io/), and click ![img](https://web-cdn.agora.io/docs-files/1551260936285) in the left navigation menu to go to the **Products & Usage** page. 
2. Select a project from the drop-down list in the upper-left corner, and click **Duration** under **RTMP Converter**. 
![](https://web-cdn.agora.io/docs-files/1569302661254)
3. Click **Enable RTMP Converter**.
4. Click **Apply** to enable the RTMP Converter service and get 500 max concurrent channels.

<div class="alert note"> The number of concurrent channels, N, means that users can push streams to the CDN with transcoding in N channels of media streams. </div>

Now, you can use the function and see the usage statistics.


## Implementation 

Before proceeding, ensure that you implement a basic live broadcast in your project. See [Start a Live Broadcast](../../en/Interactive%20Broadcast/start_live_web.md) for details.


Refer to the following steps to push streams to the CDN:

<a name="single"></a>
1. The host in a channel calls the `Client.setLiveTranscoding` method to set the transcoding parameters of the media streams (`LiveTranscoding`), such as the resolution, bitrate, frame rate, background color, and watermark position. If you need a transcoded picture, set the picture-in-picture layout for each user in the `TranscodingUser` class, as described in [Sample Code](#sample).

   > The `Client.on("liveTranscodingUpdated")` callback occurs when the `LiveTranscoding` class updates and reports the updated information to the local host.

2. The host in a channel calls the `Client.startLiveStreaming` method to add a media stream to the CDN. 

   > Use `enableTranscoding` to set whether or not to enable transcoding.

3. The host in a channel can call the `Client.stopLiveStreaming` method to remove a media stream from the CDN live broadcast.


When adding/removing a media stream in the CDN live broadcast, the SDK triggers some callbacks in `Client.on` to report  the current state of pushing streams to the local host. See [API Reference](#api).



<a name="sample"></a>
### Sample code

```javascript
// CDN transcoding settings.
var LiveTranscoding = {
            // Width of the video (px). The default value is 640.
            width: 640,
            // Height of the video (px). The default value is 360.
            height: 360,
            // Bitrate of the video (Kbps). The default value is 400.
            videoBitrate: 400,
            // Frame rate of the video (fps). The default value is 15. Agora adjusts all values over 30 to 30.
            videoFramerate: 15,
            audioSampleRate: AgoraRTC.AUDIO_SAMPLE_RATE_48000,
            audioBitrate: 48,
            audioChannels: 1,
            videoGop: 30,
            // Video codec profile. Choose to set as Baseline (66), Main (77), or High (100). If you set this parameter to other values, Agora adjusts it to the default value of 100.
            videoCodecProfile: AgoraRTC.VIDEO_CODEC_PROFILE_HIGH,
            userCount: 1,
            userConfigExtraInfo: {},
            backgroundColor: 0x000000,
		    // Adds a PNG watermark image to the video. You can add more than one watermark image at the same time.
            images: [{
                    url: "http://www.com/watermark.png",
                    x: 0,
                    y: 0,
                    width: 160,
                    height: 160,
                }],
            // Sets the output layout for each user.
            transcodingUsers: [{
                    x: 0,
                    y: 0,
                    width: 640,
                    height: 360,
                    zOrder: 0,
                    alpha: 1.0,
                   // The uid must be identical to the uid used in Client.join.
                    uid: 1232,
                  }],
          };
  
client.setLiveTranscoding(LiveTranscoding);
  
// Adds a URL to which the host pushes a stream. Set the transcodingEnabled parameter as true to enable the transcoding service. Once transcoding is enabled, you need to set the live transcoding configurations by calling the setLiveTranscoding method. We do not recommend transcoding in the case of a single host.
client.startLiveStreaming("your RTMP URL", true)
 
 
// Removes a URL to which the host pushes a stream.
client.stopLiveStreaming("your RTMP URL")
```

We also provide an open-source [Live-Streaming](https://github.com/AgoraIO/Advanced-Interactive-Broadcasting/tree/master/Live-Streaming/Agora-Interactive-Broadcasting-Live-Streaming-Web-Webpack) demo project on GitHub. You can [try the demo](https://webdemo.agora.io/agora-web-showcase/examples/Agora-Interactive-Broadcasting-Live-Streaming-Web/), or view the source code in the [index.js](https://github.com/AgoraIO/Advanced-Interactive-Broadcasting/blob/master/Live-Streaming/Agora-Interactive-Broadcasting-Live-Streaming-Web-Webpack/src/index.js) and [rtc-client.js](https://github.com/AgoraIO/Advanced-Interactive-Broadcasting/blob/master/Live-Streaming/Agora-Interactive-Broadcasting-Live-Streaming-Web-Webpack/src/rtc-client.js) files.


<a name="api"></a>
### API reference 

- [`init`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/web/interfaces/agorartc.stream.html#init)
- [`setLiveTranscoding`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/web/interfaces/agorartc.client.html#setlivetranscoding)
- [`startLiveStreaming`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/web/interfaces/agorartc.client.html#startlivestreaming)
- [`stopLiveStreaming`](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/web/interfaces/agorartc.client.html#stoplivestreaming)
- `liveTranscodingUpdated`
- `liveStreamingStarted`
- `liveStreamingFailed`


## Considerations

- Up to 17 hosts can be supported in the same live broadcast channel.

- In the case of a single host, we do not recommend transcoding. You can skip [Step1](#single), and call the `Client.startLiveStreaming` method directly with `enableTranscoding ("url", false)`.

  > Please use `AgoraRTC.createClient({mode: "live", codec: "h264"})` in this case.

- Set the value of `videoBitrate` by referring to [Video Bitrate Table](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/web/interfaces/agorartc.videoencoderconfiguration.html#bitrate). If you set a bitrate beyond the proper range, the SDK automatically adjusts it to a value within the range. 

- Agora charges a transcoding usage fee for CDN live streaming.
