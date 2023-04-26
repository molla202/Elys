## v0.4.0 güncelleme
# go tekrar kuruyoruz
```
# go tekrar kuruyoruz
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl https://dl.google.com/go/go1.19.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version 
```
# go tekrar kuruyoruz
```
cd $HOME
rm -rf elys
git clone https://github.com/elys-network/elys.git
cd elys
git checkout v0.4.0
make install
sudo systemctl restart elysd && sudo journalctl -u elysd -f 
```
