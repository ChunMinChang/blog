---
layout: post
title: Development Notes for Mozilla
category: [Work]
tags: [Firefox, Fennec]
comments: true
---
The following are some of my notes to develop the mozilla products.

- [Basic knowledge][basic]
   - XPCOM
   - WebIDL
   - Mochitest
   - others
   - <del>Firefox OS</del>
- [IPDL][ipdl]
- [(System) Add-on][addon]
- [Message Manager][mm]
- [Keyboard Event Dispatch][keyevt]
- [JPAKE over TLS][jpake]
- Build
  - Build Config/Makefile
    - [variables][build-vars]
      - [OS_TARGET][os-target]
  - [Platform build defines][pbd]
  - [Develop gecko on Windows with git][gecko-git-win]
  - [Develop gecko on Linux/MacOS with git][gecko-git-linux]
- Debugging
  - [GDB](https://developer.mozilla.org/en-US/docs/Mozilla/Debugging/Debugging_Mozilla_with_gdb)
    - [Avoid getting SIGSYS when attaching to process](https://gist.github.com/ChunMinChang/464d8e338fb6c55f081286ea0aa503f6)


[basic]: https://www.penflip.com/Chun-Min/mozilla-newbie-notes "Mozilla Newbie Notes"
[ipdl]: https://docs.google.com/presentation/d/1S4njAbl4tSFCrJ3cnE30L4PcYeWdvY-1wcghX2mWNOE/edit?usp=sharing "IPDL"
[addon]: https://docs.google.com/presentation/d/1o9qeSucSDAJO94TmZmebxRnGepY7MVy-MeVGqyaIO0s/edit?usp=sharing "Firefox (System) Add-on"
[mm]: https://docs.google.com/presentation/d/1eDjlmBdBrECQ_vXve0HgzmBilgL8HwHrcFQz9puGGr0/edit?usp=sharing "Message Manager"
[keyevt]: https://docs.google.com/presentation/d/1zxanY8xDioeIrfPyuzx9QDgggO1pC8erqhcL30gL7Wc/edit?usp=sharing "Keyboard Event Dispatch"
[jpake]: https://docs.google.com/presentation/d/15iXd4ZXy9Y1uKdXkuFsdoqXAzw7Y4pUFKlDaNiAFkb0/edit?usp=sharing "JPAKE over TLS"

[build-vars]: https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/How_Mozilla_s_build_system_works/Makefile_-_variables "Makefile - variables"
[os-target]: https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/OS_TARGET "OS_TARGET"
[pbd]: https://wiki.mozilla.org/Platform/Platform-specific_build_defines "Platform/Platform-specific build defines"

[gecko-git-win]: https://gist.github.com/ChunMinChang/bd39852e96f39fce328de812e64a5b3c#file-windows-md
[gecko-git-linux]: https://gist.github.com/ChunMinChang/bd39852e96f39fce328de812e64a5b3c#file-linux-md
