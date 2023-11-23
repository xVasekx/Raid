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
                },
                :sata5 => {
                  :dfile => './sata5.vdi',
                  :size => 250,
                  :port => 5
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
            mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
            apt install -y mdadm smartmontools hdparm gdisk parted
            mdadm --zero-superblock --force /dev/sd{b,c,d,e}
            mdadm --create --verbose --force /dev/md0 -l 10 -n 4 /dev/sd{b,c,d,e}
            echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
            mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
            parted -s /dev/md0 mklabel gpt
            parted /dev/md0 mkpart primary ext4 0% 20%
            parted /dev/md0 mkpart primary ext4 20% 40%
            parted /dev/md0 mkpart primary ext4 40% 60%
            parted /dev/md0 mkpart primary ext4 60% 80%
            parted /dev/md0 mkpart primary ext4 80% 100%
            for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
            mkdir -p /raid/part{1,2,3,4,5}
            for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
            echo "#NEW DEVICE" >> /etc/fstab
            for i in $(seq 1 5); do echo `sudo blkid /dev/md0p$i | awk '{print $2}'` /u0$i ext4 defaults 0 0 >> /etc/fstab; done
            SHELL
    end
  end
end
