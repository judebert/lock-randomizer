# MacOS screensaver randomizer
I love XScreenSaver. It has *literally* hundreds of screensavers.

But they don't all work on my system. And I can't keep track of which ones I've tried, let alone whether they worked or not, or how much I liked them.

This Python script keeps track of that for me. I pop it into my path (I created `~/bin` specifically for this), then call it from the terminal whenever I want a change. 

## Detailed Usage
### Activate the screensaver
You have multiple options here:
* `lock`
* `lock a`
* `lock activate`
I just type "lock". The script works with the native MacOS screensaver system, so you can also activate in any natively supported method: hot corners, setting a timeout, whatever.
### Choose a new screensaver
Again, multiple options.
* `lock change`
* `lock next`
These both do the same thing. They load the screensavers from the known screen saver paths (`/Library/Screen Savers`, `/System/Library/Screen Savers`, and `~/Library/Screen Savers`, rate any new ones as 3 of 5 stars, choose one based on the weighting, and set the system preferences to the new choice.

You can read more about the rating below. Ratings of 0 never get activated: if you get dizzy from the atunnel screensaver, for instance, you can rate it as 0 and never ever worry about it again.
### Show the current screensaver info
* `lock show`
* `lock info`
Besides the current screensaver's path (which usually includes its name), this also shows the weighting of the current screensaver and the most recent screensavers.
### Rate the current screensaver
* `lock like`
* `lock dislike`
These bump the weight of the current screensaver by 1. They're weighted on a 5-star scale, but you can bump the ones you love as high as 10. 

This also takes a `-d` argument, which lets you bump by more than 1. For instance, `lock dislike -d 3` will bump the current screensaver's rating down by 3, and `lock like -d 3` will bump it up by 3. Limits from 0 to 10 will be enforced.

If you don't like relative weighting, you can also set the weight absolutely:
* `lock weight`
If you don't set the `-d` argument, this will set the current screensaver rating to 1. If you'd like to make it middle-of-the-road, you could use `lock weight -d 3`.

There are also shortcuts for the extremes:
* `lock love` sets the current screensaver weight to 5
* `lock hate` sets the current screensaver weight to 0, so it will never be selected

## Dependencies
This is a Python 3 script. It uses some common libraries like argparse, json, random, and urllib. It also uses subprocess, because it needs MacOS commands to process plists and trigger daemons.

It uses the Mac commands `osascript`, `defaults`, `plutil`, `killall`, and `/usr/libexec/PlistBuddy`. 

## Sonoma
Before Sonoma, this was easy: just set the `com.apple.screensaver` with `defaults`, and boom! You're done!

And then the Fire Nation attacked.

Sonoma added a new capability, to run different screensavers on each screen. This is **not supported** by lock. Mostly because it is a *pain*.

Instead of a single screensaver preference, now Apple stores screensaver preferences per-screen. And that requires big changes to its plists. Then some bright brain thought, "Hey, we also have per-screen desktop backgrounds. Let's combine them in a single service!" So they did, and they moved they moved the preferences into a new location, and they didn't even delete the old defaults location - it just doesn't work.

Since it all moved into WallpaperAgent, the old ways of triggering the screensaver don't work either!

I discovered that [I wasn't the only person with this problem from Mac Power Users](https://talk.macpowerusers.com/t/can-we-no-longer-script-screen-savers-in-sonoma/35094). The Apple Developer Forums recommended [using AppleScript to trigger the screensaver](https://developer.apple.com/forums/thread/739314). The [iScreensaver Forum pointed me to the new plist location](https://forum.iscreensaver.com/t/understanding-the-macos-sonoma-screensaver-plist/718). It also explained that the screen saver functionality was in WallpaperAgent now. I wish I had read the rest of the article, instead of figuring out the binary plist stuff on my own.

Luckily, it turns out you can set the value as an XML plist. That way I don't have to go save prefs and things!

### Wallpaper Flicker
When you change the screen saver, the wallpaper on your desktop will flash once, turning dark for a moment and then returning to your settings. That's because the system doesn't read the new settings unless the WallpaperAgent gets reset. Since it now manages BOTH the screensaver and the wallpaper, the system "forgets" your wallpaper while the agent dies, then "remembers" it when the agent starts up near-instantly.

Technically, there is some way to do this without flickering. The system prefs don't flicker the wallpaper when you change the screensaver. I just don't seem to have access to it from Python or the terminal.
