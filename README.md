# UNIX Admin Utilities/Commands
a cheat sheet for common (and some uncommon) command line tasks required when administering web applications, Drupal in particular.  References include:
- http://www.commandlinefu.com/commands/browse

## Contents
1. [Diff/Merge](#diffmerge)
1. [Find (on file system and in file)](#find-on-file-system-and-in-file)
1. [RegEx](#regex)
1. [Text Editing](#text-editing)
1. [Drupal/Drush/Composer](#drupaldrushcomposer)
1. [General Admin](#general-admin)
1. [Backup via rsync](#backup-via-rsync)
1. [TROUBLESHOOTING](#troubleshooting)
1. [Random Things](#random-things)
1. [git Tricks](#git-tricks)
1. [SVN](#svn)

## Diff/Merge
### Merge Source Folder into Destination Folder w/o Overwriting
Reference: http://the.taoofmac.com/space/HOWTO/Merge%20Folders
These commands preserve files/directories unique to destination target. 

The standard UNIX way:
```bash
cp -R -v source/. destination
```

The geeky UNIX way (restartable):
```bash
rsync -vaEW source/ destination
```

### Another method of merging directories recursively - and handling deletes
create a list of files from both directories:
```bash
cd <path to local directory>
find . | sort > /tmp/filenames-local
cd <path to remote directory>
find . | sort > /tmp/filenames-remote
```

### compare both files, supressing lines that appear in both files and lines that are unique to either
List files that only exist in the remote directory:
```bash
comm -1 -3 /tmp/filenames-local /tmp/filenames-remote
```
List files that only exist in the local directory:
```bash
comm -2 -3 /tmp/filenames-local /tmp/filenames-remote
```

### use the output of the command above to copy the missing files
reference: http://www.no-ack.org/2010/08/merge-directories-on-unix-like-systems.html
```bash
export IFS=$'\n'
```
Copy missing files from the remote to the local directory.
```bash
tar -C <path to remote directory> -c $(comm -1 -3 /tmp/filenames-local /tmp/filenames-remote) | tar -C <path to local directory> -x
```
Copy missing files from the local to the remote directory.
```bash
tar -C <path to local directory> -c $(comm -2 -3 /tmp/filenames-local /tmp/filenames-remote) | tar -C <path to remote directory> -x
```

## Find (on file system and in file)
References: 
- http://www.thegeekstuff.com/2009/03/15-practical-linux-find-command-examples
- https://stackoverflow.com/questions/4529134/delete-files-with-string-found-in-file-linux-cli
- https://stackoverflow.com/a/11283391

### largest files in directory
```bash
find . -type f -size +50000k -exec ls -lh {} \; | awk '{ print $8 ": " $5 }'
```
or
```bash
du -ks . | sort -n -r | head -n 10
```

### find directory X (e.g. .svn) current folder and one level down
```bash
sudo find . -maxdepth 2 -type d -name '.svn'
```

### count lines in file that do not contain the word ‘.svn’
reference: http://www.ahinc.com/linux101/textfiles.htm
```bash
grep -c -v .svn ~/svn-delete-it.brownsites.log
```

### Find files with given content in a directory (and subdirectories)
```bash
grep -nrol “given content” .
```
#### Take found files and create a batch deletion script
- Find files containing `email@example.com`.
   - Create `doit.sh` script that removes each of the found files.
- Review and edit the batch remove script for any outliers.
- Run the script.
```bash
find . | xargs grep -l email@example.com | awk '{print "rm "$1}' > doit.sh
vi doit.sh // check for murphy and his law
source doit.sh
```

### Find files not containing given content in a directory
```bash
grep -L "given content" .
```
e.g. 
```bash
grep -L "shib_authmap" ./*/settings.php
```
#### Find and remove files that do not contain given content
```bash
find . -type f -print0 | xargs --null grep -Z -L 'my string' | xargs --null rm
```

### Find files containing “one content string” that doesn’t contain “another content string”
```bash
grep -rl <string-to-match> | xargs grep -L <string-not-to-match>
```

### Find files with specific name or name pattern in directory (and subdirs)
```bash
find . -name "*some-filename-pattern*" -print | xargs grep "some text pattern"
```

### Find files of specific type with specific content and only list file names
```bash
find . -iname '*php' | xargs grep 'string' -sl
```
(finds and lists php files containing the string “string”)

### Find files with specific owner
```bash
find . *.* -user $USERNAME -print
```

### Find files by specific group
reference: http://www.cyberciti.biz/faq/how-do-i-find-all-the-files-owned-by-a-particular-user-or-group/
```bash
find directory-location -group {group-name} -name {file-name}
```
e.g. 
```bash
find . -group webserv -name *.php
```

### Find and Replace in file using sed
reference: http://www.cyberciti.biz/faq/unix-linux-replace-string-words-in-many-files/
```bash
sed -i 's/old-word/new-word/g' *.txt
```

### Find files in specific files, exclude certain paths, and show first occurrence of specific string in content
```bash
sudo find . \( ! -path '*/sites/*' ! -path '*/profiles/*' ! -path '*/themes/*' ! -path '*/modules/*' \) -iname 'CHANGELOG.txt' | xargs grep -m 1 'Drupal' -s1
```

### Find files modified X days ago or after
reference: http://stackoverflow.com/questions/848293/shell-script-get-all-files-modified-after-date
```bash
find ./*/files/. -mtime -10
```
(X = 10 days ago)

## RegEx
### Search for tags in content

### Search for tags and subsearch for tag name (e.g. “a” of <a href=...> or “b” of <b>, etc)

### Search for tags and subsearch enclosed content

## Text Editing
### Inline (sed, awk, etc)

### VI
#### Search and Replace from VI command mode (:) 
Enter:
```
%s/old-string/new-string/
```
e.g. User needs to go through a settings file and replace the path to physical directory “drupal_core/drupal6/drupal-6.22” with the simpler path to a symlink “drupal-cms”:
```
%s/drupal_core\/drupal6\/drupal-6.22/drupal-cms
```
N.B. “\” used to escape “/” characters in search and/or replace strings.

#### Display Line Numbers:
```
set number
```

## Drupal/Drush/Composer
### DDev Usage
#### Setup on MacOS
##### Setup DDev
##### Setup XDebug (w/DDev and VSCode)
###### References:
- [Visual Studio Code (VS Code) Debugging Setup (w/DDEV)](https://ddev.readthedocs.io/en/stable/users/debugging-profiling/step-debugging/#visual-studio-code-vs-code-debugging-setup)
- [Debug PHP/Drupal with DDEV, XDebug, and VSCode](https://blog.devops.dev/debug-php-drupal-with-ddev-xdebug-and-vscode-7f40261c5864)
- [Setting up XDebug + VSCode for use w/Drush](https://github.com/ddev/ddev/issues/2341)
- [XDebug Step Debugging Guide](https://xdebug.org/docs/step_debug)
- [XDebug Over Command Line w/DDev](https://mglaman.dev/blog/xdebug-over-command-line-ddev)
###### Setup Steps
1. Setup XDebug on DDEV
   1. Run `ddev xdebug`.
1. Setup Browser's XDebug Listener
1. Setup XDebug in VSCode
#### Setup on Linux
##### Setup DDev
##### Setup XDebug (w/DDev and VSCode)
###### References:
- [Visual Studio Code (VS Code) Debugging Setup (w/DDEV)](https://ddev.readthedocs.io/en/stable/users/debugging-profiling/step-debugging/#visual-studio-code-vs-code-debugging-setup)
###### Setup Steps
1. Setup XDebug on DDEV
   1. Run `ddev xdebug`.
### Drush Usage
#### Import DB via Drush
```bash
drush sql-cli < /path/to/exported-db-file.sql
```
More about [drush sql-cli usage](https://www.drush.org/12.x/commands/sql_cli/).
#### Drush on MultiSite (7.x+)

#### Fun With Drush
```bash
drush php-eval 'node_access_rebuild()';
drush php-eval 'drupal_rebuild_theme_registry()';
drush php-eval 'menu_rebuild()';
```

### Composer
References:
1. [(d.o) Updating Drupal Core via Composer](https://www.drupal.org/docs/updating-drupal/updating-drupal-core-via-composer)
2. [Composer and Version Constraints](https://getcomposer.org/doc/articles/versions.md)
#### Install specific version of a package
`composer require vendor/package_name:version_id --with-all-dependencies`
#### List Available Updates for Drupal core and contrib
`composer outdated "drupal/*"`
#### Update Drupal Core via Command Line
`composer update "drupal/core-*" --with-all-dependencies`
#### Require Packages for Dev Site Installs
`composer require --dev drupal/devel`
##### Install without `dev` packages
`composer install --no-dev`

## General Admin
### Show Environment Variables
```bash
printenv #shows all environment variables
```

```bash
echo $variable_name #shows the value for the given variable
```

e.g. for PATH:
```bash
echo $PATH
```

### Get System Info
shows all information about the distribution currently installed
```bash
lsb_release -a
```

### Disk capacity
gets amount of space used on partition where ‘sites’ folder located
```bash 
df sites 
```

### Folder Size
human readable folder sizes and total folder size for ‘sites’ folder
```bash
du -ch sites 
```
human readable total size for ‘sites’ folder
```bash
du -ch sites | grep total 
```

### Search bash command history
reference: http://www.techrepublic.com/article/master-the-linux-bash-command-line-with-these-10-shortcuts/5827311
Sometimes you want to see how you might have done something before on the command line.
```
Ctrl+r
```
then type in the keyword or bash command to see previous uses of the command


### Date/Time
Show current date as UNIX timestamp: 
```bash
date -u +%s
```

### Networking Stuff
What ports are in use or assigned?
reference: http://www.cyberciti.biz/tips/linux-display-open-ports-owner.html
```bash
sudo lsof -i
```
```bash
sudo netstat -lptu
```
```bash
sudo netstat -tulpn
```

### Backup via rsync
sync directory from prod source to QA destination (assuming logged into QA server)
```bash
rsync -r -a -v -e ssh username@server.example.com:/var/www/cms/sites/example.com.base /var/www/cms/sites/example.com.base
```


## TROUBLESHOOTING

### Argument list too long
example:
"-bash: /bin/rm: Argument list too long"

Occurs when list of arguments for given command exceed system limits.  The example above occurred when rm * was run on a directory with an excessive (100s) number of files on a restrictive system.

workaround:
```bash
find . -name ‘*.jpg’ | xargs rm
```


## Random Things

### Generate Random String
reference:  http://www.howtogeek.com/howto/30184/10-ways-to-generate-a-random-password-from-the-command-line/
```bash
date | md5sum
```

## git Tricks

### Execute git Commands on Subfolders
reference: http://www.zoharbabin.com/git-pull-inside-multiple-folders
```
find basefolder -name .git -execdir git pull origin master \;
```

this finds the subdirectories under current directory with git repositories (contains .git folder), echoes the path and pulls updates (from origin to local master)
```
find . -name ".git" -exec echo {} \; -execdir git pull \;
```

## SVN
### Checkout Specific Revision from SVN
this checks out revision 1234 of somepath to ./working-directory
```bash
svn checkout svn://somepath@1234 working-directory
```

### Remove unwanted SVN directories
```bash
find . -type d -name '.svn' -print0 | xargs -0 rm -rdf
```

### Adding missing files via svn status, grep, gawk, and svn update
reference: http://www.thingy-ma-jig.co.uk/blog/11-03-2009/lazy-linux-piping
when running svn ci, kept getting error message:
"   svn: Commit failed (details follow):
   svn: Directory '/www/data/httpd/htdocs/cms/sites/all/modules/xxxxx' is missing
 used the following to re-add missing directories in bulk"
```bash
svn st all/modules | grep ^! | gawk '{ print $2 }' | xargs svn up
```

### Import in Place
references: 
- http://subversion.apache.org/faq.html#in-place-import
- http://stackoverflow.com/questions/678437/svn-in-place-import-and-checkout
- http://news.e-scribe.com/350
```bash
     svn mkdir file:///root/svn-repository/etc \
         -m "Make a directory in the repository to correspond to /etc"
      cd /etc
      svn checkout file:///root/svn-repository/etc .
      svn add apache samba alsa X11
      svn commit -m "Initial version of my config files"
```

### Set multiple values for svn:ignore property for a working copy
 reference: http://sdesmedt.wordpress.com/2006/12/10/how-to-make-subversion-ignore-files-and-folders/
 inside target directory:
```
svn propset svn:ignore -F ignore.txt .
```

### Remove files currently in SVN but retain on filesystem
reference: http://stackoverflow.com/questions/542065/svn-remove-file-from-repository-without-deleting-local-copy
```
svn delete --keep-local the_file
```

### Find and Execute to Perform SVN Cleanup
In response to:
"Can't remove directory 'example.com/mysite/themes/.svn/tmp/props': Permission denied"

entering the following command:
```
find example.com.mysite* -type d -exec sudo chown -R myuser:mygroup .svn '{}' \;; svn cleanup .
```

### Get SVN Info on Directories Newer than Given File or Folder
```
find -maxdepth 1 -type d -newer example.com.mysite -exec svn info {} \;
```
