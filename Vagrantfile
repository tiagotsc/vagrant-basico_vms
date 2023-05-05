# -*- mode: ruby -*-
# vi: set ft=ruby  :

# Definindo a quantidade e configuração de cada máquina
machines = {
  "vm1"   => {"memory" => "2048", "cpu" => "2", "ip" => "150", "image" => "centos/7"},
  "vm2"   => {"memory" => "2048", "cpu" => "2", "ip" => "151", "image" => "centos/7"},
  "vm3"   => {"memory" => "2048", "cpu" => "2", "ip" => "152", "image" => "centos/7"}
}

Vagrant.configure("2") do |config|
  # Toda a configuração do Vagrant é feita aqui. A configuração mais comum
  # opções estão documentadas e comentadas abaixo. Para uma referência completa,
  # consulte a documentação online em vagrantup.com.

  # Se true, qualquer conexão SSH feita habilitará o encaminhamento do agente.
  # Valor padrão: false
  config.ssh.forward_agent = true
  
  # Define o usuário padrão SSH
  #config.ssh.username = 'vagrant'
  #config.ssh.password = 'vagrant'
  #config.ssh.insert_key = false

  machines.each do |name, conf|
    config.vm.define "#{name}" do |machine|
	  # Todo ambiente virtual Vagrant requer uma imagem para construir.
      machine.vm.box = "#{conf["image"]}"
	  # Definindo o hostname da VM
      machine.vm.hostname = "#{name}"
	  # Crie uma rede privada, que permite acesso somente de host à máquina
      # usando um IP específico.
      machine.vm.network "private_network", ip: "192.168.56.#{conf["ip"]}"
      # Crie uma rede pública, que geralmente corresponda à rede em ponte.
      # As redes em ponte fazem com que a máquina apareça como outro dispositivo físico na
      # sua rede.
      # machine.vm.network "public_network", ip: "192.168.0.#{conf["ip"]}"
      machine.vm.provider "virtualbox" do |vb|
	    # Nome VM
        vb.name = "#{name}"
		# Define a memória RAM VM
        vb.memory = conf["memory"]
		# Defina a quantidade de CPUs VM
        vb.cpus = conf["cpu"]
		# Nome do grupo no VirtualBox que conterá as VMs
        vb.customize ["modifyvm", :id, "--groups", "/GrupoVms"]
      end
	  # Executa shell script inline
 	  machine.vm.provision "shell", inline: "hostnamectl set-hostname #{name}"
	  config.vm.provision "shell", inline: <<-SHELL
	  HOSTS=$(head -n7 /etc/hosts)
	  echo -e "$HOSTS" > /etc/hosts
	  echo '192.168.56.#{conf["ip"]} #{name}' >> /etc/hosts
	  # Habilita acesso SSH através de senha
	  sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
	  systemctl restart sshd
	  SHELL
    end
  end
  # Executa shell script externo, para o caso de querer subir as VMs instalando pacotes, configurando serviços e aplicações
  #config.vm.provision "shell", path: "script.sh"
end