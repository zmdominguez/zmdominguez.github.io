---
layout: post
title: "Extending an Interactive ADB 🔀 "
tags:
    - android
    - adb
---

A few weeks ago, I [wrote about a script](https://zarah.dev/2023/08/30/adb-deeplinks.html) for making `adb` a little bit more interactive. The script makes the process of running an `adb` command much smoother if there are multiple devices attached by presenting a chooser. For example, when sending a deeplink:
```shell
➜  ~ deeplink https://zarah.dev
Multiple devices found:
1 - R5CR7039LBJ
2 - emulator-5554
3 - emulator-5556
Select device: 
```

The `adb` command to be sent is embedded in the script. It works fine if we only need the convenience to run one command, but let's face it, in reality I use a bunch of different commands all the time. It does not make sense though to have multiple copies of the script just to support multiple `adb` commands.

I mentioned in that post that it would be nice to be able to make the script generic enough to support multiple commands, and I've given it some thought since then.

Before we dive into possible solutions, I did notice an issue with the current version of the script. This line figures out how many devices are available:
```shell
# Find how many devices we have
num_matches=$(echo $all_devices | egrep -o "([[:alnum:]-]+[[:space:]]+device$)" | wc -l)
```

To recap, it counts how many lines have some text followed by the word "devices". It works most of the time, however I noticed that if I plug in a device that has the USB authorisations revoked, that device appears as "unauthorized".
```shell
➜  ~ adb devices
List of devices attached
R5CR7039LBJ   unauthorized
emulator-5556 device
```
For this post, that line has been updated to remove any lines with "unauthorized" devices:
```shell
# Drop any unauthorised devices (i.e. USB debugging disabled or authorisations revoked)
valid_devices=$(echo $all_devices | grep -v "([[:alnum:]-]+[[:space:]]+unauthorized$)" | grep -oE "([[:alnum:]-]+[[:space:]]+device$)")
```

Back to the problem at hand: all `adb` commands are [structured](https://developer.android.com/tools/adb#issuingcommands) in a predicatable manner:
```shell
adb -s <SERIAL_NUMBER> command
```
We can take advantage of this pattern to extend the scalability of our script.

### Option 1: Pass a command in as an argument 🗣️

I first explored the option of passing in a stub of the `adb` command as an argument to the script. If we take the deeplink command for example:

```shell
adb -s <SERIAL_NUMBER> shell am start -W -a android.intent.action.VIEW -d "SOME_URL"
```
it means passing in `shell am start -W -a android.intent.action.VIEW -d "SOME_URL"` into the script. With the command stub now a parameter, we'd have to change our `alias` from this:

```shell
alias deeplink='zsh /Users/zarah/scripts/deeplink.sh $1'
```

to this:
```shell
alias deeplink='zsh /Users/zarah/scripts/deeplink.sh "shell am start -W -a android.intent.action.VIEW -d \"$1\""'
```

With this option, the script remains mostly the same except for the part where the command is actually sent. Instead of hard-coding the command, we will use the stub passed in:
```shell
command adb -s $serial_number $COMMAND
```

This works, but it's not the best. There may be instances when we need to run multiple `adb` commands one after the other. For example, when setting the screen orientation to portrait:
```shell
function rotatePortrait() {
  adb shell settings put system accelerometer_rotation 0
  adb shell settings put system user_rotation 0
}
```

If we use this version of the script, it _will_ work, but it will also ask multiple times for the serial number. That's not good because it is easy to mess it up if different devices were entered for each command.

### Option 2: Just make it get the serial number 💱

In this option, we cut back the functionality of the script to make it do one thing: get the serial number. A big chunk of the script remains the same, the only change reallly is to make the `get_devices` function skip sending the command and return the serial number chosen instead:

```shell
# If there are multiple, ask for which device to grab
if [[ $num_matches -gt 1 ]]; then
  get_from_multiple
# Otherwise just grab the serial number
else
  serial_number=$(echo $valid_devices | awk '{printf $1}')
fi

echo "$serial_number"
```

This means that issuing the actual command is up to the caller, which may sound annoying and repetitive. Do not fret though, because we can hide all the annoyingness in functions that we can use in our aliases.

In the `.zshrc` file (or wherever your `alias`es live), we can reference our `get_devices` script:
```shell
source "$(dirname "$0")/get_devices.sh"
```

The syntax to grab the returned value (the serial number) is a bit difficult to remember, so wrapping it in a function is helpful:
```shell
# Grabs a serial number from all _available_ devices
# If there is only one device, grabs that serial number automatically
# If there are multiple devices, shows a chooser with the list of serial numbers
function getSerialNumber() {
  serial_number=$(get_devices)
}
```

To make it even easier, we can make a convenience function to call through to `getSerialNumber` and then launch the `adb` command (thanks to my teammate [Ani](https://www.linkedin.com/in/aniruddhfichadia/) for suggesting this!):
```shell
# Sends an interactive ADB command
# Usage: Use the usual ADB command, replacing `adb` with `adbi`
function adbi() {
    getSerialNumber && adb -s "$serial_number" "$@"
}
```

Applying this to our deeplink `alias` (which is now a function because [Shellcheck](https://www.shellcheck.net/) will not stop complaining about it):
```shell
# Deep links
function deeplink() {
  adbi shell am start -W -a android.intent.action.VIEW -d \""$1"\"
}
```

This solution is really adaptible and works well for the `rotatePortrait` function too:
```shell
function rotatePortrait() {
  getSerialNumber
  adb -s "$serial_number" shell settings put system accelerometer_rotation 0
  adb -s "$serial_number" shell settings put system user_rotation 0
}
```
Now it only asks us to choose the device once and uses that serial number for all the `adb` commands to be executed.

I like this solution a lot for a couple of reasons:
- it's super easy to update our current aliases, i.e. `s/adb/adbi`
- the syntax is VERY similar to the usual `adb` syntax, i.e. `s/adb/adbi`


I think it's super obvious that we have a clear winner here 🥇🏋️‍♀️ Option 2 it is! And to celebrate, as always, the gist is in [Github](https://gist.github.com/zmdominguez/9a889f1c367e1a21203ce8527c81e612).
