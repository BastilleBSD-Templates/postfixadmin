# postfixadmin
Bastille Template to create a Postfixadmin jail to manage Postfix instances

 STATUS: Testing


Once the packages are installed you jail needs some configuration to function.  First create
the PostfixAdmin database:

	# mysql_secure_installation


	# mysql -u root -p
	(Enter MySQL root password)
	> CREATE DATABASE postfix;
	> CREATE USER 'postfix'@'localhost' IDENTIFIED BY 'postfix_sql_password';
	> GRANT ALL PRIVILEGES ON `postfix`.* TO 'postfix'@'localhost';
	> FLUSH PRIVILEGES;
	> QUIT;


Create and edit the /usr/local/www/postfixadmin/config.local.php file:

	<?php
	$CONF['configured'] = true;
	$CONF['setup_password'] = 'changeme';
	$CONF['database_type'] = 'mysqli';
	$CONF['database_host'] = 'localhost';
	$CONF['database_user'] = 'postfix';
	$CONF['database_password'] = 'postfix_sql_password';
	$CONF['database_name'] = 'postfix';
	$CONF['admin_email'] = 'postmaster@domain.tld';
	$CONF['smtp_server'] = 'localhost';
	$CONF['smtp_port'] = '25';
	$CONF['encrypt'] = 'md5crypt';
	$CONF['dovecotpw'] = "/usr/sbin/doveadm pw";
	$CONF['page_size'] = '50';
	$CONF['default_aliases'] = array (
	    'abuse' => 'abuse@domain.tld',
	    'hostmaster' => 'hostmaster@domain.tld',
	    'postmaster' => 'postmaster@domain.tld',
	    'webmaster' => 'webmaster@domain.tld'
	);
	$CONF['domain_path'] = 'NO';
	$CONF['domain_in_mailbox'] = 'YES';
	$CONF['aliases'] = '50';
	$CONF['mailboxes'] = '50';
	$CONF['maxquota'] = '102400';
	$CONF['domain_quota_default'] = '1024000';
	$CONF['quota'] = 'YES';
	$CONF['domain_quota'] = 'YES';
	$CONF['quota_multiplier'] = '1048576';
	$CONF['transport'] = 'NO';
	$CONF['vacation'] = 'YES';
	$CONF['vacation_domain'] = 'autoreply.domain.tld';
	$CONF['vacation_control'] ='YES';
	$CONF['vacation_control_admin'] = 'YES';
	$CONF['alias_control'] = 'YES';
	$CONF['alias_control_admin'] = 'YES';
	$CONF['show_header_text'] = 'NO';
	$CONF['header_text'] = ':: Postfix Admin ::';
	$CONF['show_footer_text'] = 'YES';
	$CONF['footer_text'] = 'Return to Company';
	$CONF['footer_link'] = 'http://www.domain.tld/';
	$CONF['welcome_text'] = <<<EOM
	Hello,

	Welcome to your new email account!

	For questions or comments regarding your mail account,
	please feel free to send an email to support@domain.tld.
	Likewise, any other inquiries regarding the Company Name 
	or our affiliates can be sent to the same address.

	Also, don't forget to check your mail settings via Maia-
	Mailguard located at https://www.domain.tld/maia/.
	Simply log into your account using your email address
	and password. That's it! From Maia-Mailguard, you can
	adjust your spam, virus, malware, whitelists, blacklists,
	etc... This will put you in full control of your email so
	you never miss anything important.

	Thank you for using Company Name and enjoy
	your new email account!
	
	Regards,
	Company Name Staff
	support@domain.tld
	EOM;
	$CONF['emailcheck_resolve_domain']='NO';
	$CONF['mailbox_postdeletion_script'] = '/usr/local/bin/sudo -u vscan /root/bin/postfixadmin-mailbox-postdeletion.sh';
	$CONF['domain_postdeletion_script'] = '/usr/local/bin/sudo -u vscan /root/bin/postfixadmin-domain-postdeletion.sh';
	?>


Create and edit /usr/local/etc/apache24/Includes/postfixadmin.conf file:

	Alias /postfixadmin/ "/usr/local/www/postfixadmin/public/"
	<Directory "/usr/local/www/postfixadmin/public/">
	   Options Indexes
	   AllowOverride AuthConfig
	   Require all granted
	</Directory>
	Reload Apache configuration:

	# apachectl configtest
	# apachectl graceful


Generate the setup password:

	Visit https://www.domain.tld/postfixadmin/setup.php. The first visit will take some 
	time to set up the database and do other checks, so PLEASE BE PATIENT!

	Generate your password hash after the page finishes loading by scrolling to the bottom 
	and entering your “setup password” twice and then clicking the “Generate password hash” 
	button. You will then be presented with a nice, long hash that you will place in the 
	“config.local.php” file we edited above. So, copy that password and edit the config file 
	one last time! You can also leave the setup page open (recommended) to set your admin 
	user up after the hash is in place.

Edit the /usr/local/www/postfixadmin/config.local.php file and put in your password hash for 
the “setup_password” setting:

	...
	$CONF['setup_password'] = 'a1ccbec61b5c1de578ba394mf93bfaa3:6c6fb18e117be3d872bf4f3f74jfh69184c79f48';
	...
Set your admin user (email address) and password for Postfixadmin:

	Going back to the Postfixadmin setup page (hopefully still open), use the password you 
	used to create the setup hash, an email address and a password (should be different from 
	the password used to create the hash) to create your admin user. Click the “Add Admin” 
	button, and you’re all set!

Now, let’s move on to finish the Postfixadmin setup before we begin adding actual email accounts 
or anything…

Install sudo:
	NOTE: This is only needed if you plan on using the Postfixadmin post-deletion scripts. If 
	you don’t plan on using these, feel free to skip to the “Create Vacation user” section 
	below and continue from there…

	# portmaster -dG security/sudo

Create and edit the /usr/local/etc/sudoers.d/www file:
	NOTE: The following line in the sudoers file allows the “www” user to execute the 
	post-deletion scripts as the “vscan” user.

	#Allow "www" to execute pfa scripts as "vscan" user.
	www ALL=(vscan)  NOPASSWD: /root/bin/postfixadmin-mailbox-postdeletion.sh, \
	                 /root/bin/postfixadmin-domain-postdeletion.sh

Create post-deletion directories and copy scripts:

	# mkdir /root/bin
	# mkdir -p /usr/local/virtual/deleted/{mailboxes,domains}
	# chown -R vscan:vscan /usr/local/virtual/deleted
	# chmod -R 0700 /usr/local/virtual/deleted
	# cp /usr/local/share/postfixadmin/ADDITIONS/postfixadmin-*deletion.sh /root/bin/
	# chmod +x /root/bin/postfixadmin*

Edit the /root/bin/postfixadmin-domain-postdeletion.sh file:

	# Change this to where you keep your virtual mail users' maildirs.
	basedir=/usr/local/virtual

	# Change this to where you would like deleted maildirs to reside.
	trashbase=/usr/local/virtual/deleted/domains

Edit the /root/bin/postfixadmin-mailbox-postdeletion.sh file:

	# Change this to where you keep your virtual mail users' maildirs.
	basedir=/usr/local/virtual

	# Change this to where you would like deleted maildirs to reside.
	trashbase=/usr/local/virtual/deleted/mailboxes

	Create Vacation user and group accounts:

	# pw groupadd vacation
	# pw useradd vacation -c Virtual\ Vacation -d /nonexistent -g vacation -s /sbin/nologin

Create, populate and secure vacation directory:

	# mkdir /var/spool/vacation
	# cp /usr/local/share/postfixadmin/VIRTUAL_VACATION/vacation.pl /var/spool/vacation/
	# chown -R vacation:vacation /var/spool/vacation/
	# chmod 700 /var/spool/vacation/
	# chmod 750 /var/spool/vacation/vacation.pl
	# touch /var/log/vacation.log /var/log/vacation-debug.log
	# chown vacation:vacation /var/log/vacation*

Edit /var/spool/vacation/vacation.pl script:
Find and edit the RED TEXT.

	#!/usr/bin/env perl
	...
	our $db_type = 'mysqli';
	our $db_host = 'localhost';
	our $db_username = 'postfix';
	our $db_password = 'postfix_sql_password';
	our $db_name = 'postfix';
	our $vacation_domain = 'autoreply.domain.tld';
	...
	our $smtp_ssl = '';
	...
	our $logfile = "/var/log/vacation.log"; # specify a file name here for example: vacation.log
	our $log_level = 0;
	our $log_to_file = 1;



Now you need to go edit your postfix servers to allow postfixadmin to attach and manage them.


