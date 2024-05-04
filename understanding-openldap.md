# Understanding OpenLDAP

When you want to setup centralized authentication in a pure Linux environment, OpenLDAP is the first thing that comes up.
Just install an OpenLDAP Server, configure your clients to use it, and you are good to go.

There are however a few things, that can be confusing, so I thought I write a bit about them.

## Access to the configuration database cn=config
The configuration of OpenLDAP ist stored in it's own database. So you can use the usual Ldap Tools like ldapmodify to set them. However
an initial Installation doesn't provide you with credentials? Surely there is some sort of authentication to access it, but whats the password?
With the old style configuration it was just 'sudo editing' a file at /etc/ldap, but now?

Well it's more or less the same. Ldap knows an authentication method called 'EXTERNAL'. With this LDAP trusts the 'surrounding' system and uses
what ever username that provides. There are two options for the surrounding system. The first ist TLS certificates, which means your username will
be the common name of your client certificate. The second is Unix sockets, which means your username will be based on your UID/GID on the system.

The Ubuntu package, and probably all others too, of OpenLDAP, configures access to the configuration database to be allowed for UID/GID = 0. This
of course only works with Unix sockets, so it is limited to the server that OpenLDAP runs on. Which does make sense, after all you don't necessarily
want to open the configuration to the outside. So you can configure the server from the system it runs on via:

    # ldapmodify -Y EXTERNAL -H ldapi:/// -f <you config file.ldif>

ldapi:/// tells ldapmodify to use the Unix socket. 