"Une organisation autonome décentralisée (DAO) est une entité sans direction centrale. Les décisions sont prises de bas en haut, gouvernées par une communauté organisée autour d'un ensemble spécifique de règles appliquées sur une blockchain".

- connecter le portefeuille
- créer un abonnement NFT
- recevoir un airdrop symbolique
- voter pour les propositions

npm install
npm start

Passer Metamask sur le réseau Goerli.

Récupérer des Faucets :
- MonCrypto	https://app.mycrypto.com/faucet	0,01	Aucun
- Goerli officiel	https://goerlifaucet.com/	0,25	24 heures
- Maillon de chaîne	https://faucets.chain.link/goerli	0,1	Aucun

# Ajoutez 'Connect to Wallet' à votre tableau de bord DAO.

Dans src/index.js, ajouter :

```
import React from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App';

// Import thirdweb provider and Goerli ChainId
import { ThirdwebProvider } from '@thirdweb-dev/react';
import { ChainId } from '@thirdweb-dev/sdk';

// This is the chainId your dApp will work on.
const activeChainId = ChainId.Goerli;

// Wrap your app with the thirdweb provider <ThirdwebProvider>
const container = document.getElementById('root');
const root = createRoot(container);
root.render(
  <React.StrictMode>
    <ThirdwebProvider desiredChainId={activeChainId}>
      <App />
    </ThirdwebProvider>
  </React.StrictMode>,
);
```

## Ajouter une connexion au portefeuille

Dans App.jsx, ajouter :

```
import { useAddress, ConnectWallet } from '@thirdweb-dev/react';

const App = () => {
  // Use the hooks thirdweb give us.
  const address = useAddress();
  console.log("👋 Address:", address);

  // This is the case where the user hasn't connected their wallet
  // to your web app. Let them call connectWallet.
  if (!address) {
    return (
      <div className="landing">
        <h1>Welcome to NarutoDAO</h1>
        <div className="btn-hero">
          <ConnectWallet />
        </div>
      </div>
    );
  }

  // This is the case where we have the user's address
  // which means they've connected their wallet to our site!
  return (
    <div className="landing">
      <h1>👀 wallet connected, now what!</h1>
    </div>);
}

export default App;
```

# Déployez votre membership NFT


