# MANUAL TEST SETUP FOR DCR AND BTC

This guide follows the [official minimal manual setup guide](https://docs.btcpayserver.org/Deployment/ManualDeployment/), with an additional database setup step added from the [extended setup guide](https://docs.btcpayserver.org/Deployment/ManualDeploymentExtended/).

Steps 1-7 below are similar to steps 1-7 in the [official minimal manual setup guide](https://docs.btcpayserver.org/Deployment/ManualDeployment/), with differences relating to running btcpayserver with dcr and btc rather than just btc.

Step 0 is borrowed from the [extended setup guide](https://docs.btcpayserver.org/Deployment/ManualDeploymentExtended/) and describes how to setup the required databases.

## Setup Steps

0. **Setup databases**

Install postgres, then run `sudo -u postgres psql` and enter the following commands to create the required databases.
Note the database names, usernames and passwords; they’ll be required in steps 6 and 7 below.

```
CREATE DATABASE nbxplorerdb TEMPLATE 'template0' LC_CTYPE 'C' LC_COLLATE 'C' ENCODING 'UTF8';
CREATE USER nbxplorerdbuser WITH ENCRYPTED PASSWORD 'nbxplorerdbpass';
GRANT ALL PRIVILEGES ON DATABASE nbxplorerdb TO nbxplorerdbuser;
\c nbxplorerdb;
GRANT ALL PRIVILEGES ON SCHEMA public TO nbxplorerdbuser;
```

```
CREATE DATABASE btcpaydb TEMPLATE 'template0' LC_CTYPE 'C' LC_COLLATE 'C' ENCODING 'UTF8';
CREATE USER btcpaydbuser WITH ENCRYPTED PASSWORD 'btcpaydbpass';
GRANT ALL PRIVILEGES ON DATABASE btcpaydb TO btcpaydbuser;
\c btcpaydb;
GRANT ALL PRIVILEGES ON SCHEMA public TO btcpaydbuser;
```

1. **Install ~~Bitcoin Core 0.19.1~~ BTC and DCR daemons**

Install bitcoind, bitcoin-cli, dcrd, dcrwallet and dcrctl.

2. **Install .NET ~~8.0~~ 10.0 SDK**

3. **Install NBXplorer**

First create a folder to house related source codes (nbitcoin, nbxplorer and btcpayserver). All 3 source codes must be in the same root folder to build successfully.

```
mkdir ~/nbtest
cd ~/nbtest
git clone https://github.com/itswisdomagain/NBitcoin.git -b dcr
git clone https://github.com/itswisdomagain/NBXplorer.git -b dcr
git clone https://github.com/itswisdomagain/btcpayserver.git -b dcr
```

Now, install NBXplorer

```
cd ~/nbtest/NBXplorer
./build.sh
```

4. **Install BTCPayServer**

```
cd ~/nbtest/btcpayserver
./build.sh
```

5. **Run ~~bitcoind~~ BTC and DCR daemons**
   Use the [dcr](https://github.com/decred/dcrdex/tree/master/dex/testing/dcr) and [btc](https://github.com/decred/dcrdex/tree/master/dex/testing/btc) [simnet harnesses from dcrdex project](https://github.com/decred/dcrdex/wiki/Simnet-Testing#start-the-dcr-and-btc-simnet-harnesses) to run the btc and dcr daemons. You’ll need to run the harnesses in separate terminal windows.

```
git clone https://github.com/decred/dcrdex.git
```

(On terminal window 1)

```
cd dcrdex/dex/testing/dcr && ./harness.sh
```

(On terminal window 2)

```
cd dcrdex/dex/testing/dcr && ./harness.sh
```

6. **Run NBXplorer**
   Create the config file at `~/.nbxplorer/RegTest/settings.config` and enter the following:

```
noauth=1
postgres=User ID=nbxplorerdbuser;Password=nbxplorerdbpass;Application Name=nbxplorer;MaxPoolSize=20;Host=localhost;Port=5432;Database=nbxplorerdb;
btc.rpc.user=user
btc.rpc.password=pass
btc.rpc.url=http://127.0.0.1:20556
btc.node.endpoint=127.0.0.1:20575
dcr.rpc.user=user
dcr.rpc.password=pass
dcr.rpc.certfile=/full/path/to/dextest-folder/dcr/alpha/rpc.cert
dcr.rpc.url=https://localhost:19562
dcr.node.endpoint=127.0.0.1:19560
```

Open a new terminal window (terminal window 3) and run NBXplorer:

```
cd ~/nbtest/NBXplorer
./run.sh --network=regtest --chains=btc,dcr
```

7. **Run BTCPay Server**
   Create the config file at `~/.btcpayserver/RegTest/settings.config` and enter the following:

```
postgres=User ID=btcpayuser;Password=btcpaypass;Application Name=btcpayserver;Host=localhost;Port=5432;Database=btcpaydb;
explorer.postgres=User ID=nbxplorerdbuser;Password=nbxplorerdbpass;Application Name=nbxplorer;MaxPoolSize=20;Host=localhost;Port=5432;Database=nbxplorerdb;
```

Open a new terminal window (terminal window 4) and run cd ~/nbtest/btcpayserver:

```
cd ~/nbtest/btcpayserver
./run.sh --network=regtest --chains=btc,dcr
```

## What to test.

-   Once btcpayserver is running, visit the url in your browser.
-   Setup your login password and create a store.
-   Setup your btc / dcr wallets and fund them.
-   Create invoices, requests and pull payments.
-   Test other features.
