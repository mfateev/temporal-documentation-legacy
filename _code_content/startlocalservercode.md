---
name: startlocalserver
---

Mac:

```bash
mkdir cadence-docker
cd cadence-docker
curl -s -L https://github.com/temporalio/temporal/releases | egrep -m 1 -o '/uber/cadence/releases/download/v[0-9]+.[0-9]+.[0-9]+/docker.tar.gz' | wget --base=https://github.com/ -i -
tar -xzvf docker.tar.gz
docker-compose up
```

Windows:

```powershell
mkdir cadence-docker
cd cadence-docker
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$R = wget https://github.com/temporalio/temporal/releases -UseBasicParsing
$L = $R.Links | where {$_.href -match '/uber/cadence/releases/download/v[0-9]+.[0-9]+.[0-9]+/docker.tar.gz'} | select href -First 1
$U = 'https://github.com' + $L.href
wget $U -OutFile docker.tar.gz
7z x .\docker.tar.gz
docker-compose up
```
