# PostSRSD and Packer

I run a small website for my beekeepers club. That includes an email address
that I need to check. The website runs on it's own - heavily over engineered
(hey, I'm a DevOps Engineer, I like automating things) - server. For the email
address I use Postfix, but I didn't want to install an imap server, as that
would mean, I have to setup another account on my mail application. A simple
forwarding should do. But with SPF and the like, forwarding mails isn't as
straight forward as it used to be. One needs to rewrite the envelope address of
each mail. Otherwise my mail server won't accept the forwarded email as it would
suspect it's spam.

This is what PostSRSD is for. It rewrites the envelope address. PostSRSD is
quite clever, as it doesn't really need a configuration for it self. Postfix
needs to be told about PostSRSD, but the later it figures out everything it
needs to know during installation. That's why it doesn't come with a config
file inside the deb package. Instead it creates it's own config file during
installation and stores it at /etc/default/postsrsd.

This works perfectly if you set it up manually when the server and especially
Postfix is already up and running. So when I first tested it on the system
everything looked fine.  All I really had to do was run apt and add a few lines
of Postfix configuration. 

So after the initial test, it was time to include PostSRSD in the packer image.
The Postfix configuration is part of the persistent disk in my setup, so just
replace the image and everything is done. Right? Well, obviously wrong,
otherwise I wouldn't write about it.

You see, when packer creates an image, the Postfix configuration isn't there
yet, so PostSRSD can't determine it's own configuration settings. So it creates
a config file with the wrong settings and therefore it doesn't work, when the
image is rolled out. 

This is also the point where PostSRSD could be improved. Creating your own
config file from by reading data from the machine it runs on is unfortunate. It
doesn't add any value, because the configuration is always checked on startup
of the service. The config file is then read and overwrites any value that has
been set in the file. So the only reason to configure something in the config
file, would be if the sysadmin needs to overwrite a value. Creating the config
from the read value therefore doesn't make sense. The better solution would be
to create the config file, but have everything commented out. This way it is
easy to reconfigure something but it wouldn't interfere with the autoconfig.

Ok, back to the story. After figuring out where things go wrong, solving it is
almost easy. The test during the manual installation showed that I don't really
need a configfile. PostSRSD gets the settings correct - at least in my setup -
so removing the config file is the simplest solution. Right? Well, almost.

As my Postfix configuration is located on the permanent disk of my server
instance I need a setup step during startup. Something that tells Postfix that
it's configuration is at /data instead on /etc.  Cloud-Init takes care of that.
But, Cloud-Init runs after Postfix and PostSRSD have started, so PostSRSD works
on the default Postfix configuration, which isn't correct just yet. After
Cloud-Init reconfigures Postfix, it also restarts it. At that point the
configuration is correct and PostSRSD can use it. So all I needed to do was to
add restart of PostSRSD in the Cloud-Init script and it worked.

A lot of work, for something so seemingly simple as an email forwarder. 
