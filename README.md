LedgerLens — README complet & pro (copier/coller)

> Outil open-source pour surveiller en temps réel les frais réseau (gas / fees / coûts de transaction) multi-blockchain. Backend Node.js/Express + Socket.IO, frontend React (Vite) + Tailwind + Recharts.
Ce README est prêt à copier/coller dans README.md.

Table des matières

1. Pitch rapide


2. Fonctionnalités clés


3. Architecture (vue d’ensemble)


4. Prérequis


5. Installation & démarrage (dev)


6. Configuration — .env (exemple)


7. Scripts utiles & endpoints


8. Déploiement (Docker / PM2 / systemd / Nginx)


9. Ajouter un plugin blockchain (exemple)


10. Alerting / seuils / webhooks (exemple)


11. Monitoring & observabilité (Prometheus / logs)


12. Sécurité — checklist pratique


13. Tests & CI (exemple GitHub Actions)


14. Contribuer


15. Licence

Pitch rapide

Crypto Fee Watcher est une application légère et extensible permettant de monitorer en direct les frais réseau (Bitcoin, Ethereum, XRPL, Solana, EVM chains) et d’émettre des notifications / alertes quand des seuils sont dépassés. Elle cible les devs, les teams infra, et les traders.

Fonctionnalités clés

Surveillance en live via WebSocket (Socket.IO).

Plugins modulaires par blockchain (RPC / REST).

Frontend graphique avec séries temporelles (Recharts).

Extensibilité : alertes webhook/Slack/Telegram, persistance time-series (InfluxDB/TimescaleDB).

Endpoints HTTP pour état/dernières mesures.

Architecture (vue d’ensemble)

[ Collector Pollers ] -> [ Express Backend + Socket.IO ] -> [ Frontend React ]
         |                                          \
         v                                           `-> [ Alerting Webhooks / DB / Prometheus ]
  (Bitcoin, Ethereum, XRPL, Solana, EVM RPCs)

Prérequis

Node.js >= 18 (recommandé)

npm (ou yarn)

(optionnel) Docker & docker-compose pour production

Clé API Etherscan (pour une meilleure fiabilité Ethereum) — recommandé

Installation & démarrage (dev)

# 1) Cloner
git clone https://github.com/<ton-org>/crypto-fee-watcher.git
cd crypto-fee-watcher

# 2) Installer dépendances
npm install

# 3) Créer .env (voir plus bas pour exemple)
cp .env.example .env

# 4) Lancer en dev (concurrently: backend + vite)
npm run dev

Accès : backend http://localhost:4000, frontend http://localhost:5173 (Vite).

Configuration — .env (exemple prêt à coller)

# serveur
PORT=4000
POLL_INTERVAL_MS=10000  # intervalle de polling en ms

# APIs
ETHERSCAN_API_KEY=TON_ETHERSCAN_KEY   # facultatif mais recommandé
SOLANA_RPC=https://api.mainnet-beta.solana.com
EXTRA_EVM_RPC=https://rpc.ankr.com/polygon  # exemple (optionnel)

# production
VITE_BACKEND_URL=http://localhost:4000    # côté frontend (vite)

🔎 Explications rapides

POLL_INTERVAL_MS : fréquence d'interrogation des APIs/RPC (10s par défaut).

ETHERSCAN_API_KEY : limite de rate améliorée pour le tracking Ethereum.

EXTRA_EVM_RPC : exemple pour ajouter une RPC EVM supplémentaire.

Scripts utiles & endpoints

package.json — Scripts usuels

{
  "scripts": {
    "dev": "concurrently \"nodemon server.js\" \"vite\"",
    "start": "node server.js",
    "build": "vite build"
  }
}

Endpoints HTTP

GET /health → { ok: true }

GET /last → { ts: <timestamp>, data: [...] } (dernières mesures)

WebSocket (Socket.IO) events:

fees:update → payload périodique avec mesures

fees:error → erreurs du poller

Exemple curl :

curl http://localhost:4000/last

Exemple client Socket.IO (node / browser)

import { io } from "socket.io-client";
const socket = io("http://localhost:4000");

socket.on("connect", () => console.log("connecté"));
socket.on("fees:update", (payload) => {
  console.log("mise à jour fees:", payload);
});

Déploiement — Docker (exemple)

Dockerfile (backend simple)

# Backend + frontend build (monolithique simple)
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app ./
RUN npm ci --production
EXPOSE 4000
CMD ["node", "server.js"]

docker-compose.yml (exemple minimal)

version: "3.8"
services:
  app:
    build: .
    ports:
      - "4000:4000"
    environment:
      - PORT=4000
      - POLL_INTERVAL_MS=10000
      - ETHERSCAN_API_KEY=${ETHERSCAN_API_KEY}
    restart: unless-stopped

Reverse proxy Nginx (snippet HTTPS)

server {
  listen 80;
  server_name fees.example.com;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl;
  server_name fees.example.com;
  ssl_certificate /etc/letsencrypt/live/fees.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/fees.example.com/privkey.pem;

  location / {
    proxy_pass http://127.0.0.1:4000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }

  location /socket.io/ {
    proxy_pass http://127.0.0.1:4000/socket.io/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
  }
}

Ajouter un plugin blockchain — template (copier/coller)

Exemple : nouveau plugin (pseudo)

// server.js (ou plugins/x_newchain.js)
const axios = require('axios');

async function fetchNewChainFees() {
  try {
    const res = await axios.get('https://api.newchain.org/v1/fees');
    // normaliser la forme renvoyée
    return { network: 'newchain', timestamp: Date.now(), data: res.data };
  } catch (err) {
    return { network: 'newchain', error: err.message, timestamp: Date.now() };
  }
}

// Dans pollAll(), ajouter fetchNewChainFees()

Bonnes pratiques plugin

Toujours retourner { network, timestamp, data } ou { network, timestamp, error }.

Gérer timeouts & retries (axios timeout + try/catch).

Valider / normaliser les unités (gwei / wei / sats/vByte).

Alerting / seuils / webhooks — exemple simple

1) Configuration (extrait .env)

ALERT_WEBHOOK=https://hooks.example.com/incoming
ALERT_THRESHOLD_ETH_GWEI=120

2) Code d’alerte (backend)

const axios = require('axios');
async function checkAlertAndNotify(measure) {
  // exemple : measure pour ethereum
  const fastGwei = Number(measure.data.FastGasPrice);
  if (fastGwei >= Number(process.env.ALERT_THRESHOLD_ETH_GWEI)) {
    await axios.post(process.env.ALERT_WEBHOOK, {
      network: 'ethereum',
      metric: 'FastGasPrice',
      value: fastGwei,
      ts: Date.now(),
    });
  }
}

3) Intégration
Appeler checkAlertAndNotify() à chaque fees:update avant d’émettre l’événement socket.

Monitoring & observabilité

Exposer métriques Prometheus (snippet avec prom-client)

const client = require('prom-client');
const gauge = new client.Gauge({ name: 'eth_fast_gas_gwei', help: 'fast gas gwei' });
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});

Logs

JSON logs (pino / winston) pour ingestion centralisée (ELK / Loki).

Niveau info / warn / error, rotation et centralisation.


Persistance historique (optionnel)

InfluxDB / TimescaleDB pour séries temporelles ; écrire mesures toutes les N minutes pour analyses historiques.

Sécurité — checklist pratique (obligatoire)

HTTPS partout (Nginx / Let's Encrypt).

Stockage sécurisé des API keys (Vault / AWS Secrets Manager).

Ne jamais committer .env → .gitignore doit contenir .env.

Limiter l’accès au backend par IP ou auth (JWT / API key) si exposé publiquement.

Audit dépendances :

npm audit
npm audit fix --force  # avec prudence

Scanner chaîne logistique (npm packages) et activer dépendabot.

Rotation des clés si une version compromise d’un package est découverte.

Rate limiting (express-rate-limit) pour éviter abuse des APIs publiques.

Tests & CI — exemple GitHub Actions (build + test)

.github/workflows/ci.yml

name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: node-version: 18
      - run: npm ci
      - run: npm run build
      - run: npm test || true

> Ajouter tests unitaires (Jest) pour les fonctions de parsing/normalisation des plugins.

Contribuer

Workflow recommandé

1. Fork → feature/xxx → commit → PR.


2. Ajoute tests si tu changes de logique.


3. Documente nouveaux plugins / variables .env.



Code style : Prettier + ESLint (recommandé). Ajouter hooks husky pour lint / tests pré-commit.

FAQ rapide

Q : Puis-je ajouter d’autres RPC privés ?
R : Oui, définis EXTRA_EVM_RPC ou ajoute un plugin avec ton RPC (auth si nécessaire).

Q : Peut-on persister toutes les mesures ?
R : Oui — connecter un writer InfluxDB / TimescaleDB dans la boucle pollAll().

Q : Est-ce sécurisé pour production ?
R : Le prototype l’est peu ; appliquer les recommandations de la section sécurité et déployer derrière reverse-proxy + auth.

Exemple rapide — « run & voir » (copy/paste)

git clone https://github.com/<ton-org>/crypto-fee-watcher.git
cd crypto-fee-watcher
cp .env.example .env
# éditer .env pour ajouter ETHERSCAN_API_KEY si besoin
npm install
npm run dev
# ouvrir http://localhost:5173

Licence

MIT — libre d’utilisation, modification et distribution.
