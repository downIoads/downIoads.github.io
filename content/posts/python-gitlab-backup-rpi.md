---
title: "Automated self-hosted GitLab backups to Dropbox using a Python script on Raspberry Pi"
date: 2024-01-02T18:00:23+02:00
description: Simple Python script that updates your Gitlab config and data to your dropbox.
draft: false
tags: [python, programming, gitlab, crontab, rpi]
---

## Introduction

I prefer self-hosted GitLab over uploading my code to GitHub. The only disadvantage is that I need to take care of backups myself in case a hard drive dies or gets stolen. Therefore, I wrote a simple Python script that runs on my Raspberry Pi twice a week, which backs up my GitLab config and data and uploads them to my Dropbox. Specifically, I am using the free community edition of GitLab on my RPI. GitLab backups consist of two parts: There is the config (which includes the keys needed to decrypt the data backup) backup and the data backup. Technically, it is recommended to store these two backups separately, but I do not have any sensitive data in my GitLab projects, I just want to prevent data loss. This is why my script backs up both config and data backup into the same Dropbox account. In order for this script to work, you must set up a Dropbox app by using the [Dropbox developer site](https://www.dropbox.com/developers). I would recommend limiting the scope of the app into a single folder. This way, the token you put into the Python script can only read and write data to this folder and the rest of the Dropbox is untouched. Note that the token you can generate on the website where you adjust your Dropbox App settings is only temporary (a few hours) and will not be used by this script. Instead, read through [this reply](https://www.dropboxforum.com/t5/Dropbox-API-Support-Feedback/Issue-in-generating-access-token/m-p/592921/highlight/true#M27586) to learn how to use this token to get a permanent refresh token. The script itself then uses this refresh token to get a new authorization token each time it runs. The authorization token you get will always be valid for a few hours (which is enough for the script) and then become invalid. To sum it up: There is a "refresh token" which is used to request new "authorization tokens" and the authorization tokens are what you need to be able to access your Dropbox. I had to read lots of threads and documentation until I found the working answer that I linked above to figure this out. 

Another note for setting up GitLab on your Raspberry Pi: If after installation you get a Nginx default page instead of your GitLab login website, it means that the port GitLab tries to use is already taken (e.g. by pihole). In the GitLab config, you have to assign a different, unused port to make GitLab work.

The script below assumes that GitLab Community edition runs on a Raspberry Pi with default OS and default GitLab backup locations.

## Python
```py

# PREREQ: sudo pip3 install dropbox
# USAGE: sudo python3 <scriptname.py>
import ast # string to dictionary
import dropbox
import os
import subprocess

# gitlab folder that stores config backup (required to decrypt data backup) and data backup
# assumes default locations for linux installation of gitlab community edition
LOCAL_GITLAB_CONFIG_PATH    = "/etc/gitlab/config_backup"
LOCAL_GITLAB_DATA_PATH      = "/var/opt/gitlab/backups"

DROPBOX_APP_KEY     = "<you-have-to-edit-this-field>"
DROPBOX_APP_SECRET  = "<you-have-to-edit-this-field>"
DROPBOX_REFRESH_TOKEN = "<you-have-to-edit-this-field>"
DROPBOX_DESTINATION_CONFIG  = "/Config"
DROPBOX_DESTINATION_DATA    = "/Data"

# access token only valid 4 hours, but refresh token always valid and used to get new access token.
# this function uses refresh token to get new valid access token
# read this to know to get refresh token in the first place: https://www.dropboxforum.com/t5/Dropbox-API-Support-Feedback/Issue-in-generating-access-token/m-p/592921/highlight/true#M27586
def getNewAccessToken():
    refreshCommand = "curl https://api.dropbox.com/oauth2/token -d grant_type=refresh_token -d refresh_token=" + DROPBOX_REFRESH_TOKEN + " -u " + DROPBOX_APP_KEY + ":" + DROPBOX_APP_SECRET
    result = subprocess.run(refreshCommand, shell=True, check=True, text=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    response = result.stdout # this is what you get back, contains the data you want
    response_dict = ast.literal_eval(response)
    return response_dict["access_token"]

# e.g. takes "gitlab_config_1702820194_2023_12_17.tar" and returns 1702820194 as int
# getTimestampFromConfigFilename("gitlab_config_1702820194_2023_12_17.tar")
def getTimestampFromConfigFilename(filename):
    # files that are not gitlab config backups are ignored
    if not "gitlab_config_" in filename:
        return -1
    return int(filename[filename.find("gitlab_config_") + 14:filename.find("_", filename.find("gitlab_config_") + 14)])


# takes path to dir and returns filename of newest config backup
def getNewestConfigBackupFilename():
    # get list of all files in gitlab config backup dir
    configBackupList = [f for f in os.listdir(LOCAL_GITLAB_CONFIG_PATH) if os.path.isfile(os.path.join(LOCAL_GITLAB_CONFIG_PATH, f))]

    # check each filename and remember index of the name with highest timestamp (newest backup)
    newestBackupIndex = -1
    newestBackupHighestTimestamp = -1
    for index, filename in enumerate(configBackupList):
        if getTimestampFromConfigFilename(filename) > newestBackupHighestTimestamp:
            newestBackupHighestTimestamp = getTimestampFromConfigFilename(filename)
            newestBackupIndex = index
    return configBackupList[newestBackupIndex]


# e.g. takes "1702758583_2023_12_16_16.5.1_gitlab_backup.tar" and returns 1702758583 as int
# getTimestampFromDataFilename("1702758583_2023_12_16_16.5.1_gitlab_backup.tar")
def getTimestampFromDataFilename(filename):
    # files that are not gitlab data backups are ignored
    if not "_gitlab_backup" in filename:
        return -1
    return int(filename[:filename.find("_")])


# takes path to dir and returns filename of newest data backup
def getNewestDataBackupFilename():
    # get list of all files in gitlab data backup dir
    dataBackupList = [f for f in os.listdir(LOCAL_GITLAB_DATA_PATH) if os.path.isfile(os.path.join(LOCAL_GITLAB_DATA_PATH, f))]

    # check each filename and remember index of the name with highest timestamp (newest backup)
    newestBackupIndex = -1
    newestBackupHighestTimestamp = -1
    for index, filename in enumerate(dataBackupList):
        if getTimestampFromDataFilename(filename) > newestBackupHighestTimestamp:
            newestBackupHighestTimestamp = getTimestampFromDataFilename(filename)
            newestBackupIndex = index
    return dataBackupList[newestBackupIndex]


def uploadToDropbox(accessToken, localFilePath, remoteFolder):
    print("Will use this access token:", accessToken)
    dbx = dropbox.Dropbox(accessToken)
    with open(localFilePath, "rb") as f:
        dbx.files_upload(f.read(), remoteFolder)
        

def main():
    # get new access token (valid 240 min)
    DROPBOX_ACCESS_TOKEN = getNewAccessToken()    

    # ensure local gitlab paths exist
    if (not os.path.exists(LOCAL_GITLAB_CONFIG_PATH)) or (not os.path.exists(LOCAL_GITLAB_DATA_PATH)):
        print("Local gitlabs paths could not be found. Terminating..")
        return

    # create gitlab backups using subprocess (must be run as sudo or will fail)
    # for this reason you must also automate this in "sudo crontab -e" (sudo is important)
    commandConfigBackup = "sudo gitlab-ctl backup-etc"
    commandDataBackup   = "sudo gitlab-backup create"

    #       make new backups  (might take few minutes but program will wait)
    subprocess.run(commandConfigBackup, shell=True)
    subprocess.run(commandDataBackup, shell=True)


    # get filename of newest gitlab config backup
    newestConfigFilename = getNewestConfigBackupFilename()
    newestConfigFullPath = LOCAL_GITLAB_CONFIG_PATH + "/" + newestConfigFilename
    print("Newest config backup:", newestConfigFullPath)
    
    # get filename of newest gitlab data backup
    newestDataFilename = getNewestDataBackupFilename()
    newestDataFullPath = LOCAL_GITLAB_DATA_PATH + "/" + newestDataFilename
    print("Newest data backup:", newestDataFullPath)

    # upload data (if it already exists nothing happens)
    uploadToDropbox(DROPBOX_ACCESS_TOKEN, newestConfigFullPath , DROPBOX_DESTINATION_CONFIG + "/" + newestConfigFilename)
    uploadToDropbox(DROPBOX_ACCESS_TOKEN, newestDataFullPath   , DROPBOX_DESTINATION_DATA   + "/" + newestDataFilename)
    
    print("Successfully backed up Gitlab Config and Data.")


main()

```

## Features

Quickly reading over the code, you probably have noticed that the script determines the newest backups and only uploads them to your Dropbox. GitLab automatically puts the current timestamp in the filenames of your backups (note that there is a slightly different naming scheme for the config and the data backup filenames) which is what this script makes use of to determine the currently newest backup. Note: If your Dropbox does not have a lot of storage, you can earn like 18 GB of free permanent storage by making new accounts with your referral link. Since they only grant the storage if the desktop Dropbox app is installed by your friends, you might get the idea that you can set up a VM which reverts each time to simplify this process. But then you might notice that Dropbox will refuse to send out invitation e-mails after you do it three times and the fix might be to use a different browser (who knows, maybe only the browser agent is being checked as the only counter measurement of this strategy). This is all just guessing, though.

## Run this script twice per week using crontab

The script needs "sudo" to trigger the GitLab backups. For this reason, you must use "sudo crontab -e" instead of just "crontab -e". This ensures, that you have sudo rights when crontab executes the script. If you want to run the script twice per week, you could add the following line to your crontab (adjust paths as needed):
```bash
"0 4 * * 3,6 /usr/bin/python3 /home/pi/gitlab-backup.py"
```
This means the backup is triggered at 4:00 AM of every Wednesday (3) and Saturday (6). Note that the output of the script will be mailed to you. Run "sudo mail" to read the output of sudo crontab tasks (e.g. for this script) and use "mail" (without sudo) for the script triggered by your non-sudo crontab.

# Conclusion

The script is very useful for me, I wrote it once, and it regularly runs ever since. In the process of writing the script, I learned about the Dropbox API and how crontab has separated schedules for sudo and non-sudo. You could improve upon this script by deleting backups older than x, but I did not need this feature as my GitLab backups are not very large, and I have enough Dropbox storage.

