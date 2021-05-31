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
KAFKA_BROKER    = settings['broker_vms']
ZOOKEEPER       = settings['zookeeper_vms']
ZOONAVIGATOR    = settings['zoonavigator_vms']
CONNECT         = settings['connect_vms']
KSQLDB          = settings['ksqldb_vms']
REST_PROXY      = settings['restproxy_vms']
SCHEMA_REGISTRY = settings['schemaregistry_vms']
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
        'brokers'             => (0..KAFKA_BROKER - 1).map { |j| "#{LABEL_PREFIX}broker#{j}" },
        'zookeepers'          => (0..ZOOKEEPER - 1).map { |j| "#{LABEL_PREFIX}ZOOKEEPER#{j}" },
        'zoonavigators'       => (0..ZOONAVIGATOR - 1).map { |j| "#{LABEL_PREFIX}ZOONAVIGATOR#{j}" },
        'connects'            => (0..CONNECT - 1).map { |j| "#{LABEL_PREFIX}CONNECT#{j}" },
        'ksqldbs'             => (0..KSQLDB - 1).map { |j| "#{LABEL_PREFIX}KSQLDB#{j}" },
        'rest_proxies'        => (0..REST_PROXY - 1).map { |j| "#{LABEL_PREFIX}REST_PROXY#{j}" },
        'schema_registries'   => (0..SCHEMA_REGISTRY - 1).map { |j| "#{LABEL_PREFIX}SCHEMA_REGISTRY#{j}" }
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

(0..KAFKA_BROKER - 1).each do |i|
    config.vm.define "#{LABEL_PREFIX}broker#{i}" do |broker|
    broker.vm.hostname = "#{LABEL_PREFIX}broker#{i}"
    if ASSIGN_STATIC_IP
        broker.vm.network :private_network,
        ip: "#{PUBLIC_SUBNET}.#{$last_ip_pub_digit+=1}"
    end
    # Libvirt
    broker.vm.provider :libvirt do |lv|
        lv.memory = MEMORY
        lv.storage_pool_name = 'pool_myhome_SSD'
        # lv.random_hostname = true
        end
    end
end

(0..ZOOKEEPER - 1).each do |i|
    config.vm.define "#{LABEL_PREFIX}zookeeper#{i}" do |zookeeper|
    zookeeper.vm.hostname = "#{LABEL_PREFIX}zookeeper#{i}"
    if ASSIGN_STATIC_IP
        zookeeper.vm.network :private_network,
        ip: "#{PUBLIC_SUBNET}.#{$last_ip_pub_digit+=1}"
    end
    # Libvirt
    zookeeper.vm.provider :libvirt do |lv|
        lv.memory = MEMORY
        lv.storage_pool_name = 'pool_myhome_SSD'
        # lv.random_hostname = true
        end
    end
end

(0..ZOONAVIGATOR - 1).each do |i|
    config.vm.define "#{LABEL_PREFIX}zoonavigator#{i}" do |zoonavigator|
    zoonavigator.vm.hostname = "#{LABEL_PREFIX}zoonavigator#{i}"
    if ASSIGN_STATIC_IP
        zoonavigator.vm.network :private_network,
        ip: "#{PUBLIC_SUBNET}.#{$last_ip_pub_digit+=1}"
    end
    # Libvirt
    zoonavigator.vm.provider :libvirt do |lv|
        lv.memory = MEMORY
        lv.storage_pool_name = 'pool_myhome_SSD'
        # lv.random_hostname = true
        end

           zoonavigator.vm.provision 'ansible', &ansible_provision if i == (ZOONAVIGATOR - 1)
       end
    end
end