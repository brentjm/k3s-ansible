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

  vm.provision "ansible", run: 'once' do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "playbook/site.yml"
    ansible.verbose = "vvv"
    ansible.groups = {
      "server" => NODE_ROLES.grep(/^server/),
      "agent" => NODE_ROLES.grep(/^agent/),
      "k3s_cluster:children" => ["server", "agent"],
    }
    ansible.extra_vars = {
      k3s_version: "v1.26.9+k3s1",
      api_endpoint: "#{NETWORK_PREFIX}.101",
      token: "myvagrant",
      # Required to use the private network configured above
      extra_server_args: "--node-external-ip #{node_ip} --flannel-iface eth1", 
      extra_agent_args: "--node-external-ip #{node_ip} --flannel-iface eth1",
      # Optional, left as reference for ruby-ansible syntax
      # extra_service_envs: [ "NO_PROXY='localhost'" ],
      # server_config_yaml: <<~YAML
      #   write-kubeconfig-mode: 644
      #   kube-apiserver-arg:
      #     - advertise-port=1234
      # YAML
    }
  end
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
