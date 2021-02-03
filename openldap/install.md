#### Install and configure openLdap

1) ##### Install the openLDAP packaages
```
 apt install slapd ldap-utils
```
You will be asked to set an admin password.  This is for the default ldap admin


2) ##### Reconfigure openLDAP
```
dpkg-reconfigure slapd
```
For the first option select **No** to write a new configuration.  For the DNS domain name choose whatever feels appropriate.  The organization name is arbitrary.  The admin password is the same thing you used in step 1.  Use the default **MDB** database and just select the defaults for the next two options.

3) ##### Create a config admin.

First we need create an ecrypted password to use in the ldif config file
```
slappasswd -f {ssha}
New Password:
Re-enter new password:
{SSHA}encryptedstring
```
Replace the oldRootPw in files/config.ldif with the string output from your command.  The password you type in is the one you will use for the remaining configure steps.

4) ##### Apply the password
As root run
```
ldapadd -Y EXTERNAL -H ldapi:/// -f files/config.ldif
```

5) ##### Enable memberOf module
Load the module with following command
```
ldapadd -W -D cn=admin,cn=config -f files/memberOfModule.ldif
```
Next configure the module with the following command
```
ldapadd -W -D cn=admin,cn=config -f files/memberOfconfig.ldif
```

6) ##### Enable the refint (reference integrity) module
Load the module with the following command
```
ldapmodify -W -D cn=admin,cn=config -f files/refintModule.ldif
```
Next configure the module with the following command
```
ldapadd -W -D cn=admin,cn=config -f files/refintConfig.ldif
```