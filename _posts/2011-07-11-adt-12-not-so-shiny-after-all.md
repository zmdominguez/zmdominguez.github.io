---
layout: post
title: 'ADT 12: Not so shiny after all'
date: '2011-07-11T12:21:00.008+10:00'
author: Zarah Dominguez
tags:
  - wtfSDK
  - eclipse
  - extras
  - adt
  - android
modified_time: '2011-07-11T21:44:34.796+10:00'
thumbnail: http://4.bp.blogspot.com/-o6cwyB6vgGM/ThpenBXHsLI/AAAAAAAAA70/XDTqdi3kR8M/s72-c/emulator_failed_to_launch_adt12.JPG
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-3638375874358053330
blogger_orig_url: http://www.zdominguez.com/2011/07/adt-12-not-so-shiny-after-all.html
---

<a href="http://4.bp.blogspot.com/-o6cwyB6vgGM/ThpenBXHsLI/AAAAAAAAA70/XDTqdi3kR8M/s1600/emulator_failed_to_launch_adt12.JPG" onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}"><img style="float:left; margin:0 10px 10px 0;cursor:pointer; cursor:hand;width: 200px; height: 132px;" src="http://4.bp.blogspot.com/-o6cwyB6vgGM/ThpenBXHsLI/AAAAAAAAA70/XDTqdi3kR8M/s200/emulator_failed_to_launch_adt12.JPG" border="0" alt="" id="BLOGGER_PHOTO_ID_5627914708826173618" /></a>

For all the shininess that ADT 12 promised, it seems that it also broke one major feature of DDMS: Launching emulators.

After updating to ADT 12, I kept on seeing that error when launching an emulator instance. Restarting Eclipse doesn't help any.

Anyway, the apparent cause of this error is that ADT 12 has some problems with the SDK location having spaces. If you have already forgotten, it's in <span style="font-weight:bold;">Window > Preferences > Android</span>.
<a href="http://3.bp.blogspot.com/-7uTigV5TDMY/ThphWlBwDAI/AAAAAAAAA78/s1a6Iume1o0/s1600/sdk_location_old.JPG" onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}"><img style="display:block; margin:0px auto 10px; text-align:center;cursor:pointer; cursor:hand;width: 200px; height: 107px;" src="http://3.bp.blogspot.com/-7uTigV5TDMY/ThphWlBwDAI/AAAAAAAAA78/s1a6Iume1o0/s200/sdk_location_old.JPG" border="0" alt="" id="BLOGGER_PHOTO_ID_5627917724877327362" /></a>

To work around this bug, it is either you move your SDK to a folder in a path without spaces; or, modify the existing path to either of the following:
- If your SDK is in Program Files: `C:/PROGRA~1/<path_to_sdk>`
- If your SDK is in Program Files(x86): `C:/PROGRA~2/<path_to_sdk>`

**Edit (20110711 1943)**: [A bug has already been filed for this issue](http://code.google.com/p/android/issues/detail?id=18317&amp;q=tools&amp;colspec=ID%20Type%20Status%20Owner%20Summary%20Stars). Star to be notified of updates.