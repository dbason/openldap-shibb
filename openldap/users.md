#### Create users and groups
> The password for these instructions will be the password for the admin account you created during installation of openLDAP.
> In all commands make sure the domain path matches what you set when you installed openLDAP

1) ##### Create OUs for users and groups
Modify files/ous.ldif and make sure the dcs in the path matches the domain name you set when you configured openLDAP.  Then create the OUs with the command
```
ldapadd -W -D cn=admin,dc=example,dc=com -f files/ous.ldif
```

2) ##### Create users
Modify files/users.ldif and make sure the domain matches yours.
```
ldapadd -W -D cn=admin,dc=example,dc=com -f files/users.ldif
```

3) ##### Create groups
Modify files/groups.ldif and make sure the domain matches yours.
```
ldapadd -W -D cn=admin,dc=example,dc=com -f files/groups.ldif
```

4) ##### Verify group membership
At this point 2 users (user1, user2) should have been created and added to groups group1 and group2 respectively.  The OUs these live in are `ou=users,dc=example,dc=com` and `ou=groups,dc=example,dc=com`.  Users are referenced using the `uid` field.

Next we need to verify the refint module has set the `memberOf` field on the users correctly.  Run the following command:
```
ldapsearch -x -D cn=admin,dc=example,dc=com -W -b "ou=users,dc=example,dc=com" memberOf
```

This should result in the following users showing up in the results
```
# user1, users, example.com
dn: uid=user1,ou=users,dc=example,dc=com
memberOf: cn=group1,ou=groups,dc=example,dc=com

# user2, users, example.com
dn: uid=user2,ou=users,dc=example,dc=com
memberOf: cn=group2,ou=groups,dc=example,dc=com
```

5) ##### Finally set the passwords on the users
These can be set with the following commands
```
ldappasswd -H ldapi:/// -x -D cn=admin,dc=example,dc=com -W -S uid=user1,ou=users,dc=example,dc=com
```
```
ldappasswd -H ldapi:/// -x -D cn=admin,dc=example,dc=com -W -S uid=user2,ou=users,dc=example,dc=com
```