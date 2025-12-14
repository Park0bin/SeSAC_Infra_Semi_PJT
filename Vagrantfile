# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_NAME = "nobreak-labs/rocky-9"
MEMORY = "2048"
CPUS = 1

NODES = {
  "controller" => {
    networks: [
      { ip: "192.168.56.100" }
    ]
  },
"LB-webs" => {
    networks: [
      { ip: "192.168.56.101" }
    ]
  },
"LB-dbs" => {
    networks: [
      { ip: "192.168.56.102" }
    ]
  },
"WEB-01" => {
    networks: [
      { ip: "192.168.56.201" }
    ]
  },
"WEB-02" => {
    networks: [
      { ip: "192.168.56.202" }
    ]
  },
"DB-01" => {
    networks: [
      { ip: "192.168.56.151" }
    ]
  },
"DB-02" => {
    networks: [
      { ip: "192.168.56.152" }
    ]
  }
}

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false  # 자동 업데이트/체크 비활성화
    config.vbguest.no_remote   = true   # 네트워크로 ISO 받아오는 것도 금지
  end

  NODES.each do |node_name, node_config|
    config.vm.define node_name do |node|
      node.vm.box = BOX_NAME

      node_config[:networks].each do |network|
        node.vm.network "private_network", ip: network[:ip]
      end

      node.vm.synced_folder ".", "/vagrant", disabled: true

      node.vm.provider "virtualbox" do |vb|
        vb.memory = MEMORY
        vb.cpus = CPUS
        vb.name = node_name
      end

      node.vm.hostname = node_name
    end
  end
end