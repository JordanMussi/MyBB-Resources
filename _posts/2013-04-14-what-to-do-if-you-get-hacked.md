---
layout: post
category: Security
title: What to do if you get hacked
---

If your forum was hacked or you're curious to know what to do when you get hacked, please read this guide thoroughly and follow the instructions precisely. Please note we try to cover every single possible situation. In most cases you won't need to follow all the steps outlined, but it is still highly recommended to do so.

1. [Get in Control](#get_in_control)
	1. [Secure Your Computer](#secure_your_computer)
	2. [Secure Your Online Accounts](#secure_your_online_accounts)
	3. [Restore Your Admin Account](#restore_your_admin_accoun)
	4. [Restrict Access to the Forum](#restrict_access_to_the_forum)
	5. [Check the Logs](#check_the_logs)

2. [Secure the Forum](#secure_the_forum)
	1. [Replace All the Files](#replace_all_the_files)
	2. [Check CHMOD Permissions](#check_chmod_permissions)
	3. [Check New Users](#check_new_users)
	4. [Check Templates for Malicious Code](#check_templates_for_malicious_code)

3. [New Security Measures](#new_security_measures)
	1. [MyBB Updates](#mybb_updates)
	2. [Fine Tuning the Admin CP](#fine_tuning_the_admin_cp)
	3. [Administrator Accounts](#administrator_accounts)
	4. [Protect the inc Directory](#protect_the_inc_directory)
	5. [Change the Default Table Prefix](#change_the_default_table_prefix)
	6. [Disallow HTML in Posts](#disallow_html_in_posts)
	7. [Hide the Version Number](#hide_the_version_number)
	8. [Keep Plugins to a Minimum](#keep_plugins_to_a_minimum)
	9. [Secure Other Software](#secure_other_software)
	10. [Backup Regularly](#backup_regularly)
	
4. [Final Words](#final_words)

<a name="get_in_control"></a>
## Get in Control

<a name="secure_your_computer"></a>
### Secure Your Computer

First things first: secure your computer. It is possible that you downloaded something off the internet that infected your computer and allowed your forum to be hacked. Therefore it is not safe to do anything else from your computer until it is 100% clean. Even if you were to recover your forum right now it could be useless because the hacker could easily know what you're up to and destroy everything again. Securing your computer is a priority at the moment. If you have another computer that you can use which is not infected then get on it immediately. Forget about the infected computer, you can take care of it later. If you have other people working with you on your forum, asking them for help is also an alternative if you don't have another computer.

However if you don't have another computer lying around or other people to help you then you will have to take care of it right now. If you're using Windows we urge you to run tools like Spybot - Search & Destroy and Malwarebytes on your system right now to eliminate any malware or spyware. HiJackThis is also something worth considering, but it's a bit more advanced. Scanning your computer with your antivirus' own tools is also a good idea. If you don't already have one, Microsoft Security Essentials or AntiVir are excellent choices. A firewall like Comodo Internet Security or ZoneAlarm is recommended as well.

<a name="secure_your_online_accounts"></a>
### Secure Your Online Accounts

Once your computer is safe it is highly recommended to reset all of your passwords. Literally. Your admin account might be safe, but the attack could have originated from a hacker gaining access to your email account, thus being able to request a password reset. Having one weak password might compromise everything related to it, therefore it is very important to use secure passwords on all of your online accounts, specifically those related to your website (i.e. FTP, SSH, database, email). A good password can take a human hacker 580 million years to crack, or up to 59 years for a supercomputer. Isn't that the kind of security you want?

Using safe passwords, however, may not be enough. Of what use is a lengthy password consisting of all sorts of special characters if someone else gets their hands on it? Having that said, it is important to store your passwords in a secure place. Writing them on a piece of paper or a txt file is not good enough. We recommend that you use a password manager program such as KeePass or LastPass - both of which feature very powerful password generators as well.

Use two factor authentication wherever possible, especially if you use the same password in multiple places (which admittedly you shouldn't). Two factor authentication helps reducing your vulnerability to weak passwords, key logging, password reuse, phishing and brute force because it means your account can't be accessed even if your password is compromised. If you have a Google account consider enabling two factor authentication.

<a name="restore_your_admin_account"></a>
### Restore Your Admin Account

Having audited your computer and reset all your passwords, the first thing you want to do is to check if you have access to the Admin CP and the server. After all, you can't do anything if the hacker kicked you out of the administrator group or changed your permissions.

Before we even look into your account let's check if the default administrator group actually has permission to access the Admin CP. To do so run the following SQL query. Replace `mybb_` with your table prefix if you have a different one.

	UPDATE `mybb_usergroups` SET `cancp`= '1' WHERE `gid` = '4';

And to make sure your account is in the administrator group, run the following SQL query. Replace X with your uid and change `mybb_` to your table prefix if you have a different one.

	UPDATE `mybb_users` SET usergroup = '4' WHERE uid = 'X';

Next, you want to open the `inc/config.php` file in a text editor, find the code below within that file and make sure your uid is within quotes. If you have multiple administrators you can separate them with a comma. I would recommend sticking with your uid only for now though.

	$config['super_admins'] = '1'; 

Finally, to reset your password, in case the hacker changed it too, run the following SQL query. Remember to replace X with your uid and `mybb_` with your table prefix. The password will be set to test so you can change it afterwards in the User CP.

	UPDATE `mybb_users` SET `password` = '098f6bcd4621d373cade4e832627b4f6', `salt` = '' WHERE `uid` = 'X';

<a name="restrict_access_to_the_forum"></a>
### Restrict Access to the Forum

Great! Your admin account is fully restored and all your passwords were reset. Now you can actually start recovering your forum. Well, first we need to make sure the hacker is unable to access your website completely, should he try to do anything else.

First off you should disable users from accessing the front-end of the forum by closing it using the global switch in the settings. To do so, go to Admin CP > Configuration > Board Online / Offline > Board Closed > Yes. Adding a reason is irrelevant at this point, but if you want to add one try to sound like you're making some changes rather than let everyone know you've been hacked.

That will prevent the hacker from interacting with MyBB, but the website itself is still accessible. If he planted an exploit somewhere, he might be able to use it. So what you should do now is disallow all visitors from accessing your website except yourself. You can do that by placing the following code in your root .htaccess file. Replace `127.0.0.1` with your IP address.

	Order deny,allow
	Deny from all
	Allow from 127.0.0.1

If you find yourself unable to access your website during this process, it is possible that you have a dynamic IP, in which case you will have to repeat the above procedure whenever your IP changes. Typically this only happens when you reset your modem or router, but your ISP might have weird policies.

<a name="check_the_logs"></a>
### Check the Logs

To have a better understanding of what happened and what actions were performed we recommend that you take a little time to go through the logs and review them. To do that go to Admin CP > Tools & Maintenance > Logs > Administrator Log. You might be surprised at what you can find. Obviously logs can be deleted, so if the hacker deleted them you are out of luck.

But this is just MyBB's side. In addition to that we also recommend that you check your server's error logs. This contains various information such as what pages the hacker tried to access and how he managed to get administrative access. If you don't know where to find the server error logs or how to interpret them, please get in touch with your web host. They should be able to fill you in on the attack.

<a name="secure_the_forum"></a>
## Secure the Forum

<a name="replace_all_the_files"></a>
### Replace All the Files

To ensure that your MyBB installation is clean and no extra files have been added, you should delete all your files and upload a fresh copy of the latest version of MyBB. If you have a lot of plugins, images, language packs or custom modifications you should take note of them. You can simply create a backup and upload these things later on, but it is possible that a vulnerability lies within these files, so be careful. In the future consider using the Patches plugin to edit all of the core files. This makes it supremely easy to restore patches when you're upgrading or replacing files. You only really need to backup your `inc/config.php`. And even that should be double checked against the default structure of the file. Pay attention to your database details specifically, as well as your admin directory, super admins, etc.

Deleting the files and re-uploading a fresh copy of MyBB also has the benefit of updating to the latest code. If you were running an older version of MyBB - which is why most forums get hacked - you will now be running on code free of any known vulnerabilities. Please note that to complete the upgrade, you need to point your browser to `install/upgrade.php` and run the upgrade script. You only need to do this if you were not running on the latest version.

<a name="check_chmod_permissions"></a>
### Check CHMOD Permissions

If certain files or folders have unnecessary permissions you may be up against a security risk. They should only have the permissions required by MyBB to run. There is no recommended set of permissions specifically. It varies from server to server, because they are configured differently. For more information read our docs on CHMOD permissions or contact your web host for their recommendations. 

<a name="check_new_users"></a>
### Check New Users

Right now no one can access your website or the MyBB backend, but you should quickly head over to the Admin CP and check all users that joined around the time of the attack. It is possible that the hacker created a new admin account and set the display usergroup to a normal user. To see all the users with admin permissions and manage them go to Admin CP > Users & Groups > Admin Permissions. It's also worth checking changes to admin permissions and other things.

<a name="check_templates_for_malicious_code"></a>
### Check Templates for Malicious Code

Lastly you should quickly go through your templates, since they are a way to inject malicious JavaScript code into your forum. If possible, delete your current theme and install a fresh copy of it. If you hired a designer to create a custom theme and don't have a copy of it at the moment, contact them for one.

However, if you made a lot of custom modifications and don't have a backup, then you'll have to go through the process of looking through the templates and look for suspicious code. Common templates for malicious code insertion are the header, footer, and headerinclude, as they are loaded globally. It is also recommended to check the index and forumdisplay templates. An easy way to do this is to click on the Options button next to a template name and select Diff Report. The code highlighted in red is what differs from the default template, and also what you should be looking at. Look specifically for code within a `<script>` tag. Not all of it is malicious, but this filters down the options. If you don't know what to look for, don't be afraid to ask for help.

<a name="new_security_measures"></a>
## New Security Measures

<a name="mybb_updates"></a>
### MyBB Updates

Your forum is safe, but you shouldn't stop here. You should take new security measures immediately to prevent this from happening again. One thing we cannot stress enough is to always have your MyBB installation up to date. If you are running an older version, you are open to security vulnerabilities that were already fixed. MyBB takes security very seriously. Whenever a security vulnerability is reported it is patched very quickly and a new release is sent out. You should never skip upgrading, as you will be opening yourself to a security risk that hackers can easily take advantage of.

<a name="fine_tuning_the_admin_cp"></a>
### Fine Tuning the Admin CP

The Admin CP is the most powerful tool in MyBB. If anyone gains access to it, they can easily deface your forum and get complete control over it. It is ultimately important to guarantee that only you can access it. For starters you should rename your Admin CP directory and hide all links to it. Once you have done that it is a good idea to install Admin CP Honeypot. This will take your previous Admin CP location and install a fake Admin CP, which will record the IP of anyone who tries to login to it and email you a small report.

Now your real Admin CP directory should look something like `Svt06wbowXgMVvFmkFaz` (which you should bookmark or take note of) and the fake Admin CP will be located at `admin`, which will record the details of anyone who tries to access it. To finalize it, you should password protect your real Admin CP with HTTP Basic Auth. Additionally you can add an extra PIN number in the Admin CP login form, but having to get through all of these steps might be a little troublesome if you just want to do some quick edits.

<a name="administrator_accounts"></a>
### Administrator Accounts

The Admin CP is indeed the most powerful tool in MyBB. But no matter how hard you try to secure it, if people other than yourself have access to it then it is a risk. You should only allow access to the Admin CP to people you know really well and trust. Do not randomly allow a user of your forum to access it, even if he promises you to install a bunch of cool plugins or themes. Administrators should be reviewed rigorously and thoroughly before being selected. Be very careful in who you trust access to the Admin CP to. If you trust no one, then perhaps you're better off as an administrator. Remember: it is the most powerful tool in MyBB.

For that reason permissions should be limited as much as possible. Distribute tasks between all the accounts. Discuss this with your admins and decide who should take care of what. For example, one of your administrators may know HTML and could therefore be in charge of making changes to templates and keeping the code clean. The other administrators may not know HTML, so why should they have access to the Templates & Style module? Similarly, if the HTML-guy doesn't like managing users and group permissions, then he definitely doesn't need access to that module. You can configure all of this in Admin CP > Users & Groups > Admin Permissions. Your administrators will be listed there, and you can specify everything they can and cannot access. Be rigorous and only allow access to the parts your administrators really need. As an example, you should probably disable all administrators other than yourself from accessing the database backups section. A backup of your database essentially contains all the information in your forum, which can be quite dangerous in the wrong hands. Provided that you have a proper backup solution (covered later on) there is no need for them to be able to create backups.

<a name="protect_the_inc_directory"></a>
### Protect the `inc` Directory

The `inc` directory in your MyBB installation is something that should not be accessible to the end user at all. It contains sensitive information such as your database details (`inc/config.php`). And even though it is almost impossible for hackers to access that data, it's always a good idea to make things extra difficult to access. And the `inc` directory certainly doesn't need to be publicly available. You should therefore protect it by completely disallowing its access, as described on MyBB Security. 

<a name="change_the_default_table_prefix"></a>
### Change the Default Table Prefix

Changing your table prefix can prove to be helpful in certain cases. If a hacker manages to run an SQL query, he can easily destroy your forum completely. But if for some reason he doesn't know what your table prefix is (and therefore doesn't have a table name to query) it would certainly slow him down. Having that said, consider changing your table prefix.

<a name="disallow_html_in_posts"></a>
### Disallow HTML in Posts

Allowing HTML to be used in posts is a terrible, terrible idea. That is why MyBB does not allow it by default. Unless you are absolutely certain that you want to use it (in which case you should install HTML Purifier) it should be disabled on all forums. To do this quickly, run the following SQL query. Replace `mybb_` with your own table prefix.

	UPDATE `mybb_forums` SET `allowhtml` = '0';

Afterwards you should go to Admin CP > Tools & Maintenance > Cache Manager > forums > Rebuild Cache to make sure this change is cached and is applied immediately.

<a name="hide_the_version_number"></a>
### Hide the Version Number

Displaying which MyBB version you're running is essentially the same as saying "hey, I'm running this specific version, which contains these specific vulnerabilities". It's almost an open invitation to hackers. If you're running on the latest version, it's probably nothing to worry about, but there is simply no point in displaying it. To hide it go to Admin CP > Configuration > Settings > General Configuration > Show Version Numbers > Off.

<a name="keep_plugins_to_a_minimum"></a>
### Keep Plugins to a Minimum

The more plugins you have installed, the more code can hackers exploit. Most plugins are fairly secure, but if one of them has a vulnerability, hackers can take advantage of it to get access to your forum. And for that simple reason it is highly recommended to keep the number of plugins to a minimum and only install those that you really need. It's also worth considering the popularity and the author of the plugin. Having that said, to improve your forum's security, we still recommend having a look at our list of security plugins.

<a name="secure_other_software"></a>
### Secure Other Software

Similarly, if you have other software like WordPress or Drupal installed on your server then you are facing a potential security risk because hackers have even more code to exploit. If they manage to hack your blog or image gallery, it's very possible that they get access to MyBB as well. This doesn't mean you should refrain from installing other software or that WordPress and the like are bad, you simply need to invest a little extra time securing them. Obviously, we can't help you with that, but things like updating the software and using strong passwords are a few key aspects you should keep in mind. For more information you should look up articles on securing the other software you have installed.

<a name="backup_regularly"></a>
### Backup Regularly

None of this stuff matters if a hacker gets access to your database and drops all tables. He probably shouldn't have gotten that far provided that you have secure passwords and all, but if he did and you don't have a backup, you are essentially screwed. If you don't have any users, posts or threads your forum is empty and you will have to start over. That is why making backups of all your files and your database is extremely important. If the task is enabled in Admin CP > Tools & Maintenance > Task Manager, MyBB will backup your forum's database to the `backups` directory. In most cases this is not ideal, as hackers can easily destroy the backups. For a more sophisticated and complete backup solution, refer to the backup section in our list of security tutorials.

<a name="final_words"></a>
## Final Words

Even if you had the world's most advanced security system, if the person in control of it does something wrong, then all work is in vain. Always keep an eye on what you install and run on your computer, make sure your passwords are safe, ensure your forum is properly patched, and never try to do anything that would expose any "flaws" to others. If you and your fellow administrators do your job properly, your website will certainly be much safer. And in order to always be prepared, make backups, frequently, and you will thank yourself in the long run.

We hope you never have to go through the "recovery" steps in this guide, what you should follow are the preparation steps - if you are prepared, it makes you a much less viable target.
