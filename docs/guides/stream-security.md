---
title: Stream security 
description: This guide explains stream security options in Ant Media Server, and how you can Enable Disable, or Accept Undefined Streams.
keywords: [Enable or Disable Undefined Streams, Accept Undefined Streams, One Time Token Control, Stream security, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 4
---

# Stream security

This guide explains stream security options in Ant Media Server. 


## One Time Token Control

One Time Token Control feature usage is in Dashboard / Application(LiveApp or etc.) / Publish/Play with One-time Tokens section.

![onetime-token](https://github.com/ant-media/ant-media-documentation/assets/86982446/2f118822-f997-4326-a5cc-f367e548bcd8)

By enabling this option, one-time tokens are required for publishing and playing. Publish/Play requests without tokens will not be streamed.

If One-Time Token control is enabled, then all publish and play requests should be sent with a token parameter.

After version 2.2 one time token security option in the Dashboard will be divided into two parts. There will be separate options for enabling/disabling one time tokens for publishing and for playing. This will allow for example using one time tokens for only players and hash-based tokens (or no security) for publishers or vice-versa.

### Create a Token in Publish & Play Scenario

The Server creates tokens with [getTokenV2](https://github.com/ant-media/Ant-Media-Server/blob/master/src/main/java/io/antmedia/rest/BroadcastRestService.java) Rest Service getting ```streamId, expireDate``` and ```type``` parameters with query parameters. Service returns tokenId and other parameters. It is important that ```streamId``` and type parameters should be defined properly. Because ```tokenId``` needs to match with both ```streamId``` and type.

The sample token creation service URL in Publish Scenario:

    http://[IP_Address]:5080/`<Application_Name>`/rest/v2/broadcasts/`<Stream_Id>`/token?expireDate=`<Expire_Date>`&type=publish

The sample token creation service URL in Play Scenario:

    http://[IP_Address]:5080/`<Application_Name>`/rest/v2/broadcasts/`<Stream_Id>`/token?expireDate=`<Expire_Date>`&type=play

Expire Date format is Unix Timestamp. Check also ->` [https://www.epochconverter.com/](https://www.epochconverter.com/)

### RTMP & SRT URL usage

    rtmp://IP_Address/Application_Name/StreamId?token=tokenId
    
    srt://IP_Address:4200?Application_Name/streamid,token=tokenId

Here are the OBS settings for the One Time Token.

![](@site/static/img/ant-media-server-one-time-token.png)

### Live Stream / VoD URL usage

```html
<!-- VOD -->
http(s)://{ant-media-server}:{port}/<Application_Name>/streams/{stream_id}.mp4?token={tokenId}
```

```html
<!-- HLS -->
http(s)://{ant-media-server}:{port}/<Application_Name>/streams/{stream_id}.m3u8?token={tokenId}
```

```html
<!-- Embedded Player -->
http(s)://{ant-media-server}:{port}/<Application_Name>/play.html?name={stream_id}&playOrder=hls&token={tokenId}
```

### WebRTC usage

#### Playing usage

Again the token parameter should be inserted to play WebSocket message. Also please have a look at the principles described in the [WebRTC playing page](https://antmedia.io/docs/guides/publish-live-stream/webrtc/webrtc-websocket-messaging-reference/#playing-webrtc-stream).

```shell
# Secure WebSocket: 
wss://{ant-media-server}:5443/WebRTCAppEE/websocket

# Non Secure WebSocket: 
ws://{ant-media-server}:5080/WebRTCAppEE/websocket
```

```json
{
  command : "play",
  streamId : "stream1",
  token : "tokenId",
}
```
#### Publishing usage

Again the token parameter should be inserted to WebSocket message. Also please have a look at the principles described in the [WebRTC publishing page](https://antmedia.io/docs/guides/publish-live-stream/webrtc/webrtc-websocket-messaging-reference/#publishing-webrtc-stream).

```shell
# Secure WebSocket: 
wss://{ant-media-server}:5443/WebRTCAppEE/websocket

# Non Secure WebSocket: 
ws://{ant-media-server}:5080/WebRTCAppEE/websocket
```

```json
{
  command : "publish",
  streamId : "stream1",
  token : "tokenId",
}
```

## Time based One Time Password

The Time-based One-time Password algorithm (TOTP) is an extension of the HMAC-based One-time Password algorithm (HOTP) that generates a one-time password (OTP) by instead taking uniqueness from the current time.

We define a publisher or player as a subscriber. If time based token enabled, a subscriber should be created for the stream to able to publish or play. Each subscriber has an ID and a code. When a subscriber requests to publish or play a stream, he should provide his ID and time based token generated for his code. Otherwise server doesn't accept the publish or play request.

### Enabling and Setting

You can enable TOTP using the web Panel or in configuration file as ```settings.timeTokenSubscriberOnly=true``` You can also set TOTP period in seconds in configuration file as ```settings.timeTokenPeriod=60```

![](@site/static/img/image-1671624507845.png)
-----------------------------------------------------------------------------------------------------------------

### Subscriber Operations

After enabling TOP in the server the following operations should be performed to publish or play by using TOTP.

*   Admin creates a new subscriber (publisher or player) by using this rest method. You should assign a base 32 secret to each subscriber at the creation. A secret should be in length of multiple of 8 characters .

#### Curl example for publisher type subscriber creation._

```shell
curl -X POST -H "Accept: Application/json" -H "Content-Type: application/json" http://localhost:5080/WebRTCAppEE/rest/v2/broadcasts/stream1/subscribers -d '{"subscriberId":"publisherA", "b32Secret":"mysecret", "type":"publish"}'
```

#### Curl example for player type subscriber creation._

```shell
curl -X POST -H "Accept: Application/json" -H "Content-Type: application/json" http://localhost:5080/WebRTCAppEE/rest/v2/broadcasts/stream1/subscribers -d '{"subscriberId":"playerB", "b32Secret":"mysecret", "type":"play"}'
```
    

*   Subscriber (Publisher or Player) needs to have a TOTP token to publish or play the stream. This token should be created using subscriber secret key. [Here](https://totp.danhersam.com/) is an example page that creates TOTP.
*   Subscriber (Publisher or Player) can request publish or play using the created TOTP.

#### Example of a publish request:_

```html
http://localhost:5080/WebRTCAppEE/?subscriberId=publisherA&subscriberCode=440456
```
    

#### Example of a play request:_

```html
http://localhost:5080/WebRTCAppEE/play.html?subscriberId=playerB&subscriberCode=438610
```

![](https://github.com/ant-media/Ant-Media-Server/wiki/images/totp_messages.png)

You can find create, delete, list REST Methods references from [REST API Reference](https://antmedia.io/rest)

### Subscriber Statistics

You can also get the some statistics like connection events, average bitrate for each subscriber with the following REST method.

```shell
curl -i -H "Accept: Application/json" -X GET "http://localhost:5080/WebRTCAppEE/rest/v2/broadcasts/stream1/subscriber-stats/list/0/5"
```

## Accepting Undefined Streams

This setting shortly is checking if the live stream is registered in Ant Media Server.

For example: If Ant Media Server accepts undefined streams, it accepts any incoming streams. If accepting undefined Streams is disabled, only streams with their stream id in the database are being accepted by Ant Media Server.

You can find in more detail [here](/guides/configuration-and-testing/ams-application-configuration)

After modifying the configuration, please add the streamId, and stream name in "broadcast" collections of your App.

![undefined-streams](https://github.com/ant-media/ant-media-documentation/assets/86982446/f456c3e9-dbae-42af-8a6f-34ee0aa177e8)

## JWT Stream Security Filter

JWT Stream Security feature is enabled/disabled in ```Dashboard/LiveApp( or any other)/ Settings/Publish/Play with JWT Filter```. Just take a look at the image for the related part. You can use JWT Stream Security Filter for Stream Publishing and Playing. Publish/Play requests without JWT tokens will not be streamed if you enable the JWT Stream Security Filter as shown below by also adding Secret Key on web panel.

![](@site/static/img/ant-media-server-jwt-stream-security-filter-dashboard.png)

After version 2.3.3 JWT Stream Security filter option in the Dashboard will be divided into two parts. There will be separate options for enabling/disabling JWT Stream Security for publishing and for playing. This will allow for example using JWT Stream Security for only players and other security (or no security) for publishers or vice-versa.

### Generate JWT Token

Let's assume that our secret key is ```zautXStXM9iW3aD3FuyPH0TdK4GHPmHq``` so that we just need to create a JWT token. Luckily, there are plenty of [libraries available for JWT](https://jwt.io/#libraries-io) for your development. For our case, we will just use [Debugger at JWT](https://jwt.io/#debugger-io).

![](@site/static/img/generate-jwt-stream-token.png)

As shown above, we use HS256 as algorithm and use our secret key ```zautXStXM9iW3aD3FuyPH0TdK4GHPmHq``` to generate the token. The payload is not critical for this authorization. You can use any payload to create the token. On the server side, it just checks the token is signed with secret key. So that our JWT token to publish/play the stream is:

![](@site/static/img/generate-jwt-stream-token-with-expiration.png)  

As shown above, the expiration time of the token is Mar 08, 2021 02:14:08 GMT+3. It means that you can use the generated token until the expiration time. The unit of expiration time is [unix timestamp](https://www.unixtimestamp.com/). When it expires, the JWT token becomes invalid.

#### Generate JWT Token with REST API

You can also generate Publish/Play JWT Token with REST API. The Server creates JWT tokens with [getJwtTokenV2](https://antmedia.io/rest/#/BroadcastRestService/getJwtTokenV2) Rest Service getting ```streamId```, ```expireDate``` and ```type``` parameters with query parameters. Service returns ```tokenId``` and other parameters. It is important that ```streamId``` and ```type``` parameters should be defined properly. Because ```tokenId``` needs to match with both ```streamId``` and ```type```.

**The sample JWT token creation service URL in Publish Scenario:**

    http://[IP_Address]:5080/`<Application_Name>`/rest/v2/broadcasts/`<Stream_Id>`/jwt-token?expireDate=`<Expire_Date>`&type=publish

**The sample JWT token creation service URL in Play Scenario:**

    http://[IP_Address]:5080/`<Application_Name>`/rest/v2/broadcasts/`<Stream_Id>`/jwt-token?expireDate=`<Expire_Date>`&type=play

Expire Date format is Unix Timestamp. Check also ->` [https://www.epochconverter.com/](https://www.epochconverter.com/)

### JWT Token (Publish)

#### RTMP and SRT usage

```shell
rtmp://{ant-media-server}/<APPLICATION_NAME>/{stream_id}?token={tokenId}

srt://{ant-media-server}:4200?<APPLICATION_NAME>/{stream_id},token={tokenId}
```

Here is the OBS setting for the JWT Token:

![](@site/static/img/ant-media-server-one-time-token.png)

#### WebRTC Publish Usage

Again the JWT token parameter should be inserted to publish WebSocket message. Also please have a look at the principles described in the [WebRTC publishing page](https://antmedia.io/docs/guides/publish-live-stream/webrtc/webrtc-websocket-messaging-reference/#publishing-webrtc-stream).

```json
{
  command : "publish",
  streamId : "stream1",
  token : "tokenId",
}
```

**This feature is available in Ant Media Server 2.3.3+ versions.**

### JWT Token (Play)

#### HLS/VoD & Embedded Player Usage


```html
<!-- VOD Playback -->
http(s)://{ant-media-server}:{port}/<Application_Name>/streams/streamID.mp4?token={JWT_tokenId}
```

```html
<!-- HLS Playback -->
http(s)://{ant-media-server}:{port}/<Application_Name>/streams/{stream_id}.m3u8?token={JWT_tokenId}
```

```html
<!-- Embedded Player -->
http(s)://{ant-media-server}:{port}/<Application_Name>/play.html?name={stream_id}&playOrder=hls&token={JWT_tokenId}
```

#### WebRTC Play Usage

Again the JWT token parameter should be inserted to play WebSocket message. Also please have a look at the principles described in the [WebRTC playing page](https://antmedia.io/docs/guides/publish-live-stream/webrtc/webrtc-websocket-messaging-reference/#playing-webrtc-stream).

```json
{
  command : "play",
  streamId : "stream1",
  token : "tokenId",
}
```

**This feature is available in Ant Media Server 2.3.3+ versions.**


## Hash-Based Token

Firstly, settings should be enabled from the settings file of the application in ```SERVER_FOLDER``` / ```webapp``` / ```{Application}``` / ```WEB-INF``` / ```red5-web.properties```

```javascript
settings.hashControlPublishEnabled=true
settings.hashControlPlayEnabled=true
tokenHashSecret=PLEASE_WRITE_YOUR_SECRET_KEY
```

Set true ```settings.hashControlPublishEnabled``` to enable secret based hash control for publishing operations, and ```settings.hashControlPlayEnabled``` for playing operations.

:::warning
Do not forget to define a secret key for generating a hash value.
:::

### Publishing Scenario

**Step 1. Generate a Hash**

You need to generate a hash value using the formula ```sha256(STREAM_ID+ROLE+SECRET)``` for your application and send to your clients. The values used for hash generation are:

    STREAM_ID: The id of stream, generated in Ant Media Server.
    ROLE: It is either "play or "publish"
    SECRET: Shared secret key (should be defined in the setting file)

**Step 2. Request with Hash**

The system controls hash validity during publishing or playing. **Keep in mind that there is NO '+' in calculating the hash in this formula** ```**sha256(STREAM_ID+ROLE+SECRET)**``` Here is an example for that.

Let's say ```STREAM_ID: stream1```, ```ROLE: publish```, ```SECRET: this_is_secret``` Your hash is the result of this calculation: ```sha256(stream1publishthis_is_secret)```

Go to [JavaScript SHA-256](https://geraintluff.github.io/sha256/) for online demo

**RTMP Publishing:** You need to add a hash parameter to RTMP URL before publishing. Sample URL:

    rtmp://IP_Address/Application_Name/StreamId?token=hash
    
    srt://IP_Address:4200?Application_Name/streamid,token=hash

Here is OBS settings for the Hash-Based Token

![](@site/static/img/ant-media-server-one-time-token(1).png)

**WebRTC Publishing:** Hash parameter should be inserted to publish WebSocket messages.

    {
    command : "publish",
    streamId : "stream1",
    token : "hash",
    }

### Playing Scenario

**Step 1. Generate a Hash**

You need to generate a hash value using the formula sha256(STREAM\_ID + ROLE + SECRET) for your application and send to your clients. The values used for hash generation are:

    STREAM_ID: The id of stream, generated in Ant Media Server.
    ROLE: It is either "play or "publish"
    SECRET: Shared secret key (should be defined in the setting file)

**Step 2. Request with Hash**

**Live Stream/VoD Playing:** Same as publishing, the hash parameter is added to the URL. Sample URL:

    http://[IP_Address]/`<Application_Name>`/streams/`<Stream_Id_or_Source_Name>`?token=hash

**WebRTC Playing:** Again the hash parameter should be inserted to play WebSocket message.

    {
    command : "play",
    streamId : "stream1",
    token : "hash",
    }

> Please have a look at the principles described in the [WebRTC WebSocket page](https://antmedia.io/docs/guides/publish-live-stream/webrtc/webrtc-websocket-messaging-reference/).

### Evaluation of the Hash

If related settings are enabled, Ant Media Server first generates hash values based on the formula sha256(STREAM\_ID + ROLE + SECRET) using streamId, role parameters and secret string which is defined in the settings file.

Then compare this generated hash value with the client's hash value during authentication.

Once the hash is successfully validated by Ant Media Server, the client is granted either to publish or play according to application setting and user request.

## CORS Filter

CORS(Cross-Origin Resource Sharing) Filter is enabled and accepts requests from everywhere by default.

If you want to customize by yourself CORS Filters in Application, you can access in ```SERVER_FOLDER``` / ```webapps``` / ```{Application}``` / ```WEB-INF``` / web.xml

```xml
<filter>
      <filter-name>CorsFilter</filter-name>
      <filter-class>io.antmedia.filter.CorsHeaderFilter</filter-class>
      <init-param>
          <param-name>cors.allowed.origins</param-name>
          <param-value>*</param-value>
        </init-param>
        <init-param>
            <param-name>cors.allowed.methods</param-name>
            <param-value>GET,POST,HEAD,OPTIONS,PUT,DELETE</param-value>
        </init-param>

        <!-- cors.allowed.origins -> * and credentials are not supported at the same time.
        If you set to cors.allowed.origins to specific domains and support credentials open the below lines
        <init-param>
            <param-name>cors.support.credentials</param-name>
            <param-value>true</param-value>
        </init-param>
          -->
        <init-param>
            <param-name>cors.allowed.headers</param-name>
            <param-value>Accept, Origin, X-Requested-With, Access-Control-Request-Headers, Content-Type, Access-Control-Request-Method, Authorization</param-value>
          </init-param>
          <async-supported>true</async-supported>
</filter>
<filter-mapping>
      <filter-name>CorsFilter</filter-name>
      <url-pattern>/*</url-pattern>
</filter-mapping>
```

If you want to customize by yourself CORS Filters in Root, you can access in ```SERVER_FOLDER``` / ```webapps``` / ```root``` / ```WEB-INF``` / web.xml

```xml
<filter>
  <filter-name>CorsFilter</filter-name>
  <filter-class>io.antmedia.filter.CorsHeaderFilter</filter-class>
  <init-param>
    <param-name>cors.allowed.origins</param-name>
    <param-value>*</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.methods</param-name>
    <param-value>GET,POST,HEAD,OPTIONS,PUT,DELETE</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>CorsFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

:::info
Quick Learn: [Tomcat CORS Filter](https://tomcat.apache.org/tomcat-8.0-doc/api/index.html?org/apache/catalina/filters/CorsFilter.html)
:::

## Publisher IP Filter

:::info
Publisher IP Filter feature is available for later versions of the 1.9.0+ version.
:::

Publisher IP filter feature allows you to specify the IP addresses allowed for publishing. You can define multiple allowed IPs in CIDR format as comma (,) separated.

To enable publisher IP filtering you must set ```settings.allowedPublisherCIDR``` in ```AMS_DIR/webapps/<App_Name>/WEB_INF/red5.properties``` file with the allowed IP addresses.

Example: 

```json
settings.allowedPublisherCIDR=10.20.30.40/24,127.0.0.1/32
``` 

allows IPs 10.20.30.\[0-255\] and 127.0.0.1.

You can [read more](https://whatismyipaddress.com/cidr/) about CIDR notation.

## Subscriber Block

The subscriber block feature allows blocking a specific user from engaging in playback, publishing, or both at any given moment. This implies that even if the user is actively publishing or playing the stream, their ability to publish or play will cease until the block is removed or expires. Block is valid for all publish and play types. Subscriber block feature can be used in version 2.7.0 and later.

### Block Play

To utilize subscriber block for play, first enable TOTP(Time-based One-Time Password) for play through web panel application settings.

![](@site/static/img/stream-security/subscriber_block_enable_play_totp.png)

Instead of using the UI for activation, you can alternatively set the parameters 
    
    timeTokenSubscriberOnly=true
    timeTokenSecretForPlay="O24FW6"
    

in the advanced section of the app settings.

By default, the TOTP generated for playback remains valid for 60 seconds after its generation. Consequently, users intending to utilize this token must send a play request to AMS within this 60-second timeframe.
You can change default TOTP time by adding
    
    timeTokenPeriod=60
to app settings.
When this setting is enabled, users without TOTP won't be able to watch the streams.

Now, you can send a **GET** request to generate TOTP for the subscriber:

```plaintext
http://localhost:5080/WebRTCAppEE/rest/v2/broadcasts/{streamId}/subscribers/{subscriberId}/totp?type=play
```
Add `"type":"play"` as a query parameter to the GET request.
![](@site/static/img/stream-security/subscriber_block_play_totp_postman.png)
Example curl:

```bash
curl --location 'http://localhost:5080/LiveApp/rest/v2/broadcasts/teststream/subscribers/lastpeony/totp?type=play'
```
Retrieve the token from the dataId field of the returned object.

As an illustration, suppose you are associating your users with `userIds` in your application. When users initiate playback on your application, you can transmit their `userId` as the `subscriberId` and issue a TOTP generation request to AMS (Ant Media Server). Once you receive the token, pass it to the Ant Media Server SDK to commence the user's playback.

Users must pass TOTP as `subscriberCode` to Ant Media Server from the SDK when sending a play request.

Example:
```plaintext
http://localhost:5080/LiveApp/play.html?id=teststream&subscriberId=lastpeony&subscriberCode=956364
```
Upon a user successfully initiating a play request with TOTP, they become authenticated and gain access to watch the stream. It's important to note that they won't be able to play the stream if they refresh the page and their TOTP has expired. However, if the TOTP is still valid and they refresh, they will be reauthenticated and able to resume playing the stream.

Now you can utilize the subscriber block feature to prevent this user from playing for the desired duration.
To block a subscriber play, send a **PUT** request to:

```plaintext
http://localhost:5080/LiveApp/rest/v2/broadcasts/{streamId}/subscribers/block/{blockDurationInSeconds}/{blockType}
```
The blockType can be `play`, `publish`, or `publish_play`.

For example, to block subscriber "lastpeony" from playing for 120 seconds, use this request:
```plaintext
http://localhost:5080/LiveApp/rest/v2/broadcasts/teststream/subscribers/lastpeony/block/120/play
```

![](@site/static/img/stream-security/subscriber_block_block_play_postman.png)

Example curl:

```bash
curl --location --request PUT 'http://localhost:5080/LiveApp/rest/v2/broadcasts/teststream/subscribers/lastpeony/block/120/play'
```
Note that the player's playback immediately stops upon a successful request.

After 120 seconds, the user will regain access to play the stream. To manually remove the block, set the block duration to 0:
```bash
curl --location --request PUT 'http://localhost:5080/LiveApp/rest/v2/broadcasts/teststream/subscribers/lastpeony/block/0/play'
```
Please be aware that playback resumes immediately after this request returns successfully.

### Block Publish

For the publish block, the steps are similar to play block. Initially, enable TOTP for publishing through the web panel and generate a secret for it.

![](@site/static/img/stream-security/subscriber_block_enable_publish_totp.png)

For publishing, similar to the play scenario, you have the option to set up TOTP without using the UI. You can achieve this by adding the following parameters in the advanced section of the app settings: 

```plaintext
timeTokenSubscriberOnly=true
timeTokenSecretForPublish="4ULgui"
```

Remember to save the application settings after making these changes.

Next, to generate a TOTP for the subscriber, send a GET request:
```plaintext
http://localhost:5080/WebRTCAppEE/rest/v2/broadcasts/{streamId}/subscribers/{subscriberId}/totp?type=publish
```
Include "type":"publish" as a query parameter in the GET request.

![](@site/static/img/stream-security/subscriber_block_publish_totp_postman.png)

Example curl:

```bash
curl --location 'http://localhost:5080/LiveApp/rest/v2/broadcasts/teststream/subscribers/lastpeony/totp?type=publish'
```
After obtaining the TOTP token, use it as secretCode in your publish requests to Ant Media Server SDKs or in RTMP publish requests.

For instance, when using the JavaScript SDK, the publish command should be called as shown below:
```
webRTCAdaptor.publish(streamId, tokenId, subscriberId, subscriberCode);
```
Example:
```
webRTCAdaptor.publish("teststream", null, "lastpeony", "451222");
// (The 2nd parameter, which is null here, represents the token(for example a JWT), not subscriber code)
```
Likewise, for RTMP publishing, ensure the inclusion of the parameters streamId, subscriberId, and subscriberCode. Your OBS configuration should resemble this setup:
![](@site/static/img/stream-security/subscriber_block_obs_publish.png)

Example ffmpeg command to publish with RTMP:

```bash
ffmpeg -re -i input_source -c:v libx264 -c:a aac -f flv "rtmp://172.17.52.70/LiveApp/teststream?subscriberId=lastpeony&subscriberCode=451222"
```
As you're now utilizing TOTP for publishing, you can block this subscriber from publishing using a block request. To prevent the user from publishing for 120 seconds, send a subscriber block PUT request to:
```
http://localhost:5080/LiveApp/rest/v2/broadcasts/teststream/subscribers/lastpeony/block/120/publish
```
![](@site/static/img/stream-security/subscriber_block_block_publish_postman.png)

Example curl:

```bash
curl --location --request PUT 'http://localhost:5080/LiveApp/rest/v2/broadcasts/teststream/subscribers/lastpeony/block/120/publish'
```
Upon a successful return of this request, the subscriber's publishing will immediately stop, and they will be blocked for 120 seconds.

To remove the block, set the block duration to 0 seconds:
```bash
curl --location --request PUT 'http://localhost:5080/LiveApp/rest/v2/broadcasts/teststream/subscribers/lastpeony/block/0/publish'
```
Remember, if you previously blocked subscribers from publishing and then unblocked them, they might encounter an "unauthorized_access" error if their TOTP has expired. In such cases, generating a new TOTP becomes necessary for them to publish again.
