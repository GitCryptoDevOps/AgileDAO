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

Le contrat va être créé avec ThirdWeb (https://github.com/thirdweb-dev/contracts).

Le tableau de bord fourni par ThirdWeb est https://thirdweb.com/dashboard.

QuickNode permet de diffuser notre transaction de création de contrat afin qu'elle puisse être récupérée par les mineurs sur le testnet le plus rapidement possible. Une fois la transaction extraite, elle est ensuite diffusée sur la blockchain en tant que transaction légitime.

Créer un compte QuickNode (https://www.quicknode.com/).

Créer un endpoint sur le Polygon sur le réseau Mumbai Testnet et sélectioner le mode `Discover`.

Créer un fichier `.env` avec ce contenu :

```
QUICKNODE_API_URL=<YOUR_API_URL>
WALLET_ADDRESS=<YOUR_WALLET>
0x04158d6B8bc018D41d910494c0f8d15813c8e638

```

## Initialiser le SDK

Modifier `scripts/1-initialize-sdk.js` :

```
import { ThirdwebSDK } from "@thirdweb-dev/sdk";

// Importing and configuring our .env file that we use to securely store our environment variables
import dotenv from "dotenv";
dotenv.config();

// Some quick checks to make sure our .env is working.
if (!process.env.PRIVATE_KEY || process.env.PRIVATE_KEY === "") {
  console.log("🛑 Private key not found.");
}

if (!process.env.QUICKNODE_API_URL || process.env.QUICKNODE_API_URL === "") {
  console.log("🛑 QuickNode API URL not found.");
}

if (!process.env.WALLET_ADDRESS || process.env.WALLET_ADDRESS === "") {
  console.log("🛑 Wallet Address not found.");
}

const sdk = ThirdwebSDK.fromPrivateKey(
  // Your wallet private key. ALWAYS KEEP THIS PRIVATE, DO NOT SHARE IT WITH ANYONE, add it to your .env file and do not commit that file to github!
  process.env.PRIVATE_KEY,
  // RPC URL, we'll use our QuickNode API URL from our .env file.
  process.env.QUICKNODE_API_URL
);

(async () => {
  try {
    const address = await sdk.getSigner().getAddress();
    console.log("👋 SDK initialized by address:", address)
  } catch (err) {
    console.error("Failed to get apps from the sdk", err);
    process.exit(1);
  }
})();

// We are exporting the initialized thirdweb SDK so that we can use it in our other scripts
export default sdk;
```

```
npm init -y
npm i --save-dev node@17
npm config set prefix=$(pwd)/node_modules/node
export PATH=$(pwd)/node_modules/node/bin:$PATH
```

```
node scripts/1-initialize-sdk.js
```

## Créer une collection ERC-1155

Créer et déployer un contrat ERC-1155 à Goerli

Avec un ERC-1155, plusieurs personnes peuvent être titulaires du même NFT.

Au lieu de créer un nouveau NFT à chaque fois, nous pouvons simplement attribuer le même NFT à tous nos membres.

C'est aussi plus économe en gaz.

Créer `scripts/2-deploy-drop.js` :

```
import { AddressZero } from "@ethersproject/constants";
import sdk from "./1-initialize-sdk.js";
import { readFileSync } from "fs";

(async () => {
  try {
    const editionDropAddress = await sdk.deployer.deployEditionDrop({
      // The collection's name, ex. CryptoPunks
      name: "NarutoDAO Membership",
      // A description for the collection.
      description: "A DAO for fans of Naruto.",
      // The image that will be held on our NFT! The fun part :).
      image: readFileSync("scripts/assets/naruto.png"),
      // We need to pass in the address of the person who will be receiving the proceeds from sales of nfts in the contract.
      // We're planning on not charging people for the drop, so we'll pass in the 0x0 address
      // you can set this to your own wallet address if you want to charge for the drop.
      primary_sale_recipient: AddressZero,
    });

    // this initialization returns the address of our contract
    // we use this to initialize the contract on the thirdweb sdk
    const editionDrop = await sdk.getContract(editionDropAddress, "edition-drop");

    // with this, we can get the metadata of our contract
    const metadata = await editionDrop.metadata.get();

    console.log(
      "✅ Successfully deployed editionDrop contract, address:",
      editionDropAddress,
    );
    console.log("✅ editionDrop metadata:", metadata);
  } catch (error) {
    console.log("failed to deploy editionDrop contract", error);
  }
})();
```

Modifier name, descriptionet primary_sale_recipient, et image

`image` est chargé à partir d'un fichier local sous scripts/assets

```
node scripts/2-deploy-drop.js
```

Un contrat ERC-1155 a été déployé sur Goerli.

Sur https://goerli.etherscan.io/, coller l'adresse du contrat editionDrop.

On voit une transaction correspondant au déploiement du contrat.

ThirdWeb a automatiquement uploadé l'image de votre collection sur IPFS. Un lien commençant par https://gateway.ipfscdn.io est affiché.

IPFS est essentiellement un système de stockage décentralisé.

# Déployez les métadonnées NFT

## Configurer les données NFT


Déployer les métadonnées associées au membership NFT :

Créer le fichier scripts/3-config-nft.js :

```
import sdk from "./1-initialize-sdk.js";
import { readFileSync } from "fs";

(async () => {
  try {
    const editionDrop = await sdk.getContract("INSERT_EDITION_DROP_ADDRESS", "edition-drop");
    await editionDrop.createBatch([
      {
        name: "Leaf Village Headband",
        description: "This NFT will give you access to NarutoDAO!",
        image: readFileSync("scripts/assets/headband.png"),
      },
    ]);
    console.log("✅ Successfully created a new NFT in the drop!");
  } catch (error) {
    console.error("failed to create the new NFT", error);
  }
})();
```

Modifier `INSERT_EDITION_DROP_ADDRESS`.

Ensuite :
- accéder au contrat editionDrop, qui est un ERC-1155.
- configurer le NFT sur notre ERC-1155 en utilisant createBatch
- Configurer les propriétés :
  - name : Le nom de notre NFT.
  - description : La description de notre NFT
  - image : L'image de notre NFT. C'est à l'image du NFT que les utilisateurs prétendront pouvoir accéder à votre DAO.

```
node scripts/3-config-nft.js
```

## Configurer la condition de réclamation

nous devons mettre en place nos "conditions de réclamation". 

Créer scripts/4-set-claim-condition.js :

```
import sdk from "./1-initialize-sdk.js";
import { MaxUint256 } from "@ethersproject/constants";

(async () => {
  try {
    const editionDrop = await sdk.getContract("INSERT_EDITION_DROP_ADDRESS", "edition-drop");
    // We define our claim conditions, this is an array of objects because
    // we can have multiple phases starting at different times if we want to
    const claimConditions = [{
      // When people are gonna be able to start claiming the NFTs (now)
      startTime: new Date(),
      // The maximum number of NFTs that can be claimed.
      maxClaimable: 50_000,
      // The price of our NFT (free)
      price: 0,
      // The amount of NFTs people can claim in one transaction.
      maxClaimablePerWallet: 1,
      // We set the wait between transactions to unlimited, which means
      // people are only allowed to claim once.
      waitInSeconds: MaxUint256,
    }]

    await editionDrop.claimConditions.set("0", claimConditions);
    console.log("✅ Sucessfully set claim condition!");
  } catch (error) {
    console.error("Failed to set claim condition", error);
  }
})();
```

remplacer INSERT_EDITION_DROP_ADDRESSpar l'adresse de votre contrat ERC-1155.

`startTime` est le moment où les utilisateurs sont autorisés à commencer à frapper des NFT

`maxClaimable` est le nombre maximum de NFT de membership qui peuvent être frappés

`maxClaimablePerWallet` spécifie combien de jetons quelqu'un peut réclamer dans une seule transaction

`price` fixe le prix de notre NFT

`waitInSeconds` est le temps entre les transactions. Parce que nous ne voulons que les gens réclament une fois, nous le fixons au nombre maximum autorisé par la blockchain.

`editionDrop.claimConditions.set("0", claimConditions)` interagit avec notre contrat déployé en chaîne et ajuste les conditions

`0` : notre adhésion NFT a un `tokenId` de 0 puisque c'est le premier jeton de notre contrat ERC-1155. Avec ERC-1155, plusieurs personnes peuvent frapper le même NFT. Dans ce cas, tout le monde frappe un NFT avec id 0. Mais, nous pourrions aussi avoir un NFT différent avec peut-être un identifiant 1et peut-être que nous donnerions le NFT aux membres de notre DAO qui sont exceptionnels.

```
node scripts/4-set-claim-condition.js
```

# Laissez les utilisateurs frapper vos NFT

si nous détectons que notre utilisateur a un abonnement NFT, montrez-lui notre écran "DAO Dashboard" où il peut voter sur les propositions et voir les informations liées à DAO.

si nous détectons que l'utilisateur n'a pas notre NFT, nous lui donnerons un bouton pour en fabriquer un.

## Vérifiez si l'utilisateur possède un abonnement NFT

Dans App.jsx, ajouter :

```
import { useAddress, ConnectWallet, useContract, useNFTBalance } from '@thirdweb-dev/react';
import { useState, useEffect, useMemo } from 'react';
```

Sous `console.log("👋 Address:", address);`, ajouter :

```
 // Initialize our Edition Drop contract
  const editionDropAddress = "INSERT_EDITION_DROP_ADDRESS"
  const { contract: editionDrop } = useContract(editionDropAddress, "edition-drop");
  // Hook to check if the user has our NFT
  const { data: nftBalance } = useNFTBalance(editionDrop, address, "0")

  const hasClaimedNFT = useMemo(() => {
    return nftBalance && nftBalance.gt(0)
  }, [nftBalance])

  // ... include all your other code that was already there below.
  ```
  
- initialiser notre editionDropcontrat.
- utiliser `useNFTBalance` pour vérifier combien de NFT le mur connecté détient. Cela interrogera en fait notre contrat intelligent déployé pour les données. "0" c'est le tokenIdde notre adhésion NFT.
- Si un utilisateur n'a pas de NFT, créer un bouton pour permettre à l'utilisateur d'en créer un.

## Construire un bouton "Mint NFT"

Dans App.jsx, ajouter :

```
import { Web3Button } from '@thirdweb-dev/react';

const App = () => {
  ...
  return (
    <div className="mint-nft">
      <h1>Mint your free 🍪DAO Membership NFT</h1>
      <div className="btn-hero">
        <Web3Button 
          contractAddress={editionDropAddress}
          action={contract => {
            contract.erc1155.claim(0, 1)
          }}
          onSuccess={() => {
            console.log(`🌊 Successfully Minted! Check it out on OpenSea: https://testnets.opensea.io/assets/${editionDrop.getAddress()}/0`);
          }}
          onError={error => {
            console.error("Failed to mint NFT", error);
          }}
        >
          Mint your NFT (FREE)
        </Web3Button>
      </div>
    </div>
  );
}
```

- utiliser `Web3Button` pour créer un bouton qui appellera notre fonction `claim` sur notre contrat `editionDrop`.
- transmettre `address` de l'utilisateur, `1` pour la quantité (`amount`) des NFT à mint, et `0` pour le `tokenId` du NFT à mint.

Une transaction est réalisée sur notre contrat intelligent pour créer le NFT.

Nous transmettons également des callbacks `onSuccess`et `onError` pour gérer les cas de réussite et d'erreur.

Pour mint un NFT, Metamask apparaît pour payer le gaz. Le lien vers le Testnet OpenSea (https://testnets.opensea.io/) est alors affiché dans la console.





