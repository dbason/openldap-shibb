### Configure and run Shibboleth
This will configure and run a dockerized version of shibboleth.

1) #### Install docker
```
curl https://releases.rancher.com/install-docker/19.03.sh | sh
```

2) #### Run the Shibboleth set up
```
docker run -it -v $(pwd):/ext-mount --rm unicon/shibboleth-idp init-idp.sh
```
Answer the following question in the interactive script
```
Hostname: The DNS/hosts record entry for the server
SAML EntityID: The default is fine
Attribute Scope: The domain name of the openLDAP install
Backchannel PKCS12 Password: Arbitrary password
Cookie Encryption Key Password: Arbitrary password
```
Then rename the created directory
```
mv customized-shibboleth-idp shibboleth-idp
```
3) #### Replace/create the following files with those in the files directory in the repo
 * `./shibboleth-idp/conf/admin/general-admin.xml`
 * `./shibboleth-idp/system/conf/admin/general-admin-system.xml`
 * `./shibboleth-idp/conf/attribute-filter.xml`
 * `./shibboleth-idp/conf/attribute-resolver.xml`
 * `./shibboleth-idp/conf/metadata-provider.xml`
 * `./shibboleth-idp/conf/services.xml`
 * `./shibboleth-idp/credentials/metaroot.pem`

4) #### Update ./shibboleth-idp/conf/ldap.properties
 Update the follwing settings:
 ```
 idp.authn.LDAP.authenticator                   = bindSearchAuthenticator
 idp.authn.LDAP.ldapURL                          = ldap://local.ip:389 # This needs to be the local interface address not localhost
 idp.authn.LDAP.useStartTLS                     = false
 idp.authn.LDAP.useSSL                          = false
 idp.authn.LDAP.returnAttributes                 = sn,uid,givenName,memberOf
 idp.authn.LDAP.baseDN                           = ou=users,dc=example,dc=com
 idp.authn.LDAP.userFilter                       = (uid={user})
 idp.authn.LDAP.bindDN                           = cn=admin,dc=example,dc=com
 idp.authn.LDAP.bindDNCredential                 = password
 idp.authn.LDAP.dnFormat                         = uid=%s,ou=users,dc=example,dc=com
 ```

5) #### Update ./shibboleth-idp/conf/metadata-provider.xml
 Replace rancher-server.url with the hostname of your rancher server

6) #### Customize ./shibboleth-idp/metadata/idp-metadata.xml

This is the metadata you will provide to Rancher telling it how to communicate to your IdP

 Uncomment and update the IdP details:
 ```xml
 <!--
    Fill in the details for your IdP here

            <mdui:UIInfo>
                <mdui:DisplayName xml:lang="en">A Name for the IdP at shibboleth.danlab.test</mdui:DisplayName>
                <mdui:Description xml:lang="en">Enter a description of your IdP at shibboleth.danlab.test</mdui:Description>
                <mdui:Logo height="80" width="80">https://shibboleth.danlab.test/Path/To/Logo.png</mdui:Logo>
            </mdui:UIInfo>
-->
```
You will need to remove the BackChannel certs which is the first `<KeyDescriptor use="signing">` block in each set.  Also remove the `<KeyDescriptor use="encryption">` keys.

7) #### Generate idp-browser certs
```
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
openssl pkcs12 -inkey key.pem -in certificate.pem -export -out idp-browser.p12
mv idp-browser.p12 shibboleth-idp/credentials/
```

8) #### Generate service certs
```
openssl req -newkey rsa:2048 -nodes -keyout myservice.key -x509 -days 365 -out myservice.cert
openssl pkcs12 -inkey key.pem -in certificate.pem -export -out idp-browser.p12
mv myservice.* shibboleth-idp/credentials/
```

9) #### Build and run the docker container
```
docker build -f Dockerfile -t "rancher/shibboleth-idp:latest" .
docker run -d --name="shibb" -p 443:4443 -p 2443:8443     -e TZ=America/Phoenix     -e JETTY_BROWSER_SSL_KEYSTORE_PASSWORD=password     -e JETTY_BACKCHANNEL_SSL_KEYSTORE_PASSWORD=password     rancher/shibboleth-idp:latest
```