---
layout: post
title: Automatic Password Trigger for iTerm2
tags: [Tech, How-to]
author: zigsphere
excerpt_separator: <!--more-->
---

Working in tech requires me to preety much live in the command line. Doing this tends to prompt me for my password quite often, since much of my work requires to run things as sudo either on my Mac on Linux systems accessed with SSH. I'd have to say, typing in my passowrd over and over and over gets pretty annoying, especially if my password takes some time to type (it does).

I'd like to share something with you all that I've been using for about 5ish years now. First of all, if you don't use [iTerm2](https://iterm2.com/), I highly recommend it. If you don't want to, well I guess this post wouldn't make much sense to read then :).

What I discovered is something called [triggers](https://iterm2.com/documentation-triggers.html), which is a built-in feature within iTerm2. A trigger is an action that is performed when text matching some regular expression is received in a terminal session. Although you can do many things with triggers, I will be discussing how to leverage triggers to automatiically enter in your password each time iTerm prompts you for it.

<center><img src="https://github.com/zigsphere/zigsphere.github.io/blob/main/assets/images/triggers/trigger.png" width="800" alt="Trigger"></center>

## Instructions

1. Open iTerm
2. In the taskbar, go to **iTerm** -> **Preferences..**
3. Click the **Profiles** tab
4. Click your profile in the left pane. If you dont have one, you can keep the default selected or create a new one
5. Click the **Advanced** tab in the right pane
6. Under **Triggers**, click **Edit**
7. Click the **+** in the lower left corner of the window to create a new trigger
8. In the **Regular Expression** column, enter `^(\[sudo\]|(P|p)assword)`. This will trigger if the word password or sudo is shown in iTerm
9. In the **Action** column dropdown, select **Open Password Manager**
10. For the **Parameters** column, we will use this in a minute, but for now, this will contain no items. Leave this blank for now
11. Check both the **Instant** and **Enabled** checkboxes
12. Click **Close**
12. Now, within the iTerm window, enter `echo password` then press **enter**
13. A window will be displayed. Press the **+** in the lower left corner and enter an account name. This can be anything, but really just a description. I just use my name
14. For the username, enter your name or username. This field will not be used either, so enter what you want. 
15. For the password field, this is where you want to enter your computer / server password(s). You can add as many accounts here if you want if you have different passwords for different systems
16. Click **Close**
17. Go back and edit the triggers in iTerm.
18. Now, within the **parameters** column, select the account you added in step #13.
19. Click **Close**
20. Now, if you enter `sudo echo hi`, for example, in the iTerm window, when your password is prompted, a window will be displayed. You can simply just press enter to select the account you created in the triggers.


## Questions
1. Is this safe?
    - Yes. After your computer is rebooted or iTerm is restarted, when a password is prompted for the first time, you do have to enter your computer password to unlock your keychain.
2. I like triggers. What else can I use it for?
    - There is quite a few articles about this such as https://github.com/scottdware/iterm2-triggers. There are many cool things you can do with this.