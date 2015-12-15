# UNIX Admin Utilities/Commands
a cheat sheet for common (and some uncommon) command line tasks required when administering web applications, Drupal in particular.  References include:
- http://www.commandlinefu.com/commands/browse



##Diff/Merge
###Merge Source Folder into Destination Folder w/o Overwriting
These commands preserve files/directories unique to destination target. Reference:
http://the.taoofmac.com/space/HOWTO/Merge%20Folders

The standard UNIX way:
```bash
cp -R -v source/. destination
```

The geeky UNIX way (restartable):
```bash
rsync -vaEW source/ destination
```

###Another method of merging directories recursively - and handling deletes
create a list of files from both directories:
```bash
cd <path to local directory>
find . | sort > /tmp/filenames-local
cd <path to remote directory>
find . | sort > /tmp/filenames-remote
```

###compare both files, supressing lines that appear in both files and lines that are unique to either
List files that only exist in the remote directory:
```bash
comm -1 -3 /tmp/filenames-local /tmp/filenames-remote
```
List files that only exist in the local directory:
```bash
comm -2 -3 /tmp/filenames-local /tmp/filenames-remote
```

###use the output of the command above, like below in order to copy the missing files
export IFS=$'\n'
Copy missing files from the remote to the local directory.
tar -C <path to remote directory> -c $(comm -1 -3 /tmp/filenames-local /tmp/filenames-remote) | tar -C <path to local directory> -x
Copy missing files from the local to the remote directory.
tar -C <path to local directory> -c $(comm -2 -3 /tmp/filenames-local /tmp/filenames-remote) | tar -C <path to remote directory> -x

reference: http://www.no-ack.org/2010/08/merge-directories-on-unix-like-systems.html



##Find (on file system and in file)
(see http://www.thegeekstuff.com/2009/03/15-practical-linux-find-command-examples/)

###largest files in directory
find . -type f -size +50000k -exec ls -lh {} \; | awk '{ print $8 ": " $5 }'

du -ks . | sort -n -r | head -n 10


###find directory X (e.g. .svn) current folder and one level down
sudo find . -maxdepth 2 -type d -name '.svn'


###count lines in file that do not contain the word ‘.svn’
from http://www.ahinc.com/linux101/textfiles.htm
grep -c -v .svn ~/svn-delete-it.brownsites.log


###Find files with given content in a directory (and subdirectories)
grep -nrol “given content” .

###Find files not containing given content in a directory
grep -L "given content" .
e.g. grep -L "shib_authmap" ./*/settings.php

###Find files containing “one content string” that doesn’t contain “another content string”
grep -rl <string-to-match> | xargs grep -L <string-not-to-match>


###Find files with specific name or name pattern in directory (and subdirs)
find . -name "*some-filename-pattern*" -print | xargs grep "some text pattern"


###Find files of specific type with specific content and only list file names
find . -iname '*php' | xargs grep 'string' -sl
(finds and lists php files containing the string “string”)

###Find files with specific owner
find . *.* -user $USERNAME -print

###Find files by specific group
find directory-location -group {group-name} -name {file-name}
e.g. find . -group webserv -name *.php

http://www.cyberciti.biz/faq/how-do-i-find-all-the-files-owned-by-a-particular-user-or-group/

###Find and Replace in file using sed
(http://www.cyberciti.biz/faq/unix-linux-replace-string-words-in-many-files/)
sed -i 's/old-word/new-word/g' *.txt

###Find files in specific files, exclude certain paths, and show first occurrence of specific string in content
sudo find . \( ! -path '*/sites/*' ! -path '*/profiles/*' ! -path '*/themes/*' ! -path '*/modules/*' \) -iname 'CHANGELOG.txt' | xargs grep -m 1 'Drupal' -s1

###Find files modified X days ago or after
find ./*/files/. -mtime -10
(X = 10 days ago)

http://stackoverflow.com/questions/848293/shell-script-get-all-files-modified-after-date



##RegEx

###Search for tags in content


###Search for tags and subsearch for tag name (e.g. “a” of <a href=...> or “b” of <b>, etc)


###Search for tags and subsearch enclosed content



##Text Editing
###Inline (sed, awk, etc)

###VI
Search and Replace - from VI command mode (:) enter:
%s/old-string/new-string/
e.g. User needs to go through a settings file and replace the path to physical directory “drupal_core/drupal6/drupal-6.22” with the simpler path to a symlink “drupal-cms”:

%s/drupal_core\/drupal6\/drupal-6.22/drupal-cms
(N.B. “\” used to escape “/” characters in search and/or replace strings)

Display Line Numbers - :set number


##Drupal/Drush
###Drush on MultiSite (7.x+)


###Fun With Drush
drush php-eval 'node_access_rebuild()';
drush php-eval 'drupal_rebuild_theme_registry()';
drush php-eval 'menu_rebuild()';



##General Admin

###Show Environment Variables
printenv - shows all environment variables

echo $variable_name - shows the value for the given variable, e.g. for PATH:
echo $PATH


###Get System Info
lsb_release -a
shows all information about the distribution currently installed


###Disk capacity
df sites - gets amount of space used on partition where ‘sites’ folder located


###Folder Size
du -ch sites - human readable folder sizes and total folder size for ‘sites’ folder

du -ch sites | grep total - human readable total size for ‘sites’ folder


###Search bash command history
Sometimes you want to see how you might have done something before on the command line.

from: http://www.techrepublic.com/article/master-the-linux-bash-command-line-with-these-10-shortcuts/5827311
ctrl+r
then type in the keyword or bash command to see previous uses of the command


###Date/Time
Show current date as UNIX timestamp: date -u +%s


###Networking Stuff
What ports are in use or assigned?
(from: http://www.cyberciti.biz/tips/linux-display-open-ports-owner.html)
sudo lsof -i
sudo netstat -lptu
sudo netstat -tulpn


##Backup via rsync
sync directory from prod source to QA destination (assuming logged into QA server)
rsync -r -a -v -e ssh anwosu@matador.services.brown.edu:/www/data/httpd/htdocs/cms/sites/brown.edu.it.brownsites.base /www/data/www/data/httpd/htdocs/cms/sites/brown.edu.it.brownsites.base



##TROUBLESHOOTING

###Argument list too long
example:
-bash: /bin/rm: Argument list too long

Occurs when list of arguments for given command exceed system limits.  The example above occurred when rm * was run on a directory with an excessive (100s) number of files on a restrictive system.

workaround:

find . -name ‘*.jpg’ | xargs rm



##Random Things

###Generate Random String
date | md5sum

from:  http://www.howtogeek.com/howto/30184/10-ways-to-generate-a-random-password-from-the-command-line/



##git Tricks

###Execute git Commands on Subfolders
http://www.zoharbabin.com/git-pull-inside-multiple-folders
find basefolder -name .git -execdir git pull origin master \;

this finds the subdirectories under current directory with git repositories (contains .git folder), echoes the path and pulls updates (from origin to local master)

find . -name ".git" -exec echo {} \; -execdir git pull \;



##SVN
###Checkout Specific Revision from SVN
this checks out revision 1234 of somepath to ./working-directory
svn checkout svn://somepath@1234 working-directory

###Remove unwanted SVN directories
find . -type d -name '.svn' -print0 | xargs -0 rm -rdf

###Adding missing files via svn status, grep, gawk, and svn update
found at http://www.thingy-ma-jig.co.uk/blog/11-03-2009/lazy-linux-piping
when running svn ci, kept getting error message:
   svn: Commit failed (details follow):
   svn: Directory '/www/data/httpd/htdocs/cms/sites/all/modules/xxxxx' is missing
 used the following to re-add missing directories in bulk

svn st all/modules | grep ^! | gawk '{ print $2 }' | xargs svn up


###Import in Place
found at: http://subversion.apache.org/faq.html#in-place-import
and: http://stackoverflow.com/questions/678437/svn-in-place-import-and-checkout
and: http://news.e-scribe.com/350

     svn mkdir file:///root/svn-repository/etc \
         -m "Make a directory in the repository to correspond to /etc"
      cd /etc
      svn checkout file:///root/svn-repository/etc .
      svn add apache samba alsa X11
      svn commit -m "Initial version of my config files"


###Set multiple values for svn:ignore property for a working copy
 from: http://sdesmedt.wordpress.com/2006/12/10/how-to-make-subversion-ignore-files-and-folders/
 inside target directory:
svn propset svn:ignore -F ignore.txt .

###Remove files currently in SVN but retain on filesystem
reference: http://stackoverflow.com/questions/542065/svn-remove-file-from-repository-without-deleting-local-copy
svn delete --keep-local the_file


###Find and Execute to Perform SVN Cleanup
In response to:
Can't remove directory 'brown.edu.research.projects.center-for-fluid-mechanics/themes/.svn/tmp/props': Permission denied

entering the following command:
find brown.edu.research.projects* -type d -exec sudo chown -R anwosu:webserv .svn '{}' \;; svn cleanup .


###Get SVN Info on Directories Newer than Given File or Folder
e.g. find -maxdepth 1 -type d -newer brown.edu.Student_Services.Greek_Council -exec svn info {} \;

