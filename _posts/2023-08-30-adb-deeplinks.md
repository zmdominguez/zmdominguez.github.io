---
layout: post
title: "Making ADB a little bit dynamic 📱"
tags:
    - android
    - deeplinks
    - adb
---

Android has a lot of tools for developers and one that has been around for as long as I can remember is [Android Debug Bridge](https://developer.android.com/tools/adb) (`adb`). It allows you to issue commands to an attached device, such as installing an app or starting an `Activity`.

If I want to test deeplinks, for example, I can issue an `adb` command that simulates the system sending an `Intent` directed to my app:
```shell
➜  ~ adb shell am start -W -a android.intent.action.VIEW -d "https://zarah.dev"
Starting: Intent { act=android.intent.action.VIEW dat=https://zarah.dev/... }
Status: ok
LaunchState: WARM
Activity: dev.zarah.sdksample/.DetailActivity
TotalTime: 165
WaitTime: 168
Complete
```

I usually test on a real device, but sometimes I have to spin up an emulator to test on a different screen size or OS version, and sometimes I also attach my personal phone to charge. I have lost count of how many times I have tried to run an `adb` command and forgot that I have multiple devices attached.

When the deeplink command is sent again in these circumstances:
```shell
➜  ~ adb shell am start -W -a android.intent.action.VIEW -d "https://zarah.dev"
adb: more than one device/emulator
```

One of the quirks of `adb` is that it tells us there is more than one device, but it doesn't tell us _what_ those devices are. To make the command work again, we need to include the serial number of the target device.

We query for all devices via `adb devices` and then add the `-s <SERIAL_NUMBER>` option when running the command:

```shell
➜  ~ adb devices
List of devices attached
emulator-5554	device
emulator-5556	device

➜  ~ adb -s emulator-5554 shell am start -W -a android.intent.action.VIEW -d "https://zarah.dev"
Starting: Intent { act=android.intent.action.VIEW dat=https://zarah.dev/... }
Status: ok
LaunchState: WARM
Activity: dev.zarah.sdksample/.DetailActivity
TotalTime: 289
WaitTime: 306
Complete
```

Wouldn't it be nice if `adb` just straight up notifies us of the problem (multiple devices found), asks us how we want to fix the problem (which device should be the target), and then try again?

After years and years of dealing with this, I finally gave in and wrote a script that just does that. 🙊

With a super handy `deeplink` alias, I can launch the script and provide it with a URI. If there's only one device, it issues the command directly:
```shell
➜  ~ deeplink https://zarah.dev
Starting: Intent { act=android.intent.action.VIEW dat=https://zarah.dev/... }
Status: ok
LaunchState: WARM
Activity: dev.zarah.sdksample/.DetailActivity
TotalTime: 165
WaitTime: 168
Complete
```

But when there are multiple devices, it shows the list of devices available and asks for which one to target:
```shell
➜  ~ deeplink https://zarah.dev
Multiple devices found:
1 - R5CR7039LBJ
2 - emulator-5554
3 - emulator-5556
Select device: 
```

There is no need to faff about copying serial numbers, as entering the option should be enough. I added an actual device to the mix, and if I want to send the `Intent` to that device I can type in `1` and press enter:
```shell
➜  ~ deeplink https://zarah.dev
Multiple devices found:
1 - R5CR7039LBJ
2 - emulator-5554
3 - emulator-5556
Select device: 1
Starting: Intent { act=android.intent.action.VIEW dat=https://zarah.dev/... }
Status: ok
LaunchState: WARM
Activity: dev.zarah.sdksample/.DetailActivity
TotalTime: 648
WaitTime: 667
Complete
```

I did talk about using the `deeplink` `alias` [before](https://zarah.dev/2022/02/08/android12-deeplinks.html), but I have since updated it to run the script instead:
```shell
alias deeplink='zsh /Users/zarah/scripts/deeplink.sh $1'
```

### The nuts and bolts of it 🔩

There is nothing truly special about how the script works, but it is doing a bunch of RegEx (which should tell you that it took me waaaaaay to long to figure out 😝).

First, we call `adb devices` to figure out how many devices are available:
```shell
all_devices=$(command adb devices)

# Drop the title ("List of devices attached")
all_devices=${all_devices#"List of devices attached"}
```

Figure out how many recognised devices there are:
```shell
num_matches=$(echo $all_devices | egrep -o "([[:alnum:]-]+[[:space:]]+device$)" | wc -l)
```

If there's only one device, send the command immediately; otherwise, we need to ask which device to send the command to:
```shell
# If there are multiple, ask for which device to send the command to
if [[ $num_matches -gt 1 ]]; then
  deeplink_with_multiple
# Otherwise just send the ADB command
else
  command adb shell am start -W -a android.intent.action.VIEW -d \"$URL\"
fi
```

In this case `$URL` is the variable that holds the input parameter (the URL passed into the script).

If there are multiple devices, we do more string manipulation to present the list:
```shell
# Display device serial numbers
find_matches=$(echo $all_devices | egrep -io "([[:alnum:]-]+[[:space:]]+device$)" | awk '{print NR " - " $1}')
printf "Multiple devices found:\n%s\n" "$find_matches"
```

Notice the syntax is very similar to the alias I use for displaying the [recently-checked out branches in git](https://zarah.dev/2021/08/10/magic-reflog.html). Thank you 2021 Zarah for figuring that out!

We then ask for the input:
```shell
# Present chooser
echo -n "Select device: "
read -r selected_device
```

Find the matching serial number chosen and issue the command:
```shell
# Send the ADB command with the serial number
serial_number=$(echo $find_matches | egrep "${selected_device} - (.*)" | awk '{print $3}')
command adb -s $serial_number shell am start -W -a android.intent.action.VIEW -d \"$URL\"
```

### Do this for all the things! 💨
The best thing about this script is it's super extensible. By changing the issued `adb` commands in the script, I can have this convenience apply to basically any `adb` commands I usually use.

It is especially handy for those things that require a bunch of `adb` commands, such as [forwarding](https://developer.android.com/tools/adb#forwardports) or reversing ports. A bunch of commands mean a bunch of places where `-s <SERIAL_NUMBER>` needs to be added and letting the script do it means we won't miss adding it to any of them:
```shell
adb -s $serial_number wait-for-device && adb -s $serial_number reverse tcp:9000 tcp:9000 && adb -s $serial_number reverse tcp:3000 tcp:3000
```

I am 💩 at shell scripting (as evidenced by how much time I spent writing this tiny script), but I imagine it may be possible to make this work without having to have one version of the script for each `adb` command. Maybe a lookup map with the command name as the key and the `adb` command for a single device and the `adb` command for multiple devices as the values? Is that even possible? Maybe? It'd be nice.

But for now, the script is [available on Github](https://gist.github.com/zmdominguez/1b74a2fa6bb027870362a3ca5202a8df).