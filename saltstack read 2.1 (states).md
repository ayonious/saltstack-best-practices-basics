States
-------------

This is from:
http://bencane.com/2013/09/03/getting-started-with-saltstack-by-example-automatically-installing-nginx/


Suppose in our system there is only one master and 100 minions. We have set up images quickly using vagrant/packer now its time to install things inside those images. You definitely want to automate this installation process.


>__Idea__:
>Why not create 1 image then clone this image 100 times
>__Problem:__
>	1.cloning takes time
>	2. what will you do when you want to do some changes? Will you destroy all the images and then create them again and clone them again 100 times? (BTW I think docker can help us with something like this, images handled like git, I need to learn docker more to know this).
>	3. Suppose even with docker what happens is that you get your upgraded image without copying entire image. After your image is updated you need to restart the image(I think). So in this case you will lose the intermediate states of those nodes. [again Im not sure about this]
>__Solution: __
>	So only good way to do this is using some configure management tool like saltstack,puppet,ansible,chef


###1. What is States?
In salt states means configuration of images (minions).
Salt uses state files of format __[name].sls__ to describe states of the minions.
So inside this file master will write command like:
```
"I want all the minions to install this package"
"I want all the minions to remove this package"
"I want all the minions to copy this file "
"I want all the minions to copy this file and whenever I change anything in this file minions will change their copy of file according to latest file in master"

"I want all the minions to copy all these source files of a program and whenever I change any of these files all minions will make the change, then rebuild the project, then run the program again"
```


### 2. simple example using states

before starting this example I must assume that you already have 3 machines running and installed those machines according to My  prev reads on saltstack
now enter the 3 machines, open 3 terminals:

####2.1 Setting the states file_roots directory

From master:


>```bash
>vagrant@salt-master /> $ cd /etc/salt/
>vagrant@salt-master /> $ ls
>```
>
>output: (make sure you see the file master here)
>```
>master  master.d/  pki/
>```

Now edit this file:
```bash
vagrant@salt-master /> $ sudo nano /etc/salt/master
```
you will get a huge editor with lots of lines commented (this may not be true for all versions of salt)

Find:
```
#file_roots:
#base:
#- /srv/salt
```
Replace with:
```
file_roots:
  base:
    - /salt/states/base
```

__Note__: we are going to place the base file in location `/salt/states/base`


####2.2 Restart the salt-master service:
From master:
>```bash
>vagrant@salt-master /> $ sudo service salt-master restart
>```
>output:
>```
>salt-master stop/waiting
>salt-master start/running, process 1036
>```



####2.3 Creating the salt states directories and files:

#####Folder structure:
	/
	|
	salt/
	|
	states/
	|
	base/
	├────top.sls
	└────allMinions.sls


From master:
```bash
vagrant@salt-master /> $ sudo mkdir -p /salt/states/base
vagrant@salt-master /> $ sudo touch /salt/states/base/top.sls
vagrant@salt-master /> $ sudo nano /salt/states/base/top.sls
```
In editor:
```
###############salt/states/base/top.sls#############
base:             ## tell salt that this is safe for all machines can be used as base
  '*':            ## apply these to all minions
    - allMinions  ## run the 'allMinions.sls' file, this name can be anything
```
now save and exit
`ctrl+x -> Y -> enter`

So salt-master knows about this directory, It by default knows about __top.sls__
inside __top.sls__ we tell salt about all other __.sls__ files

Time to create and edit allMinions.sls file
```bash
vagrant@salt-master /> $ sudo touch /salt/states/base/allMinions.sls
vagrant@salt-master /> $ sudo nano /salt/states/base/allMinions.sls
```
In editor:
```
######/salt/states/base/allMinions.sls#############

## what im trying to do is just sudo apt-get install fish in all minions
fish-install-ayon:          ## this is just an id, used for refering only
  pkg:                      ## tel salt this is a package
    - installed             ## install the package
    - name: fish            ## exact name of the package
```


>__NOTE:__
>All the scripts in __.sls__ file is written in __YAML__ format. its a very common mistake to
>make syntax error in __YAML__. Every space/new line in YAML has meaning. you must check your syntax, by any __online YAML editor__
>My favourite: http://yaml-online-parser.appspot.com/



Time to run the changes:
From master terminal:

```bash
vagrant@salt-master /> $ sudo salt '*' state.highstate
#master telling all minions to update according to new new state files
```
output:
```
salt-minion2:
----------
          ID: fish-install-ayon
    Function: pkg.installed
        Name: fish
      Result: True
     Comment: Package fish is already installed.
     Started: 08:21:51.363905
    Duration: 407.672 ms
     Changes:

Summary
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
salt-minion1:
----------
          ID: fish-install-ayon
    Function: pkg.installed
        Name: fish
      Result: True
     Comment: The following packages were installed/updated: fish.
     Started: 08:21:51.448387
    Duration: 26592.773 ms
     Changes:
              ----------
              fish:
                  ----------
                  new:
                      2.0.0-1
                  old:


Summary
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1

```


####3. Excuting only a particular state file

```bash
vagrant@salt-master /e/salt> $ sudo salt '*' state.sls allMinions
```

>__Note__:
>Just as one could call the ``test.ping`` or ``disk.usage`` execution modules,
>``state.sls`` is simply another execution module. It simply takes the name of an
>SLS file as an argument.
