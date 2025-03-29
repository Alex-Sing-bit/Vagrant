# -*- mode: ruby -*-
# vi: set ft=ruby :
# Magic-переменная окружения, которая укажет сервер, где хранятся боксы для Vagrant (обходим блокировку)
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'


BoxName = "bento/ubuntu-24.04"
# Название провайдера. По умолчанию -- VirtualBox
Provider = "virtualbox"
# Путь до публичного ключа. Генерируется через утилиту ssh-keygen
SSH_PUB_KEY = "./lab.pub"
# Vagrantfile поддерживает синтаксис Ruby, создаем структуру, которая описывает наши виртуальные машины
VirtMachine = Struct.new(:name, :ip, :cpus, :memory)
# Объект VM для виртуальной машины с Ansible
ansible = VirtMachine.new("ansible-host", "10.10.10.20", 2,
2048)
# Объект VM для Worker, где будет исполняться код
worker = VirtMachine.new("task-service", "10.10.10.30", 4,
2048)
# Делаем список из двух наших объектов - виртуальных машин
vms = [ansible, worker]

# Определяем конфигурацию Vagrant версии 2
Vagrant.configure("2") do |config|
# Проходим по каждой виртуальной машине в foreach цикле. Каждая машина в переменной i
    vms.each do |i|
        config.vm.define "server#{i.name}" do |cfg|
            # Настраиваем сеть с статическим IP
            # cfg.vm.network "private_network", ip: "#{i.ip}"
            cfg.vm.network "private_network", virtualbox__intnet: "private_network", auto_config: false

            # Выбираем базовую коробку
            cfg.vm.box = "#{BoxName}"

            # Устанавливаем hostname
            cfg.vm.hostname = "#{i.name}.local"

            # Подкладываем созданный нами заранее SSH ключ (публичную часть) на виртуальную машину, копируя на хост
            cfg.vm.provision "file", source: "#{SSH_PUB_KEY}", destination: "~/.ssh/ansible_key.pub"
            # И теперь записываем его в Authorized Keys
            cfg.vm.provision "shell", inline: "cat /home/vagrant/.ssh/ansible_key.pub >> /home/vagrant/.ssh/authorized_keys"

            cfg.vm.provision "shell", inline: <<-SHELL
                # Отключаем systemd-networkd
                systemctl stop systemd-networkd
                systemctl disable systemd-networkd

                # Настраиваем /etc/network/interfaces
                cat > /etc/network/interfaces 
                EOF
                # The loopback network interface
                auto lo
                iface lo inet loopback

                # The primary network interface
                auto eth0
                iface eth0 inet dhcp

                # Static IP configuration
                auto eth1
                iface eth1 inet static
                    address #{i.ip}
                    netmask 255.255.255.0
                    network 10.10.10.0
                    broadcast 10.10.10.255
                EOF

                        # Перезапускаем сетевые интерфейсы
                        ifdown eth1 || true
                        ifup eth1
            SHELL

            # Настраиваем параметры провайдера
            cfg.vm.provider "#{Provider}" do |v|
              # Количество доступных для ВМ ядер
              v.cpus = "#{i.cpus}"
              # Количество доступной оперативной памяти
              v.memory = "#{i.memory}"
              # Имя машины в интерфейсе VirtualBox
              v.name = "#{i.name}"
            end
        end
    end
end
