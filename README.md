YAMS
====

*Yet Another MPD Scrobbler (For Last.FM)*

## Fork
This fork adds functionality to scrobble music from internet radio streams. Can't believe I actually had to fork a project and add this functionality myself, instead of it just being implemented in the 3-4 or so Last.fm scrobblers that exist for MPD.

There's a *very* slight caveat with the implementation and that is that scrobbles will only be sent if you've fully finished listening to the song that is playing on the internet radio. So scrobbles are sent on track changes. I don't think you can really solve this problem in any other way honestly.

The version has not been changed, so it's still 0.7.3 as it was when I forked YAMS.

If Berulacks wants to use the code I wrote for this fork then go right ahead. I have not created a pull request for the original repository because I'm unsure of how good of an implementation this really is, but you know, it works at least. I made sure to separate the code and README changes into separate commits, so it should be relatively painless to merge just my commit. By the way, I did not use Black to format the code, so you may want to do that before merging any of my code.

I've modified the Installation section below, but other than that, the README is pretty much unchanged, aside from this big wall of text that is the Fork section.

[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

YAMS is exactly what its name says it is.

## Features
YAMS is just your run of the mill Last.FM scrobbler. But, if you *really* need to know, it can do the following:

* Authenticate with the new Last.FM [Scrobbling API v2.0](https://www.last.fm/api/scrobbling) - without the need to input/store your username/password locally.
* Update your profile's "Now Playing" track via Last.FM's "Now Playing" [API](https://www.last.fm/api/show/track.updateNowPlaying)
* Save failed scrobbles to a disk and upload them at a later date.
* Timing configuration (e.g. scrobble percentage, real world timing values for scrobbling, etc.).
* Prevent accidental duplicate scrobbles on rewind/playback restart/etc.
* Automatic daemonization and config file generation.

## Requirements
`PyYAML`, `psutil` and `python-mpd2` are required. YAMS is written for `python3` *only*.

## Installation
### From Source
Clone this repo and run `pip3 install --user -e <path_to_repo>` (omit the `-e` flag if you don't want changes in the repo to be reflected in your local installation; likewise one can omit the `--user` flag for a system-wide installation, though it's really not recommended).

Unsure what the -e flag actually does, but y'know it works so whatever.

```
$ git clone https://github.com/aukuste/yams
$ cd yams
$ pip3 install --user -e ./
```

## Running

The script includes a `yams` script that will be installed by pip.

`yams` runs as a daemon by default (`yams -N` will run it in the foreground).

`yams -k` will kill the current running instance.

`yams -a` will attach to the current running instance's log file, allowing you to watch the daemon's output.

`yams -h` will print all the options (also available below).

 *NB: (If you can't access the `yams` script, maybe because pip's script install directory isn't in your `$PATH` or something, `python3 -m yams` will also do the trick.)*

### Via Systemd

A Systemd user service unit file is included in the root of this repository (named `yams.service`). This can be used to automate starting/stopping YAMS on startup, for a specific user. (Note for users who installed from the AUR: The service file is automatically installed with the PKGBUILD, you just need to start it with `systemctl`)

To install, copy it to `~/.config/systemd/user/` and run `systemctl --user enable --now yams` to enable/start it. Note that you should also [start mpd as a Systemd service](https://wiki.archlinux.org/index.php/Music_Player_Daemon#Autostart_with_systemd) to ensure YAMS actually loads up at the right time. You might also need to edit the path to the python binary in the unit file if your system python version is installed anywhere other than `/usr/bin/python3`.

### Via Launchd (OS X)

OS X users can use the `yams.plist` file at the root of this repository to automate starting/stopping YAMS via OS X's built-in `launchd`. Note that this file must be installed manually.

To load the program, copy `yams.plist` to: `$HOME/Library/LaunchAgents/`

To start YAMS, either re-login, or run `launchctl load ~/Library/LaunchAgents/yams.plist`

Once loaded, check that everything is running fine with `yams -a`

*Note: The `plist` expects a Homebrew installation of Python to be available at `/usr/local/bin/python3` to work. For systems not using Homebrew's python, edit `yams.plist` and change the first  string in the `ProgramArguments` array to point to your Python binary (or just call YAMS directly). I've left some comments in there to help you out.*

## Setup

YAMS will use the usual `$MPD_HOST` and `$MPD_PORT` environment variables to connect to `mpd`, if they exist.

Run `yams` and follow the printed instructions to authenticate with Last.FM

### Configuration Files

If it can't find a config file by default, YAMS will create a default config file, log, cache, and session file in `$HOME/.config/yams`, however it will also accept config files in `$HOME/.yams` or `./.yams` (theoretically configs in `$HOME` or the current working directory can be read in, as well).

YAMS will only create its own directory/configuration file if none of the previous directories exist.

### Help

Here's the output for `--help`:

    usage: YAMS [-h] [-m 127.0.0.1] [-p 6600] [-s ./.lastfm_session]
                [--api-key API_KEY] [--api-secret API_SECRET] [-t 50] [-r] [-d]
                [-g] [-l /path/to/log] [-c /path/to/cache] [-C ~/my_config] [-N]
                [-D] [-k] [--disable-log] [-a]

    Yet Another Mpd Scrobbler, v0.5. Configuration directories are either
    ~/.config/yams, ~/.yams, or your current working directory. Create one of
    these paths if need be.

    optional arguments:
      -h, --help            show this help message and exit
      -m 127.0.0.1, --mpd-host 127.0.0.1
                            Your MPD instance's host
      -p 6600, --mpd-port 6600
                            Your MPD instance's port
      -s ./.lastfm_session, --session-file-path ./.lastfm_session
                            Where to read in/save your session file to. Defaults
                            to inside your config directory.
      --api-key API_KEY     Your last.fm api key
      --api-secret API_SECRET
                            Your last.fm api secret
      -t 50, --scrobble-threshold 50
                            The minimum point at which to scrobble, defaults to 50
                            percent
      -r, --real-time       Use real times when calculating scrobble times? (e.g.
                            how long you've been running the app, not the track
                            time reported by mpd). Default: True
      -d, --allow-duplicate-scrobbles
                            Allow the program to scrobble the same track multiple
                            times in a row? Default: False
      -g, --generate-config
                            Save the entirety of the running configuration to the
                            config file, including command line arguments. Use
                            this if you always run yams a certain fashion and want
                            that to be the default. Default: False
      -l /path/to/log, --log-file /path/to/log
                            Full path to a log file. If not set, a log file called
                            "yams.log" will be placed in the current config
                            directory.
      -c /path/to/cache, --cache-file /path/to/cache
                            Full path to the scrobbles cache file. This stores
                            failed scrobbles for upload at a later date. If not
                            set, a log file called "scrobbles.cache" will be
                            placed in the current config directory.
      -C ~/my_config, --config ~/my_config
                            Your config to read
      -N, --no-daemon       If set to true, program will not be run as a daemon
                            (e.g. it will run in the foreground) Default: False
      -D, --debug           Run in Debug mode. Default: False
      -k, --kill-daemon     Will kill the daemon if running - will fail otherwise.
                            Default: False
      --disable-log         Disable the log? Default: False
      -a, --attach          Runs "tail -F" on a running instance of yams' log
                            file. "Attaches" to it, for all intents and purposes.
                            NB: You will still need to kill it by hand. Default:
                            False

## Contributing
- Pull requests are always welcome.
- YAMS uses [Black](https://github.com/psf/black) for formatting its code.
- Not much else to say, really - code's riddled with comments, should be (relatively) legible!

## Other Information
- YAMS will try to re-send failed scrobbles every minute during playback, or on every subsequent scrobble. YAMS does not try to re-send failed "Now Playing" requests
- YAMS will wait on MPD's idle() command *only* when not playing a track. The `update_interval` configruation option controls the rate, in seconds, at which YAMS polls MPD for the currently playing track.
- YAMS will not crash when an MPD connection is lost but will attempt to re-connect every 10 seconds. Kill the daemon if this behaviour is undesirable, though the reconnect behaviour shouldn't significantly affect system resources.
- YAMS suppresses most error messages by default, run with `--debug` to see them all.
- `-g` is pretty useful, you should probably use it once to not have to keep typing in command line parameters.
- Windows support is not guaranteed. YAMS works fine under Elementary OS Juno and OS X Mojave (presumably all variants of Linux and OSX with python3 should work fine).
- YAMS is developed with Python `3.7`, if you're encountering a bug with a lower version, report it.
- YAMS works fine with Libre.FM. Just make sure to do the following:
   * Set the `base_url` config variable to "https://libre.fm/2.0/" (don't forget the trailing slash!)
   * Delete any leftover ".lastfm_session" files
   * Authenticate like you normally would with Last.FM, however replace "last.fm" with "libre.fm" in the authorization URL printed out by YAMS
