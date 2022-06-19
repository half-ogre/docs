# Making a First Empty Commit

I like to make my very first commit in a repo an empty commit. This avoids a potential issue later if you don't like your first "real" commit and want to "undo" it.

Git won't allow you to make an empty commit by default, so you need to add the `--allow-empty` arg, like so:

```
git init
git commit -m "empty" --allow-empty
```