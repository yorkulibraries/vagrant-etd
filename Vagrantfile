app = ENV['APP'] || 'etd'

rails_env = ENV['RAILS_ENV'] || 'development'

Vagrant.configure("2") do |config|
  config.vm.hostname = "vagrant"
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.define "vagrant-#{app}"
  config.vm.provider "virtualbox" do |vb|
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
    vb.name = "vagrant-#{app}"
  end
  config.vm.network :private_network, ip: "192.168.168.168"
  config.vm.provision "ansible" do |ansible| 
    ansible.playbook="vagrant_provision.yml" 
    ansible.extra_vars = {
      rails_env: rails_env,
      mysql_host: "localhost"
    } 
  end 

  if rails_env == 'development'
    config.vm.synced_folder "#{app}", "/vagrant/#{app}", mount_options: ["dmode=775,fmode=664"]
    config.vm.synced_folder "#{app}/bin", "/vagrant/#{app}/bin", mount_options: ["dmode=775,fmode=775"]
  end

end
