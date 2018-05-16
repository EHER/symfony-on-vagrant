Vagrant.configure("2") do |config|
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.synced_folder ".", "/symfony"
  config.vm.provider "docker" do |d|
    d.build_dir = "."
  end
end
