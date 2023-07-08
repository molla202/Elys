cd $HOME/elys
git fetch --all
git checkout v0.8.0
make build
sudo mv $HOME/elys/build/elysd $(which elysd)
sudo systemctl restart elysd && sudo journalctl -u elysd -f
