# Snapshot Nolus
Snapshots allows a new node to join the network by recovering application state from a backup file. Snapshot contains compressed copy of chain data directory. To keep backup files as small as plausible, snapshot server is periodically beeing state-synced.

My snapshot: http://89.58.33.98/downloads/nolus.tar.lz4

# Instructions
1. Stop the service and reset the data
 ```bash
sudo systemctl stop nolusd
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
rm -rf $HOME/.nolus/data
```
2. Download latest snapshot
```bash
curl -L http://89.58.33.98/downloads/nolus.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.nolus
mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
```
3. Restart the service and check the log
```bash
sudo systemctl start nolusd && sudo journalctl -u nolusd -f --no-hostname -o cat
```
## How to create a snapshot
1. Execute this command to make snapshot
```bash
sudo apt-get update
  sudo apt install snapd -y
  sudo apt install lz4 jq

  sudo systemctl stop nolusd

  cd $HOME/.nolus/
  tar cfv - data/ | lz4 > "nolus.tar.lz4"
```
2. Move snapshot to dir download
```bash
sudo mkdir -p /var/www/html/downloads
  mv $HOME/.nolus/nolus.tar.lz4 /var/wwww/html/downloads/nolus.tar.lz4
chmod 755 /var/www/html/downloads
```

3. Install nginx
https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04
4. Execute this command
```bash
cat <<EOF | sudo tee /etc/nginx/sites-available/nolus.conf
server {
    listen 80;
    server_name your_ip; # change with your own domain or IP Address
#example server_name 10.40.23.20;
    location /downloads {
        root /var/www/html;
        autoindex on;
    }
}
EOF
```

5. Restart nginx
```bash
sudo systemctl restart nginx
```

6. Your snapshot is: http://your_ip/downloads/nolus.tar.lz4
