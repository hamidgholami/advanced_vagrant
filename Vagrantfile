# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'
VAGRANTFILE_API_VERSION = '2'
# current_dir      = File.dirname(File.expand_path(__FILE__))
# configs          = YAML.load_file("#{current_dir}/config.yaml")
# vagrant_config   = configs['configs'][configs['configs']['use']]

# Vagrant.configure('2') do |config|
#     config.vm.network

config_file=File.expand_path(File.join(File.dirname(__FILE__), 'vagrant_variables.yaml'))
settings=YAML.load_file(config_file)

LABEL_PREFIX    = settings['label_prefix'] ? settings['label_prefix'] + "-" : ""
MACHINE_H       = settings['h']
MACHINE_A       = settings['a']
MACHINE_M       = settings['m']
MACHINE_I       = settings['i']
MACHINE_D       = settings['d']
BOX             = ENV['ANSIBLE_VAGRANT_BOX'] || settings['vagrant_box']
BOX_URL         = ENV['ANSIBLE_VAGRANT_BOX_URL'] || settings['vagrant_box_url']
MEMORY          = settings['memory']
CPU             = settings['cpu_count']
PUBLIC_SUBNET   = settings['public_subnet']
CLUSTER_SUBNET  = settings['cluster_subnet']
USER            = settings['ssh_username']
DEBUG           = settings['debug']
ETH             = settings['eth']
SYNC_DIR        = settings['vagrant_sync_dir']
DOCKER          = settings['docker']

$last_ip_pub_digit = 9
$last_ip_cluster_digit = 9

ASSIGN_STATIC_IP = !(BOX == 'openstack' or BOX == 'linode')
DISABLE_SYNCED_FOLDER = settings.fetch('vagrant_disable_synced_folder', false)

ansible_provision = proc do |ansible|
    if DOCKER then
      ansible.playbook = 'site-container.yml'
      if settings['skip_tags']
        ansible.skip_tags = settings['skip_tags']
      end
    else
      ansible.playbook = 'playbook.yml'
    end

    ansible.groups = {
        'h'       => (0..MACHINE_H - 1).map { |j| "#{LABEL_PREFIX}h#{j}" },
        'a'       => (0..MACHINE_A - 1).map { |j| "#{LABEL_PREFIX}a#{j}" },
        'm'       => (0..MACHINE_M - 1).map { |j| "#{LABEL_PREFIX}m#{j}" },
        'i'       => (0..MACHINE_I - 1).map { |j| "#{LABEL_PREFIX}i#{j}" },
        'd'       => (0..MACHINE_D - 1).map { |j| "#{LABEL_PREFIX}d#{j}" },
     }

     ansible.extra_vars = {
        cluster_network: "#{CLUSTER_SUBNET}.0/24",
        journal_size: 100,
        public_network: "#{PUBLIC_SUBNET}.0/24",

    }

    if DEBUG then
        ansible.verbose = '-vvvv'
      end
      ansible.limit = 'all'
    end
    
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = BOX
    config.vm.box_url = BOX_URL
    config.ssh.insert_key = false
    config.ssh.private_key_path = settings['ssh_private_key_path']
    # config.ssh.username = USER

    config.vm.provider :libvirt do |lv|
        lv.cpu_mode = 'host-passthrough'
        lv.disk_driver :cache => 'unsafe'
        lv.graphics_type = 'none'
        lv.cpus = CPU
    end

if DISABLE_SYNCED_FOLDER
    config.vm.provider :virtualbox do |v,override|
        override.vm.synced_folder '.', SYNC_DIR, diabled: true
       end
    config.vm.provider :libvirt do |v,override|
        override.vm.synced_folder '.', SYNC_DIR, diabled: true
       end
end

(0..MACHINE_H - 1).each do |i|
    config.vm.define "#{LABEL_PREFIX}h#{i}" do |h|
    h.vm.hostname = "#{LABEL_PREFIX}h#{i}"
    if ASSIGN_STATIC_IP
        h.vm.network :private_network,
        ip: "#{PUBLIC_SUBNET}.#{$last_ip_pub_digit+=1}"
    end
    # Libvirt
    h.vm.provider :libvirt do |lv|
        lv.memory = MEMORY
        lv.storage_pool_name = 'pool_myhome_SSD'
        # lv.random_hostname = true
        end
    end
end

(0..MACHINE_A - 1).each do |i|
    config.vm.define "#{LABEL_PREFIX}a#{i}" do |a|
    a.vm.hostname = "#{LABEL_PREFIX}a#{i}"
    if ASSIGN_STATIC_IP
        a.vm.network :private_network,
        ip: "#{PUBLIC_SUBNET}.#{$last_ip_pub_digit+=1}"
    end
    # Libvirt
    a.vm.provider :libvirt do |lv|
        lv.memory = MEMORY
        lv.storage_pool_name = 'pool_myhome_SSD'
        # lv.random_hostname = true
        end
    end
end

(0..MACHINE_M - 1).each do |i|
    config.vm.define "#{LABEL_PREFIX}m#{i}" do |m|
    m.vm.hostname = "#{LABEL_PREFIX}m#{i}"
    if ASSIGN_STATIC_IP
        m.vm.network :private_network,
        ip: "#{PUBLIC_SUBNET}.#{$last_ip_pub_digit+=1}"
    end
    # Libvirt
    m.vm.provider :libvirt do |lv|
        lv.memory = MEMORY
        lv.storage_pool_name = 'pool_myhome_SSD'
        # lv.random_hostname = true
        end
    end
end

(0..MACHINE_I - 1).each do |i|
    config.vm.define "#{LABEL_PREFIX}i#{i}" do |i|
    i.vm.hostname = "#{LABEL_PREFIX}i#{i}"
    if ASSIGN_STATIC_IP
        i.vm.network :private_network,
        ip: "#{PUBLIC_SUBNET}.#{$last_ip_pub_digit+=1}"
    end
    # Libvirt
    i.vm.provider :libvirt do |lv|
        lv.memory = MEMORY
        lv.storage_pool_name = 'pool_myhome_SSD'
        # lv.random_hostname = true
        end
    end
end

(0..MACHINE_D - 1).each do |i|
    config.vm.define "#{LABEL_PREFIX}d#{i}" do |d|
    d.vm.hostname = "#{LABEL_PREFIX}d#{i}"
    if ASSIGN_STATIC_IP
        d.vm.network :private_network,
        ip: "#{PUBLIC_SUBNET}.#{$last_ip_pub_digit+=1}"
    end
    # Libvirt
    d.vm.provider :libvirt do |lv|
        lv.memory = MEMORY
        lv.storage_pool_name = 'pool_myhome_SSD'
        # lv.random_hostname = true
        end

           d.vm.provision 'ansible', &ansible_provision if i == (MACHINE_D - 1)
       end
    end
end