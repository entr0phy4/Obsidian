A Wireless packet capture tool for aircrack-ng
First we must remember have the network card in [[Monitor mode]] ([[Airmon-ng]]) and we can start capture packets just with:

```bash
sudo airodump-ng wlo1mon
```

with `airodump-ng` we can filter by channel using:
```bash
sudo airodump-ng wlo1mon -c11
```
this filter networks and stations in channel 11.

We also can filter by [[IEEE]] standards
```bash
sudo airodump-ng wlo1mon --band abg
```