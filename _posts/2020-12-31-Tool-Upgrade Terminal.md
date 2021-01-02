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

Let's upgrade to a tty shell and continue enumeration.

```yaml
SHELL=/bin/bash script -q /dev/null
```

If the target has python installed you can upgrade shell with this:

```yaml
python3 -c "import pty;pty.spawn{'/bin/bash'}"
```

Additional Notes for general upgrading terminal are here:

[Upgrade a Dumb Shell to a Fully Interactive Shell for More Flexibility][upgrade-term]

[upgrade-term]: https://null-byte.wonderhowto.com/how-to/upgrade-dumb-shell-fully-interactive-shell-for-more-flexibility-0197224/
