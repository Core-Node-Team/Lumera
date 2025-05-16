```
sudo systemctl stop lumerad
cp $HOME/.lumera/data/priv_validator_state.json $HOME/.lumera/priv_validator_state.json.backup
lumerad tendermint unsafe-reset-all --home $HOME/.lumera --keep-addr-book
curl -L http://37.120.189.81/lumera_testnet/lumera_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.lumera
mv $HOME/.lumera/priv_validator_state.json.backup $HOME/.lumera/data/priv_validator_state.json
sudo systemctl restart lumerad
sudo journalctl -u lumerad -f -o cat
```
