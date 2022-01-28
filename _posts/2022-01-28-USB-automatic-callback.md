---
title: "USB Automatic Callback: building a USB honeypot"
author: Murphy
date: 2022-01-28 00:00:00 +0100
categories: [Misc]
tags: [usb, honeypot]
---

Recently I had an idea for a research project that involves dropping USB sticks around my university campus and documenting how many were picked up and plugged into a device. By doing so, I would be able to determine how likely a malicious USB would be to succeed in compromising a machine when dropped in a targeted location.

## Canary Tokens

From previous research in this field, I discovered a site called [CanaryTokens](https://canarytokens.org/generate) which can be used to generate documents that, when opened, call back to a webhook (or send an email) to alert you that the document has been opened by someone. There are many different options for these canary tokens, such as:
- Word documents
- Excel documents
- Windows folder
- Adobe reader PDF
- and more...

The one that interested me the most was the Windows folder option. How were they able to make someone reach out to a website just by entering a folder? I looked into their documentation and it explains that the folder contains a *desktop.ini* file that sets the icon resource of the file to an external link (the callback URL). This intrigued me as I never realised it was possible to set the icon of a windows file or folder to a remote location.

## Autorun.inf

USB drives or any runnable media can contain a file called *autorun.inf* which will tell Windows what to execute when the device is inserted into the machine. Unfortunately (or fortunately, in the sense of security), this feature is disabled by default on Windows 10 and USB drives can no longer execute commands when plugged in (probably for the best...).

However, there are a few functions in autorun that still work; such as setting the icon and label of the USB device. I tried performing a similar attack by setting the icon location to a remote URL using the following code:
```
[Autorun]
icon=\\webhook.site\3bcd8153-894a-453b-b17b-9ad6784856c4
```
When the USB is plugged in, the icon should be set to the remote location and should trigger my webhook.

Unfortunately, this was not the case. This is because Windows made sure that the icon file must be contained within the USB and cannot access locations outwith the local scope of the USB.

## Shortcuts

Since I was unable to change the icon of the USB itself, I decided to put a file with a custom icon inside the root folder of the USB so that it would be loaded when the USB was inserted. In Windows, you can set a custom URL for shortcuts; so I create a shortcut to `calc.exe` using the path `C:\Windows\System32\calc.exe` which should be present on every Windows 10 machine (not that it really matters, because the shortcut doesn't need to be run anyway). In the properties of the shortcut I was able to change the icon and set it to my webhook URL `\\webhook.site\3bcd8153-894a-453b-b17b-9ad6784856c4`. The window froze for a few seconds and I got a successful request on my webhook, but the window returned an error stating that the file could not be found, so I could not apply my changes.

To get around this, I decided to bypass Windows' user interface and create a shortcut manually using Powershell. I found a script online and modified it to the following:
```ps
$ShortcutPath = "test.lnk"
$IconLocation = "\\webhook.site\3bcd8153-894a-453b-b17b-9ad6784856c4"
$Shell = New-Object -ComObject ("WScript.Shell")
$Shortcut = $Shell.CreateShortcut($ShortcutPath)
$Shortcut.TargetPath = "C:\Windows\System32\calc.exe"
$Shortcut.IconLocation = "$IconLocation"
$Shortcut.Save()
```

This generated a shortcut called `test` which had the default file icon. I went to right click and check the properties of the shortcut, but the properties window didn't show up. It took about 10 seconds before it finally showed up, and I got a hit on my webhook:

![Request]({{ "/assets/img/3/request.png" | relative_url }})

I removed the USB device and plugged it back in again, and after about 5 or 10 seconds I got the request again without having to touch anything. As the USB device is plugged in, it tries to resolve the shortcut's icon as it's in the root directory, which reaches out to my webhook.

## Summary

Even though Windows have locked down the icon permissions in autorun, you can still create shortcuts with custom icons that reach out to a remote location.

## Inspirations

- [CanaryTokens](https://canarytokens.org/generate)
- [Users Really Do Plug in USB Drives They Find](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45597.pdf)