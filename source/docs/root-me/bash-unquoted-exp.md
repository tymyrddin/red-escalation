# Bash: unquoted expression injection

[root-me challenge: Bash - unquoted expression injection](https://www.root-me.org/en/Challenges/App-Script/Bash-unquoted-expression-injection): Bypass this script’s security to recover the validation password.

----

```text
./somescript "0 -o foo"
```

This makes any condition become

    test 1234 -eq 0 -o foo

This is the equivalent of `1234 == 0 || "foo"` in other languages, with one irrelevant comparison `OR`'d with the truth value of the string `foo`.

Since test considers all non-empty strings to be true, this expression is always true.
```
