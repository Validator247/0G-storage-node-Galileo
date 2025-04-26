# 0G-storage-node-Galileo

 Download and install binary

     git clone -b v1.0.0 https://github.com/0glabs/0g-storage-node.git
    cd 0g-storage-node
    cargo build --release

        
Download config file:

    curl -o $HOME/0g-storage-node/run/config-testnet-turbo.toml https://cdn.bangcode.id/0g/v3_config.toml

        
nano config-testnet-turbo.toml , edit file config:

    # Network Config Options
    network_dir = "network"
    network_listen_address = "0.0.0.0"

    # DB Config Options
    db_dir = "db"

    # Log configuration file.
    log_config_file = "log_config"

    #Log directory.
    log_directory = "log"

    # transaction gas fee.
    miner_key = "Your_private-key_wallet"

    # RPC Config Options
    [rpc]

    # Whether to provide RPC service.
     enabled = true

    # HTTP server address to bind for public RPC.
     listen_address = "0.0.0.0:5678"
​
# Create service

    sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
    [Unit]
    Description=ZGS Node
    After=network.target

    [Service]
    User=$USER
    WorkingDirectory=$HOME/0g-storage-node/run
    ExecStart=$HOME/0g-storage-node/target/release/zgs_node --config $HOME/0g-storage-node/run/config-testnet-turbo.toml
    Restart=on-failure
    RestartSec=10
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF
​

Run node

    sudo systemctl daemon-reload && \ sudo systemctl enable zgs && \ sudo systemctl restart zgs && \ sudo systemctl status zgs
​

Check logSync & Peer via RPC

    while true; do
    response=$(curl -s -X POST http://localhost:5678 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"zgs_getStatus","params":[],"id":1}')
    logSyncHeight=$(echo $response | jq '.result.logSyncHeight')
    connectedPeers=$(echo $response | jq '.result.connectedPeers')
    echo -e "logSyncHeight: \033[32m$logSyncHeight\033[0m, connectedPeers: \033[34m$connectedPeers\033[0m"
    sleep 5;
    done
​
Stop & Delete node

    sudo systemctl stop zgs
    sudo systemctl disable zgs
    sudo rm /etc/systemd/system/zgs.service
    rm -rf $HOME/0g-storage-node
