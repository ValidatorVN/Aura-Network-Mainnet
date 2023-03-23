# Aura-Network-Mainnet

    Cấu hình khuyến nghị:
    8core
    16gb ram
    500

1/ Cập nhật hệ thống:

    sudo apt update && apt upgrade -y
    
Cài đặt các package cần thiết:

    sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
    
2/ Cài đặt Golang: khuyến khích bản v19:

    ver="1.19.1"
    cd $HOME
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
    rm "go$ver.linux-amd64.tar.gz"

    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
    source $HOME/.bash_profile

Kiểm tra phiên bản Go:

    go version
    
3/ Cài đặt Node:

    git clone --branch aura_v0.4.4 https://github.com/aura-nw/aura

    cd aura

    make install

4/ Thêm thông tin node: thay Name = tên bạn muốn đặt:

    aurad init <name> --chain-id=xstaxy-1
    
 Tải về khối Genesis:
 
    curl -s https://raw.githubusercontent.com/aura-nw/mainnet-artifacts/main/xstaxy-1/genesis.json > /root/.aura/config/genesis.json
    
 Kiểm tra sha256Sum xem chính xác thông tin này: 90b9404d38167e3b40f56ddc11a1565f0107b89008742425e44905871699febc  -
 
    jq -S -c -M '' /root/.aura/config/genesis.json | sha256sum
    
 Set thông tin Seed & Peer & Gas:
 
    SEEDS="22a0ca5f64187bb477be1d82166b1e9e184afe50@18.143.52.13:26656,0b8bd8c1b956b441f036e71df3a4d96e85f843b8@13.250.159.219:26656"
    PEERS=""

    sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.aura/config/config.toml

    sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.aura/config/app.toml
    sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.aura/config/app.toml
    sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.aura/config/app.toml
    sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.aura/config/app.toml

    sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uaura"|g' $HOME/.aura/config/app.toml
    sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.aura/config/config.toml
    aurad tendermint unsafe-reset-all --home $HOME/.aura --keep-addr-book
    
 5/ Tạo hệ thống:

    sudo tee /etc/systemd/system/aurad.service > /dev/null <<EOF
    [Unit]
    Description=aurad Daemon
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which aurad) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF

Tải snapshot, mình mượn của bên khác cho đỡ phải Sync từ đầu:

Link1:

    SNAP_NAME=$(curl -s https://snapshots.nodestake.top/aura/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
    curl -o - -L https://snapshots.nodestake.top/aura/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.aura
    
Link2:

    SNAP_NAME=$(curl -s https://snapshots1.nodejumper.io/aura/info.json | jq -r .fileName)
    curl "https://snapshots1.nodejumper.io/aura/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C "$HOME/.aura"

Khởi động hệ thống:
    
    sudo systemctl daemon-reload
    sudo systemctl enable aurad
    sudo systemctl restart aurad
    sudo journalctl -u aurad -f --no-hostname -o cat

Lệnh kiểm tra trạng thái Sync:

    aurad status 2>&1 | jq .SyncInfo.catching_up
    
Trang chủ Aura để kiểm tra số Block:

    https://aurascan.io/dashboard
   
6/ Tạo ví:

    aurad keys add wallet
    
Nếu đã có ví, dùng lệnh khôi phục;

    aurad keys add wallet --recover
    
Tạo validator:

    auradd tx staking create-validator \
    --amount=1000000uaura \
    --pubkey=$(aurad tendermint show-validator) \
    --moniker="Node & Validator VietNam" \
    --identity=6CB6AC3E672AAB9D \
    --details="https://t.me/NodeValidatorVietNam" \
    --chain-id=xstaxy-1 \
    --commission-rate=0.10 \
    --commission-max-rate=0.20 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1 \
    --from=wallet \
    --gas-prices=0.1uaura \
    --gas-adjustment=1.5 \
    --gas=auto \
    -y

Cộng đồng chạy Node & Validator VietNam, nơi thảo luận và chia sẻ kinh nghiệm về chạy node/validator, không bàn luận chính trị.

    Chanel: https://t.me/RunnodeVietNamese
    Youtube: https://www.youtube.com/@nodevalidatorvietnam
