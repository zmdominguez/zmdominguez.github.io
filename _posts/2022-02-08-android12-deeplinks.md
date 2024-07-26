---
layout: post
title: "Debugging App Links in Android 12 🔗"
tags:
    - android
    - deeplinks
---
I have been working with deeplinks lately and I noticed that quite a few things have changed since I last worked with them. The most important change is quoted in the list of [Android 12 behaviour changes](https://developer.android.com/about/versions/12/behavior-changes-all#web-intent-resolution):

> Starting in Android 12 (API level 31), a generic web intent resolves to an activity in your app **only if your app is approved for the specific domain** contained in that web intent. If your app isn't approved for the domain, the web intent resolves to the user's default browser app instead.
> 
> <footer>Emphasis mine</footer>

There's enough documentation on the Android developer site on how to go about handling this approval. But to recap:
- [Add intent filters in the AndroidManifest file](https://developer.android.com/training/app-links/deep-linking#adding-filters)
- [Make sure `autoVerify` is set to `true`](https://developer.android.com/training/app-links/verify-site-associations#add-intent-filters)
- [Associate your website with your app](https://developer.android.com/training/app-links/verify-site-associations#web-assoc)

If all goes well, clicking on a link should open the corresponding screen in the app:
<figure class="align-center">
    <a href="https://imgur.com/4Kn6N5T"><img src="https://imgur.com/4Kn6N5T.gif" width="320"/></a><br />
    <figcaption>Deep linking into the product details screen</figcaption>
</figure>

If things do *not* go well, Google has provided ways to [test deeplinks](https://developer.android.com/training/app-links/verify-site-associations#testing). There are lots of ways to figure out where things went wrong, but they are scattered in different sections. For my sanity, I have collated the steps I have found so that they are all in one place.

### Website Linking
If your website is not verified to work with the app, auto-verification will fail. Head on over to the [Statement List Generator and Tester](https://developers.google.com/digital-asset-links/tools/generator), put in the required details, and click on "Test statement".

<figure class="align-center">
    <a href="https://imgur.com/T9J8qI8"><img src="https://i.imgur.com/T9J8qI8.png" title="source: imgur.com" width="450" /></a><br />
    <figcaption>Successful linking!</figcaption>
</figure>


You can also use the Digital Assets API to confirm that the `assetlinks.json` file is properly hosted:
```
https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=[YOUR_WEBSITE_URL]&relation=delegate_permission/common.handle_all_urls
```

Remember that verification should pass for **all** hosts declared in the `AndroidManifest` file on Android 11 and below, so make sure to test each of them.

If any of these tests fail, review the [Digital Asset Links documentation](https://developers.google.com/digital-asset-links/v1/create-statement) and make sure that the file is formatted properly.

We found out the hard way that the value for your certificate's `sha256_cert_fingerprints` in `assetlinks.json` **SHOULD** be in ALL CAPS
{: .notice--info}
(Thanks to [Ben Trengrove](https://twitter.com/bentrengrove) for debugging that issue with me!)

On the device-side of things, we can also check the status of domain verification:
```shell
adb shell pm get-app-links [YOUR_PACKAGE_NAME]
```

This will show results similar to this:
```shell
  com.woolworths:
    ID: fb789c89-1d2e-403a-be0c-a8871a8e5b76
    Signatures: [41:0F:9A:43:72:FC:C0:76:BD:90:AC:C4:A0:6F:96:D5:24:CC:1E:69:2E:79:18:1F:05:0C:78:21:8C:39:27:D5]
    Domain verification state:
      woolworths.app.link: verified
      woolworths-alternate.app.link: verified
      www.woolworths.com.au: verified
```

There are various states for domain verification. Check out the [documentation](
https://developer.android.com/training/app-links/verify-site-associations#review-results) for what each of those may mean.


### User Permissions
If everything on the website side of things is setup properly, check that the user has allowed opening your app's supported links.

The easiest way to do this is to use the ADB command to check the domain verification status and add [flags to show the user's side of things](https://developer.android.com/training/app-links/verify-site-associations#user-prompt-command-line-program):
```shell
adb shell pm get-app-links --user cur [YOUR_PACKAGE_NAME]
```

Running this command will spit out the verification status and if the user has given your app permission to open declared URLs:
```shell
  com.woolworths:
    ID: fb789c89-1d2e-403a-be0c-a8871a8e5b76
    Signatures: [41:0F:9A:43:72:FC:C0:76:BD:90:AC:C4:A0:6F:96:D5:24:CC:1E:69:2E:79:18:1F:05:0C:78:21:8C:39:27:D5]
    Domain verification state:
      woolworths.app.link: verified
      woolworths-alternate.app.link: verified
      www.woolworths.com.au: verified
    User 0:
      Verification link handling allowed: true
      Selection state:
        Disabled:
          woolworths.app.link
          woolworths-alternate.app.link
          www.woolworths.com.au
```


To see the status of *ALL* apps on the device, run the following ADB command to [check all link policies](https://developer.android.com/training/app-links/verify-site-associations#check-link-policies):
```shell
adb shell dumpsys package d
// OR
adb shell dumpsys package domain-preferred-apps
```
I find the information this shows to be very interesting! Maybe that's just me though, I'm weird like that. :nerd_face:

Note that even if auto-verification fails, the user can manually allow your app to open links. Take this output for the debug variant of our app for example:
```shell
com.woolworths.debug:
  ID: 99e87cda-e951-4e7a-ba6a-894a31718add
  Signatures: [AF:35:FE:62:F8:11:02:16:8D:B4:7F:15:91:A3:9B:43:0E:9C:B0:93:F7:57:AC:99:B2:FC:19:2E:C1:A8:E3:96]
  Domain verification state:
    woolworths-alternate.test-app.link: legacy_failure
    www.woolworths.com.au: verified
    woolworths.test-app.link: legacy_failure
  User 0:
    Verification link handling allowed: true
    Selection state:
      Enabled:
        woolworths-alternate.test-app.link
        woolworths.test-app.link
      Disabled:
        www.woolworths.com.au
```


Despite two hosts failing the verification process:
```shell
woolworths-alternate.test-app.link: legacy_failure
woolworths.test-app.link: legacy_failure
```

I can go into the app's settings and manually approve these URLs:
<figure class="align-center">
    <a href="https://i.imgur.com/OYEKHYO"><img src="https://i.imgur.com/OYEKHYO.png" title="Screenshot showing links" width="320"/></a><br />
    <figcaption>Manual permission for supported links</figcaption>
</figure>


### Resetting Verification
There are also ADB commands to facilitate going through the whole validation process.

First [reset the app links state](https://developer.android.com/training/app-links/verify-site-associations#reset-state) of the app:
```shell
adb shell pm set-app-links --package [YOUR_PACKAGE_NAME] 0 all
```

Then [manually trigger re-verification](https://developer.android.com/training/app-links/verify-site-associations#invoke-domain-verification):
```shell
adb shell pm verify-app-links --re-verify [YOUR_PACKAGE_NAME]
```

If you want to test out the auto-verification process but do not target Android 12 yet, it can be [enabled for your app](https://developer.android.com/training/app-links/verify-site-associations#support-updated-domain-verification):
```shell
adb shell am compat enable 175408749 [YOUR_PACKAGE_NAME]
```


### Testing Intents
Finally, to ensure that we have correctly configured the Intent filters in the `AndroidManifest.xml` file and our app can open intended links, [send an implicit Intent via ADB](https://developer.android.com/training/app-links/verify-site-associations#auto-verification):
```shell
adb shell am start -a android.intent.action.VIEW -c android.intent.category.BROWSABLE -d "[URL_HERE]"
```

Since I'm lazy and that's long command to remember, I added an alias for it:
```shell
alias deeplink='() { adb shell am start -a android.intent.action.VIEW -c android.intent.category.BROWSABLE -d "$1" ;}'
```

So I can do this:
```shell
➜  ~ deeplink https://www.woolworths.com.au/shop/productdetails/670560
Starting: Intent { act=android.intent.action.VIEW cat=[android.intent.category.BROWSABLE] dat=https://www.woolworths.com.au/... }
```


### Install-time Logs
Back in 2017, I [wrote about another way](https://zarah.dev/2017/01/20/testing-autoverify.html) to troubleshoot `autoVerify` . You would need to keep an eye on Logcat for the domain verification logs. For our debug variant, these logs look like this:
```shell
I/IntentFilterIntentOp: Verifying IntentFilter. verificationId:180 scheme:"https" hosts:"woolworths-alternate.test-app.link www.woolworths.com.au woolworths.test-app.link" package:"com.woolworths.debug". [CONTEXT service_id=244 ]
I/AppLinksUtilsV1: Legacy cross-profile verification enabled [CONTEXT service_id=244 ]
I/SingleHostAsyncVerifier: Verification result: checking for a statement with source # cfkq@55fed08a, relation delegate_permission/common.handle_all_urls, and target # cfkq@7ce31cea --> true. [CONTEXT service_id=244 ]
I/SingleHostAsyncVerifier: Verification result: checking for a statement with source # cfkq@5c3d4ef1, relation delegate_permission/common.handle_all_urls, and target # cfkq@7ce31cea --> false. [CONTEXT service_id=244 ]
I/SingleHostAsyncVerifier: Verification result: checking for a statement with source # cfkq@9705d4b3, relation delegate_permission/common.handle_all_urls, and target # cfkq@7ce31cea --> false. [CONTEXT service_id=244 ]
I/IntentFilterIntentOp: Verification 180 complete. Success:false. Failed hosts:woolworths-alternate.test-app.link,woolworths.test-app.link. [CONTEXT service_id=244 ]
```

It looks like the output formatting has changed since 2017 and the individual URLs are not cleartext anymore (for example, `cfkq@55fed08a`). There's really not much reason to look for these logs aside from checking that *some* form of auto-verification is happening. The ADB commands we've gone through in the previous sections show the same information in a much more readable format.

---

Unfortunately, it is difficult to ascertain the inner workings of domain verification. Hopefully the steps outlined here help narrow down possible causes for when your app links fail to cooperate. Good luck and happy (app) linking! :handshake:
