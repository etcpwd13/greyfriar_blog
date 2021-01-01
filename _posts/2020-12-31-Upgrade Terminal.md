---
title: "Tool: Upgrade Terminal"
date: 2020-12-31T14:34:30-04:00
excerpt_separator: "<!--more-->"
categories:
  - Blog
  - Tool
tags:
  - shell
  - tool
  - commands
  - enumeration
---

```yaml
SHELL=/bin/bash script -q /dev/null
Ctrl-Z
stty raw -echo
fg
reset
xterm
```

Notes for general pugrading terminal are here:

[Upgrade a Dumb Shell to a Fully Interactive Shell for More Flexibility][upgrade-term]

[upgrade-term]: https://null-byte.wonderhowto.com/how-to/upgrade-dumb-shell-fully-interactive-shell-for-more-flexibility-0197224/
