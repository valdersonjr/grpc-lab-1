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

Verifique se o serviço está **ativo** e **rodando**.

## 3. Criação da Interface Virtual (dic
doc
macvlan)

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
    
- Ajustar a rede e gateway se a faixa IP for diferente.
    

## 4. Criação das Máquinas Virtuais com o Virt-Manager

Para cada VM (VM1, VM2, VM3):

- Abra o `virt-manager`
- Clique em **"Create a new virtual machine"**
- Escolha **"Local install media (ISO image or CDROM)"**
- Selecione a ISO do Ubuntu (ex: `ubuntu-24.04.2-desktop-amd64.iso`)
- Configure Memória e CPUs (ex: 4096 MiB RAM, 2 CPUs)
- Crie um disco qcow2 (ex: 25 GiB)
- Configure **Network source** como **Macvtap** baseado na interface Wi-Fi (`wlp0s20f3`)
- Finalize a criação.
    

Repita o processo para criar **VM1**, **VM2** e **VM3**.

## 5. Configuração de Rede nas VMs
**Importante:** Cada VM precisa de um IP diferente, e a rota deve apontar para o Gateway da rede (normalmente o IP do roteador).

### VM1:

```bash
sudo ip addr add 192.168.1.201/24 dev enp1s0
sudo ip link set enp1s0 up
sudo ip route add default via 192.168.1.1
```

### VM2:

```bash
sudo ip addr add 192.168.1.202/24 dev enp1s0
sudo ip link set enp1s0 up
sudo ip route add default via 192.168.1.1
```

### VM3:

```bash
sudo ip addr add 192.168.1.203/24 dev enp1s0
sudo ip link set enp1s0 up
sudo ip route add default via 192.168.1.1
```

## 6. Testes de Conectividade

Dentro das VMs:

- Teste ping entre VM1, VM2 e VM3:
    

```bash
ping 192.168.1.202  # VM1 para VM2
ping 192.168.1.203  # VM1 para VM3
ping 192.168.1.201  # VM2 para VM1
ping 192.168.1.203  # VM2 para VM3
ping 192.168.1.201  # VM3 para VM1
ping 192.168.1.202  # VM3 para VM2
```

- Teste ping entre cada VM e o Host (Linux Mint):
    

```bash
ping 192.168.1.100  # (Host via macvlan0)
```

- Teste ping do Host para cada VM:
    

```bash
ping 192.168.1.201
ping 192.168.1.202
ping 192.168.1.203
```

## 7. Adaptando para outro Ambiente (Checklist)

Se for usar outro PC ou outra rede Wi-Fi:

1. **Verificar nome da interface Wi-Fi** no Host:
    

```bash
ip a
```

2. **Verificar o IP do Gateway** (roteador) no Host:
    

```bash
ip route
```

3. **Criar o macvlan** baseado na interface Wi-Fi correta:
    

```bash
sudo ip link add link <nova-interface> name macvlan0 type macvlan mode bridge
sudo ip addr add <novo-ip-host>/24 dev macvlan0
sudo ip link set macvlan0 up
```

4. **Ajustar IPs nas VMs** para uma faixa compátivel com a nova rede e novo gateway.
    
5. **(Opcional)** Atualizar MACVTAP no Virt-Manager para vincular ao novo nome de interface.
    

---

# Fim 📅

Agora suas VMs estão prontas para se comunicar entre si, com o Host e com a rede externa se quiser!