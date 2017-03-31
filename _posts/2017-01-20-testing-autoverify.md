---
layout: default
title: Troubleshooting autoVerify
date: '2017-01-20T21:30:00.002+11:00'
author: Zarah Dominguez
tags:
- ux
- quick tips
- app links
- android
modified_time: '2017-01-20T21:41:04.397+11:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-2482718186145075003
blogger_orig_url: http://www.zdominguez.com/2017/01/testing-autoverify.html
---

So you implement [app links](https://developer.android.com/training/app-links/index.html) and you are 300% sure you have implemented everything correctly. The important thing to remember here is that verification is all or nothing. From the docs:

> For app link verification to succeed, the system must be able to verify your app with all of the websites that you specify in your app’s intent filters, and that meet the criteria for app links. 

Google has outlined [several steps](https://developers.google.com/digital-asset-links/tools/generator) on how to test your implementation and they have provided several samples as well. As with life, however, things can go awry no matter how hard you try.

If automatic link handling does not work, chances are one of the hosts declared in your Manifest failed verification. If you have a bunch (subdomains are considered unique and separate), it can be hard to figure out which one is failing. Thankfully, there is a way for us to see which is causing the failure.

Fire up your terminal, start `logcat`, install your APK (`adb install <path-to-apk>`) and keep an eye out for the verifier logs. Here is a sample output:

```shell
I SingleHostAsyncVerifier: Verification result: checking for a statement with source a <
I SingleHostAsyncVerifier:   a: "https://domain.com.au"
I SingleHostAsyncVerifier: >
I SingleHostAsyncVerifier: , relation delegate_permission/common.handle_all_urls, and target b <
I SingleHostAsyncVerifier:   a: "com.fairfax.domain"
I SingleHostAsyncVerifier:   b <
I SingleHostAsyncVerifier:     a: "AA:B4:3F:0F:A7:49:F8:90:F3:D6:64:30:FB:5E:69:54:7B:BA:EB:85:7D:9D:04:57:83:5F:FD:58:E7:B9:70:6A"
I SingleHostAsyncVerifier:   >
I SingleHostAsyncVerifier: >
I SingleHostAsyncVerifier:  --> true.
I SingleHostAsyncVerifier: Verification result: checking for a statement with source a <
I SingleHostAsyncVerifier:   a: "https://www.domain.com.au"
I SingleHostAsyncVerifier: >
I SingleHostAsyncVerifier: , relation delegate_permission/common.handle_all_urls, and target b <
I SingleHostAsyncVerifier:   a: "com.fairfax.domain"
I SingleHostAsyncVerifier:   b <
I SingleHostAsyncVerifier:     a: "AA:B4:3F:0F:A7:49:F8:90:F3:D6:64:30:FB:5E:69:54:7B:BA:EB:85:7D:9D:04:57:83:5F:FD:58:E7:B9:70:6A"
I SingleHostAsyncVerifier:   >
I SingleHostAsyncVerifier: >
I SingleHostAsyncVerifier:  --> true.
I SingleHostAsyncVerifier: Verification result: checking for a statement with source a <
I SingleHostAsyncVerifier:   a: "https://m.domain.com.au"
I SingleHostAsyncVerifier: >
I SingleHostAsyncVerifier: , relation delegate_permission/common.handle_all_urls, and target b <
I SingleHostAsyncVerifier:   a: "com.fairfax.domain"
I SingleHostAsyncVerifier:   b <
I SingleHostAsyncVerifier:     a: "AA:B4:3F:0F:A7:49:F8:90:F3:D6:64:30:FB:5E:69:54:7B:BA:EB:85:7D:9D:04:57:83:5F:FD:58:E7:B9:70:6A"
I SingleHostAsyncVerifier:   >
I SingleHostAsyncVerifier: >
I SingleHostAsyncVerifier:  --> false.
I IntentFilterIntentSvc: Verification 6 complete. Success:false. Failed hosts:m.domain.com.au.
```

In this case one host (`https://m.domain.com.au`) fails, which means automatic link handling will not work at all. If this happens, hit up your friendly web developers and go through the troubleshooting steps outlined in the developer guide.

Once successful, the last tine in the verification process should say `Success:true`:

```shell
  I IntentFilterIntentSvc: Verification 7 complete. Success:true. Failed hosts:.
```

H/T to [Wojtek Kaliciński](https://twitter.com/wkalic) for the tip!
