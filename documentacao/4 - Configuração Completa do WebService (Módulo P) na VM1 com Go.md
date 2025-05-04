# 4 - Configuração Completa do WebService (Módulo P) na VM1 com Go

Este documento registra todo o processo para configurar o módulo **P** (Web Server com API HTTP + Cliente gRPC) em Go, rodando na **VM1 (192.168.1.201)**.

---

## Visão Geral

* O módulo P é um servidor HTTP que oferece diversos endpoints:

  * `/sum?a=5&b=3`, `/subtract?a=5&b=3`, `/multiply?a=5&b=3`, `/divide?a=5&b=3` — operações básicas que usam o gRPC Server A (VM2)

  * `/square?number=7`, `/sqrt?number=9`, `/factorial?number=5`, `/power?base=2&exponent=3` — operações matemáticas que usam o gRPC Server B (VM3)

* O servidor foi implementado em **Go**, usando as bibliotecas `grpc` e `net/http`.

---

## 1. Instalar o Go

```bash
sudo apt update
sudo apt install golang-go
```

Verifique:

```bash
go version
```

---

## 2. Criar o Projeto Go

```bash
mkdir grpc-client-p
cd grpc-client-p
go mod init grpc-client-p
```

---

## 3. Instalar Plugins do gRPC

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

Adicione ao PATH:

```bash
export PATH="$PATH:$(go env GOPATH)/bin"
```

Instale as dependências do projeto:

```bash
go get google.golang.org/grpc
```

---

## 4. Copiar os Arquivos `.proto`

Coloque na pasta do projeto:

* `sum.proto` (do módulo A)
* `square.proto` (do módulo B)

Adicione esta linha ao topo dos arquivos `.proto`:

```proto
option go_package = "./sum"; // ou ./square no outro
```

---

## 5. Gerar os Códigos Go

```bash
protoc --go_out=. --go-grpc_out=. sum.proto
protoc --go_out=. --go-grpc_out=. square.proto
```

Serão criadas as pastas:

* `sum/`
* `square/`

---

## 6. Criar o Servidor HTTP (main.go)

Use o código completo com todos os handlers implementados: `sum`, `subtract`, `multiply`, `divide`, `square`, `sqrt`, `factorial`, `power`.

---

## 7. Executar o Web Server

```bash
go build -o webserver main.go
./webserver
```

Deve aparecer:

```
HTTP Server running on :8080
```

---

## 8. Testar no Navegador ou Terminal

### Operações Básicas com Server A (VM2):

```bash
curl "http://192.168.1.201:8080/sum?a=5&b=3"
curl "http://192.168.1.201:8080/subtract?a=5&b=3"
curl "http://192.168.1.201:8080/multiply?a=5&b=3"
curl "http://192.168.1.201:8080/divide?a=10&b=2"
```

### Operações Avançadas com Server B (VM3):

```bash
curl "http://192.168.1.201:8080/square?number=7"
curl "http://192.168.1.201:8080/sqrt?number=9"
curl "http://192.168.1.201:8080/factorial?number=5"
curl "http://192.168.1.201:8080/power?base=2&exponent=3"
```

---

## Observações Finais

* Certifique-se de que:

  * O gRPC Server A (Node.js) esteja ativo na VM2
  * O gRPC Server B (Python) esteja ativo na VM3
  * A rede entre as VMs está funcional (ping entre elas)
