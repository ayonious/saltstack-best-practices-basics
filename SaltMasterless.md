According to this:
http://docs.saltstack.com/en/latest/topics/tutorials/standalone_minion.html



According to doc to set masterless minion:
1. install only salt-minion
2. set file_client: local
3. dont start minion daemon [Im not sure about this line actually, I think this means sudo service salt-minion restart]



vagrant file:
```perl

$script = <<SCRIPT
  set -x
  sudo apt-get -y install python-software-properties
  sudo apt-get -y install software-properties-common
  sudo add-apt-repository -y ppa:saltstack/salt
  sudo apt-get -y update
	sudo apt-get -y install salt-minion
	sudo bash -c " echo 'file_client: local' >> /etc/salt/minion "
SCRIPT
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.network "private_network", ip: "192.168.33.18"
  config.vm.hostname = "salt-masterlessMinion"
  config.vm.provision :shell, :inline => $script
end
```





give a vagrant up and then vagrant ssh

everything is ready

now create states/pillars as if this is the masterless

when execute just use salt-call instead of salt

salt-call --local state.highstate
