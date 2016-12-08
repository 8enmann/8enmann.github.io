---
layout: post
title: Introducing Webtendo — open source, multiplayer, in-person game platform
---

**tl;dr** Visit [https://webtendo.herokuapp.com](https://webtendo.herokuapp.com) on your laptop and phone (ideally on the same wifi network and with friends)

# Inspiration
When I was growing up, game consoles were the best way to play games. Nintendo, SEGA, and PlayStation dominated the gaming scene with titles like Mario Kart, Sonic Adventure, and Final Fantasy. But just as Chase Jarvis said "The best camera is the one that's with you", it seems the masses have judged that the best console is the one in your pocket. This year mobile game revenue [is expected](http://www.cnbc.com/2016/04/22/mobile-game-revenue-to-pass-console-pc-for-first-time.html) to surpass PC and console revenue for the first time, and I think it's pretty safe to use revenue as a proxy for usage.

We've lost something in the process: mobile games tend to be solitary. Although there are bright exceptions like Pokémon Go, it's unusual that games bring us physically together like Super Smash or Mario Kart used to. And indeed, Nintendo is and has always been the king of in-person multiplayer video games. I think this is part of the reason for the [resurgence in board game popularity](http://www.vice.com/read/rise-of-board-games). It's not enough to talk to someone over a headset. We need that face to face interaction. And it is especially games like [Mafia](https://en.wikipedia.org/wiki/Mafia_(party_game)), where our personalities most come to the fore, that get me excited. In these cases, we're not just staring at playing cards, we're actively trying to read our opponents, understand who they are, and use all of our faculties to compete with them.

A few weeks ago, a few friends and I realized that using our ubiquitous phones and laptops, we can replicate the multiplayer console experience. The phones become the controllers and the laptop is the console. The experience can be fundamentally the same as 20 years ago, but we can ditch most of the wires, support any number of players, and play anywhere. We can once again laugh and shout with each other.

# Architecture
To support the broadest range of games, latency between the controllers and the host needs to be minimal, preferably below 30 ms, which is the limit of human perception. So for the transport layer [WebRTC](https://webrtc.org/) seemed to be the natural choice. WebRTC allows devices to communicate on the shortest possible network path between them, even if there's NAT or firewalling in the way. It's supported in Chrome on Windows, Mac, and Android, but unfortunately not on Chrome iOS (more on that below). However both iOS and Android have native WebRTC support.

In my tests using Chrome 54 on a Mac and Chrome 54 on Android on the same wifi network, I saw round trip times of about 12 ms, though this spiked much higher if the device was not active. This was the first hint that building a high quality platofrm might be viable. The codelab and most online resources focused on transmitting realtime audio and video rather than data, so getting an absolute minimum example going was an exercise in chopping down and modifying the examples on the net to their minimum possible.

We also want cross platform support, so developers can reach the largest potential audience without extra effort. 

## Learn once, write everywhere
A few years ago I spent months fighting with [PhoneGap/Cordova](https://en.wikipedia.org/wiki/Apache_Cordova). I loved how fast I could iterate and test suite execution speed, but at the end of the day the platform itself was too buggy. When we launched, users complained of random network disconnects even when other apps were chugging smoothly. It was hard to follow native UI paradigms. Eventually I bit the bullet and rewrote everything natively on Android, swearing never again to put so much energy into a platform not proven to work at scale.

Fast forward to spring of this year and all my dev friends are talking about  React Native, and how magical it is to be able to use all our JS tooling but still write native apps. It seemed too good to be true, but as I tried it in exploring a side project it turned out to be a wonderful development experience. It has its quirks but waiting for native compilation and deploy, rather than just hitting save and instantly seeing changes, is unforgiveable.

That said, React native isn't really built to share code across vanilla React and React Native, and on top of that I didn't want to tie developers down to a single layout engine. Maybe someone will make a game in WebGL, another in Canvas, and another in ThreeJS. So I made an semi-awful hack where the transport layer, which doesn't work in iOS web, is in React Native, and everything else is a webview in React Native. There's some [tricksy bridge code](https://github.com/8enmann/webtendo/blob/master/public/scripts/webtendo.js#L79) that allows the webview to talk to the native environment and a small cost for serializing and deserializing the JS, but in practice performance seems unaffected!

```javascript
// Connect to the signaling server if we have webrtc access.
var socket;
try {
  new RTCSessionDescription();
  socket = io.connect();
  attachListeners(socket);
} catch (e) {
  console.log('No WebRTC detected, bailing out.');
  setTimeout(() => {
    try {
      WebViewBridge;
    } catch (e) {
      console.log('Dis don look like ReactNative...');
      alert('This browser is not supported. Please use Android Chrome or iOS native app.');
    }
  }, 500);
}
```

The first try statement will bail out before it establishes the socket connection if it can't find WebRTC. The second try statement will bail out if it can't find the WebViewBridge after half a second, since it takes some time for the bridge to get injected. This means we'll bail instantly on a native client, but not throw the alert unless we're in an iOS browser.

Luckily I was able to copy over the transport code with just a few small tweaks, and in the future, thanks to Babel transpilation, I intend to reuse the code entirely.

## Webpack and ES6
As my friends started making games on the platoform, the vanilla HTML5 project structure started causing pain. Which JS files should be imported? What is the public interface for each? Can we have optional typing? What about classes? Etc. 

I started out trying to use [SystemJS](https://github.com/systemjs/systemjs) but it was a nightmare. I kept getting strange errors, and in general there was just too much magic. How did the files get generated? Where were they coming from? How do I modify the behavior? Eventually I threw it out and switched to [Webpack](https://webpack.github.io/).

I had heard bad things about Webpack, and nearly all the examples I could find online were too simple or too complicated. Starting simple, however, I appreciated the relative lack of magic in the explicit config and presence of a bunch of features I needed, like transpilation, source-maps, rebuild-on-save, and module loading.

## Hosting


## iOS support
https://itunes.apple.com/us/app/webtendo/id1180349310

### dev/prod config

# Games

## Spacewar

## Scrabble

## Poker

# Contribute
https://github.com/8enmann/webtendo

