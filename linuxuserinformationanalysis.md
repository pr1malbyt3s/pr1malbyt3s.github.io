## User Information
Information pertaining to individual users is commonly viewed as one of the most useful forensic data sources for security incidents.
Not only does it provide a wealth of information, but it also provides a concept of attribution.
Additionally, accounts and their permissions are easily tracked for users and administrators alike for standalone systems.
If one were to see a user that they have never created nor used, or a user performing actions they're not authorized for, then it's easily perceived that something is awry.
However, the use of legitimate credentials to blend-in with normal actions is usually harder, but not impossible to detect.
Using credentials, whether through abuse or manipulation, covers a lot of well-known adversary tactics.
It is important to be knowledgable of user information because of the significant data it provides.

__Users:__
```bash
cat /etc/passwd
# /etc/passwd is the file used for system user configuration.
# Username is displayed in the first field.
# Password is the second field, usually occupied by an x, indicating the user's encrypted password is store in the /etc/shadow file.
# User ID (UID) is the third field.
# Group ID (GID) is the fourth field.
# User Info is in the fifth field.
# The user's absolute path home directory is the sixth field.
# The user's absoltue path command or shell is the seventh field.
# Source: https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format/
```
__Groups:__
```bash
cat /etc/group
# /etc/group is the filed used for system group configuration.
# Group name is displayed in the first field.
# Group password, if any, is displayed in the second field.
# Group ID (GID) is the third field.
# Group members are listed in the fourth field.
```
__Sudoers:__
```bash
cat /etc/sudoers
# /etc/sudoers is the file used to configure super user privileges.
# Multiple aliases are available such as hostnames, users, services, commands, etc.
# By default, the root user has the ability to run any commands from any location.
# By default, members of the wheel group can run all commands with a password.
```
__Open Files for User:__
```bash
lsof -u <USER>
# The -u option is used to select listings of files for a specific user.
```
__User Command History:__
```bash
cat ~/.bash_history
# Displays command history stored in users bash_history file.
```
```bash
history
# Part of the bash utility that displays command history.
```
__SSH Files:__
```bash
ls -alh ~/.ssh/
# Lists all files in the .ssh directory.
# The known_hosts file list hosts the user connected to from this machine.
# The authorized_keys file lists public keys used for logins this machine.
# The id_rsa file lists private keys used to log in to remote hosts.
```
__History of Websites Visited:__
```bash
sqlite3 ~/.mozilla/firefox/<PROFILE>.default/places.sqlite "SELECT url FROM moz_places;"
# Lists all the websites visited by the user on FireFox.
# FireFox browser artifacts are stored in a SQLite database file titled places.sqlite.
# The sqlite3 utility is a command line interface for the SQLite database service.
```
```bash
sqlite3 .config/google-chrome/Default/History "SELECT url FROM urls"
# Lists all the websites visited by the user on Chrome
```
__History of Downloads:__
```bash
sqlite3 ~/.mozilla/firefox/<PROFILE>.default/places.sqlite "SELECT url, content FROM  moz_annos JOIN moz_places ON moz_places.id=moz_annos.place_id;"
# Lists all items downloaded by the user on FireFox.
```
```bash
sqlite3 .config/google-chrome/Default/History "SELECT referrer, target_path FROM downloads"
# Lists all items downloaded by the user on Chrome.
```
  
#### Demonstration:
We'll cover the power of sudo and how it can turn any average Joe into a king. We'll first investigate how we can add a user to the privileged 'wheel' group to gain sudo privileges.
Then, we'll take a look at the effects of adding specific users to the /etc/sudoers file. First, let's create our user, 'joe':
```bash
adduser joe
```
We'll then change the password for user 'joe' (feel free to pick any password, just remember that you'll need it):
```bash
passwd joe
```
Take a look at some of the mentioned file locations to familiarize with user 'joe'.
Next, we'll add 'joe' to the privileged 'wheel' group:
```bash
usermod -G wheel joe
```
Take another look at the '/etc/groups' file to observe where 'joe' is now present.
At this point, 'joe' should be able to execute any command with elevated privileges using the prefix 'sudo'. Let's explore this. First, change your current user to user 'joe':
```bash
su joe
```
Now use the accounts privileges to become the user 'root':
```bash
sudo su
```
Any command would have worked, but if one were to obtain a shell as 'root', all other commands are moot at that point.
We'll now remove 'joe' from the 'wheel' group (make sure you exit out of your current session as 'joe' first):
```bash
gpasswd -d joe wheel
```
And again, change your user back to 'joe' and try to use sudo:
```bash
su joe
sudo su
```
Did it work? All of the sudo privileges for user 'joe' are now removed.
Let's take a look at the other alternative- adding a specific user to the /etc/sudoers file.
Issue the following command to place permissions in the /etc/sudoers file for 'joe' to run the '/bin/bash' command with elevated privileges:
```bash
echo "joe ALL=(ALL) /bin/bash" >> /etc/sudoers
```
Change back to user 'joe' and first try to look at the '/etc/sudoers' file:
```bash
su joe
sudo cat /etc/sudoers
```
This demonstrates that 'joe' definitely doesn't have sudo privileges for everything. Let's try the actual command we added for 'joe':
```bash
sudo bash
```
User 'joe' now has his elevated privileges back. Take a look at some of the mentioned files again to confirm that 'joe' looks just like an ordinary user. So what are the inherent dangers with what we just displayed?
Can you think of any ways this might be leveraged for various attack tactics?
Remember, it's not advocated to execute any commands directly as user root. Always try to use sudo and limit privileges as much as possible.
When you're finished investigating the power of sudo, you can clean up your '/etc/sudoers' file by issuing the command:
```bash
sed -i '/joe ALL=(ALL) \/bin\/bash/d' /etc/sudoers
```
You can then delete user 'joe' with the command:
```bash
userdel -r joe
```
