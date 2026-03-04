# Bitcoin Compose Stack

Dieser Stack startet:

- `bitcoind`
- `electrs`
- `tor`
- `c-lightning` (`lightningd`)
- `mempool` (`backend`, `frontend`, `mariadb`)

## Start

1. Beispieldatei kopieren:

   ```bash
   cp .env.example .env
   ```

2. Passwoerter und Alias in `.env` anpassen.

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
