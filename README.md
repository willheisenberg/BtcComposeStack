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

## Bitcoind Troubleshooting (`chmod ... Input/output error`)

Wenn `bitcoind` in einer Restart-Schleife mit so etwas haengt:

```text
chmod: changing permissions of '/home/bitcoin/.bitcoin': Input/output error
```

dann liegt das fast immer am Host-Mount fuer `BITCOIN_DATA_PATH`, nicht an Bitcoin Core selbst.
Das Image `bitcoin/bitcoin:29.0` fuehrt beim Start ein `chmod` auf `/home/bitcoin/.bitcoin` aus. Das funktioniert nur auf einem Dateisystem mit normalen Linux-ACLs/Unix-Rechten.

Typische Problemfaelle:

- `BITCOIN_DATA_PATH` zeigt auf `NTFS`, `exFAT` oder ein gemountetes Windows-Laufwerk
- der Pfad liegt auf `CIFS/SMB`, manchen `NFS`-Mounts oder FUSE-basiertem Storage
- das Zielverzeichnis ist zwar beschreibbar, unterstuetzt aber kein echtes `chmod/chown`

Fix:

1. Lege den Bitcoin-Datenpfad auf ein natives Linux-Dateisystem wie `ext4` oder `xfs`.
2. Erstelle das Verzeichnis vor dem Start.
3. Setze den Owner auf die UID/GID des Containers (`101:101`).

Beispiel:

```bash
sudo mkdir -p /srv/bitcoin-data
sudo chown -R 101:101 /srv/bitcoin-data
```

Dann in `.env`:

```bash
BITCOIN_DATA_PATH=/srv/bitcoin-data
```

Danach den Container neu starten:

```bash
docker compose up -d bitcoind
```

Wenn du bewusst ein nicht-POSIX-faehiges Laufwerk verwenden willst, ist ein Docker Named Volume die robustere Option als ein direkter Bind-Mount.

## Aktueller Stack-Status

Aktuell laeuft das Setup als komplette self-hosted Bitcoin-Node-Plattform mit:

- Tor
- Bitcoin Core
- Electrs
- Core Lightning
- MariaDB
- Mempool Backend
- Mempool Frontend
