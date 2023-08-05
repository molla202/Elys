## Blok yükseliği 2041000 geldiğinde yapacağız.

```
cd $HOME/elys
git fetch --all
git checkout v0.9.0
make build
sudo mv $HOME/elys/build/elysd $(which elysd)
sudo systemctl restart elysd && sudo journalctl -u elysd -f
```

## Ardından snap atıyoruz..
```
sudo systemctl stop elysd

cp $HOME/.elys/data/priv_validator_state.json $HOME/.elys/priv_validator_state.json.backup

rm -rf $HOME/.elys/data 
curl https://testnet-files.itrocket.net/elys/snap_elys.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.elys

mv $HOME/.elys/priv_validator_state.json.backup $HOME/.elys/data/priv_validator_state.json
```
```
sudo systemctl restart elysd && sudo journalctl -u elysd -f
```
