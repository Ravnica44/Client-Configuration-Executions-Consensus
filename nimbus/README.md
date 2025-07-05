# Generate a secure JWT secret for Nimbus eth1 (run only once)
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

# Import pre-synced Era chain data and start Nimbus eth1 client
docker compose -f docker-compose_m.yml run --rm nimbus_import && \
docker compose -f docker-compose_m.yml up --build -d nimbus_eth1_m
