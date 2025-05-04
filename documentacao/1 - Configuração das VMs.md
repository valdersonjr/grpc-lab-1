# 1 - Configuração das VMs

## 1. Instalação dos pacotes necessários

```bash
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

## 2. Verificação do status do libvirt

```bash
sudo systemctl status libvirtd
```

Verifique se o serviço está ativo e rodando.

## 3. Criação da Interface Virtual (macvlan)

No Host (Linux Mint):

```bash
sudo ip link add link wlp0s20f3 name macvlan0 type macvlan mode bridge
sudo ip addr add 192.168.1.100/24 dev macvlan0
sudo ip link set macvlan0 up
```

Explicação:

- `wlp0s20f3` é a interface Wi-Fi real. (Pode mudar em outros PCs)
- `macvlan0` é a interface virtual usada para comunicar o Host com as VMs.
- `192.168.1.100` é o IP do Host na rede virtual.

**Observação:** Em outro ambiente (outro PC ou Wi-Fi), você deve:

- Ajustar `wlp0s20f3` para o nome correto da interface Wi-Fi (veja com `ip a`).
- Ajustar a rede se a faixa IP for diferente.

## 4. Criação das Máquinas Virtuais com o Virt-Manager

Para cada VM (VM1, VM2, VM3):

- Abra o `virt-manager`
- Clique em "Create a new virtual machine"
- Escolha "Local install media (ISO image or CDROM)"
- Selecione a ISO do Ubuntu (ex: `ubuntu-24.04.2-desktop-amd64.iso`)
- Configure Memória e CPUs (ex: 4096 MiB RAM, 2 CPUs)
- Crie um disco qcow2 (ex: 25 GiB)
- Configure "Network source" como Macvtap baseado na interface Wi-Fi (`wlp0s20f3`)
- Finalize a criação.

Repita o processo para criar VM1, VM2 e VM3.