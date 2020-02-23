---
layout: post
title: Mounting Google Cloud Storage FUSE with fstab
tags: [Tech, How-to]
author: zigsphere
excerpt_separator: <!--more-->
---

I'm extremely new to GCP, but I explicitly joined it because I wanted a storage solution where I could mount directly to my machine and just work normally without having to manually drag and drop things to ensure they are backed up. Not only is it easy, it's really cheap. 
To ensure my day-to-day is smooth, I ensure that, upon booting up my machine, my GCP bucket is mounted automatically. In order to do this, there are a few things I had to tinker with in order to make it happen. 

The configuration file /etc/fstab contains the necessary information to automate the process of mounting partitions. For mounting GCP storage in this case, I will be using fstab. Since the fstab configuration is read by the root user, this is the part where I ran into a few issues because root is not the local user when signing into to the machine, it's me.

##### Dependencies
 - Before we begin, please have a look at https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/docs/installing.md for installing the gcsfuse application installed.
 - Secondly, look at the main article for configuring gcsfuse https://cloud.google.com/storage/docs/gcs-fuse. You will need to go through all of the steps, including 2a. A service account will be needed for the root user to mount under your Google account. 
 - Installation of the Google SDK: https://cloud.google.com/sdk/docs/

##### Instructions
1. After installing the gcsfuse application, create a service account as noted in https://cloud.google.com/storage/docs/gcs-fuse in 2a. When doing this, ensure that the project you select is the same project as the bucket. If you do not select the same project, the json file generated will NOT work. Download the json file.
2. There may be a workaround for this step, but this way worked for me. Change to the root user `sudo su` and then run `gcloud auth application-default login`. When doing this, it will provide a URL. Copy the URL and paste into the web browser. Select the Google account that the bucket is associated with.
3. Credentials will be written to `/root/.config/gcloud/application_default_credentials.json`, but those are YOUR credentials. Really, we want to use the service account credentials here for security reasons. Replace the contents of `application_default_credentials.json` with the contents of the json file downloaded in #1.
4. Switch back to your user account by running `exit`.
5. Make a directory on your machine that the bucket will be mounting to. In this example, let's use /media/bucket:
`sudo mkdir /media/bucket && sudo chown $WHOAMI:$WHOAMI /media/bucket`. This will ensure that the directory created will be owner by your user account.
6. Edit `/etc/fstab` and add the following line, replacing attributes to match your own:
`BUCKETNAME /media/bucket gcsfuse rw,gid=1000,uid=1000,noauto,user,allow_other,_netdev`. Replace `BUCKETNAME` with your bucket name and replace `1000` with your uid and gid. You can determine this by running `id $WHOAMI`
7. Edit `/etc/fuse.conf` and uncomment the line `user_allow_other`.
8. Finally, test to ensure that the bucket mounts correctly: `sudo mount /media/bucket` (Replacing /media/bucket with the directory you created in #5).