
# 使用```js <video> ``` 遇到异常RangeNotSatisfiableError
It's a valid request. It's also quite common when the client is requesting media or resuming a download.

A client will often test to see if the server handles ranged requests other than just looking for an Accept-Ranges response. Chrome always sends a Range: bytes=0- with its first GET request for a video, so it's something you can't dismiss.

Whenever a client includes Range: in its request, even if it's malformed, it's expecting a partial content (206) response. When you seek forward during HTML5 video playback, the browser only requests the starting point. For example:

Range: bytes=3744-
So, in order for the client to play video properly, your server must be able to handle these incomplete range requests.

You can handle the type of 'range' you specified in your question in two ways:

First, You could reply with the requested starting point given in the response, then the total length of the file minus one (the requested byte range is zero-indexed). For example:

Request:
```js
GET /BigBuckBunny_320x180.mp4 
Range: bytes=100-
Response:

206 Partial Content
Content-Type: video/mp4
Content-Length: 64656927
Accept-Ranges: bytes
Content-Range: bytes 100-64656926/64656927
```
Second, you could reply with the starting point given in the request and an open-ended file length (size). This is for webcasts or other media where the total length is unknown. For example:

Request:
```js
GET /BigBuckBunny_320x180.mp4
Range: bytes=100-
Response:

206 Partial Content
Content-Type: video/mp4
Content-Length: 64656927
Accept-Ranges: bytes
Content-Range: bytes 100-64656926/*
```
# Tips:

You must always respond with the content length included with the range. If the range is complete, with start to end, then the content length is simply the difference:

```js

Request: Range: bytes=500-1000

Response: Content-Range: bytes 500-1000/123456
```

Remember that the range is zero-indexed, so Range: bytes=0-999 is actually requesting 1000 bytes, not 999, so respond with something like:

```js
Content-Length: 1000
Content-Range: bytes 0-999/123456
```
Or:
```js
Content-Length: 1000
Content-Range: bytes 0-999/*
```
But, avoid the latter method if possible because some media players try to figure out the duration from the file size. If your request is for media content, which is my hunch, then you should include its duration in the response. This is done with the following format:
```js
X-Content-Duration: 63.23 
```
This must be a floating point. Unlike Content-Length, this value doesn't have to be accurate. It's used to help the player seek around the video. If you are streaming a webcast and only have a general idea of how long it will be, it's better to include your estimated duration rather than ignore it altogether. So, for a two-hour webcast, you could include something like:
```js
X-Content-Duration: 7200.00 
```
With some media types, such as webm, you must also include the content-type, such as:
```js
Content-Type: video/webm 
```
All of these are necessary for the media to play properly, especially in HTML5. If you don't give a duration, the player may try to figure out the duration (to allow for seeking) from its file size, but this won't be accurate. This is fine, and necessary for webcasts or live streaming, but not ideal for playback of video files. You can extract the duration using software like FFMPEG and save it in a database or even the filename.

X-Content-Duration is being phased out in favor of Content-Duration, so I'd include that too. A basic, response to a "0-" request would include at least the following:
```js
HTTP/1.1 206 Partial Content
Date: Sun, 08 May 2013 06:37:54 GMT
Server: Apache/2.0.52 (Red Hat)
Accept-Ranges: bytes
Content-Length: 3980
Content-Range: bytes 0-3979/3980
Content-Type: video/webm
X-Content-Duration: 2054.53
Content-Duration: 2054.53
One more point: Chrome always starts its first video request with the following:

Range: bytes=0-
```
Some servers will send a regular 200 response as a reply, which it accepts (but with limited playback options), but try to send a 206 instead to show than your server handles ranges. RFC 2616 says it's acceptable to ignore range headers.
