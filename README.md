# aztec

## 第 1 步：安装 Aztec 工具

```bash
bash -i <(curl -s https://install.aztec.network)
```

```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

📌 运行完后**重启终端**

### 验证是否安装成功：

```bash
aztec
```

---

## 第 2 步：更新网络配置

```bash
aztec-up alpha-testnet
```

---

## 第 3 步：获取 RPC URL 和 Beacon URL

Alchemy或ankr    
    
## 第 4 步：获取服务器公网 IP

```bash
curl ipv4.icanhazip.com
```

保存下来，稍后需要用。

---

## 第 5 步：开启防火墙并开放端口

```bash
ufw allow 22
ufw allow ssh
ufw allow 40400
ufw allow 8080
ufw enable
```

---

## 第 6 步：启动 Sequencer 节点

1.docker方法

新建aztec文件夹

```bash
mkdir aztec
cd aztec
```

创建.env文件

```bash
nano .env
```

```bash
ETHEREUM_RPC_URL=http://
CONSENSUS_BEACON_URL=http://
VALIDATOR_PRIVATE_KEY=[privatekey带0x]
COINBASE=[wallet address]
P2P_IP=[第4步获取的ip地址，不带http]
```

创建docker-compose.yml文件

```bash
nano docker-compose.yml
```

```bash
services:
  aztec-node:
    container_name: aztec-sequencer
    network_mode: host
    image: aztecprotocol/aztec:alpha-testnet
    restart: unless-stopped
    environment:
      ETHEREUM_HOSTS: ${ETHEREUM_RPC_URL}
      L1_CONSENSUS_HOST_URLS: ${CONSENSUS_BEACON_URL}
      DATA_DIRECTORY: /data
      VALIDATOR_PRIVATE_KEY: ${VALIDATOR_PRIVATE_KEY}
      COINBASE: ${COINBASE}
      P2P_IP: ${P2P_IP}
      LOG_LEVEL: debug
    entrypoint: >
      sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network alpha-testnet --node --archiver --sequencer'
    ports:
      - 40400:40400/tcp
      - 40400:40400/udp
      - 8080:8080
    volumes:
      - /root/.aztec/alpha-testnet/data/:/data
```

运行容器

```bash
docker compose up -d
```

查看日志

```bash
docker compose logs -fn 1000
```

暂停容器

```bash
docker compose down -v
```

2.CLI方法

```bash
screen -S aztec
```

执行以下命令（替换对应变量）：

```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls http:// \
  --l1-consensus-host-urls http:// \
  --sequencer.validatorPrivateKey [privatekey带0x] \
  --sequencer.coinbase [address] \
  --p2p.p2pIp 9142.171.223.46
  --p2p.maxTxPoolSize 1000000000
```

**Aztec 节点同步完成之后**，再运行以下 `curl` 命令来获取 proven block number：

```bash
sudo apt install jq -y
```

```bash
sudo ss -tlnp | grep :8080
```

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
[http://localhost:8080](http://localhost:8080/) | jq -r ".result.proven.number"
```

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["6009","6009"],"id":67}' \
http://localhost:8080 | jq -r ".result"
```
