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

    git clone https://github.com/aura-nw/aura.git

    cd aura

    git checkout Aura_v0.4.4

    make install

4/ Thêm thông tin node: thay Name = tên bạn muốn đặt:

    aurad init <name> --chain-id=xstaxy-1
    
 Tải về khối Genesis:
 
    curl -s https://raw.githubusercontent.com/aura-nw/mainnet-artifacts/main/xstaxy-1/pre-genesis.json >~/.aura/config/genesis.json
    
 Kiểm tra sha256Sum xem chính xác thông tin này: 90b9404d38167e3b40f56ddc11a1565f0107b89008742425e44905871699febc  -
 
    jq -S -c -M '' /root/.aura/config/genesis.json | sha256sum
    
 Set thông tin gas:
 
    sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.001uaura\"|" /root/.aura/config/app.toml
    
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

    SNAP_NAME=$(curl -s https://snapshots.nodestake.top/aura/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")

    curl -o - -L https://snapshots.nodestake.top/aura/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.aura

Khởi động hệ thống:
    
    sudo systemctl daemon-reload
    sudo systemctl enable aurad
    sudo systemctl restart aurad
    sudo journalctl -u aurad -f --no-hostname -o cat

Lệnh kiểm tra trạng thái Sync:

    d status 2>&1 | jq .SyncInfo.catching_up
   
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
    --keyring-backend test \
    --gas-prices=0.1uaura \
    --gas-adjustment=1.5 \
    --gas=auto \
    -y

Cộng đồng chạy Node & Validator VietNam, nơi thảo luận và chia sẻ kinh nghiệm về chạy node/validator, không bàn luận chính trị.

    Chanel: https://t.me/RunnodeVietNamese
    Youtube: https://www.youtube.com/@nodevalidatorvietnam
