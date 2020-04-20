---
layout: post
title: Running multiple Firefox at the same time
category: [Work]
tags: [Firefox]
comments: true
---
# Running multiple Firefox at the same time

I have two firefox: Firefox(normal version) and Firefox Developer Edition.
Today morning, I deleted the Firefox Developer Edition by _AppCleaner_
and re-installed it, then something weird happened.

When I opened the new Firefox Developer Edition
then launched my unchanged Firefox, the Firefox cannot be opened
and it showed an error: __A copy of Firefox is already open.
Only one copy of Firefox an be opened at a time.__

![][a-copy-of-firefox-is-alreay-open]

AFAIK, the multiple Firefox instances are able to run at the same time,
if they use __different profiles__.
Therefore, I opened ```about:profiles``` on Firefox to check it.

![][firefox-about-profiles-wrong]

From above, the setting was incorrect.
It should use _Profile: default_ instead of _Profile: dev-edition-default_.
That's why I cannot open Firefox and FirefoxDeveloperEdition at the same time.

To fix the problem, I simply click _Set a default profile_
under _Profile: default_, then closed it, and relaunched it to check the setting.

![][firefox-about-profiles-right]

I also checked the FirefoxDeveloperEdition's setting
to make sure everything is fine.
![][firefoxdeveloperedition-about-profiles]

I wrote this to remember how I fix it,
in case the same problem happens again.

[a-copy-of-firefox-is-alreay-open]: ../images/posts/a-copy-of-firefox-is-alreay-open.png "A copy of Firefox is already open"
[firefox-about-profiles-wrong]: ../images/posts/firefox-about-profiles-wrong.png "Firefox - about:profiles(incorrect)"
[firefox-about-profiles-right]: ../images/posts/firefox-about-profiles-right.png "Firefox - about:profiles(correct)"
[firefoxdeveloperedition-about-profiles]: ../images/posts/firefoxdeveloperedition-about-profiles.png "FirefoxDeveloperEdition - about:profiles"
