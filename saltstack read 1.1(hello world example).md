###Simple hello world example with salt(without shell provision)

This example is same as before just without shell provision. In practice I dont prefer this
way. Im showing this only to explain what really happened in the WITH SHELL PROVISION.


Our target is to make environment to stacksalt so that we can install+work with salt from multiple machines(virtual) and after we
have done everything we can delete the environment. This way our ubuntu can be safe from harmful installations.


This is from:
http://www.giantflyingsaucer.com/blog/?p=5001

###0. Install vagrant virtualbox guest additions 
install virtualbox:
```bash
sudo apt-get install virtualbox dkms virtualbox-guest-additions-iso
```
install vagrant by downloading the proper version from https://www.vagrantup.com/downloads.html
or you can just use this
```bash
#you can download from 
wget https://dl.bintray.com/mitchellh/vagrant/vagrant_1.6.3_x86_64.deb
#now you should check the checksum
#sha256sum vagrant_1.6.3_x86_64.deb | grep 0fc3259cf08b693e3383636256734513ee93bf258f8328efb64e1dde447aadbe
sudo dpkg -i vagrant_1.6.3_x86_64.deb
#since you dont need this you can do this
#rm vagrant_1.6.3_x86_64.deb
```


__NOTE__:
latest version maybe:  __vagrant_1.7.1_x86_64.deb__

__NOTE:__ 
__vagrant_1.7.1_x86_64.deb__ package has some issues, So a more stable version is I think __vagrant_1.6.3_x86_64.deb__


###1. Create folders

Lets make 3 folders in our desktop where we want to make the environment:

```bash
$ cd
$ cd VagrantBoxes
$ mkdir salt-master
$ mkdir salt-minion1
$ mkdir salt-minion2

$ cd salt-master
$ vagrant init ubuntu/trusty64
$ cd ..

$ cd salt-minion1
$ vagrant init ubuntu/trusty64
$ cd ..

$ cd salt-minion2
$ vagrant init ubuntu/trusty64
```
Open 3 separate __terminals__ for each master/minion1/minion2

###2. Edit Vagrant files
in Vagrant file of master:
```
Vagrant.configure(2) do |config|
	config.vm.box = "ubuntu/trusty64"
	config.vm.network "private_network", ip: "192.168.33.10"
	config.vm.hostname = "salt-master"
end
```
in Vagrant file of minion1:
```
Vagrant.configure(2) do |config|
	config.vm.box = "ubuntu/trusty64"
	config.vm.network "private_network", ip: "192.168.33.11"
	config.vm.hostname = "salt-minion1"
end
```
in Vagrant file of minion2:
```
Vagrant.configure(2) do |config|
	config.vm.box = "ubuntu/trusty64"
	config.vm.network "private_network", ip: "192.168.33.12"
	config.vm.hostname = "salt-minion2"
end
```

###3. Start and enter into machines
in all 3 __terminals__:
```bash
$ vagrant up
...
...
...
$ vagrant ssh
```
you will notice in __master terminal__ `vagrant@salt-master:~$`
you will notice in __minion terminals__ `vagrant@salt-minion1:~$`   `vagrant@salt-minion2:~$`
which means our Vagrant file config:`config.vm.hostname = <name>` is working


###4. Install salt in machines
Start the virtual box GUI see that all 3 machines are running.

In all 3 terminal:
```bash
$ sudo apt-get install python-software-properties
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:saltstack/salt
$ sudo apt-get update
```
In master terminal:
```bash
$ sudo apt-get install salt-master
```

In both minion terminals:
```bash
$ sudo apt-get install salt-minion
```



###4. Make connection between machines


Once the salt-minion package is installed run the following on the __Minion Terminal1__:
>```bash
>$ sudo nano /etc/salt/minion
>```
>Change:
>`\# master: salt`
>to:
>`master: 192.168.33.10`
>
>Save and exit
>`ctrl+x -> Y -> enter`

Once we updated the /etc/salt/minion file we need to restart the service. From __Minion Terminal1__:
>```bash
>$ sudo service salt-minion restart
>```
>Expected output:
>```
>salt-minion stop/waiting
>salt-minion start/running, process 6207
>```

Ok, now we need to add the salt-minion1 key to the salt-master. From __Master Terminal__:

>```bash
>$ sudo salt-key -L
>```
>This should display the following:
>```
>Accepted Keys:
>Unaccepted Keys:
>salt-minion1
>Rejected Keys:
>```

From __Master Terminal__:
>```bash
>$ sudo salt-key -a 'salt-minion1'
>```
>__Note__: You will be promoted with a yes/no.
>
>Verify the key was added. From the __Master Terminal__:
>```bash
>$ sudo salt-key -L
>```
>Expected output:
>```
>Accepted Keys:
>salt-minion1
>Unaccepted Keys:
>Rejected Keys:
>```

Do the exact same thing to add __salt-minion2__. After adding __minion2__ from __salt-master__:
```
Accepted Keys:
salt-minion1
salt-minion2
Unaccepted Keys:
Rejected Keys:
```


###5. Test the connection

Assuming everything went well, lets try to ping our minions (in this case just one). From __Master Terminal__ run the following:

>```bash
>$ sudo salt '*' test.ping
>```
>Expected output:
>```
>salt-minion1:
>    True
>salt-minion2:
>    True
>```


Let’s find the Minion’s IP address. From the __Master Terminal__:
>```bash
>$ sudo salt 'salt-minion1' network.ip_addrs
>```
>Expected output:
>```
>salt-minion1:
>    - 10.0.2.15
>    - 192.168.33.11
>```

Try some other commands for fun from the Master Terminal:
```bash
$ sudo salt 'salt-minion1' cmd.run 'ls -l /etc/salt'
$ sudo salt 'salt-minion1' disk.percent
$ sudo salt 'salt-minion1' network.interfaces
$ sudo salt '*' pkg.install vim
$ sudo salt '*' test.echo 'foo: bar'
```
__NOTE__ : `test.ping` or `disk.usage`  `pkg.install` are called __execution modules__.

That’s it. You now have a simple Salt playground to experiment with. Stay tuned for my next tutorial to show you how to do more with Salt.

>__Final Notes__: If you have UFW enabled (it won’t be enabled by default) you need to do the following on both terminals:
>```bash
>$ sudo ufw allow salt
>```
>__see more here__: http://docs.saltstack.com/en/latest/topics/tutorials/firewall.html


Thanks to: http://www.giantflyingsaucer.com/blog/?p=5001




###6. Changing the Output Format

The default output format used for most Salt commands is called the `nested`
outputter, but there are several other outputters that can be used to change
the way the output is displayed. For instance, the `pprint` outputter can be
used to display the return data using Python's `pprint` module:

>default output:
>```bash
>vagrant@salt-master /e/salt> $ sudo salt 'salt-minion1' test.ping
>```
>
>```
>salt-minion1:
>    True
>```
>pprint format:
>```bash
>vagrant@salt-master /e/salt> $ sudo salt 'salt-minion1' test.ping  --out=pprint
>```
>
>```
>{'salt-minion1': True}
>```
>json format:
>```bash
>vagrant@salt-master /e/salt> sudo salt 'salt-minion1' test.ping  --out=json
>```
>```
>{
>    "salt-minion1": true
>}
>```

you can use other formats like in --out=
`json yaml txt raw highstate`
see the full list here:  http://docs.saltstack.com/en/latest/ref/output/all/index.html


This is taken from: http://fossies.org/linux/salt/doc/topics/tutorials/walkthrough.rst
