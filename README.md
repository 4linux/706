# Laboratório 706 - Preparatório LPIC-2

Repositório para armazenar o Laboratório do curso **706 - Preparatório LPIC-2** da [4Linux][1]

Dependências
------------

Para a criação do laboratório é necessário ter pré instalado os seguintes softwares:

* [Git][2]
* [VirtualBox][3]
* [Vagrant][4]

> Para o máquinas com Windows aconselhamos, se possível, que as instalações sejam feitas pelo gerenciador de pacotes **[Cygwin][5]**.

> Para as máquinas com MAC OS aconselhamos, se possível, que as instalações sejam feitas pelo gerenciador de pacotes **brew**.

Laboratório
-----------

O Laboratório será criado utilizando o [Vagrant][6]. Ferramenta para criar e gerenciar ambientes virtualizados (baseado em Inúmeros providers) com foco em automação.

Nesse laboratório, que está centralizado no arquivo [Vagrantfile][7], sera criada 4 maquinas com a seguinte característica:

| Nome | vCPUs | Memória RAM | IP1 | IP2 | IP3 | S.O.¹ |
|---|:---:|:---:|:---:|:---:|:---:|---:|
| `admin-client` | 2 | 3072MB | `192.168.56.10` | `172.16.56.10` | - | ubuntu-24.04-amd64 |
| `infra-core-server` | 2 | 3072MB | `192.168.56.11` | `172.16.56.11` | - | oracle-9-amd64 |
| `application-server` | 2 | 3072MB | `192.168.56.12` | `10.20.30.10` | - | ubuntu-24.04-amd64 |
| `gateway-server` | 2 | 2048MB | `192.168.56.254` | `172.16.56.254` | `10.20.30.254` | oracle-9-amd64 |

> **¹**: Esses Sistemas operacionais estão sendo utilizado no formato de Boxes, é a forma como o vagrant chama as imagens do sistema operacional utilizado.

Criação do Laboratório 
----------------------

Para criar o laboratório é necessário fazer o `git clone` desse repositório e, dentro da pasta baixada realizar a execução do `vagrant up`, conforme abaixo:

```bash
git clone https://github.com/4linux/706
cd 706
vagrant up
```

_O Laboratório **pode demorar**, dependendo da conexão de internet e poder computacional, para ficar totalmente preparado._

> Em caso de erro na criação das máquinas sempre valide se sua conexão está boa, os logs de erros na tela e, se necessário, o arquivo **/var/log/vagrant_provision.log** dentro da máquina que apresentou a falha.

Por fim, para melhor utilização, abaixo há alguns comandos básicos do vagrant para gerencia das máquinas virtuais.

Comandos                | Descrição
:----------------------:| ---------------------------------------
`vagrant init`          | Gera o VagrantFile
`vagrant box add <box>` | Baixar imagem do sistema
`vagrant box status`    | Verificar o status dos boxes criados
`vagrant up`            | Cria/Liga as VMs baseado no VagrantFile
`vagrant provision`     | Provisiona mudanças logicas nas VMs
`vagrant status`        | Verifica se VM estão ativas ou não.
`vagrant ssh <vm>`      | Acessa a VM
`vagrant ssh <vm> -c <comando>` | Executa comando via ssh
`vagrant reload <vm>`   | Reinicia a VM
`vagrant halt`          | Desliga as VMs

> Para maiores informações acesse a [Documentação do Vagrant][8]

[1]: https://4linux.com.br
[2]: https://git-scm.com/downloads
[3]: https://www.virtualbox.org/wiki/Downloads
[4]: https://www.vagrantup.com/downloads
[5]: https://cygwin.com/install.html
[6]: https://www.vagrantup.com/
[7]: ./Vagrantfile
[8]: https://www.vagrantup.com/docs
