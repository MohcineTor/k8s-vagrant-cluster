Vagrant.configure("2") do |config|
    config.vm.box = "debian/bullseye64"

    # Configuration du node master 
    config.vm.define "master" do |master|
        master.vm.hostname = "master"
        master.vm.network "private_network", ip: "192.168.56.10"
        master.vm.provider "virtualbox" do |vb|
            vb.memory = "2048"
            vb.cpus = 2
        end
    end

    # Configuration des nodes worker
    (1..2).each do |i|
        config.vm.define "worker#{i}" do |worker|
            worker.vm.hostname = "worker#{i}"
            worker.vm.network "private_network", ip: "192.168.56.1#{i}"
            worker.vm.provider "virtualbox" do |vb|
                vb.memory = "2048"
                vb.cpus = 2
            end
        end
    end
end
