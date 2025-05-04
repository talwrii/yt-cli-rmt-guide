# Youtube command-line remote control
**@readwithai** - [X](https://x.com/readwithai) - [blog](https://readwithai.substack.com/) - [machine-aided reading](https://www.reddit.com/r/machineAidedReading/) - [📖](https://readwithai.substack.com/p/what-is-reading-broadly-defined)[⚡️](https://readwithai.substack.com/s/technical-miscellany)[🖋️](https://readwithai.substack.com/p/note-taking-with-obsidian-much-of)

This pages describes how to remotely control youtube running within a browser from the command-line.

In practice this sort of script may not be used directly from the command-line but rather attached to a [keyboard shortcut](https://en.wikipedia.org/wiki/Keyboard_shortcut) or run from another script that forms a larger process.

This approach only works for Linux due to the limitations of the Chrome extension it uses. If you have programming experience you may be able to adapt the approach for another operating system. I would direct anyone interested in doing this to [https://github.com/egovelox/mozeidon/](mozeidon) as an example of a remote control from *Chrome* which can be run from the command-line.

This approach only works for Chromium-based browsers. I got it working with Brave, but Vivalid is another Chrome-based browser.

# How this works
This entire page is effectively alternative and more complete documentation for [streamkeys](https://github.com/egovelox/mozeidon/) which, almost incidentally provides this feature. This page, however, focuses on command-line usage for Linux - documents and works around some issues in streamkeys, and is an easy-to-find article on the topic of controlling youtube from the command-line.

Streamkeys was written with the intention of controlling youtube, or other media players, running with Chrome from any other tabs. However, incidentally it also provides support for MPRIS which is a generally linux protocol for controlling multimedia players from the command-line. This standard can be used to cotrol youtube playback from the command-line.

You can do things like:
* Stop the player
* Skip forward
* Get the current position in the audio. Suitable for adding to notes or shared with others.
* Get the length of the video

Obtaining the may be particularly useful when combined [yt-dlp][yt-dlp] to obtain subtitles for the video.

# Setting everything up
Unfortunately stream has been removed from the Chrome extension store, has a build process that depends on deprecated libraries, and requires an out of date build process. It does however still work.

You need to download streamkeys, build it, and manually add it to Chrome. This may be good experience for you if you ever intend to develop a browser extension.

Requirements:
1. [uv](https://github.com/astral-sh/uv) is needed to use an old version of python required by `node-gyp` which is required by `node-sass`
1. [nvm](https://github.com/nvm-sh/nvm) is required to use an old version of `node` used for building the extension
1. `python3-dbus` is needed for the interface used to communicate with the extension from the command-line
1. `playerctl` is a command-line tool to communicate with mediaplayers (includig streamkeys after some configuration).

Both of these tools are standard, broadly used develoment tools

To install requiments:
1. `sudo apt-get install pipx`
1. `sudo apt-get install python3-dbus`
1. `pipx install uv`
1. [Follow the instructions here](https://github.com/nvm-sh/nvm) to install `nvm`. Note that `nvm` updaes you shell configuration and requires you to reload this configuraiton aor restart your shell


To make a build of streamkeys:

1. cd `~/.local`. I like to keep third-party installed code here - but you may store your code elsewhere.
1. Clone streamkeys. `git clone git@github.com:berrberr/streamkeys.git`
1. `cd streamkeys`
1. Create a  uv virtualenv with an old python `uv venv --python=3.10`
1. Activate the uv virtual virtualenv `source .venv/bin/activate`
1. Install node 16 `nvm install 16`
1. Enable node 16 `nvm use 16`
1. Add this python `python --version` should return 3.10 and `node --version` should 16.
1. Set up the build environment with `npm install`
1. Create the extension `npm run dev`

To install streamkeys:

1. Locate the extension `./build/unpacked-dev`
1. Go to the extension page in chrome "chrome://extensions"
1. Add a developer extension with the path of `build/unpacked-dev`


To enable streamkeys for use with debug (MPIRS), you must run a script.
1. Find the id of the streamkeys extension. Go to `brave://extensions/` and click on the Details of streamkeys package. This should have an ID - copy this ID.
2. Run the script `python3 ./build/unpacked-dev/native/mpris_host_setup.py install $ID` with this ID.


The command `playerctl pause` should now pause youtube and `playerctl play` start it.

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

## Alternatives
[ytmdesktop](https://github.com/ytmdesktop/ytmdesktop/releases) is a desktop app for youtube which can be [remotely controlled via http](https://github.com/ytmdesktop/ytmdesktop/wiki/Remote-Control-API). There tends to be a risk that such players attached to a changing website like google will stop working unless they have dedicated maintainers.

You may prefer to download media from youtube using [yt-dlp](https://github.com/yt-dlp/yt-dlp) to download media and play it in another player.

A shortcut to jump to the youtube tab may be sufficient for your uses. I have down this before with a combination of key simulation with `xdotool` interacting with the tab selection feature and `wmctrl` to raise windows. You can aso used mozeidon to select tabs

It is to be noted that the youtube website provides a variety of social and recommednation featues that other approaches do not support.

## References
[yt-dlp](https://github.com/yt-dlp/yt-dlp)

## About me
I am **@readwithai**. I create tools for reading, research and agency sometimes using the markdown editor [Obsidian](https://readwithai.substack.com/p/what-exactly-is-obsidian).

I also create a [stream of tools](https://readwithai.substack.com/p/my-productivity-tools) that are related to carrying out my work.

I write about lots of things - including tools like this - on [X](https://x.com/readwithai).
My [blog](https://readwithai.substack.com/) is more about reading and research and agency.
