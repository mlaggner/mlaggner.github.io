---
title: "Config file backup with GIT"
categories:
  - linux
tags:
  - linux
  - git
  - shellscript
summary:
  - image: tux.png
---
At work I've stumbled across the problem to save configuration files of some Linux servers I am administrating. Since I am a developer I am familiar to GIT and thus I wanted to use the benefits of GIT to solve this problem.

There are several solutions to use GIT for a config file backup. The most obvious one would be to use a git repository directly in /etc (like [etckeeper](https://etckeeper.branchable.com/) does), but there are some disadvantages with this method:

- the whole /etc will be stored and not only the relevant ones
- not all configuration files are in /etc (e.g. some services have their configs in /opt)
- you could bring wrong versions of the config files via ```git pull``` from the upstream repo into the live /etc

I've decided to use another approach: I wrote a script which copies all specified files/folders from a separate config file to a temporary folders which contains the git repository. This approach solves the shortcomings from above:

- only specified files will be backed up
- we can use any file from the file system
- ```git pull``` cannot destroy productive config files

For this approach you simply need one script and a config file called ```config_files.txt``` which contains a list of all files to be backed up. The script does a cleanup of the temporary folder and copies all relevant config files:

```
#!/bin/bash

find . -maxdepth 1 -mindepth 1 ! -regex "./\(config_files.txt\|.git\|copy.sh\)" | xargs rm –r
cat config_files.txt | xargs -I {} cp -- parents -R {} .
```

Having this script, you can set up a backup of your config files with a few commands:

1. create the temporary folder:

        / # mkdir –p /backup/conf        

2. create the git repository inside the temporary folder

        / # cd /backup/conf
        /backup/conf # git init
        /backup/conf # git remote add origin <path to git repo>

3. add the file ```config_files.txt``` to the temporary folder containing a list of files/folders to backup. E.g.

        /etc/nginx/nginx.conf
        /etc/nginx/conf.d/

4. execute the copy script inside the backup folder

        /backup/conf # copy_conf.sh

5. check if config files have been changed since your last commit

        /backup/conf # git status
        On branch master
        Initial commit

        Untracked files:
        (use "git add <file>..."" to include in what will be committed)

            config_files.txt
            etc/

        nothing added to commit but untracked files present (use "git add" to track)

6. commit/push the new backup

        /backup/conf # git commit
        /backup/conf # git push
