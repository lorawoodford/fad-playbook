# -*- mode: ruby -*-
# vi: set ft=ruby :

# set version of ubuntu server and path to inventory for vagrant builds
BOX       = ENV.fetch('FAD_VAGRANT_BOX', 'ubuntu/xenial64')
INVENTORY = ENV.fetch('FAD_VAGRANT_INVENTORY', 'inventory/local/hosts')

PLAYS = [
  # 'app',
  { file: 'solr', restart: {} },
].freeze

Vagrant.configure('2') do |config|
  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.define 'fad' do |fad|
    fad.vm.box           = BOX
    config.disksize.size = '20GB'
    fad.vm.hostname      = 'fad'
    # http://10.11.12.200:8983
    fad.vm.network :private_network, ip: '10.11.12.200'

    fad.vm.provider 'virtualbox' do |v|
      v.customize ['modifyvm', :id, '--memory', 4096]
      unless RUBY_PLATFORM.downcase.include?('mswin')
        v.customize [
          'modifyvm', :id, '--cpus',
          `awk "/^processor/ {++n} END {print n}" /proc/cpuinfo 2> /dev/null || sh -c 'sysctl hw.logicalcpu 2> /dev/null || echo ": 2"' | awk \'{print \$2}\' `.chomp
        ]
      end
    end
  end

  config.vm.provision "shell", inline: "sudo apt-get -y install python-minimal swapspace"

  PLAYS.each do |play|
    config.vm.provision :ansible do |ansible|
      ansible.playbook = "#{play[:file]}.yml"
      ansible.inventory_path = INVENTORY
      ansible.limit = 'all'
    end
  end
end
