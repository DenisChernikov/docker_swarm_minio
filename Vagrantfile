NODES = [
    { :hostname => "api", :ip => "192.168.0.207" },
    { :hostname => "api1", :ip => "192.168.0.208" },
    { :hostname => "api2", :ip => "192.168.0.209" },
    { :hostname => "api3", :ip => "192.168.0.210" }
]

Vagrant.configure("2") do |config|
  NODES.each do |node|
    config.vm.define node[:hostname] do |nodeconfig|
      nodeconfig.vm.hostname = node[:hostname]
      nodeconfig.vm.box = "generic/centos8"
      nodeconfig.vm.synced_folder ".", "/vagrant", disabled: true
      nodeconfig.vm.synced_folder "app/", "/opt/app"
      nodeconfig.vm.network "public_network", bridge: "Intel(R) Dual Band Wireless-AC 8260", ip: node[:ip]
      nodeconfig.vm.provider "virtualbox" do |v|
        v.cpus = 1
        v.memory = "1024"
      end
      nodeconfig.vm.provision "shell", inline: <<-SHELL
        yum install -y yum-utils
        yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
        systemctl enable --now docker
        systemctl enable --now containerd
        firewall-cmd --zone=public --add-masquerade --permanent
        firewall-cmd --add-port=2376/tcp --permanent
        firewall-cmd --add-port=7946/tcp --permanent
        firewall-cmd --add-port=7946/udp --permanent
        firewall-cmd --add-port=4789/udp --permanent
        firewall-cmd --reload
      SHELL
      if node[:hostname] == NODES[0][:hostname]
        nodeconfig.vm.provision "shell", env: {"IPADDR" => node[:ip]}, inline: <<-SHELL
          firewall-cmd --permanent --add-port=2377/tcp --permanent
          firewall-cmd --reload
          docker swarm init --advertise-addr $IPADDR | head -5 | tail -1 | sed -e 's/^[ \t]*//' > /opt/app/worker
          docker swarm join-token manager | head -3 | tail -1 | sed -e 's/^[ \t]*//' > /opt/app/manager
        SHELL
      elsif node[:hostname] == NODES[-1][:hostname]
        nodeconfig.vm.provision "shell", inline: <<-SHELL
          firewall-cmd --permanent --add-port=2377/tcp --permanent
          firewall-cmd --reload
          bash /opt/app/manager
          docker stack deploy -c /opt/app/minio.yml minio
          docker stack deploy -c /opt/app/viz.yml viz
          rm -f /opt/app/manager /opt/app/worker
        SHELL
      else
        nodeconfig.vm.provision "shell", inline: <<-SHELL
          bash /opt/app/worker
        SHELL
      end
  end
  end
end