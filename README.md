Crypto Fee Watcher (Outil de surveillance live des frais réseau crypto)

Un outil open-source permettant de surveiller en temps réel les frais de sécurité (gas, fees, coûts de transaction) sur plusieurs blockchains (Bitcoin, Ethereum, Solana, XRPL, etc.) avec un backend Node.js/Express + Socket.IO et un frontend React/Tailwind + Recharts.

✨ Fonctionnalités

Surveillance en temps réel des frais réseau sur plusieurs blockchains.

Frontend moderne en React (Vite, TailwindCSS).

Graphiques dynamiques (Recharts) pour visualiser l’évolution des frais.

Backend extensible basé sur des « plugins » par blockchain.

Notifications live via Socket.IO.

Extensible avec webhooks/alerting et bases de données time-series.


📊 Blockchains prises en charge (par défaut)

Bitcoin : API mempool.space (frais recommandés en sats/vByte).

Ethereum : API Etherscan (Safe/Propose/Fast Gas en gwei).

XRPL (XRP Ledger) : API officielle Ripple (data.ripple.com).

Solana : via RPC public (getRecentBlockhash).

EVM génériques : possibilité d’ajouter n’importe quel RPC (Polygon, Arbitrum, BSC, etc.).


📂 Structure du projet

crypto-fee-watcher/
├── server.js           # Backend Express + Socket.IO
├── package.json
├── .env.example        # Variables d’environnement
├── src/
│   ├── App.jsx         # Frontend React principal
│   ├── index.js        # Bootstrap React
│   └── index.css       # TailwindCSS

🚀 Installation & démarrage

1. Cloner le projet

git clone https://github.com/ton-user/crypto-fee-watcher.git
cd crypto-fee-watcher

2. Installer les dépendances

npm install

3. Configurer les variables d’environnement

Créer un fichier .env à la racine :

PORT=4000
POLL_INTERVAL_MS=10000
ETHERSCAN_API_KEY=ta_clef
SOLANA_RPC=https://api.mainnet-beta.solana.com
EXTRA_EVM_RPC=https://rpc.ankr.com/polygon   # (optionnel)

4. Lancer en mode développement

npm run dev

Le backend s’exécute sur http://localhost:4000 et le frontend Vite sur http://localhost:5173 (par défaut).

5. Lancer en mode production

npm run build
npm start

🔧 Personnalisation

Ajouter un nouveau plugin blockchain → éditer server.js et ajouter une nouvelle fonction de récupération des frais.

Modifier l’intervalle de polling → variable POLL_INTERVAL_MS.

Déployer sur VPS/Docker/Heroku/Railway.


🔐 Sécurité recommandée

Ne jamais exposer directement ton backend sans HTTPS ni authentification.

Stocker les clés API dans un gestionnaire de secrets (Vault, AWS Secrets Manager, etc.).

Mettre en place du rate-limiting sur le backend.

Vérifier régulièrement les dépendances NPM (audit, Snyk, Dependabot).

Rotation des clés API et surveillance des journaux.


📈 Améliorations possibles

Sauvegarde des données historiques dans InfluxDB ou TimescaleDB.

Ajout d’alertes (Slack, Telegram, e-mail, PagerDuty).

Dashboard avancé avec filtres, comparaisons et seuils personnalisés.

Intégration d’autres blockchains (Avalanche, Cosmos, Near, etc.).


🤝 Contribution

Les contributions sont les bienvenues !

1. Fork le repo.


2. Crée une branche feature/ma-feature.


3. Commit tes modifications.


4. Ouvre une Pull Request.



📜 Licence

MIT — libre d’utilisation et de modification.


---

💡 Exemple d’utilisation

Lancer le projet, ouvrir ton navigateur et observer :

les frais Bitcoin (fastest, half-hour, hour)

le gas Ethereum (safe/propose/fast)

les frais XRPL et Solana


Cet outil est utile pour :

les traders qui veulent choisir le meilleur moment pour envoyer des transactions,

les développeurs qui intègrent des DApps,

les équipes infra qui veulent monitorer plusieurs réseaux en parallèle.


