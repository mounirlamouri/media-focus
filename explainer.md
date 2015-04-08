# Media Focus

## Problem scope

The Web Platform doesn't provide a good user experience with regards to media playing. Media played on the Web are
usually left as second class citizen on the embedding platform. On desktop, some basic functionalities like enabling
media keys are ignored. On mobile, these media wouldn't show up in the lockscreen, making any mobile web media content
consuming particularly tedious.

## Use cases

The list of use cases this proposal is trying to address is:
* As a user, I want to be able to switch to _Bar Player_ and start playing music without having to stop _Foo Player_ if
this is the expectation on the platform I am using (for example, Android or iOS).
* As a user, I want to be able to press the _Pause_ key on my keyboard and stop music playing from _Foo Player_.
* As a user, I want to be able to change tracks using _Next_ and _Previous_ keys.
* As a user, I want _Foo Player_ to duck or stop playing music when something requiring my attention happens on the
system.
* As a user, I want to be able to control _Foo Player_ using my watch, headphones, keyboards indistinctively.
* As a user, I want to play a game with sound effects while listening to content from _Foo Player_.
* As a user, I want the content I am listening to showing up in my device's lockscreen with information about the
content such as name of artist/author, cover, etc.
* As a user, I want my "play" media key to start music playback in the active tab if no other app or tab is playing music.

## Proposal

### Media focus definition

* Following platform conventions, one or more media can be focused at the same time.
* When a media lose its focus state, it must be stopped.
* Following platform conventions, a focused media might be paused and resumed at any time by the UA.
* When a media is focused, all media key events should be sent to the HTMLMediaElement even if its document isn't
focused. In the case of AudioContext, the event should be sent to its document.

### Backward compatibility

* When played, if they match some heuristics, ```<audio>``` and ```<video>``` become media focused. For example,
heuristic might include the media length to prevent short audio files (intended as sound effects or notifications) to take media focus out of another media.

### Channels

A media channel defines the priority of the media and some rules around it like how many media of the same types can be
played or when it should be paused or ducked. This proposal doesn't yet list a channel list but considers Mozilla's
AudioChannels to be a good start: https://developer.mozilla.org/en-US/docs/Web/API/AudioChannels_API

In addition to the list that Mozilla has, a ```default``` state needs to be created which would allow the UA to treat the source as `normal` or `content` based on heuristics. The ```normal``` state might need to be renamed. The ```content``` state is the
one that would get media focus.

### Media metadata

In order to have a good experience on the lockscreen, this proposal would like to introduce a way for web pages to get
and set metadata on their media streams. Reading metadata will allow them to know the metadata embedded in the file
and show this information on the page if needed. Setting metadata will allow web pages to set additional information
about the media. It will also allow web pages to set metadata on ```AudioContext```.

Their is no fleshed out API but it would likely look like:  
```js
interface MediaMetadata {
  attribute DOMString title;
  attribute DOMString artist;
  attribute DOMString album;
  /// [...]
};

partial interface HTMLMediaElement {
  Promise setMetadata(MediaMetadata metadata);
  Promise<MediaMetadata> getMetadata();
};
```

## Compatibility with other audio sources

This proposal intentionally doesn't provide specific support for legacy plugins such as Flash. However, it would be possible for a silent `AudioContext` to act as a proxy.

```js
// Silent audio context to receive focus
var context = new AudioContext();
context.channel = 'content';

document.addEventListener('keyup', function(event) {
  if (event.key == 'MediaPlayPause') {
    flashObj.playPause();
  }
});
```

The Flash movie would call `suspend`/`resume` on the `AudioContext` to reflect its own playback state, or set the channel to `normal` to release focus.
