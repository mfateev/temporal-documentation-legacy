---
name: startlocalserver
---

Mac:

```bash
mkdir temporal-docker
cd temporal-docker
curl -O https://raw.githubusercontent.com/temporalio/temporal/master/docker/docker-compose.yml
docker-compose up
```

Windows:

```powershell
mkdir temporal-docker
cd temporal-docker
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$R = wget https://github.com/temporalio/temporal/releases -UseBasicParsing
$L = $R.Links | where {$_.href -match '/uber/cadence/releases/download/v[0-9]+.[0-9]+.[0-9]+/docker.tar.gz'} | select href -First 1
$U = 'https://github.com' + $L.href
wget $U -OutFile docker.tar.gz
7z x .\docker.tar.gz
docker-compose up
```
