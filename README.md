# Bitcoin Compose Stack

Dieser Stack startet:

- `tor`
- `bitcoind`
- `electrs`
- `c-lightning` (`lightningd`)
- `mempool` (`backend`, `frontend`, `mariadb`)

## Start

1. Beispieldatei kopieren:

   ```bash
   cp .env.example .env
   ```

2. Passwoerter und Alias in `.env` anpassen.
   Optional kannst du die Storage-Pfade trennen:
   - `BITCOIN_DATA_PATH` fuer die Blockchain/bitcoind-Daten
   - `ELECTRS_DATA_PATH` fuer den Electrs-Index

   Beispiel:
   ```bash
   BITCOIN_DATA_PATH=/mnt/ssd1/bitcoin
   ELECTRS_DATA_PATH=/mnt/ssd2/electrs
   ```

3. Stack starten:

   ```bash
   docker compose up -d --build
   ```

## Erreichbarkeit

- Electrum: `50001`
- Core Lightning P2P: `9735`
- Mempool UI: `http://localhost:4080`

## Hinweise

- `bitcoind` nutzt `txindex=1`, damit `electrs` und `mempool` sauber arbeiten.
- `bitcoind` ist auf Tor-only gestellt (`onlynet=onion`) und exponiert keinen Clearnet-P2P-Port am Host.
- `tor` stellt fuer `bitcoind` einen Onion Service bereit. Die Adresse steht nach Start in `/var/lib/tor/bitcoin-service/hostname` im `tor`-Container.
- `tor` wird als interner SOCKS-Proxy eingebunden. `bitcoind` und `lightningd` nutzen ihn fuer ausgehende Verbindungen.
- Die erste Synchronisation von `bitcoind`, `electrs` und `mempool` dauert je nach Hardware und Storage deutlich.

## Tor Troubleshooting (Permissions)

Wenn `tor` mit folgendem Fehler crasht:

```text
Couldn't create private data directory "/var/lib/tor"
```

ist meist der Volume-Owner falsch. Ursache: Das Verzeichnis gehoert `root`, `tor` laeuft aber als UID/GID `100`.

Fix:

```bash
sudo chown -R 100:100 /var/lib/docker/volumes/btccomposestack_tor-data/_data
```

Zusätzlich ist im Compose explizit gesetzt:

```yaml
services:
  tor:
    user: "100:100"
```

Damit passen Container-User und Volume-Berechtigungen zusammen.

## Aktueller Stack-Status

Aktuell laeuft das Setup als komplette self-hosted Bitcoin-Node-Plattform mit:

- Tor
- Bitcoin Core
- Electrs
- Core Lightning
- MariaDB
- Mempool Backend
- Mempool Frontend
