---
layout: post
title: "How to install Jasig CAS SSO plugged on Active Directory using SSL"
description: ""
category: Tutorial 
tags: 
- cas 
- sso 
- active directory 
- ad 
- ssl 
- apache 
- tomcat
---
{% include JB/setup %}

This article is a tutorial on how to deploy the Jasig CAS Single-Sign-On
solution, and to integrate it with Apache2 as web server, Tomcat as servlet
container, Active Directory as authentication backend. Every communication link
will be secured using SSL i.e. the client will communicate using https with the
CAS and the CAS will communicate using ldaps with the Active Directory.

### Requirements
In this article I suppose you got a running Windows Server 2008 R2 with active
directory over SSL enabled. You will also need a running linux distribution with
apache2 and vim to host Tomcat6, and Jasig CAS 3.4.2. 

First download on your linux distribution (On my installation, I put those two
tarballs in `/opt/` directory):

    # cd /opt

- last final release tarball of Jasig CAS at this address : <http://www.jasig.org/cas/download>
    
    # wget http://www.ja-sig.org/downloads/cas/cas-server-3.4.2-release.tar.gz 

- tomcat6 tarball at this address : http://tomcat.apache.org/download-60.cgi
   
   # wget http://apache.multidist.com/tomcat/tomcat-6/v6.0.33/bin/apache-tomcat-6.0.33.tar.gz -O tomcat6 
   
Note : If you are behind a proxy, remember to export environment variables :

    # export http_proxy = "http://your_proxy:3128/"
    # export https_proxy = "http://your_proxy:3128/"
    # export ftp_proxy = "http://your_proxy:3128/" 
    
Once you have untarred it you should have this :

    # ls -l /opt
    total 23268
    -rw-r--r--  1 root root  6531699 Aug 18 15:11 apache-tomcat-6.0.33.tar.gz
    drwxr-xr-x 18 root root     4096 Aug 24 18:30 cas-server-3.4.2
    -rw-r--r--  1 root root 17246325 Mar 25  2010 cas-server-3.4.2-release.tar.gz
    drwxr-xr-x  9 root root     4096 Aug 24 17:34 tomcat6

### SSL Certificate and Key generation
In our context, we are going to use a signed certificate using our own home-made 
certificate authority (for instructions on how to setup a certificate authority,
you can check here <http://www.eclectica.ca/howto/ssl-cert-howto.php>).

First, we'll need to write a simple configuration file for openssl :

    [ req ]
    default_bits = 1024 # Size of keys
    default_keyfile = key.pem # name of generated keys
    default_md = md5 # message digest algorithm
    string_mask = nombstr # permitted characters
    distinguished_name = req_distinguished_name
    req_extensions = v3_req
    [ req_distinguished_name ]
    # Variable name Prompt string
    #---------------------- ----------------------------------
    0.organizationName = Organization Name (company)
    organizationalUnitName = Organizational Unit Name (department, division)
    emailAddress = Email Address
    emailAddress_max = 40
    localityName = Locality Name (city, district)
    stateOrProvinceName = State or Province Name (full name)
    countryName = Country Name (2 letter code)
    countryName_min = 2
    countryName_max = 2
    commonName = Common Name (hostname, IP, or your name)
    commonName_max = 64

    # Default values for the above, for consistency and less typing.
    # Variable name Value
    #------------------------------ ------------------------------
    0.organizationName_default = The Sample Company
    localityName_default = Metropolis
    stateOrProvinceName_default = New York
    countryName_default = US

    [ v3_req ]
    basicConstraints = CA:FALSE
    subjectKeyIdentifier = hash

Write out to openssl.cnf and issue the following command :

    # openssl req -new -nodes -out req.pem -config ./openssl.cnf 

Fill the Common Name with the domain name or the ip adress of the server for
which you want to obtain a certificate.

This will generate a private key in key.pem and a certificate signing request
in req.pem.

In order to create a certificate, using your newly created certificate request
with your certificate authority :

Upload your request on your certificate authority
Connect to your home-made certificate authority
Execute this command to generate and sign the certificate :

    # openssl ca -out cert.pem -config ./openssl.cnf -infiles req.pem 
    
This will produce the certificate for your server in cert.pem. Now retrieve it
on your web server.

### Apache2 SSL configuration
To enable SSL within apache2, you have to tweak it a little. First enable ssl 
module for apache :

    # a2enmod ssl 

Then edit apache configuration file named default-ssl :

     # vim /etc/apache2/sites-available/default-ssl 

And add the following :

    <VirtualHost xxx.xxx.xxx.xxx:443>
    ...
        SSLEngine on
        SSLCertificateFile /etc/ssl/certs/cert.pem
        SSLCertificateKeyFile
        /etc/ssl/private/key.pem
    ...
    </VirtualHost> 

You must then place your private key and your certificate in their appropriate
folder :

    # cp cert.pem /etc/ssl/certs/
    # cp key.pem /etc/ssl/private/ 

Reload your apache : 

    # service apache2 reload 

### Jasig CAS configuration
Now let's configure the CAS itelf. Configurations files are stored in this 
directory :

    # cd /opt/cas-server-3.4.2/cas-server-webapp/src/main/webapp/WEB-INF 

First we are going to configure the connexion to the active directory. So edit
the `deployerConfigContext.xml` file :

    # vim deployerConfigContext.xml 

Supposing that your Active Directory is based on mydomain.com domain and that
you want your user to identify with their Active Directory identifier (a.k.a. 
cn), you must add the following bean within property named `authenticationHandlers` 
of the authenticationManager bean. It will be used  by the CAS to communicate
with the Active Directory:

    ...
    <bean class="org.jasig.cas.adaptors.ldap.BindLdapAuthenticationHandler">
        <property name="filter" value="cn=%u" />
        <property name="searchBase" value="dc=domain,dc=com" />
        <property name="contextSource" ref="contextSource" />
        <property name="ignorePartialResultException" value="yes" />
    </bean>
    ...

In the same file, at the end of the beans tag you must add the following bean
which will tell the CAS how to contact the Active Directory. The urls property
gives the AD url, and the `baseEnvironmentProperties` property specify whether
or not to connect using SSL, and which kind of authentication to use. You have
to be careful to a few things. First if you want to connect to your Active
Directory using SSL, you have to specify ldaps in the url in addition to the
`baseEnvironmentProperties` entry. Furthermore, you must have had a read-only
user to your Active Directory. In my example here, the latter is lecteur
connecting using lecteur as a password.

    ...
    <bean id="contextSource" class="org.springframework.ldap.core.support.LdapContextSource">
        <property name="anonymousReadOnly" value="false" />
        <property name="pooled" value="true" />
        <property name="urls">
            <list>
                <value>ldaps://ad.mydomain.com/</value>
            </list>
        </property>
        <!-- uncomment the following lines if you nead to bind to the directory. Adjust
        to your needs.-->
        <property name="password" value="lecteur" />
        <property name="userDn" value="cn=lecteur,cn=users,dc=mydomain,dc=com" />
        <property name="baseEnvironmentProperties">
            <map>
                <entry>
                    <key><value>java.naming.security.protocol</value></key>
                    <value>ssl</value>
                </entry>
                <entry>
                    <key><value>java.naming.security.authentication</value></key>
                    <value>simple</value>
                </entry>
            </map>
        </property>
    </bean>
    ...
Now that you have configured your CAS installation you have to build it. Since
it is a final release we can skip the tests :

    # cd /opt/cas-server-3.4.2
    # mvn package install -DskipTests

Note: if you are running maven behind a proxy, you have to add the following
to your `settings.xml` file (if the latter does not exist, create it):

    # vim ~/.m2/settings.xml

    <settings>
        <proxies>
            <proxy>
                <active>true</active>
                <protocol>http</protocol>
                <host>your_proxy</host>
                <port>3128</port>
            </proxy>
            <proxy>
                <active>true</active>
                <protocol>https</protocol>
                <host>your_proxy</host>
                <port>3128</port>
            </proxy>
            <proxy>
                <active>true</active>
                <protocol>ftp</protocol>
                <host>your_proxy</host>
                <port>3128</port>
            </proxy>
        </proxies>
    </settings>

Now that you have built it, you can copy it in tomcat webapp folder, and restart
tomcat. The CAS will be available at <http://localhost:8080/cas/>:

    # cp /opt/cas-server-3.4.2/cas-server-webapps/target/cas.war /opt/tomcat6/webapps/
    # cd /opt/tomcat6
    # ./bin/shutdown.sh
    # ./bin/startup.sh

### Apache2 and Tomcat integration (mod_jk)
In order to get the certificate available to java applications, running or not
in Tomcat, you have to add your Active Directory certificate to the java keystore.
So retrieve this certificate and copy it in `/etc/ssl/certs`. Let's say it is
named `ad.crt`:

    # keytool -import -keystore $JAVA_HOME/jre/lib/security/cacerts -file /etc/ssl/certs/ad.crt 

There is a specific protocol name ajp used by apache and tomcat to communicate
with each other. The following is the procedure to tell apache to use this
protocol to communicate with your tomcat instance.
First install mod_jk and enable it:

    # aptitude install libapache2-mod-jk
    # a2enmod jk 

Then we are going to edit `jk.load` file to specify where properties are
specified by adding a simple line :

    # vim /etc/apache2/mods-available/jk.load

    ...
    JkWorkersFile /etc/apache2/workers.properties 

We are now going to fill workers.properties file :

    # vim /etc/apache2/workers.properties

    worker.list=worker1
    worler.worker1.port=8009
    worker.worker1.host=localhost
    worker.worker1.type=ajp13 

At last, we are going to configure apache SSL VirtualHost. First Change the
document root by commenting the following line:

    DocumentRoot /var/www

And adding this one :

    DocumentRoot /opt/tomcat6/webapps/

Now let's add the tomcat part:

    JkMount /*.jsp worker1
    JkMount /cas/* worker1
    JkExtractSSL On
    # What is the indicator for SSL (default is HTTPS) 
    JkHTTPSIndicator HTTPS 
    # What is the indicator for SSL session (default is SSL_SESSION_ID) 
    JkSESSIONIndicator SSL_SESSION_ID 
    # What is the indicator for client SSL cipher suit (default is SSL_CIPHER) 
    JkCIPHERIndicator SSL_CIPHER 
    # What is the indicator for the client SSL certificated (default is SSL_CLIENT_CERT) 
    JkCERTSIndicator SSL_CLIENT_CERT

And since on our web server, we only have CAS running on apache ssl, we will
tweak it such as users are redirected to cas when they are pointing on root :

    <Directory />
        Options FollowSymLinks -Indexes
        AllowOverride None
        RedirectMatch permanent ^/$ /cas/
    </Directory>

Reload your apache : 

    # service apache2 reload 

You should now have a working Jasig CAS SSO based on Active Directory Authentication 
backend and using SSL to secure each communication link.

Have fun :)


