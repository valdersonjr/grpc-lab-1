# 5 - Scripts de Rede Manual para as VMs (VM1, VM2 e VM3)

Este documento explica o funcionamento dos scripts de rede criados para cada VM, localizados em `/usr/local/bin/`, e como executá-los manualmente para garantir a comunicação entre as máquinas virtuais com IPs fixos.

---

## Objetivo

- Configurar IP fixo e rota padrão manualmente
- Garantir que as VMs possam se comunicar entre si
- Executar os scripts sob demanda após o boot (sem automação)

---

## Localização dos scripts

Os scripts estão localizados em:

```
/usr/local/bin/vm1-network.sh
/usr/local/bin/vm2-network.sh
/usr/local/bin/vm3-network.sh
```

Cada script configura a interface de rede da respectiva VM com um IP fixo e uma rota padrão.

---

## Conteúdo dos scripts

### VM1 – `vm1-network.sh`

```bash
#!/bin/bash
ip addr flush dev enp1s0
ip addr add 192.168.1.201/24 dev enp1s0
ip link set enp1s0 up
ip route add default via 192.168.1.1
```

### VM2 – `vm2-network.sh`

```bash
#!/bin/bash
ip addr flush dev enp1s0
ip addr add 192.168.1.202/24 dev enp1s0
ip link set enp1s0 up
ip route add default via 192.168.1.1
```

### VM3 – `vm3-network.sh`

```bash
#!/bin/bash
ip addr flush dev enp1s0
ip addr add 192.168.1.203/24 dev enp1s0
ip link set enp1s0 up
ip route add default via 192.168.1.1
```

> 🔎 Todos usam a interface `enp1s0`, e atribuem IPs fixos compatíveis com a rede `192.168.1.0/24`, além de definir o roteador padrão como `192.168.1.1`.

---

##  Como executar manualmente

### 1. Acesse o terminal da respectiva VM

### 2. Execute o script com permissões elevadas:

#### VM1

```bash
sudo /usr/local/bin/vm1-network.sh
```

#### VM2

```bash
sudo /usr/local/bin/vm2-network.sh
```

#### VM3

```bash
sudo /usr/local/bin/vm3-network.sh
```

---

## Resultado esperado após execução

- A interface `enp1s0` será ativada
- A VM receberá seu IP fixo
- A rota default será configurada corretamente
- A VM conseguirá pingar as outras

---

### Exemplo de teste após executar o script:

```bash
ping 192.168.1.201  # Teste de conectividade entre VMs
```

Esses scripts podem ser rodados sempre que a rede estiver desconfigurada.