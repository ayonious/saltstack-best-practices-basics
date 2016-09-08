More examples with states
-------------


Taken from:
https://blog.talpor.com/2014/07/saltstack-beginners-tutorial/
I strongly suggest that you read each line of the above link, it was really helpful for me.


### __An Example__

###Folder Structure (on master of course)

/
|
salt/
|
states/
|
base/
├────top.sls
├────allMinions.sls
├────fileTransfer.sls
├────installAllShFiles.sls
├────minion1Previleges.sls
├────minion2Previleges.sls
├────fl.txt
└────f2.txt


### Edit __.sls__ files:
```bash
################################top.sls#############################
base:                    ## tell salt that this is safe
                         ## for all machines can be used as base
  '*':                   ## apply these to all minions
    - allMinions         ## run the 'allMinions.sls' file,
                         ## this name can be anything
    - fileTransfer       ## 'fileTransfer.sls'
    - installAllShFiles  ## 'installAllShFiles.sls'

  'salt-minion1':        ## apply this only to salt-minion1
    - minion1Previleges  ## run minion1Previleges.sls
  '*2':                  ## run this on any minion that has an
                         ##  ending character 2 (minion2 in this
                         ## example)
    - minion2Previleges  ## run minion2Previleges.sls

################################allMinions.sls############################

## what im trying to do is just sudo apt-get install fish in all minions
fish-install-ayon:          ## this is just an id, used for refering only
  pkg:                      ## tell salt this is a package
    - installed             ## install the package
    - name: fish            ## exact name of the package


## this will actually manage the file which is located in
## 'salt://fl.txt' in the salt-master. salt-MMinions will
## try to keep this file updated with the master. Whever
## master changes anything here minions will try to update
## their copy like git

take-file-by-nk:            ## id of this function
  file:                     ## tell salt this is a file
    - managed               ## Tells salt to mange this file
    - name: /etc/fl.txt     ## where to put the file in salt-minion
    - source: salt://fl.txt ## Tells salt where to find it in master
                            ## can find a local copy on the master
    - user: root            ## Tells salt to ensure the
                            ## owner of the file is root
    - group: root           ## Tells salt to ensure the
                            ## group of the file is root
    - mode: 644             ## Tells salt to ensure the permissions
                            ## of the file is 644

################################fileTransfer.sls############################


## this is same as the file manage  of allMinions.sls
## but here this manage requires you have already managed
## take-file-by-nk. Another thing to watch here is
## take-file-by-nk id was declared in another file
## but this file can see it




take-file-after-f1:           ## id of this function
  file:                       ## tell salt this is a file
    - managed                 ## Tells salt to mange this file
    - name: /etc/f2.txt       ## where to put the file in salt-minion
    - source: salt://f2.txt   ## Tells salt where to find it in master
                              ## can find a local copy on the master
    - user: root              ## Tells salt to ensure the
                              ## owner of the file is root
    - group: root             ## Tells salt to ensure the
                              ## group of the file is root
    - mode: 644               ## Tells salt to ensure the permissions
                              ## of the file is 644
    - require:                ## this operation depends on another
      - file: take-file-by-nk ## dependency
```

The other __.sls__ should be created. Its not mandatory to write anything in those files. You can keep other __.sls__ files blank

Remember:
For creating file: `sudo touch <file>`
For opening to edit file `sudo nano <file>`
After each file edit save and exit `ctrl+x -> Y -> enter`


>__NOTE:__
>Make sure the __YAML__ syntax is correct.
>Online YAML validator: http://codebeautify.org/yaml-validator


### make the changes in Minions :

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
     Started: 11:57:48.514196
    Duration: 384.391 ms
     Changes:
----------
          ID: take-file-by-nk
    Function: file.managed
        Name: /etc/fl.txt
      Result: True
     Comment: File /etc/fl.txt is in the correct state
     Started: 11:57:48.898736
    Duration: 3.183 ms
     Changes:
----------
          ID: take-file-after-f1
    Function: file.managed
        Name: /etc/f2.txt
      Result: True
     Comment: File /etc/f2.txt updated
     Started: 11:57:48.902246
    Duration: 6.084 ms
     Changes:
              ----------
              diff:
                  New file
              mode:
                  0644

Summary
------------
Succeeded: 3 (changed=1)
Failed:    0
------------
Total states run:     3
salt-minion1:
----------
          ID: fish-install-ayon
    Function: pkg.installed
        Name: fish
      Result: True
     Comment: Package fish is already installed.
     Started: 11:57:48.585544
    Duration: 391.671 ms
     Changes:
.....
.....
              ----------
              diff:
                  New file
              mode:
                  0644

Summary
------------
Succeeded: 3 (changed=1)
Failed:    0
------------
Total states run:     3
```

###__Improving folder structure__

states do have to be in files, but you can place them in files called 'init.sls' and place those in directories with the name of the state. For example, if I want to make a SSH state, I wouldn't make a file called 'ssh.sls', but I would create a 'ssh/' directory inside '/srv/salt/':

The above __Example__ can have folder structure like this: (you dont need to change anything in __.sls__ files in the above example)

	/
	│
	salt/
	│
	states/
	│
	base/
	├────top.sls
	├────allMinions────init.sls
	├────install────init.sls
	├────installAllShFiles────init.sls
	├────minion1Previleges────init.sls
	├────minion2Previleges────init.sls
	├────fl.txt
	└────f2.txt


###__Nested folder structure example__
Your master salt folder structure can be this:

	/
	│
	salt/
	│
	states/
	│
	base/
	├────top.sls
	├────allMinions────init.sls
	├─install
	│      ├────bashfile.sls
	│      ├────database──init.sls
	│	   └────systemfile.sls
	├────minion1Previleges────init.sls
	├────minion2Previleges────init.sls
	├────fl.txt
	└────f2.txt

In this case the __top.sls__ should be:
```perl
################################top.sls#############################
base:
  '*':
    - allMinions
    - install.bashfile        #note that . means inside folder
    - install.systemfile      #note that . means inside folder
    - install.database        #note that . means inside folder
  'salt-minion1':
    - minion1Previleges       #note that init.sls dont need .
                              #salt can see that by default
  '*2':
    - minion2Previleges
```


###__Some more states modules__:


```
common-pkgs:            # State ID
  pkg.installed:        # Make sure all of these are installed
    - names:
      - emacs
      - openssh-server
      - nginx
docker-kernel-pkgs:  # Install these kernel packages
 pkg.latest:
   - pkgs:
     - linux-image-generic-lts-raring
     - linux-headers-generic-lts-raring
docker-apt-https-transport-method: # Run this command...
  cmd.run:
    - name: apt-get update & apt-get install -y apt-transport-https
    - unless: [ ! -e /usr/lib/apt/methods/https ] # ... unless this is true
    - require:
      - pkg: docker-kernel-pkgs # This state has to run successfully first
docker-repo: # Install the Ubuntu PPA for Docker...
  pkgrepo.managed:
    - name: deb https://get.docker.io/ubuntu docker main
    - file: /etc/apt/sources.list.d/docker.list
    - keyserver: hkp://keyserver.ubuntu.com:80
    - keyid: 36A1D7869245C8950F966E92D8576A8BA88D21E9
    - require:
      - cmd: docker-apt-https-transport-method # ...only if this state ran
docker-pkg: # Install the package lxc-docker...
  pkg.latest:
    - name: lxc-docker
    - require:
      - pkgrepo: docker-repo # ...only if you ran the state docker-repo already
docker-serv:            # Make sure that the 'docker' is up and
 service.running:      # running...
   - name: docker
   - enable: True      # ... also set it to start at boot.
   - watch:
     - pkg: docker-pkg # If this package changes, restart this service.




python-soft-installer:
  cmd:
    - run
    - name: apt-get update
  pkg:
    - installed
    - name: python-software-properties
    - require:
      - cmd: php-installer
php-installer:
  pkgrepo:
    - managed
    - ppa: ondrej/php5-5.6
    - require:
      - pkg: python-soft-installer
  cmd:
    - run
    - name: apt-get update
    - require:
      - pkgrepo: php-installer
  pkg:
    - installed
    - pkgs:
      - php5
      - php5-mcrypt
      - php5-xdebug
      - php5-pgsql
      - php5-sqlite
    - require:
      - cmd: php-installer





git-install:
  pkg:
    - installed 
    - pkgs:
      - git
      - git-gui

git-set-email:
  module:
    - run
    - name: git.config_set
    - setting_name: user.email
    - setting_value: "my@mey.com"
    - is_global: True
    - require:
      - pkg: git-install

set_git_user_name:
  module:
    - run
    - name: git.config_set
    - setting_name: user.name
    - setting_value: "my"
    - is_global: True
    - require:
      - module: git-set-email
```



__NOTE:__

This
```
pkg.latest:
  - pkgs:
    - linux-image-generic-lts-raring
    - linux-headers-generic-lts-raring

docker-serv:            # Make sure that the 'docker' is up and
  service.running:      # running...
    - name: docker
	- enable: True      # ... also set it to start at boot.
	- watch:
	  - pkg: docker-pkg # If this package changes, restart this service.
```
is same as
```
pkg:
  -latest
  - pkgs:
    - linux-image-generic-lts-raring
    - linux-headers-generic-lts-raring
docker-serv:            # Make sure that the 'docker' is up and
  service:      # running...
    - running
    - name: docker
	- enable: True      # ... also set it to start at boot.
	- watch:
	  - pkg: docker-pkg # If this package changes, restart this service.
```
Just make sure YAML syntax is correct
