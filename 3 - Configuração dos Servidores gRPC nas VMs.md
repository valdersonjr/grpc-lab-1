# 3 - Configuração dos Servidores gRPC nas VMs

Este documento detalha o passo a passo realizado para configurar as VMs com seus respectivos servidores gRPC.

---

# VM2 - gRPC Server A (Node.js)

**Objetivo:** Criar um servidor gRPC em Node.js que realize operações matemáticas: soma, subtração, multiplicação e divisão.

## Ambiente

- Node.js: v18.19.1
    
- npm: 9.2.0
    

## 1. Instalação das dependências

```bash
sudo apt update
sudo apt install nodejs npm -y
```

## 2. Criação do projeto

```bash
mkdir grpc-server-a
cd grpc-server-a
npm init -y
npm install @grpc/grpc-js @grpc/proto-loader
```

## 3. Definir o arquivo `calculator.proto`

Crie o arquivo `calculator.proto`:

```proto
syntax = "proto3";

option go_package = "./sum";

service Calculator {
  rpc Sum (SumRequest) returns (SumResponse);
  rpc Subtract (SumRequest) returns (SumResponse);
  rpc Multiply (SumRequest) returns (SumResponse);
  rpc Divide (SumRequest) returns (DivisionResponse);
}

message SumRequest {
  int32 num1 = 1;
  int32 num2 = 2;
}

message SumResponse {
  int32 result = 1;
}

message DivisionResponse {
  double result = 1;
}
```

## 4. Gerar os arquivos gRPC

```bash
grpc_tools_node_protoc \
  --js_out=import_style=commonjs:. \
  --grpc_out=grpc_js:. \
  --proto_path=. \
  calculator.proto
```

## 5. Criar o servidor `server.js`

```javascript
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const PROTO_PATH = './calculator.proto';

const packageDefinition = protoLoader.loadSync(PROTO_PATH);
const calculatorProto = grpc.loadPackageDefinition(packageDefinition).Calculator;

function sum(call, callback) {
  const { num1, num2 } = call.request;
  const result = num1 + num2;
  console.log(`Sum: ${num1} + ${num2} = ${result}`);
  callback(null, { result });
}

function subtract(call, callback) {
  const { num1, num2 } = call.request;
  const result = num1 - num2;
  console.log(`Subtract: ${num1} - ${num2} = ${result}`);
  callback(null, { result });
}

function multiply(call, callback) {
  const { num1, num2 } = call.request;
  const result = num1 * num2;
  console.log(`Multiply: ${num1} * ${num2} = ${result}`);
  callback(null, { result });
}

function divide(call, callback) {
  const { num1, num2 } = call.request;
  if (num2 === 0) {
    return callback({
      code: grpc.status.INVALID_ARGUMENT,
      message: 'Division by zero is not allowed',
    });
  }
  const result = num1 / num2;
  console.log(`Divide: ${num1} / ${num2} = ${result}`);
  callback(null, { result });
}

const server = new grpc.Server();
server.addService(calculatorProto.service, {
  Sum: sum,
  Subtract: subtract,
  Multiply: multiply,
  Divide: divide,
});

const address = '0.0.0.0:50051';
server.bindAsync(address, grpc.ServerCredentials.createInsecure(), () => {
  console.log(`gRPC Server running at ${address}`);
  server.start();
});
```

## 6. Rodar o servidor

```bash
node server.js
```

Servidor rodando na porta `50051`.

---

# VM3 - gRPC Server B (Python)

**Objetivo:** Criar um servidor gRPC em Python com operações matemáticas como quadrado, raiz quadrada, potência e fatorial.

## Ambiente

- Python: 3.12.3
    

## 1. Instalação das dependências

```bash
sudo apt update
sudo apt install python3-pip -y
pip3 install grpcio grpcio-tools
```

## 2. Criação do projeto

```bash
mkdir grpc-server-b
cd grpc-server-b
```

## 3. Definir o arquivo `square.proto`

```proto
syntax = "proto3";

option go_package = "./square";

service SquareService {
  rpc Square (SquareRequest) returns (SquareResponse);
  rpc Sqrt (SqrtRequest) returns (SqrtResponse);
  rpc Power (PowerRequest) returns (PowerResponse);
  rpc Factorial (FactorialRequest) returns (FactorialResponse);
}

message SquareRequest {
  int32 number = 1;
}
message SquareResponse {
  int32 result = 1;
}

message SqrtRequest {
  double number = 1;
}
message SqrtResponse {
  double result = 1;
}

message PowerRequest {
  int32 base = 1;
  int32 exponent = 2;
}
message PowerResponse {
  int64 result = 1;
}

message FactorialRequest {
  int32 number = 1;
}
message FactorialResponse {
  int64 result = 1;
}
```

## 4. Gerar os arquivos gRPC

```bash
python3 -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. square.proto
```

## 5. Criar o servidor `server.py`

```python
from concurrent import futures
import grpc
import math
import square_pb2
import square_pb2_grpc

class SquareService(square_pb2_grpc.SquareServiceServicer):
    def Square(self, request, context):
        result = request.number * request.number
        return square_pb2.SquareResponse(result=result)

    def Sqrt(self, request, context):
        number = request.number
        if number < 0:
            context.set_code(grpc.StatusCode.INVALID_ARGUMENT)
            context.set_details("Negative number cannot have a real square root.")
            return square_pb2.SqrtResponse()
        result = math.sqrt(number)
        return square_pb2.SqrtResponse(result=result)

    def Power(self, request, context):
        result = int(math.pow(request.base, request.exponent))
        return square_pb2.PowerResponse(result=result)

    def Factorial(self, request, context):
        n = request.number
        if n < 0:
            context.set_code(grpc.StatusCode.INVALID_ARGUMENT)
            context.set_details("Cannot calculate factorial of a negative number.")
            return square_pb2.FactorialResponse()
        result = math.factorial(n)
        return square_pb2.FactorialResponse(result=result)

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    square_pb2_grpc.add_SquareServiceServicer_to_server(SquareService(), server)
    server.add_insecure_port('[::]:50052')
    server.start()
    print("gRPC Server B running at 0.0.0.0:50052")
    server.wait_for_termination()

if __name__ == '__main__':
    serve()
```

## 6. Rodar o servidor

```bash
python3 server.py
```

Servidor rodando na porta `50052`.

---