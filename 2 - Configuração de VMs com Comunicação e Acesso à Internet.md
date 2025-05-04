# 2 - Configuração de VMs com Comunicação e Acesso à Internet

Este documento descreve o processo completo para configurar três máquinas virtuais (VMs) no Virt-Manager/QEMU, garantindo que:

- Todas as VMs consigam **se pingar**.
    
- Todas as VMs tenham **acesso à Internet**.x
    

## 1. Instalação dos pacotes necessários no Host (Linux Mint)

```bash
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

## 2. Criação da Interface Virtual macvlan

No Host, execute:

```bash
sudo ip link add link wlp0s20f3 name macvlan0 type macvlan mode bridge
sudo ip addr add 192.168.1.100/24 dev macvlan0
sudo ip link set macvlan0 up
```

Notas:

- `wlp0s20f3` é a interface Wi-Fi real (confirme usando `ip a`).
    
- `192.168.1.100` é o IP fixo para comunicação com as VMs.
    

## 3. Criação das VMs

Criar três VMs pelo `virt-manager`:

- Usar a ISO do Ubuntu 24.04.
    
- 4096 MiB RAM, 2 CPUs, 25 GiB de disco.
    
- Nome das VMs:
    
    - `ubuntu24.04` (VM1)
        
    - `ubuntu24.04-2` (VM2)
        
    - `ubuntu24.04-3` (VM3)
        

**Importante:** Configurar duas interfaces de rede para cada VM:

## 4. Configuração de Rede nas VMs (via `virsh edit`)

Editar cada VM com:

```bash
virsh edit <nome-da-vm>
```

### XML para cada VM:

#### VM1 (`ubuntu24.04`)

```xml
<interface type='direct'>
  <mac address='52:54:00:3b:7d:cd'/>
  <source dev='wlp0s20f3' mode='bridge'/>
  <model type='virtio'/>
</interface>
<interface type='network'>
  <mac address='52:54:00:aa:bb:c1'/>
  <source network='default'/>
  <model type='virtio'/>
</interface>
```

#### VM2 (`ubuntu24.04-2`)

```xml
<interface type='direct'>
  <mac address='52:54:00:4c:8e:de'/>
  <source dev='wlp0s20f3' mode='bridge'/>
  <model type='virtio'/>
</interface>
<interface type='network'>
  <mac address='52:54:00:aa:bb:c2'/>
  <source network='default'/>
  <model type='virtio'/>
</interface>
```

#### VM3 (`ubuntu24.04-3`)

```xml
<interface type='direct'>
  <mac address='52:54:00:5d:9f:ef'/>
  <source dev='wlp0s20f3' mode='bridge'/>
  <model type='virtio'/>
</interface>
<interface type='network'>
  <mac address='52:54:00:aa:bb:c3'/>
  <source network='default'/>
  <model type='virtio'/>
</interface>
```

Salve e feche o editor (`:wq`).

## 5. Startar as VMs

```bash
virsh start ubuntu24.04
virsh start ubuntu24.04-2
virsh start ubuntu24.04-3
```

## 6. Configuração de IPs fixos nas VMs

Dentro de cada VM:

### Na VM1:

```bash
sudo ip addr add 192.168.1.201/24 dev enp1s0
sudo ip link set enp1s0 up
```

### Na VM2:

```bash
sudo ip addr add 192.168.1.202/24 dev enp1s0
sudo ip link set enp1s0 up
```

### Na VM3:

```bash
sudo ip addr add 192.168.1.203/24 dev enp1s0
sudo ip link set enp1s0 up
```

A interface `enp2s0` (NAT) pegará automaticamente IP via DHCP para acesso à internet.

## 7. Testes de Conectividade

Dentro das VMs, testar:

- Pings entre VMs (através da rede Macvtap):
    

```bash
ping 192.168.1.201
ping 192.168.1.202
ping 192.168.1.203
```

- Ping para o Host (192.168.1.100):
    

```bash
ping 192.168.1.100
```

- Ping para a Internet (Google):
    

```bash
ping 8.8.8.8
ping google.com
```

## 8. Resumo Final

|VM|IP Fixo (enp1s0)|Internet (enp2s0)|
|:--|:--|:--|
|VM1|192.168.1.201|DHCP NAT|
|VM2|192.168.1.202|DHCP NAT|
|VM3|192.168.1.203|DHCP NAT|

---

Pronto! Agora todas as VMs se comunicam diretamente e possuem acesso à Internet.