# Github Actions Mini Howto

## Introduction
Github Actions is an easy and simple way to automate tasks for your software
project. Yet, opening the Actions tab on Github can be a bit overwhelming. Let
me give you a simple example to get you started.

## Background
I run a personal website at https://benjaminfleckenstein.name - nothing
special, just a few links and articles. The site is stored in a (private)
Github repository and consists of markdown files, images, and template files. I
use a [small script](https://github.com/nbkr/sitegenerator2) to convert the
markdown pages to HTML and combine them with the templates. All that, plus the
static content, gets uploaded via SFTP to my server.

## What I want to do
I want to write a Github Action that runs the script and uploads the site every
time I commit something.

## How-to
* First, I create a directory `.github/workflows` within my content repository.
* In that directory, I put a file called `main.yml` with the following content:

```
name: Build
on: [workflow_dispatch, push]
jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
     - uses: actions/checkout@v4
       with: 
         repository: nbkr/sitegenerator2
         path: './sitegenerator2'
     - uses: actions/checkout@v4
       with:
         path: './website'
     - run: 'python3 -m venv venv'
     - run: 'source venv/bin/activate'
     - run: 'pip3 install -r ./sitegenerator2/requirements.txt'
     - run: './sitegenerator2/generator.py ./website gen'
     - uses: wlixcc/SFTP-Deploy-Action@v1.2.4
       with:
         username: sftp_benjaminfleckenstein
         server: benjaminfleckenstein.name
         local_path: ./website/build/*
         remote_path: /data/www/benjaminfleckenstein/droot/
         sftp_only: true
         ssh_private_key: {% raw %}${{ secrets.SFTP_BENJAMINFLECKENSTEIN_KEY }}{% endraw %}
```
* I add a secret to my repository called SFTP_BENJAMINFLECKENSTEIN_KEY that
  contains the private SSH key needed to access the document root on my server.
* Done.

Ok, let’s go into more detail.

### .github/workflows
This is the location where Github expects workflows or actions. Every .yml file
will be checked for workflows.

## The main.yml
Let’s go through every line.

---
```
name: Build
```
Defines the name of the action. Naming is up to you.

---
```
on: [workflow_dispatch, push]
```
The triggers, aka when that workflow should start. There are dozens of
triggers. You can specify more than one, just put them in a list, like in the
example. `push` triggers, well when you push something to the repo. `workflow_dispatch`
lets you start the workflow manually. If you are testing the new workflow, you
probably don’t want to use push right from the beginning. After all, the
main.yml is part of the repo, so every small change would trigger the workflow.

---
```
jobs:
  build:
```
This part defines the jobs that the workflow should actually run. You can have
multiple ones, by default, they run in parallel. As building and deploying my
site is done one after the other, I go with just one job. `build` is the
identifier of the job. You can call it - within reason - anything you like.

---
```
runs-on: ubuntu-24.04
```
The jobs need to be executed somewhere. That somewhere is called a runner, and
it’s essentially a virtual machine. What you can do depends on the operating
system this VM runs on. For example, if you need to install specific software,
it matters if you run it on Ubuntu or Red Hat. With the former, you will need
to use apt, with the latter, dnf. So you need to specify the operating system.
`runs-on` does exactly that. The value ubuntu-24.04 says that you want Ubuntu
version 24.04. You can also specify `ubuntu-latest` if you don’t care about the
specifics.

---
```
steps:
```
Now we are getting somewhere. `steps` is the section where the actual commands
are specified. `steps` is a list, so every step starts with a hyphen.

---
```
- uses: actions/checkout@v4
  with:
    repository: nbkr/sitegenerator2
    path: './sitegenerator2'
```
First step and it already gets a bit confusing. Besides running actual commands
on the runner, you can also use what is called ‘actions’. Yes, it’s unfortunate
to call it the same as the tab in the repo. I would describe them as plugins or
modules. Everyone can write those, and there are more ‘official’ ones like this
one. checkout does exactly what you would expect. It checks out a repository.
@v4 is the version tag. So if the action ever changes, it won’t break your
project.

Those actions can have additional parameters. Those are defined in the `with:`
section. `repository` specifies the repo I want to check out. Relative to
Github. So the above value would check out https://github.com/nbkr/sitegenerator2.

`path` specifies where the repo will be checked out to on the runner. This time
in the directory `sitegenerator2`. Nicely tucked away in its own folder.

---
```
- uses: actions/checkout@v4
  with:
    path: './website'
```
Again, checking out a repo. This time the `repository` isn’t defined. So it
defaults to ‘this one’. Aka the same one the `main.yml` workflow file is in. `path`
is, again, the destination. So the contents of this very repository can be
found at ‘./website’ on the runner.

---
```
- run: 'python3 -m venv venv'
- run: 'source venv/bin/activate'
- run: 'pip3 install -r ./sitegenerator2/requirements.txt'
- run: './sitegenerator2/generator.py ./website gen'
```
Now it gets a bit simpler. Each `run:` line is a command that is executed as is
on the runner. The sitegenerator2 is a Python script that requires some
modules. So I create a virtual Python environment (first line), activate it
(second line), install all required modules (third line), and finally run the
script with its required parameters. If it works, there will be a new directory
on the runner called ‘./website/build’ that contains my personal site.

---
```
- uses: wlixcc/SFTP-Deploy-Action@v1.2.4
  with:
    username: sftp_benjaminfleckenstein
    server: benjaminfleckenstein.name
    local_path: ./website/build/*
    remote_path: /data/www/benjaminfleckenstein/droot/
    sftp_only: true
    ssh_private_key: ${{ secrets.SFTP_BENJAMINFLECKENSTEIN_KEY }}
```
Finally, I can upload the contents of the build folder. I use another action
for this: wlixcc/SFTP-Deploy-Action in version 1.2.4. I found that particular
action by searching for it on the [Github Marketplace](https://github.com/marketplace?type=actions).

This specific action requires several parameters like username and destination.
The ssh_private_key is a bit of a problem. I need a private key to access my
web space. But the main.yml file is part of the repository. Sure, it’s private,
but I don’t really want to copy credentials there. Who knows, maybe I will make
the repo public one day, or someone else helping with the site who shouldn’t be
able to access the server. So putting the key directly into the file is not
good.

Instead, I’m using Github Secrets. It’s essentially a storage for strings that
will replace the `{% raw %}${{ secrets.SFTP_BENJAMINFLECKENSTEIN_KEY }}{% endraw %}` variable once the
workflow runs. The secret, therefore, won’t show up in the main.yml and is -
hopefully - kept safe by Github.

You can find the secrets in the ‘settings’ tab of your repository. There are
different ‘levels’ of secrets: repository secrets and environment secrets. This
particular action seems to work with repository secrets.

Oh, and also, if you use it, make sure your ssh_private_key is in PEM format.
It needs to start with `-----BEGIN RSA PRIVATE KEY-----`. Newer OpenSSH versions
generate a key in the OpenSSH format. At first glance, they look similar
because they are both ASCII protected strings, but the ‘wrong’ version has a
`-----BEGIN OPENSSH PRIVATE KEY-----` at the top, not RSA. See [here](https://stackoverflow.com/questions/54994641/openssh-private-key-to-rsa-private-key) on how to
convert an existing key. Spoiler: It’s annoying.

That’s all! Good luck with your Github actions!


