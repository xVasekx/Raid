MACHINES = {
  :"raid" => {
              :box_name => "generic/debian10",
              :box_version => "4.3.6",
              :cpus => 2,
              :memory => 1024,
              :ip_addr => '192.168.56.150',
              :disks => {
                :sata1 => {
                  :dfile => './sata1.vdi',
                  :size => 250,
                  :port => 1
                },
                :sata2 => {
                  :dfile => './sata2.vdi',
                  :size => 250,
                  :port => 2
                },
                :sata3 => {
                  :dfile => './sata3.vdi',
                  :size => 250,
                  :port => 3
                },
                :sata4 => {
                  :dfile => './sata4.vdi',
                  :size => 250,
                  :port => 4
                }
              }
            },
          }

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s
      box.vm.network "private_network", ip: boxconfig[:ip_addr]
      box.vm.provider "virtualbox" do |vb|
        vb.memory = boxconfig[:memory]
        vb.cpus = boxconfig[:cpus]
#        vb.customize ['storagectl', :id, '--name', "OSD Controller", '--add', 'sas' ]
	boxconfig[:disks].each do |dname, dconf|
	  unless File.exist?(dconf[:dfile])
	    vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
          end
          vb.customize ['storageattach', :id,  '--storagectl', "SATA Controller", '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
	end
      end
      box.vm.provision "shell", inline: <<-SHELL
            apt install -y mdadm smartmontools hdparm gdisk
            SHELL
    end
  end
end
