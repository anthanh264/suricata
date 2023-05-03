# Suricata 
## Prerequisites
- Ubuntu 20.04
## THUC NGHIEM 
INSTALL UBUNTU SERVER

```
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata jq nano -y

ip addr 
sudo nano /etc/suricata/suricata.yaml
line 589

sudo suricata-update

sudo nano /var/lib/suricata/rules/test.rules

Alert ICMP any any -> $HOME_NET any (msg: "Phat hien Ping"; sid:2; rev:1;)

sudo nano /etc/suricata/suricata.yaml
line 1925 add test.rules

sudo suricata -T -c /etc/suricata/suricata.yaml -v

![](https://i.imgur.com/FZZA5QZ.png)

```















## Install 
- Add repository 
```shell
sudo add-apt-repository ppa:oisf/suricata-stable
```
![](https://i.imgur.com/Lc83qvP.png)

- Install via apt
```shell
sudo apt install suricata -y
sudo apt install jq -y
```
![](https://i.imgur.com/2eE7NZy.png)

- Enable auto start
```shell
sudo systemctl enable suricata.service
```
![](https://i.imgur.com/9betmKX.png)

- Stop service 
```shell!
sudo systemctl stop suricata.service
```
## Config basic (IDS)
- Determine network interfaces
```shell!
ip -p -j route show default
```
![](https://i.imgur.com/RgKfONO.png)


- Edit Suricata's configuration file
```shell!
sudo nano /etc/suricata/suricata.yaml
```
> - Modify interface line
![](https://i.imgur.com/kYnl8mA.png)


> - Inspect traffic on additional interfaces insert it before the -interface: default section
> ![](https://i.imgur.com/qcon87C.png)

>- Configuring Live Rule Reloading (To edit rules without needing to restart the running Suricata process.)
> ![](https://i.imgur.com/cnV0Uu4.png)

## Update Suricata Rulesets
```shell
sudo suricata-update list-sources
sudo suricata-update
```
- Add rule slkippp
```shell
sudo suricata-update enable-source tgreen/hunting 
```
## Validate Suricata’s Configuration
```shell
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```
## Running Suricata
```shell
sudo systemctl start suricata.service
```
- Log check
```shell
sudo tail -f /var/log/suricata/suricata.log
```
- Correct output 
```shell
<Info> - All AFP capture threads are running.
```

## Test 
```shell
curl http://testmynids.org/uid/index.html
```
Check log (2100498: rule identifier)
```shell
grep 2100498 /var/log/suricata/fast.log
```
![](https://i.imgur.com/UZosPLR.png)
```shell
jq 'select(.alert .signature_id==2100498)' /var/log/suricata/eve.json
```
![](https://i.imgur.com/v3hn5uQ.png)


## Config IPS
- Include Custom Signatures

    + Get Public ipv4
    ```shell
    ip -brief address show
    ```
    ![](https://i.imgur.com/BuqP38D.png)
    + Create custom signature
    ```shell
    sudo nano /var/lib/suricata/rules/local.rules
    ```
    ```shell
    alert ssh any any -> 192.168.44.128 !22 (msg:"SSH TRAFFIC on non-SSH port"; flow:to_client, not_established; classtype: misc-attack; target: dest_ip; sid:1000000;)
    alert ssh any any -> fe80::491f:cea2:47aa:c2bf/64 !22 (msg:"SSH TRAFFIC on non-SSH port"; flow:to_client, not_established; classtype: misc-attack; target: dest_ip; sid:1000001;)
    drop ssh any any -> 192.168.44.128 !22 (msg:"SSH TRAFFIC on non-SSH port"; classtype: misc-attack; target: dest_ip; sid:1000000;)
    drop ssh any any -> fe80::491f:cea2:47aa:c2bf/64 !22 (msg:"SSH TRAFFIC on non-SSH port"; classtype: misc-attack; target: dest_ip; sid:1000001;)
    ```
    ![](https://i.imgur.com/yeWguEi.png)
    + Add custom rule 
    ```shell
    sudo nano /etc/suricata/suricata.yaml
    ```
    + Add local.rules  line 1879
    ![](https://i.imgur.com/eliYBZQ.png)
    + Verify config 
    ```shell
    sudo suricata -T -c /etc/suricata/suricata.yaml -v
    ```
- Enable nfqueue Mode
```shell
sudo nano /etc/default/suricata
```
 Find the LISTENMODE=af-packet and edit LISTENMODE=nfqueue
![](https://i.imgur.com/dZrLYQY.png)
Restart suricata
```shell
sudo systemctl restart suricata.service
sudo systemctl status suricata.service
```
![](https://i.imgur.com/veOZa1F.png)
- Configure UFW To Send Traffic to Suricata
```shell
sudo nano /etc/ufw/before.rules
```
Add folowing line (ipv4)
```shell
## Start Suricata NFQUEUE rules
-I INPUT 1 -p tcp --dport 22 -j NFQUEUE --queue-bypass
-I OUTPUT 1 -p tcp --sport 22 -j NFQUEUE --queue-bypass
-I FORWARD -j NFQUEUE
-I INPUT 2 -j NFQUEUE
-I OUTPUT 2 -j NFQUEUE
## End Suricata NFQUEUE rules
```
![](https://i.imgur.com/tfK7D7j.png)

```shell
sudo nano /etc/ufw/before6.rules
```
Add folowing line (ipv6)
```shell
## Start Suricata NFQUEUE rules
-I INPUT 1 -p tcp --dport 22 -j NFQUEUE --queue-bypass
-I OUTPUT 1 -p tcp --sport 22 -j NFQUEUE --queue-bypass
-I FORWARD -j NFQUEUE
-I INPUT 2 -j NFQUEUE
-I OUTPUT 2 -j NFQUEUE
## End Suricata NFQUEUE rules
```
Restart ufw
```shell
sudo systemctl restart ufw.service
```
line 2410
