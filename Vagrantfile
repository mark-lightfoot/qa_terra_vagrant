# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'
require 'json'

filename = "config"

if File.file? "#{filename}.json"
    machines = JSON.parse(File.read("#{filename}.json"))
else
    machines = YAML.load_file("#{filename}.yaml")
end

Vagrant.configure("2") do |config| 
  machines.each do |machine|
	config.vm.define "#{machine['name']}" do |new_vm|
		set_box(machine, new_vm)
		set_private_ip(machine, new_vm)
		set_forwarded_ports(machine, new_vm)
		set_cpus_and_memory(machine, new_vm)	
		update_package_manager(machine, new_vm)
		install_packages(machine, new_vm)	
		set_shared_folders(machine, new_vm)
		run_scripts(machine, new_vm)
	end
  end
end

def set_box(machine, new_vm)
	 new_vm.vm.box = "#{machine['box']}"
end

def set_private_ip(machine, new_vm)
    new_vm.vm.nework "private_network", ip: machine['private_ip']
end

def set_forwarded_ports(machine, new_vm)
    unless machine['forwarded_ports'].nil?
	    machine['forwarded_ports'].each do |ports|
                        new_vm.vm.network "forwarded_port", guest: ports['guest'], host: ports['host']
            end
    end
end

def set_cpus_and_memory(machine, new_vm)
	new_vm.vm.provider "virtualbox" do |vb|
	    unless machine['cpus'].nil?
		vb.cpus = "#{machine['cpus']}"
	    end
	    unless machine['memory'].nil?
        	vb.memory = "#{machine['memory']}"
	    end
	end
end

def update_package_manager(machine, new_vm)
    unless machine['package_manager'].nil?	 
	 if machine['package_manager'] == "apt-get"
	         new_vm.vm.provision "shell", inline: "sudo #{machine['package_manager']} upgrade -y"
	 else
		 new_vm.vm.provision "shell", inline: "sudo #{machine['package_manager']} update -y"
	 end
    end
end

def install_packages(machine, new_vm)
    unless machine['packages'].nil?
	machine['packages'].each do |pkg|
                if machine['package_manager'] == "apk"
			new_vm.vm.provision "shell", inline: <<-SHELL
				sudo #{machine['package_manager']} add #{pkg}
			SHELL
		else
			new_vm.vm.provision "shell", inline: <<-SHELL
                		sudo #{machine['package_manager']} install -y "#{pkg}"
        		SHELL
		end
	end
    end
end

def set_shared_folders(machine, new_vm)
    unless machine['shared_folders'].nil?
	machine['shared_folders'].each do |dir|
		new_vm.vm.synced_folder dir['host'], dir['guest']
	end
    end
end

def run_scripts(machine, new_vm)
    unless machine['scripts'].nil?
	machine['scripts'].each do |script|
                        new_vm.vm.provision "shell", path: "vagrant_scripts/#{script}"
        end
    end
end
