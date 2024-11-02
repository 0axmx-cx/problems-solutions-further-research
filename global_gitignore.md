# Global Gitignore
Typically, when you have a git repository on your system, all the files in that folder will be "up for grabs" to be committed. 

You can configure git to ignore certain files so that they aren't "up for grabs" in a sense.

This is typically done using **.gitignore** files at the root of a given repository.

However, if you have certain files that are consistent across repos, it may be better to have a global **.gitignore** file so that you don't have to think about it.

>A good use case for this would be whenever you open a given repository using obsidian (if you use obsidian to write your markdown or some other related reason).

>Obsidian automatically includes a .obsidian folder in any folder that you open as a "vault", so this can create many .obsidian file systems over time.

To create a global **.gitignore** at your home directory (which will impact all directories recursively under it):
```bash
touch ~/.gitignore

git config --global core.excludesFile ~/.gitignore
```

After that, you can add files to this **.gitignore**, just remember that they will be excluded globally, so be mindful of the files you write into this file.

## Source/More information:
[Atlassian](https://www.atlassian.com/git/tutorials/saving-changes/gitignore)