# ENV['VAGRANT_NO_PARALLEL'] = 'no'
NODE_ROLES = ["server-0", "agent-0", "agent-1"]
NODE_BOXES = ['generic/ubuntu2004', 'generic/ubuntu2004', 'generic/ubuntu2004']
SSH_PORTS = [2201, 2202, 2203]
NODE_CPUS = 2
NODE_MEMORY = 2048
# Virtualbox >= 6.1.28 require `/etc/vbox/network.conf` for expanded private networks 
NETWORK_PREFIX = "192.168.56"

def provision(vm, role, node_num)
  vm.box = NODE_BOXES[node_num]
  vm.hostname = role
  vm.provider "virtualbox" do |v|
    v.name = role
  end
  # We use a private network because the default IPs are dynamically assigned 
  # during provisioning. This makes it impossible to know the server-0 IP when 
  # provisioning subsequent servers and agents. A private network allows us to
  # assign static IPs to each node, and thus provide a known IP for the API endpoint.
  node_ip = "#{NETWORK_PREFIX}.#{100+node_num+1}"
  # An expanded netmask is required to allow VM<-->VM communication, virtualbox defaults to /32
  vm.network "private_network", ip: node_ip, netmask: "255.255.255.0"
  vm.network "forwarded_port", guest: 22, host: SSH_PORTS[node_num], id: "ssh"

end

Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |v|
    v.cpus = NODE_CPUS
    v.memory = NODE_MEMORY
    v.linked_clone = true
  end
  
  NODE_ROLES.each_with_index do |name, i|
    config.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{ENV['HOME']}/.ssh/id_rsa.pub").first.strip
      s.inline = <<-SHELL
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      SHELL
    end
    config.vm.define name do |node|
      provision(node.vm, name, i)
    end
  end
end







  # change the default ssh user to be the same as the user on the host
  # this allows us to use the same ssh keys
  # https://www.vagrantup.com/docs/other/environmental-variables#vagrant_default_ssh_user
#  config.ssh.username = ENV['USERNAME']
#  config.ssh.private_key_path = '~/.ssh/id_rsa'
