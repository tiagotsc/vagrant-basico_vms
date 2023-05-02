
## Vagrant - Provisionando VMs para testes ou desenvolvimento

Criar de forma rápida um conjunto de VMs para testes ou desenvolvimento de algo com tudo já configurado conforme a necessidade.

No exemplo a seguir serão criados 3 VMs no VirtualBox, que terão as seguintes definições:

| Hostname   | IP       |
| :---------- | :--------- |
| vm1 | 192.168.56.150 |
| vm2 | 192.168.56.151 |
| vm3 | 192.168.56.152 |

### Observação

Vagrant só é recomendado para ambientes de testes e desenvolvimento, conforme pode ser visto no link abaixo.

https://developer.hashicorp.com/vagrant/intro/vs/terraform

### Requisitos mínimos

Ter em seu SO os seguintes softwares.
Abaixo de cada um segue o link para download.

- VirtualBox (Software de virtualização)

  https://www.virtualbox.org/wiki/Downloads

- Vagrant (Software para configurar ambientes)

  https://developer.hashicorp.com/vagrant/downloads

- Putty (Software de cliente remoto)

  https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

### Etapas a se seguir

1 - Crie uma pasta com nome de sua preferência em qualquer lugar do seu SO. No meu caso vou chama lá de “**projeto**”.

2 - Entre na pasta “**projeto**” e dentro dela crie um arquivo chamado **Vagrantfile**, apenas o nome sem extensão, adicione o seguinte conteúdo e salve.

![App Screenshot](https://github.com/tiagotsc/vagrant-basico_vms/blob/main/images/img1.png)

Nome arquivo: **Vagrantfile**
```Vargrantfile
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
	# Executa shell script externo, para o caso de querer subir as VMs instalando pacotes, configurando serviços e aplicações
	#config.vm.provision "shell", path: "script.sh"
    end
  end
end
```
3 – Ainda dentro da pasta “**projetos**”, vamos subir nossas VMs, via linha de comando, execute:

![App Screenshot](https://github.com/tiagotsc/vagrant-basico_vms/blob/main/images/img2.png)

```bash
# Liga ou cria as VMs, caso ainda não existam
vagrant up
```
O processo demora um pouco e todos os passos que são executados podem ser acompanhados em tempo real linha de comando.

![App Screenshot](https://github.com/tiagotsc/vagrant-basico_vms/blob/main/images/img3.png)

Vms disponibilizadas no VirtualBox

![App Screenshot](https://github.com/tiagotsc/vagrant-basico_vms/blob/main/images/img4.png)

4 - Quando todo o processo terminar, abra o putty e forneça o IP de umas das VMs, no exemplo, vou acessar a VM1:

IP: 192.168.56.150
Usuário: root ou vagrant
Senha: vagrant

![App Screenshot](https://github.com/tiagotsc/vagrant-basico_vms/blob/main/images/img5.png)

Logado com sucesso. VM pronta para uso!

![App Screenshot](https://github.com/tiagotsc/vagrant-basico_vms/blob/main/images/img6.png)

O mesmo pode ser feito nas outras VMs, caso queira acessá-las.

### Alguns comandos úteis

Via linha de comando, é preciso estar na pasta "**projeto**", pasta aonde está o Vagrantfile,  para executar os comando.

```bash
# Desliga VMs
vagrant up

# Reiniciar VMs
vagrant reload

# Destruir VMs
vagrant destroy
```

#### Documentação oficial do Vagrant

https://developer.hashicorp.com/vagrant/docs
