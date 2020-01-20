---
layout: default
title: SQLite Browser Artifacts
parent: Forensics
nav_order: 2
---

# Using SQLite to Gather Browser Artifacts
  
Often times, a security incident involves actions taken by a user, such as opening a malicious email attachment or downloading a file from a visited website. In order to further understand what initiated the event, we concern ourselves with what sites the user has browsed and what he or she has downloaded, whether intentionally or inadvertently. According to http://gs.statcounter.com/, Google Chrome currently (as of January 2020) holds a 63.62% market share of browsers used and Mozilla Firefox holds a 4.39% share. If you're an avid Linux user like myself, you very well know that almost every distro comes with either of these browsers installed. Additionally, a quick poll of my colleagues, family, and friends who are Windows users revealed that when available, most users will choose to use Chrome. 
  
Since there is a high probability of a security incident spawning from an end-user workstation, it's important to know where to find artifacts these browsers create because of their potential usefulness in an investigation. You might be thinking- "Can't I just open up Chrome or Firefox and check the history in the browser?". The simple answer is "sure", but consider two scenarios where that might be infeasible:
1. You can only access the suspected compromised machine remotely (think Secure Shell or PowerShell Remoting).
2. You are analyzing a forensic copy taken of the suspected compromised machine's drive. (*If you care in the slightest about maintaining forensic integrity, you know this is the correct scenario!*)  
  
Luckily, both browsers utilize a SQLite format file to store this information to disk. As defined by sqlite.org- "SQLite is a C-language library that implements a small, fast, self-contained, high-reliability, full-featured, SQL database engine". Basically, it is a database application whose file format is used by many other applications to store persistent data, which is easily accessed without the burden of an external service (such as MySQL, MSSQL, etc.). We'll explore how SQLite is similarly used by each browswer and how to access each their respective data using native shell commands.

*If you do not have SQLite installed, you can download here for Windows: https://sqlite.org/download.html and using the following commands for Linux:*
```sh
sudo apt-get install sqlite
#Debian based distributions.
```
```sh
sudo yum install sqlite
#RPM based distributions.
```
  
## Google Chrome
Starting with Google Chrome, we'll take a look at some of the files available, the information they store, and provide an example query on how to find downloaded files along with the URL they came from for both Linux and Windows operating systems. 
  
### Chrome SQLite Files
First, it is pertinent to know where these SQLite files are stored at. On Windows, we can find these files in the following directory:
```PS
C:\Users\<USER>\AppData\Local\Google\Chrome\User Data\Default
```
And on Linux, these files can be found in the following directory:
```sh
/home/<USER>/.config/google-chrome/Default
```

Now that we know where to find the files, we need to know what information is stored in each:
- *Cookies*: stores cookie data for visited sites.
- *Favicons*: stores favicon information for visited sites.
- *History*: stores site history information, such as downloads, URLs, and visits.
- *Login Data*: stores login information for saved login sites.
- *Shortcuts*: stores shortcut information for frequently visited sites.
- *Top Sites*: stores information regarding top visited sits.
- *Web Data*: stores miscellaneous information such as autofill, credit card, or keyword data.  
*Note: This list was compiled from multiple sources. Not all files for every system are listed, and some files on this list may not be found on other systems*
  
### Gathering Chrome Download History:
Knowing now where the relevant Chrome SQLite files are located and what information is stored in each, let's take a look at the table structure to help form a userful query. Chrome is rather simplistic in its order structure, thus making it easy for us to only deal with one file- __History__. Inside the __History__ SQLite file, the __downloads__ table stores relevant information for identifying downloaded files. Its table structure is shown below:
  
__downloads:__

| id (pk) | guid | current_path | target_path | start_time | received_bytes | total_bytes | state | danger_type | interrupt_reason | hash | end_time | opened | last_access_time | transient | referrer | site_url | tab_url | tab_referrer_url | http_method | by_ext_id | by_ext_name | etag | last_modified | mime_type| original_mime_type |  
| ------- | ---- | ------------ | ----------- | ---------- | -------------- | ----------- | ----- | ----------- | ---------------- | ---- | -------- | ------ | ---------------- | --------- | -------- | -------- | ------- | ---------------- | ----------- | --------- | ----------- | ---- | ------------- | -------- | ------------------ |  
  
That's a lot of columns! Although there are twenty-six total columns, we're only concerned with two. The two columns we care about retrieving data for are the __referrer__ and __target_path__ columns. The __referrer__ value describes where a request for download originated from. The __target_path__ value tells us what was downloaded and to what full path on the system.
  
The example query in Windows:
```PS
.\sqlite3.exe C:\Users\<USER>\AppData\Local\Google\Chrome\User Data\Default\History "SELECT referrer, target_path FROM downloads;"
```
The example query in Linux:
```sh
sqlite3 ~/.config/google-chrome/Default/History "SELECT referrer, target_path FROM downloads"
```

## Mozilla Firefox
Keeping with the same theme from the Google Chrome section, we'll now take a look at the Firefox files, their stored information, and provide a sample query to retrieve downloaded files and associated URLs.
  
### Firefox SQLite Files:
Unfortunately, Firefox SQLite files are not stored in the same location as Chrome, and also are not as intuitive to get information from. On Windows, we can find these files in the following directory:
```PS
C:\Users\<USER>\AppData\Roaming\Mozilla\Firefox\Profiles\<PROFILE>default
```
And on Linux, these files can be found in the following directory:
```sh
/home/<USER>/.mozilla/firefox/<PROFILE>.default
```
  
Now that we know where to find the files, we need to know what each file actually stores. Notice that unlike Chrome, Firefox includes the file extension type in the name:
- *content-prefs.sqlite*: stores site-specific content preferences.
- *cookies.sqlite*: stores cookie data.
- *formhistory.sqlite*: stores form input data.
- *permissions.sqlite*: stores site-specific permissions.
- *places.sqlite*: stores the history of visited sites.
- *webappstore.sqlite*: stores Document Object Model (DOM) storage data.  
*Note: This list was compiled from multiple sources. Not all files for every system are listed, and some files on this list may not be found on other systems.*
  
### Gathering Firefox Download History
So now that we know which Firefox SQLite file contains visited site history (__places.sqlite__), let's take a look at a couple of tables in that database file to understand how to structure a useful query. Inside the __places.sqlite__ database file, there are two tables, __moz_places__ and __moz_annos__, that store information needed to find downloaded files. Their table structures are shown below: 
  
__moz_places:__  
|---------|-----|-------|----------|-------------|--------|-------|-----------------|----------|  
| id (pk) | url | title | rev_host | visit_count | hidden | typed | favicon_id (fk) | frecency |  
|---------|-----|-------|----------|-------------|--------|-------|-----------------|----------|  
  
__moz_annos:__

| id (pk) | place_id (fk) | anno_attribute_id (fk) | mime_type | content | flags | expiration | type | dateAdded | lastModified |  
|---------|---------------|------------------------|-----------|---------|-------|------------|------|-----------|--------------|  
  
Each table has an __id__ field used as its primary key (pk). The two columns we care about retrieving information for are the __url__ column found in __moz_places__ and the __content__ column found in __moz_annos__. Luckily, __moz_annos__ contains a foreign key (fk) field, __place_id__, that references the primary key (pk) __id__ field found in __mod_places__. Using this relationship, we can use an effective JOIN statement to retrieve urls for which downloaded content has been recorded.  
The example query in Windows:
```PS
.\sqlite3.exe C:\Users\<USER>\AppData\Roaming\Mozilla\Firefox\Profiles\<PROFILE>default\places.sqlite "SELECT url, content FROM  moz_annos JOIN moz_places ON moz_places.id=moz_annos.place_id;"
```
The example query in Linux:
```sh
sqlite3 ~/.mozilla/firefox/<PROFILE>.default/places.sqlite "SELECT url, content FROM  moz_annos JOIN moz_places ON moz_places.id=moz_annos.place_id;"
```
  
Enjoy!  
  
### Sources:
http://www.forensicswiki.org/wiki/Google_Chrome  
https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Database   
https://forensicswiki.org/wiki/Mozilla_Firefox
