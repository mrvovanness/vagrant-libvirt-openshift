cluster = [
  {
    name: 'master',
    memory: 4096,
    cpus: 2,
  },
  {
    name: 'node1',
    memory: 2048,
    cpus: 1,
  }
]

cluster_domain = '.openshift.local'

script = <<-SHELL
  yum install epel-release dnsmasq bind-utils -y
  master_ip=$(host master | awk '{print $4}')
  cp -f /vagrant/nm.conf /etc/NetworkManager/conf.d/dns.conf || true
  echo "address=/apps#{cluster_domain}/$master_ip"  > /etc/NetworkManager/dnsmasq.d/dns.conf
  chmod 664 /etc/NetworkManager/conf.d/dns.conf
  chmod 664 /etc/NetworkManager/dnsmasq.d/dns.conf
  service NetworkManager restart
SHELL

Vagrant.configure('2') do |config|

  config.vm.provision :shell, inline: script
  config.vm.box = 'centos/7'

  cluster.each do |cluster_member|
    config.vm.define cluster_member[:name] do |platform|
      platform.vm.hostname = cluster_member[:name] + cluster_domain
      platform.vm.provider :libvirt do |domain|
        domain.memory = cluster_member[:memory]
        domain.cpus = cluster_member[:cpus]
      end
    end
  end
end
