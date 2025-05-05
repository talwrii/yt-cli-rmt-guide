# Youtube command-line remote control
**@readwithai** - [X](https://x.com/readwithai) - [blog](https://readwithai.substack.com/) - [machine-aided reading](https://www.reddit.com/r/machineAidedReading/) - [📖](https://readwithai.substack.com/p/what-is-reading-broadly-defined)[⚡️](https://readwithai.substack.com/s/technical-miscellany)[🖋️](https://readwithai.substack.com/p/note-taking-with-obsidian-much-of)

This pages describes how to remotely control youtube running within a browser from the command-line. Previously this sort of tool required an extension like [streamkeys](https://github.com/talwrii/building-streamkeys). However, Chrome now has native support for a protocol called [MPRIS](https://specifications.freedesktop.org/mpris-spec/latest/) which allows any application to control playing media (via DBUS).

Using the MPRIS interface in chrome, you can:

You can do things like:
* Stop the player
* Skip forward
* Get the current position in the audio. Suitable for adding to notes or shared with others.
* Get the length of the video

Obtaining the position may be particularly useful when combined [yt-dlp](https://github.com/yt-dlp/yt-dlp) to obtain subtitles for the video.

## Usage
You can use `playerctl` from the command-line. You may like to explore the documentation. This script can be wrapped for ther purposes.

I use the following scripts (amongst others):

**skip**: Skip 10 seconds forward (or number of seconds given)
```
#!/bin/bash
playerctl position | perl -pe "\$_ = \$_ + ${1-10}" | xargs playerctl position; playerctl position
```

**back**: Skip 10 seconds backwards (or number of seconds given)
```
#!/bin/bash
skip ${1:--10}
```

**pos**: Return the position into the video
```
#!/bin/bash
playerctl position
```

**yt-position**: Return the percentage through the video
```
#!/bin/bash
length=$(playerctl metadata  | grep length | awk '{ print $3 }')
pos=$(playerctl position)
perl -e 'my ($a,$b) = @ARGV; print(int($a * 1000000 / $b * 100) . "%\n")' "$pos" "$length"
```

**yt-link**: Return a link to the video 10 seconds ago
```
#!/bin/bash
mozeidon tabs  get  | jq '.[] | .[] | select(.url | test("youtube\\.com")) | .url ' -rc | { read line ; echo $line\&t=$(playerctl position | perl -pe '$_ -= 10') ; }
```

I use this script (and others) to refer to videos while taking notes in [Obsidian](https://readwithai.substack.com/p/what-exactly-is-obsidian). If you are interested in note taking you might like to read my [review of note taking in Obsidian](https://readwithai.substack.com/p/note-taking-with-obsidian-much-of).

## Caveats
The interface presented by MPRIS only supports a single playing source of media. If you have more than one thing playing (such as background music and a video that you are reviewing), then the most recently used tab is controlled. To change the media that is controlled by MPRIS you can select the current tab.

[mozeidon](https://github.com/egovelox/mozeidon/) can remotely fetch information about the tabs (including the currently selected tab) and switch between tabs. This could be used to find the playing tab if the media is playing.

## Alternatives
You may prefer to download media from youtube using [yt-dlp](https://github.com/yt-dlp/yt-dlp) to download media and play it in another player.

A shortcut to jump to the youtube tab may be sufficient for your uses. I have down this before with a combination of key simulation with `xdotool` interacting with the tab selection feature and `wmctrl` to raise windows. You can aso used mozeidon to select tabs

It is to be noted that the youtube website provides a variety of social and recommendation featues that other approaches do not support.

## References
1. [yt-dlp](https://github.com/yt-dlp/yt-dlp)
2. [mozeidon](https://github.com/egovelox/mozeidon/)

## About me
I am **@readwithai**. I create tools for reading, research and agency sometimes using the markdown editor [Obsidian](https://readwithai.substack.com/p/what-exactly-is-obsidian).

I also create a [stream of tools](https://readwithai.substack.com/p/my-productivity-tools) that are related to carrying out my work.

I write about lots of things - including tools like this - on [X](https://x.com/readwithai).
My [blog](https://readwithai.substack.com/) is more about reading and research and agency.
