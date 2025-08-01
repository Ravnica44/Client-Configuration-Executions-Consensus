```shell
# Generate a secure JWT secret for Nimbus setup (run once only)
openssl rand -hex 32 > /root/ethereum/jwt_m.hex

# Create execution and consensus data directories with correct ownership and permissions
mkdir -p /root/ethereum/execution_m /root/ethereum/consensus_m
chown -R 1000:1000 /root/ethereum/execution_m /root/ethereum/consensus_m
chmod -R 755 /root/ethereum/execution_m /root/ethereum/consensus_m

# Create directories for Era state archive files
mkdir -p /root/ethereum/era_m /root/ethereum/era1_m

# Download *.era files recursively, continue interrupted downloads, show progress
wget -r -c -q --show-progress --no-parent -nd -A '*.era' https://mainnet.era.nimbus.team/ -P /root/ethereum/era_m
wget -r -c -q --show-progress --no-parent -nd -A '*.era' https://mainnet.era1.nimbus.team/ -P /root/ethereum/era1_m

# Download docker-compose_m.yml from GitHub
curl -sSL https://raw.githubusercontent.com/Ravnica44/Client-Configuration-Executions-Consensus/main/nimbus/docker-compose_m.yml -o /root/ethereum/docker-compose_m.yml

# Download mainnet_bootstrap.txt from GitHub
curl -sSL https://raw.githubusercontent.com/Ravnica44/Client-Configuration-Executions-Consensus/main/mainnet_bootstrap.txt -o /root/ethereum/mainnet_bootstrap.txt

# Create .env file with your external IP (required for NAT configuration)
echo "EXTERNAL_IP=$(curl -s ifconfig.me)" > /root/ethereum/.env

# Start import pre-synced Era chain data (run once before starting the execution node)
docker compose --env-file /root/ethereum/.env -f docker-compose_m.yml run --rm nimbus_import_m

# Start Nimbus execution node (execution client)
docker compose --env-file /root/ethereum/.env -f docker-compose_m.yml up --build -d nimbus_eth1_m

# Start sync consensus client with trustedNodeSync (run once before starting the beacon node)
docker compose --env-file /root/ethereum/.env -f docker-compose_m.yml run --rm nimbus_sync_m

# Start Nimbus beacon node (consensus client)
docker compose --env-file /root/ethereum/.env -f docker-compose_m.yml up --build -d nimbus_eth2_m

# (Optional) Start all services together
docker compose --env-file /root/ethereum/.env -f docker-compose_m.yml up --build -d
```
