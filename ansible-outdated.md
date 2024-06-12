# If you rely on Ansible, your TechStack might be outdated!

Sorry for the clickbaity title. It thought I try clickbait for once. But let
it make it up to you by giving you the TL;DR

## TL;DR
Ansible used to be THE tool, now it's just A tool. A good one, but nowadays
there is so much more. So if you rely on Ansible as the only tool, you are
missing out.

## Ansible was THE tool

Ansible was a bit of a revolution in the sysadmin world. Not that it was the
first tool for configuration management, Puppet was there earlier and others
were even before that, but Ansible had the capability to work via SSH. Puppet
needed an agent and an additional network connection.  So you had to ask for
permission from the firewall admins and had to go through all the hops that
adding a new tool required.

But ansible didn't need all of that. You could just install it on your notebook
and let it use the SSH connection you had anyway. No need to change the
existing system just for the tool. It also wasn't an all or nothing tool. You
could use it to automate some task, while doing everything else just like you
did before.

That probably helped it's acceptance and transformed the type of work sysadmins
did. Instead of login into systems, change things manually and update the
documentation you wrote an ansible playbook which did the changes for you and,
due to the YAML structure of the config files, was mostly self documenting.

So, for quite some time ansible was THE tool. You manually created the server,
either by physically adding it to the datacenter, or by creating a VM. After
that you installed the OS, with the help of some tools like Kickstart, but
those where rather rudimentary. Once the OS ran you could start
`ansible-playbook` and watch the screen yellow from all the `changed` message
that ansible produced when it, well, changed something on the system. Run it
another time and now enjoy all the green `Ok` messages when ansible noticed
that the system was as it should be. Good times!

But times are changing and so did the tools. VMs became more and more common,
the various cloud providers and their ever improving APIs made more detailed VM
setups possible. Docker made VMs look bloated and while actually going to the
datacenter was already a thing of the past, the new tools made ssh-ing into a
server something you only did if things went wrong.

That might all sound a bit nostalgic, but it's not. The new world has good
things. Precision, repeatabilty and interconnected tools. Automation that
actually deservers it's name. While Ansible caused a smirk on the Sysadmins
face, tools like packer, terraform, az, gcloud, cloud-init, Github Actions and
many more paint a big, bright grin on a DevOps-Engineer's face. 

Oh and don't be sad for Ansible. It's still there, for example as provisioner
in packer. They all play along nicely.

So if Ansible is still THE tool for you: Come to the new world, you wont regret
it.

