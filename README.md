# Hyperledger Besu - Permissioned QBFT Network for Production Networks

Este guia descreve a configuração de uma rede permissionada utilizando o mecanismo de consenso QBFT (QBFT Consensus Protocol) do Hyperledger Besu, ideal para ambientes de produção.


## Pré-requisitos
Certifique-se de ter as seguintes ferramentas instaladas:

- Java
- Besu v25.10.0
- curl, wget, tar
- Docker
- Docker-Compose

### Instalação das Dependências

#### Besu

> [!IMPORTANT]
> <sup>Estamos utilizando a versão 25.10.0 do Besu. Para utilizar outra versão, altere a URL de download e atualize as variáveis de ambiente conforme necessário.</sup>

``` 
wget https://github.com/hyperledger/besu/releases/download/25.10.0/besu-25.10.0.tar.gz
tar -xvf besu-25.10.0.tar.gz 
rm besu-25.10.0.tar.gz 
export PATH=$(pwd)/besu-25.10.0/bin:$PATH
```

#### JAVA

> [!IMPORTANT]
> <sup>Certifique-se de que o diretório `jdk-21.0.9/` foi extraído corretamente na raiz do projeto.</sup>

```
wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz 
tar -xvf jdk-21_linux-x64_bin.tar.gz
rm jdk-21_linux-x64_bin.tar.gz
export JAVA_HOME=$(pwd)/jdk-21.0.9
```

> [!NOTE]
> <sup>Se atente a versão do jdk baixada para criar a variável de ambiente corretamente.</sup>

Verifique a versão da besu instalada:
```
besu --version
```
> [!NOTE]
> <sup>Este tutorial foi baseado na doc oficial da Besu [Hyperledger Besu Tutorial QBFT](https://besu.hyperledger.org/private-networks/tutorials/qbft) e [Hyperledger Besu Tutorial Permissioning](https://besu.hyperledger.org/private-networks/tutorials/permissioning)</sup>

## Etapa 1: Geração das Chaves Criptográficas e Arquivos de Configuração
### 1. Geração dos arquivos da blockchain e chaves privadas

```
besu operator generate-blockchain-config \
  --config-file=genesis_QBFT.json \
  --to=networkFiles \
  --private-key-file-name=key
```

### 2. Copiar o arquivo genesis.json com extraData
```
cp networkFiles/genesis.json ./Permissioned-Network
```

### 3. Geração do arquivo permissions_config.toml
Certifique-se de que o script de geração está com permissão de execução:

```
chmod +x generate-nodes-config.sh
./generate-nodes-config.sh
```
Formato esperado do arquivo permissions_config.toml:

```
nodes-allowlist=[
  "enode://<public-key-1>@<ip-node-1>:30303",
  ...
  "enode://<public-key-6>@<ip-node-6>:30308"
]
accounts-allowlist=[
  "0x<account-id-node-1>",
  ...
  "0x<account-id-node-6>"
]
```
> [!NOTE]
> <sup>Os account-ids são os nomes das pastas geradas automaticamente em networkFiles/.</sup>


### 4. Crie a estrutura de diretórios para os Nodes
Organize os arquivos conforme a estrutura:

```
Permissioned-Network/
├── genesis.json
├── Node-1/
│   └── data/
│       ├── key
│       ├── key.pub
│       └── permissions_config.toml
├── Node-2/
│   └── data/
│       ├── ...
├── ...
├── Node-6/
│   └── data/
```
> [!IMPORTANT]
> <sup>Certifique-se de verficar se os arquivos corretos foram copiados para cada um dos nós da rede (config.toml, key ...).</sup>

### 5. Atualize o endereços e IPs do nós da rede
Atualize todas as flags --bootnodes para os endereços e IPs dos nós 1 e 3 respectivamente
Os endereços podem ser verificados em Node-1/data/key.pub e Node-3/data/key.pub respectivamente

## Etapa 2: Execução da Rede

### 1. Construção da Imagem Docker
Crie a imagem Docker personalizada do Besu:

```
docker build --no-cache -f Dockerfile -t besu-image-local:25.10.0 .
```

### Para Docker Desktop
### 2. Inicialização dos Nós
Suba os nós da rede:
```
docker-compose up -d
```

### 3. Finalização da Rede
Para derrubar todos os containers:

```
docker-compose down
```
### Para Docker CE
Suba os nós da rede:
docker compose up -d

Ver os logs
docker compose logs -f

Containers ativos:
docker ps

Containers ativos e parados:
docker ps -a

Ver as imagens
docker images

Apagar container
docker rm -f <container_id_ou_nome>

Apagar todos containers:
docker compose down

Apagar imagens:
docker rmi <image_id_ou_nome>

Informações:
docker system df
docker stats (porcentagens cpu docker ps)

## Etapa 3: Testes de Conectividade e Estado da Rede 
Utilize os comandos abaixo para validar o estado da rede:

> [!NOTE]
> <sup>
```
# Métricas Prometheus
curl http://localhost:9545/metrics

# Métricas internas (via RPC)
curl -X POST --data '{"jsonrpc":"2.0","method":"debug_metrics","params":[],"id":1}' http://127.0.0.1:8545 | jq

# Informações do nó
curl -X POST --data '{"jsonrpc":"2.0","method":"admin_nodeInfo","params":[],"id":1}' http://127.0.0.1:8545 | jq

# Escuta de rede
curl -X POST --data '{"jsonrpc":"2.0","method":"net_listening","params":[],"id":53}' http://127.0.0.1:8545 | jq

# Enode do nó
curl -X POST --data '{"jsonrpc":"2.0","method":"net_enode","params":[],"id":1}' http://127.0.0.1:8545 | jq

# Serviços de rede
curl -X POST --data '{"jsonrpc":"2.0","method":"net_services","params":[],"id":1}' http://127.0.0.1:8545 | jq

# Contagem de peers
curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}' http://127.0.0.1:8545 | jq
```
</sup>

