## Scheduled Jobs (Cron)
One of the most common tasks associated with Linux systems is creating scheduled tasks, or cron jobs.
The cron utility is a staple for any Linux administrator and is even often used by regular Linux users.
Cron jobs make redundant tasks manageable by providing a mechanism to automate these tasks.
On the contrary, these automated jobs also provide a potential pathway for persistence.
Knowing how cron jobs are organized, stored, and used allows for identification of cron jobs built for malicous intent.  
  
__Cron Jobs for User:__
```bash
crontab -l
# The crontab utility is used control table files that serve the cron daemon.
# The -l option displays jobs for current user.
```
```bash
crontab -u <USERNAME> -l
# The -u option is used to specify a user whose jobs are to be viewed.
```
```bash
ls -alh /var/spool/cron/<USER_NAME>/
# The /var/spool/cron/ directory is the location of personal crontab files for users.
```
__Cron Directory Listings__
```bash
ls -alh /etc/cron.hourly/
# The /etc/cron.hourly/ directory is configured to execute its stored files hourly.
```
```bash
ls -alh /etc/cron.daily/
# The /etc/cron.daily/ directory is configured to execute its stored files daily.
```
```bash
ls -alh /etc/cron.weekly/
# The /etc/cron.weekly/ directory is configured to execute its stored files weekly.
```
```bash
ls -alh /etc/cron.monthly/
# The /etc/cron.monthly/ directory is configured to execute its stored files monthly. 
```
```bash
ls -alh /etc/cron.d/
# The /etc/cron.d/ directory is used to configure cron jobs that do not fit any other available locations. It is often used by packages or for custom cron jobs.
```
__Viewing Cron File__
```bash
less /etc/cron.<DIRECTORY>/<FILE_NAME>
# The less utility is used to view output in sections instead of all at once.
```
  
#### Demonstration:
Similar to the previous demonstration with daemons, we will create our own cron job that demonstrates malicious intent.
First, take a look at any existing job in the cron.d directory to understand the structure.
The first five fields, likely some indicated by '\*' set the actual time values for cronjobs.
They represent minute, hour, day (of the month), month, day (of the week). Any '\*' value represents every value.
These fields can be assigned with specific values, lists, ranges, and steps.
A good reference for creating and/or making sense of cronjob configurations is https://crontab.guru/.
The remaining portion of the job contains the designated user for the job and the actual command to run.
With the text editor of your choice, go ahead and create a job file titled "airbud" with the contents:
```bash
* * 20 * * /usr/bin/wget user https://pics.me.me/air-bud-i-havent-heard-that-name-in-years-3810671.png -O /tmp/malwares
```
Be sure to replace 20 with whatever day of the month it is and to replace user with an actual username you intend to run the job with on your system.
After one minute, you should see a new file titled "malwares" in your /tmp directory.
Don't worry, it's not actually malware in this instance, unless you hate animal memes. But be mindful that advanced threat actors are commonly utilizing images for malicious embeds.
If you have a desktop and want to take a look at the photo, you can do so with the command:
```bash
xdg-open /tmp/malwares
```
or by going to the link: https://pics.me.me/air-bud-i-havent-heard-that-name-in-years-3810671.png.
Using some of the mentioned methods, take a look at your created cron job in comparison to other jobs.
Altough this a pretty simple scenario, use some critical thinking to think of some cases where access to cron may be detrimental.
