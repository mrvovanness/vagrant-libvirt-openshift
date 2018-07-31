cluster = [
  {
    name: 'master',
    memory: 8192,
    cpus: 2,
  },
  {
    name: 'node1',
    memory: 4096,
    cpus: 1,
  },
  {
    name: 'node2',
    memory: 4096,
    cpus: 1,
  },
  {
    name: 'coordinator',
    memory: 2048,
    cpus: 1,
  },

]

cluster_domain = '.openshift.local'

script = <<-SHELL
  yum install dnsmasq bind-utils git -y
  master_ip=$(host master | awk '{print $4}')
  cp -f /vagrant/nm.conf /etc/NetworkManager/conf.d/dns.conf || true
  echo "address=/apps#{cluster_domain}/$master_ip"  > /etc/NetworkManager/dnsmasq.d/dns.conf
  chmod 664 /etc/NetworkManager/conf.d/dns.conf
  chmod 664 /etc/NetworkManager/dnsmasq.d/dns.conf
  service NetworkManager restart
  sed -i '/openshift.local/d' /etc/hosts
SHELL

Vagrant.configure('2') do |config|

  config.vm.provision :shell, inline: script
  config.vm.box = 'centos/7'
  config.vm.synced_folder '.', '/vagrant', type: 'rsync',
    rsync__args: ['--delete', '--recursive', '--include', '.vagrant**'],
    rsync__auto: true

  cluster.each do |cluster_member|
    config.vm.define cluster_member[:name] do |platform|
      platform.vm.hostname = cluster_member[:name] + cluster_domain
      platform.vm.provider :libvirt do |domain|
        domain.memory = cluster_member[:memory]
        domain.cpus = cluster_member[:cpus]
        domain.storage :file, :size => '10G', :type => 'raw'
      end
      if cluster_member[:name] == 'coordinator'
        platform.vm.provision :shell, path: 'local_setup.sh'
        ['prerequisites', 'deploy_cluster'].each do |playbook|
          platform.vm.provision :ansible_local do |ansible|
            ansible.playbook = "/home/vagrant/openshift-ansible/playbooks/#{playbook}.yml"
            ansible.limit = 'all'
            ansible.become = true
            ansible.verbose = true
            ansible.inventory_path = '/vagrant/inventory/hosts'
          end
        end
      end
    end
  end
end
