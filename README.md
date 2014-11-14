Compilation Notes
=================
* mvn package would not run by default because multi-context-broker v1.0.0 could not be found in the remote repository
* downloaded multi-context-broker v1.0.0 release from internet2's github repo here:

```
https://github.com/Internet2/Shibboleth-Multi-Context-Broker
```
* unpacked the multi-context-broker jar and saved off the POM file from it
* used mvn install-file to put the jar and pom in my local maven repo:

```
 mvn install:install-file -DgroupId=edu.internet2.middleware.assurance.mcb -DgeneratePom=false -DartifactId=multi-context-broker -Dversion=1.0.0 -Dpackaging=jar -Dfile=multi-context-broker-1.0.0.jar -DpomFile=pom.xml
```
* would not compile now because multi-context-broker's 1.0.0 pom file pointed to unavailable SNAPSHOT releases of 3 projects.  I updated these three projects as follows:

```
<dependency>
            <groupId>edu.internet2.middleware</groupId>
            <artifactId>shibboleth-common</artifactId>
            <version>1.4.1-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.opensaml</groupId>
            <artifactId>opensaml</artifactId>
            <version>2.6.1-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>edu.internet2.middleware</groupId>
            <artifactId>shibboleth-identityprovider</artifactId>
            <version>2.4.1-SNAPSHOT</version>
        </dependency>
```
To

```
<dependency>
            <groupId>edu.internet2.middleware</groupId>
            <artifactId>shibboleth-common</artifactId>
            <version>1.4.4-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.opensaml</groupId>
            <artifactId>opensaml</artifactId>
            <version>2.6.5-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>edu.internet2.middleware</groupId>
            <artifactId>shibboleth-identityprovider</artifactId>
            <version>2.4.4-SNAPSHOT</version>
        </dependency>

```
mcb-duo
=======

Shibboleth Multi-Context Broker Duo Authentication.  This is a plugin for the 
[Shibboleth Multi-Context Broker](https://wiki.shibboleth.net/confluence/display/SHIB2/Multi-Context+Broker). 
It provides support for [DUO](http://www.duosecurity.com/) second factor authentication.

# Requirements
This module requires at least multi-context-broker-1.1.2.jar or later.

# Installation
1. Copy the jar file to your shibboleth-source-dir/lib.  
2. run install.sh
3. Copy the *duo.vm* file to the directory holding the rest of your MCB velocity templates

# Configuration

Before you can configure, you will need to create a [WebSDK Integration](https://www.duosecurity.com/docs/duoweb) and 
generate an [Application Secret Key](https://www.duosecurity.com/docs/duoweb#1.-generate-an-akey).  After that you need
to edit the *mcb-spring.xml* file and add the following block.


    <bean id="mcb.duo" class="edu.uchicago.identity.mcb.authn.provider.duo.DuoLoginSubmodule">
        <!-- application key -->
        <constructor-arg index="0" value="APPKEY GOES HERE" />
        <!-- integration key -->
        <constructor-arg index="1" value="IKEY GOES HERE" />
        <!-- secret key -->
        <constructor-arg index="2" value="SKEY GOES HERE" />
        <!-- host -->
        <constructor-arg index="3" value="HOST GOES HERE" />
        <!-- duo login template -->
        <constructor-arg index="4" value="duo.vm" />
    </bean>

Next you need to edit the *mcb.Configuration* bean and add a 

    <ref bean="mcb.duo" />

Next you need to edit the *multi-context-broker.xml* file and add the duo method to the authmethods:

    <method name="duo" bean="mcb.duo">
            Duo
    </method> 

Then, map it to a context in the authnContexts block:

     <context name="http://yourinstitution.edu/duo" method="duo">
                <allowedContexts>
                </allowedContexts>
        </context>

Finally, edit your *handler.xml* file and add the duo context as an AuthenticationMethod to your MCB LoginHandler declaration:

     <LoginHandler xsi:type="mcb:MultiContextBroker" authenticationDuration="PT8H0M0.000S" previousSession="true"
    depends-on="mcb.Configuration">
       <AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified</AuthenticationMethod>
       <AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthenticationMethod>
       <AuthenticationMethod>http://id.incommon.org/assurance/silver</AuthenticationMethod>
       <AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</AuthenticationMethod>
       <AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:PreviousSession</AuthenticationMethod>
       <AuthenticationMethod>http://yourinstitution.edu/duo</AuthenticationMethod>
    </LoginHandler>


### Note: you need to ensure that you do NOT specify duo as a default initial context.  In order to function, the user must already have established their identity to the MCB via another context.
