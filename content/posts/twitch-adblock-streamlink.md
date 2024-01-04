---
title: "Blocking Twitch Ads by piping the stream to VLC"
date: 2023-10-14T08:00:23+02:00
description: Your browser's ad blocker is not working on Twitch? Read this.
draft: false
tags: [windows, tutorial]
---

## Introduction
Twitch keeps changing their backend, and it recently became more difficult to block ads. A few years ago, any ad blocker seemed to work just fine. Then they changed their code, and uBlock Origin, Adblock Plus, etc. all fail to remove ads. While there are still browser plugins that promise to block Twitch ads, many of them are potentially shady, as they route your traffic over some proxy server that does who-knows-what. Since I don't like routing my traffic over shady servers and I want to avoid testing browser extensions that have way too many privileges, I tried to find a different approach. In this post, I will talk about two approaches: 
* [Streamlink](https://github.com/streamlink/streamlink), which is an excellent tool for routing a stream to a video player you have installed locally. It supports plugins, and there are [many plugins](https://streamlink.github.io/plugins.html#twitch) available
* [Yt-dlp](https://github.com/yt-dlp/yt-dlp), which is a better version of the well-known youtube-dl.

## Setup
Install VLC and add it to your path. This just looks better than having to type out the full path to the VLC executable in the commands later.

### Streamlink
The setup on Windows is simple: install the required Python version, download the binary for your version, add VLC to the path, and then just use the following command in the terminal:
```cmd
streamlink.exe -p vlc <twitch-channel-url> best --twitch-disable-ads
```
This will automatically open VLC with the Twitch stream playing at the highest quality. The last parameter "--twitch-disable-ads" is taken from the Twitch Plugin documentation and sadly does not work as it should: while an ad is playing, the stream stops (frame freezes) and only continues after the ad is over (you don't see or hear anything of the ad). The same behavior happens if you don't use the "--twitch-disable-ads" flag. The only difference here is that the frozen frame shown is not the last frame of the stream you are watching, but instead the Twitch banner shown below:

![targets](/images/twitch-stream-adblocking.png "Screenshot")

Okay, but why does the "--twitch-disable-ads" flag not work as expected? I did read in the documentation that VLC is not a recommended video player because of how it handles stream discontinuities. If you look at the CMD output during an ad, it will show:

![targets](/images/streamlink-adblock-error.png "Screenshot")

If you read through the [Twitch Plugin Documentation](https://streamlink.github.io/cli/plugins/twitch.html#embedded-ads), you will notice that the authors mention this problem. While VLC did not crash for me, there were stream discontinuity warnings shown in the command line. But using a different player than VLC will not fix this issue, the authors of the documentation write: "Unfortunately, entirely preventing embedded ads is not possible unless a loophole on Twitch gets discovered which can be exploited. This has been the case a couple of times now and ad-workarounds have been implemented in Streamlink (see #3210) and various ad-blockers, but the solutions did only last for a couple of weeks or even days until Twitch patched these exploits."

The only hope seems to be to set special API request headers using the "--twitch-access-token-param" flag that might or might not prevent embedded ads. However, even if you find a value that works, it is likely that Twitch will patch the workaround shortly after. Since my goal was to not be annoyed by repetitive ads, I am happy with the current solution of having an automatically paused stream.

### Yt-dlp
I already had this tool on my machine, so after you download it just open a CMD in the folder where yt-dlp.exe lives and run the following command:
```cmd
yt-dlp.exe --get-url "https://www.twitch.tv/mission_anime" | vlc -
```
This stream behaves almost the same as Streamlink without "--twitch-disable-ads". When there is an ad break, you will see the following:

![targets](/images/yt-dlp-twitch.png "Screenshot")

The difference to before is that now this image has an animated background (who cares), but there also is a countdown bottom-right which shows how long the current ad would take. But it turns out that this is useless, because it does not tell you how many ads you would get in a row. So it might go up to 15 seconds three times, which means that even with the countdown you have no idea how long the ad break will take. Sound is muted just like before.

## Conclusion
Yt-dlp and streamlink are both good tools for using locally installed video players like VLC instead of having to visit bloated streaming websites. While they do allow you to mute ads (by stopping the video and silencing audio), Twitch does all it can to make it difficult to remove embedded ads. If you truly care about an ad-free Twitch experience, subscribing to Twitch Nitro or using your monthly Twitch Prime (assuming you have Amazon Prime) on your favorite channel  is the easiest and most boring solution. I do think that this post is now more relevant than ever, because Youtube [recently started taking a more aggressive stance in detecting ad blockers...](https://www.ghacks.net/2023/10/10/youtube-is-cracking-down-on-ad-blockers-more-aggressively-heres-how-to-bypass-it/)