git structure:

- master brnach is a copy of the original master branch.
- fixes branch contains your fixes to the original work. can be your own fixes or from someone else. Next time you update master, you can cherrypick these commits (if they hadn't been applied to master yet)
- builds branch contains the changes done by the build system, this way you reduce the unnecessary merge conflicts to a minimum

- update master with upstream master
- merge master into fixes branch
