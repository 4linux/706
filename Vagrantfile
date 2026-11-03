# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# Laboratório LPIC-2
#

Vagrant.configure("2") do |config|

  config.vm.box_check_update = true

  config.vm.synced_folder ".", "/vagrant"


  # ============================================================
  # ADMIN CLIENT
  # ============================================================

  config.vm.define "admin-client" do |admin|

    admin.vm.box = "bento/ubuntu-24.04"
    admin.vm.hostname = "admin-client"

    # Rede de administração.
    admin.vm.network "private_network",
      ip: "192.168.56.10"

    # LAN interna.
    admin.vm.network "private_network",
      ip: "172.16.56.10",
      virtualbox__intnet: "lpic2-lan"

    # Disco principal.
    admin.vm.disk :disk,
      size: "40GB",
      primary: true

    admin.vm.provider "virtualbox" do |vb|

      vb.name = "admin-client"
      vb.memory = 3072
      vb.cpus = 2
      vb.gui = false

    end
  end


  # ============================================================
  # INFRA CORE SERVER
  # ============================================================

  config.vm.define "infra-core-server" do |infra|

    infra.vm.box = "oraclelinux/9"

    infra.vm.box_url =
      "https://oracle.github.io/vagrant-projects/boxes/oraclelinux/9.json"

    infra.vm.hostname = "infra-core-server"


    # ----------------------------------------------------------
    # Redes
    # ----------------------------------------------------------

    #
    # Adapter 1:
    # NAT da box Oracle.
    # Permanece virtio para garantir o primeiro acesso SSH.
    #
    # Adapter 2:
    # Rede de administração.
    # Intel PRO/1000 MT Desktop.
    #
    infra.vm.network "private_network",
      ip: "192.168.56.11",
      nic_type: "82540EM",
      auto_config: false

    #
    # Adapter 3:
    # LAN interna.
    # Intel PRO/1000 MT Desktop.
    #
    infra.vm.network "private_network",
      ip: "172.16.56.11",
      virtualbox__intnet: "lpic2-lan",
      nic_type: "82540EM",
      auto_config: false


    # ----------------------------------------------------------
    # Armazenamento
    # ----------------------------------------------------------

    infra.vm.disk :disk,
      name: "infra-core-disk1",
      size: "5GB"

    infra.vm.disk :disk,
      name: "infra-core-disk2",
      size: "5GB"

    infra.vm.disk :disk,
      name: "infra-core-disk3",
      size: "5GB"

    infra.vm.disk :disk,
      name: "infra-core-disk4",
      size: "5GB"


    # ----------------------------------------------------------
    # VirtualBox
    # ----------------------------------------------------------

    infra.vm.provider "virtualbox" do |vb|

      vb.name = "infra-core-server"
      vb.memory = 3072
      vb.cpus = 2
      vb.gui = false

    end


    # ----------------------------------------------------------
    # Driver e1000 e interfaces Oracle Linux
    # ----------------------------------------------------------

    infra.vm.provision "shell",
      privileged: true,
      inline: <<-SHELL

        echo "Instalando módulos adicionais do kernel UEK..."

        dnf install -y \
          kernel-uek-modules-$(uname -r)

        echo "Carregando módulo e1000..."

        modprobe e1000

        udevadm settle


        #
        # A box Oracle Linux pode enumerar as interfaces como
        # eth1, eth2, eth3 em ordem diferente da ordem dos
        # adapters do VirtualBox.
        #
        # Por isso identificamos cada interface através do nome
        # previsível baseado no caminho PCI:
        #
        # Adapter 2 = enp0s8
        # Adapter 3 = enp0s9
        #

        find_interface()
        {
          TARGET="$1"

          for DEVPATH in /sys/class/net/*; do

            DEV="$(basename "${DEVPATH}")"

            [ "${DEV}" = "lo" ] && continue

            if udevadm info \
              -q property \
              -p "${DEVPATH}" \
              | grep -q "^ID_NET_NAME_PATH=${TARGET}$"
            then
              echo "${DEV}"
              return 0
            fi

          done

          return 1
        }


        configure_interface()
        {
          PATH_NAME="$1"
          ADDRESS="$2"
          CONNECTION="$3"

          IFACE="$(find_interface "${PATH_NAME}")"

          if [ -z "${IFACE}" ]; then
            echo "ERRO: interface ${PATH_NAME} não encontrada."
            exit 1
          fi

          echo \
            "Configurando ${PATH_NAME} (${IFACE}) com ${ADDRESS}..."

          nmcli connection delete \
            "${CONNECTION}" \
            >/dev/null 2>&1 || true

          nmcli connection add \
            type ethernet \
            ifname "${IFACE}" \
            con-name "${CONNECTION}" \
            ipv4.method manual \
            ipv4.addresses "${ADDRESS}" \
            ipv4.never-default yes \
            ipv6.method disabled

          nmcli connection up \
            "${CONNECTION}"
        }


        # Adapter 2 - Rede de administração.
        configure_interface \
          enp0s8 \
          192.168.56.11/24 \
          "lpic2-admin"

        # Adapter 3 - LAN.
        configure_interface \
          enp0s9 \
          172.16.56.11/24 \
          "lpic2-lan"


        echo "Interfaces configuradas:"

        ip -br addr

        echo "Rotas configuradas:"

        ip route

      SHELL
  end


  # ============================================================
  # APPLICATION SERVER
  # ============================================================

  config.vm.define "application-server" do |app|

    app.vm.box = "bento/ubuntu-24.04"
    app.vm.hostname = "application-server"

    # Rede de administração.
    app.vm.network "private_network",
      ip: "192.168.56.12"

    # DMZ interna.
    app.vm.network "private_network",
      ip: "10.20.30.10",
      virtualbox__intnet: "lpic2-dmz"

    # Disco principal.
    app.vm.disk :disk,
      size: "40GB",
      primary: true

    # Discos adicionais.
    app.vm.disk :disk,
      name: "application-disk1",
      size: "5GB"

    app.vm.disk :disk,
      name: "application-disk2",
      size: "5GB"

    app.vm.provider "virtualbox" do |vb|

      vb.name = "application-server"
      vb.memory = 3072
      vb.cpus = 2
      vb.gui = false

    end
  end


  # ============================================================
  # GATEWAY SERVER
  # ============================================================

  config.vm.define "gateway-server" do |gw|

    gw.vm.box = "oraclelinux/9"

    gw.vm.box_url =
      "https://oracle.github.io/vagrant-projects/boxes/oraclelinux/9.json"

    gw.vm.hostname = "gateway-server"


    # ----------------------------------------------------------
    # Redes
    # ----------------------------------------------------------

    #
    # Adapter 1:
    # NAT da box Oracle.
    # Permanece virtio.
    #
    # Adapter 2:
    # Rede de administração.
    # Intel PRO/1000 MT Desktop.
    #
    gw.vm.network "private_network",
      ip: "192.168.56.254",
      nic_type: "82540EM",
      auto_config: false

    #
    # Adapter 3:
    # LAN interna.
    # Intel PRO/1000 MT Desktop.
    #
    gw.vm.network "private_network",
      ip: "172.16.56.254",
      virtualbox__intnet: "lpic2-lan",
      nic_type: "82540EM",
      auto_config: false

    #
    # Adapter 4:
    # DMZ interna.
    # Intel PRO/1000 MT Desktop.
    #
    gw.vm.network "private_network",
      ip: "10.20.30.254",
      virtualbox__intnet: "lpic2-dmz",
      nic_type: "82540EM",
      auto_config: false


    # ----------------------------------------------------------
    # VirtualBox
    # ----------------------------------------------------------

    gw.vm.provider "virtualbox" do |vb|

      vb.name = "gateway-server"
      vb.memory = 2048
      vb.cpus = 2
      vb.gui = false

    end


    # ----------------------------------------------------------
    # Driver e1000 e interfaces Oracle Linux
    # ----------------------------------------------------------

    gw.vm.provision "shell",
      privileged: true,
      inline: <<-SHELL

        echo "Instalando módulos adicionais do kernel UEK..."

        dnf install -y \
          kernel-uek-modules-$(uname -r)

        echo "Carregando módulo e1000..."

        modprobe e1000

        udevadm settle


        #
        # Não dependemos dos nomes eth1, eth2 e eth3.
        #
        # Os adapters são identificados pelo caminho PCI:
        #
        # Adapter 2 = enp0s8  - administração
        # Adapter 3 = enp0s9  - LAN
        # Adapter 4 = enp0s10 - DMZ
        #

        find_interface()
        {
          TARGET="$1"

          for DEVPATH in /sys/class/net/*; do

            DEV="$(basename "${DEVPATH}")"

            [ "${DEV}" = "lo" ] && continue

            if udevadm info \
              -q property \
              -p "${DEVPATH}" \
              | grep -q "^ID_NET_NAME_PATH=${TARGET}$"
            then
              echo "${DEV}"
              return 0
            fi

          done

          return 1
        }


        configure_interface()
        {
          PATH_NAME="$1"
          ADDRESS="$2"
          CONNECTION="$3"

          IFACE="$(find_interface "${PATH_NAME}")"

          if [ -z "${IFACE}" ]; then
            echo "ERRO: interface ${PATH_NAME} não encontrada."
            exit 1
          fi

          echo \
            "Configurando ${PATH_NAME} (${IFACE}) com ${ADDRESS}..."

          nmcli connection delete \
            "${CONNECTION}" \
            >/dev/null 2>&1 || true

          nmcli connection add \
            type ethernet \
            ifname "${IFACE}" \
            con-name "${CONNECTION}" \
            ipv4.method manual \
            ipv4.addresses "${ADDRESS}" \
            ipv4.never-default yes \
            ipv6.method disabled

          nmcli connection up \
            "${CONNECTION}"
        }


        # Adapter 2 - Rede de administração.
        configure_interface \
          enp0s8 \
          192.168.56.254/24 \
          "lpic2-admin"

        # Adapter 3 - LAN.
        configure_interface \
          enp0s9 \
          172.16.56.254/24 \
          "lpic2-lan"

        # Adapter 4 - DMZ.
        configure_interface \
          enp0s10 \
          10.20.30.254/24 \
          "lpic2-dmz"


        echo "Interfaces configuradas:"

        ip -br addr

        echo "Rotas configuradas:"

        ip route

      SHELL
  end


  # ============================================================
  # /etc/hosts
  # ============================================================

  config.vm.provision "shell",
    privileged: true,
    inline: <<-SHELL

      sed -i \
        '/# LPIC2-LAB-BEGIN/,/# LPIC2-LAB-END/d' \
        /etc/hosts

      cat >> /etc/hosts <<'EOF'

# LPIC2-LAB-BEGIN

# Rede de administracao
192.168.56.10   admin-client
192.168.56.11   infra-core-server
192.168.56.12   application-server
192.168.56.254  gateway-server

# LPIC2-LAB-END
EOF

    SHELL

end
