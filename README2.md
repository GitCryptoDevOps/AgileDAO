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

Alchemy permet de diffuser notre transaction de création de contrat afin qu'elle puisse être récupérée par les mineurs sur le testnet le plus rapidement possible. Une fois la transaction extraite, elle est ensuite diffusée sur la blockchain en tant que transaction légitime.

Créer un compte Alchemy sur https://dashboard.alchemy.com/.

Créer un endpoint sur Ethereum sur le réseau Goerli.

Créer un fichier `.env` avec ce contenu :

```
ALCHEMY_API_URL=<YOUR_API_URL>
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

if (!process.env.ALCHEMY_API_URL || process.env.ALCHEMY_API_URL === "") {
  console.log("🛑 Alchemy API URL not found.");
}

if (!process.env.WALLET_ADDRESS || process.env.WALLET_ADDRESS === "") {
  console.log("🛑 Wallet Address not found.");
}

const sdk = ThirdwebSDK.fromPrivateKey(
  // Your wallet private key. ALWAYS KEEP THIS PRIVATE, DO NOT SHARE IT WITH ANYONE, add it to your .env file and do not commit that file to github!
  process.env.PRIVATE_KEY,
  // RPC URL, we'll use our Alchemy API URL from our .env file.
  process.env.ALCHEMY_API_URL
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

Dans l'onglet `Internal txns`, on voit une transaction correspondant au déploiement du contrat.

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

Cliquer sur `MINT YOUR NFT (FREE)`.

Dans la console, le lien vers OpenSea est affiché.

Cliquer sur le lien ou chercher sur OpenSea le contrat.

#  CREATE TOKEN + GOVERNANCE

# Deploy an ERC-20 contract

un jeton de gouvernance permet aux utilisateurs de voter sur des propositions.

## Déployez votre token

scripts/5-deploy-token.js :

```
import { AddressZero } from "@ethersproject/constants";
import sdk from "./1-initialize-sdk.js";

(async () => {
  try {
    // Deploy a standard ERC-20 contract.
    const tokenAddress = await sdk.deployer.deployToken({
      // What's your token's name? Ex. "Ethereum"
      name: "NarutoDAO Governance Token",
      // What's your token's symbol? Ex. "ETH"
      symbol: "HOKAGE",
      // This will be in case we want to sell our token,
      // because we don't, we set it to AddressZero again.
      primary_sale_recipient: AddressZero,
    });
    console.log(
      "✅ Successfully deployed token contract, address:",
      tokenAddress,
    );
  } catch (error) {
    console.error("failed to deploy token contract", error);
  }
})();
```

`sdk.deployer.deployToken` déploie un contrat de jeton standard ERC-20
`name` : nom du token
`symbol` : symbole du token

Le contrat déployé par ThirdWeb est https://github.com/thirdweb-dev/contracts/blob/main/contracts/token/TokenERC20.sol

```
node scripts/5-deploy-token.js
```

Un nouveau contrat de token est déployé.

Sur https://goerli.etherscan.io/, vous voyez une nouvelle transaction sur votre contrat.

Le nouveau contrat peut aussi être trouvé.

Dans Metamask, ajouter le token associé à ce nouveau contrat.

## Créez votre réserve de jetons

il n'y a aucun jeton disponible pour les personnes réclamant

6-print-money.js :

```
import sdk from "./1-initialize-sdk.js";

(async () => {
  try {
    // This is the address of our ERC-20 contract printed out in the step before.
    const token = await sdk.getContract("<TOKEN-ADDRESS>", "token");
    // What's the max supply you want to set? 1,000,000 is a nice number!
    const amount = 1_000_000;
    // Interact with your deployed ERC-20 contract and mint the tokens!
    await token.mint(amount);
    const totalSupply = await token.totalSupply();

    // Print out how many of our token's are out there now!
    console.log("✅ There now is", totalSupply.displayValue, "$HOKAGE in circulation");
  } catch (error) {
    console.error("Failed to print money", error);
  }
})();
```

Remplacer <TOKEN-ADDRESS> par l'adresse du contrat du token créé.

`amount` indique le montant à mint.
  
```
node scripts/6-print-money.js
```

Aller voir le contrat ERC-20 dans Etherscan

Des informations sont affichées :
- qui détient votre jeton
- qui transfère des jetons
- combien de jetons sont déplacés.
- Max Total Supply

## Airdrop
  
7-airdrop-token.js :
  
```
import sdk from "./1-initialize-sdk.js";

(async () => {
  try {
    // This is the address to our ERC-1155 membership NFT contract.
    const editionDrop = await sdk.getContract("INSERT_EDITION_DROP_ADDRESS", "edition-drop");
    // This is the address to our ERC-20 token contract.
    const token = await sdk.getContract("INSERT_TOKEN_ADDRESS", "token");
    // Grab all the addresses of people who own our membership NFT, which has 
    // a tokenId of 0.
    const walletAddresses = await editionDrop.history.getAllClaimerAddresses(0);

    if (walletAddresses.length === 0) {
      console.log(
        "No NFTs have been claimed yet, maybe get some friends to claim your free NFTs!",
      );
      process.exit(0);
    }

    // Loop through the array of addresses.
    const airdropTargets = walletAddresses.map((address) => {
      // Pick a random # between 1000 and 10000.
      const randomAmount = Math.floor(Math.random() * (10000 - 1000 + 1) + 1000);
      console.log("✅ Going to airdrop", randomAmount, "tokens to", address);

      // Set up the target.
      const airdropTarget = {
        toAddress: address,
        amount: randomAmount,
      };

      return airdropTarget;
    });

    // Call transferBatch on all our airdrop targets.
    console.log("🌈 Starting airdrop...");
    await token.transferBatch(airdropTargets);
    console.log("✅ Successfully airdropped tokens to all the holders of the NFT!");
  } catch (err) {
    console.error("Failed to airdrop tokens", err);
  }
})();
```
  
- récupérer les détenteurs de notre NFT auprès du editionDrop
- émettre leur jeton à l'aide de fonctions sur le token
- `getAllClaimerAddresses` saisit les `walletAddresses` des personnes qui ont le NFT membership NFT avec un tokenId à "0".
- exécute transferBatch sur tous les fichiers airdropTargets. transferBatch boucle automatiquement sur toutes les cibles et envoie le jeton

```
  node scripts/7-airdrop-token.js
```

Dans le monde réel , un airdrop ne se produit généralement qu'une seule fois. 

Sur Etherscan, sur mon contrat ERC-20, on peut voir les nouveaux détenteurs de jetons et combien ils en possèdent.

# Montrez les détenteurs de jetons sur le tableau de bord DAO
  
## Récupérer les détenteurs de jetons sur l'application Web

Dans App.jsx, ajoutez le fichier token :

```
// Initialize our token contract
const { contract: token } = useContract('INSERT_TOKEN_ADDRESS', 'token');
```

Depuis l'ERC-1155, nous obtiendrons les adresses de tous nos membres. À partir de l'ERC-20, nous récupérerons le nombre de jetons que possède chaque membre.

Sous `const hasClaimedNFT`, ajouter :
  
```
// Holds the amount of token each member has in state.
const [memberTokenAmounts, setMemberTokenAmounts] = useState([]);
// The array holding all of our members addresses.
const [memberAddresses, setMemberAddresses] = useState([]);

// A fancy function to shorten someones wallet address, no need to show the whole thing.
const shortenAddress = (str) => {
  return str.substring(0, 6) + '...' + str.substring(str.length - 4);
};

// This useEffect grabs all the addresses of our members holding our NFT.
useEffect(() => {
  if (!hasClaimedNFT) {
    return;
  }

  // Just like we did in the 7-airdrop-token.js file! Grab the users who hold our NFT
  // with tokenId 0.
  const getAllAddresses = async () => {
    try {
      const memberAddresses = await editionDrop?.history.getAllClaimerAddresses(
        0,
      );
      setMemberAddresses(memberAddresses);
      console.log('🚀 Members addresses', memberAddresses);
    } catch (error) {
      console.error('failed to get member list', error);
    }
  };
  getAllAddresses();
}, [hasClaimedNFT, editionDrop?.history]);

// This useEffect grabs the # of token each member holds.
useEffect(() => {
  if (!hasClaimedNFT) {
    return;
  }

  const getAllBalances = async () => {
    try {
      const amounts = await token?.history.getAllHolderBalances();
      setMemberTokenAmounts(amounts);
      console.log('👜 Amounts', amounts);
    } catch (error) {
      console.error('failed to get member balances', error);
    }
  };
  getAllBalances();
}, [hasClaimedNFT, token?.history]);

// Now, we combine the memberAddresses and memberTokenAmounts into a single array
const memberList = useMemo(() => {
  return memberAddresses.map((address) => {
    // We're checking if we are finding the address in the memberTokenAmounts array.
    // If we are, we'll return the amount of token the user has.
    // Otherwise, return 0.
    const member = memberTokenAmounts?.find(({ holder }) => holder === address);

    return {
      address,
      tokenAmount: member?.balance.displayValue || '0',
    };
  });
}, [memberAddresses, memberTokenAmounts]);
```

- appeler getAllClaimerAddressespour avoir toutes les adresses de nos membres titulaires d'un NFT de notre contrat ERC-1155.
- appeler getAllHolderBalancespour obtenir les soldes de jetons de tous ceux qui détiennent notre jeton sur notre contrat ERC-20.
- combiner les données dans memberListun joli tableau qui combine à la fois l'adresse du membre et son solde de jetons

`useMemo` (https://reactjs.org/docs/hooks-reference.html#usememo) est une façon élégante dans React de stocker une variable calculée.
  
`getAllHolderBalances` ne répond pas à notre besoin car quelqu'un peut être dans notre DAO et détenir zéro jeton.
 
## Afficher les données des membres sur le tableau de bord DAO
  
Avant le `return` final, insérer :
  
```
// If the user has already claimed their NFT we want to display the internal DAO page to them
// only DAO members will see this. Render all the members + token amounts.
if (hasClaimedNFT) {
  return (
    <div className="member-page">
      <h1>🍪DAO Member Page</h1>
      <p>Congratulations on being a member</p>
      <div>
        <div>
          <h2>Member List</h2>
          <table className="card">
            <thead>
              <tr>
                <th>Address</th>
                <th>Token Amount</th>
              </tr>
            </thead>
            <tbody>
              {memberList.map((member) => {
                return (
                  <tr key={member.address}>
                    <td>{shortenAddress(member.address)}</td>
                    <td>{member.tokenAmount}</td>
                  </tr>
                );
              })}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  );
}
```
  
# Construire une trésorerie + gouvernance
  
mettre en place un contrat de gouvernance qui permet aux gens de voter sur des propositions en utilisant leurs jetons
  
## Déployer un contrat de gouvernance
  
Qui a le droit de voter ? Combien de temps les gens ont-ils pour voter ? Quel est le nombre minimum de jetons dont une personne a besoin pour créer une proposition ?

8-deploy-vote.js:
  
```
import sdk from "./1-initialize-sdk.js";

(async () => {
  try {
    const voteContractAddress = await sdk.deployer.deployVote({
      // Give your governance contract a name.
      name: "My amazing DAO",

      // This is the location of our governance token, our ERC-20 contract!
      voting_token_address: "INSERT_TOKEN_ADDRESS",

      // These parameters are specified in number of blocks. 
      // Assuming block time of around 13.14 seconds (for Ethereum)

      // After a proposal is created, when can members start voting?
      // For now, we set this to immediately.
      voting_delay_in_blocks: 0,

      // How long do members have to vote on a proposal when it's created?
      // we will set it to 1 day = 6570 blocks
      voting_period_in_blocks: 6570,

      // The minimum % of the total supply that need to vote for
      // the proposal to be valid after the time for the proposal has ended.
      voting_quorum_fraction: 0,

      // What's the minimum # of tokens a user needs to be allowed to create a proposal?
      // I set it to 0. Meaning no tokens are required for a user to be allowed to
      // create a proposal.
      proposal_token_threshold: 0,
    });

    console.log(
      "✅ Successfully deployed vote contract, address:",
      voteContractAddress,
    );
  } catch (err) {
    console.error("Failed to deploy vote contract", err);
  }
})();
```

`deployer.deployVote` établit le contrat

`voting_token_address` est le contrat qui sait quel jeton de gouvernance accepter

`voting_delay_in_blocks` permet de donner aux gens le temps d'examiner la proposition avant qu'ils ne soient autorisés à voter dessus
 
`voting_period_in_blocks` spécifie combien de temps quelqu'un doit voter une fois qu'une proposition est mise en ligne. Nous le faisons en blocs, ce qui, selon la blockchain sur laquelle vous vous trouvez, peut prendre plus de temps, pour Ethereum/Goerli, il y a un bloc toutes les 13 secondes ou alors, donc en moyenne, il y a 6570 blocs par jour.

`voting_quorum_fraction` : Disons qu'un membre crée une proposition et que les  199 autres  membres de DAO sont en vacances à Disney World et ne sont pas en ligne. Eh bien, dans ce cas, si ce membre du DAO crée la proposition et vote "OUI" sur sa propre proposition - cela signifie que 100% des votes ont dit "OUI" (puisqu'il n'y a eu qu'un seul vote) et la proposition  passerait une fois `voting_period_in_blocks`  est en haut! Pour éviter cela, nous utilisons un quorum qui dit "Pour qu'une proposition soit acceptée, un minimum de x % de jetons doit être utilisé lors du vote".

Si `voting_quorum_fraction` vaut 0, cela signifie que la proposition passera quel que soit le % de jeton utilisé lors du vote. Cela signifie qu'une personne pourrait techniquement passer une proposition elle-même si les autres membres sont en vacances. Le quorum que vous définissez dans le monde réel dépend de votre approvisionnement et de la quantité que vous avez initialement larguée.

`proposal_token_threshold` à : "0" permet à quiconque de créer une proposition même s'il ne détient aucun jeton de gouvernance.
  
```
node scripts/8-deploy-vote.js
```

Un nouveau contrat a été déployé pour voter sur des propositions en chaîne. Il s'agit d'un  contrat de gouvernance standard (https://docs.openzeppelin.com/contracts/4.x/api/governance). https://github.com/thirdweb-dev/contracts/blob/main/contracts/vote/VoteERC20.sol
  
Voir le contrat sur https://goerli.etherscan.io/.
  
Nous avons donc maintenant trois contrats : notre contrat NFT, notre contrat de jeton et notre contrat de vote
  
## Configurez votre trésorerie
  
Le contrat de vote lui-même n'a pas la capacité de déplacer nos jetons. 
  
Parce que  vous avez créé la fourniture de jetons. Votre portefeuille possède l'accès à l'ensemble de l'offre. Donc, vous seul avez le pouvoir d'accéder à la supply, de déplacer des jetons, de faire des airdrops, etc. 
  
Donc nous allons transférer 90 % de tous nos jetons vers le contrat de vote
  
Exemple d'ENS :
  
- 50% de la supply à leur trésorerie communautaire
- 25% de airdrops,
- 25% à l'équipe principale + contributeurs.

9-setup-vote.js:

```
import sdk from "./1-initialize-sdk.js";

(async () => {
  try {
    // This is our governance contract.
    const vote = await sdk.getContract("INSERT_VOTE_ADDRESS", "vote");
    // This is our ERC-20 contract.
    const token = await sdk.getContract("INSERT_TOKEN_ADDRESS", "token");
    // Give our treasury the power to mint additional token if needed.
    await token.roles.grant("minter", vote.getAddress());

    console.log(
      "Successfully gave vote contract permissions to act on token contract"
    );
  } catch (error) {
    console.error(
      "failed to grant vote contract permissions on token contract",
      error
    );
    process.exit(1);
  }

  try {
    // This is our governance contract.
    const vote = await sdk.getContract("INSERT_VOTE_ADDRESS", "vote");
    // This is our ERC-20 contract.
    const token = await sdk.getContract("INSERT_TOKEN_ADDRESS", "token");
    // Grab our wallet's token balance, remember -- we hold basically the entire supply right now!
    const ownedTokenBalance = await token.balanceOf(
      process.env.WALLET_ADDRESS
    );

    // Grab 90% of the supply that we hold.
    const ownedAmount = ownedTokenBalance.displayValue;
    const percent90 = Number(ownedAmount) / 100 * 90;

    // Transfer 90% of the supply to our voting contract.
    await token.transfer(
      vote.getAddress(),
      percent90
    ); 

    console.log("✅ Successfully transferred " + percent90 + " tokens to vote contract");
  } catch (err) {
    console.error("failed to transfer tokens to vote contract", err);
  }
})();
```
  
- `token.balanceOf` récupère le nombre total de jetons que nous avons dans notre portefeuille
- A l'heure actuelle, notre portefeuille contient essentiellement la totalité de la supply, à l'exception du jeton que nous avons airdropé.
- Il reste donc 90%.
- `token.transfer` transfère ces 90% au module de vote.

```
node scripts/9-setup-vote.js
```

Aller sur https://goerli.etherscan.io et voir ce nouveau contrat.

Cliquer sur le menu déroulant à côté du mot `Erc20 Token Txns`. On voit le transfert de 900.000 tokens.
  
On peut confirmer cela en cliquant dans la section `Contract Overview`, dans la liste déroulante `Token`, on retrouve bien la valeur 900.000.
  
# Laissez les utilisateurs voter sur les propositions
  
## Créez les deux premières propositions de votre DAO

10-create-vote-proposals.js :
  
```
import sdk from "./1-initialize-sdk.js";
import { ethers } from "ethers";

(async () => {
  try {
    // This is our governance contract.
    const vote = await sdk.getContract("INSERT_VOTE_ADDRESS", "vote");
    // This is our ERC-20 contract.
    const token = await sdk.getContract("INSERT_TOKEN_ADDRESS", "token");
    // Create proposal to mint 420,000 new token to the treasury.
    const amount = 420_000;
    const description = "Should the DAO mint an additional " + amount + " tokens into the treasury?";
    const executions = [
      {
        // Our token contract that actually executes the mint.
        toAddress: token.getAddress(),
        // Our nativeToken is ETH. nativeTokenValue is the amount of ETH we want
        // to send in this proposal. In this case, we're sending 0 ETH.
        // We're just minting new tokens to the treasury. So, set to 0.
        nativeTokenValue: 0,
        // We're doing a mint! And, we're minting to the vote, which is
        // acting as our treasury.
        // in this case, we need to use ethers.js to convert the amount
        // to the correct format. This is because the amount it requires is in wei.
        transactionData: token.encoder.encode(
          "mintTo", [
          vote.getAddress(),
          ethers.utils.parseUnits(amount.toString(), 18),
        ]
        ),
      }
    ];

    await vote.propose(description, executions);

    console.log("✅ Successfully created proposal to mint tokens");
  } catch (error) {
    console.error("failed to create first proposal", error);
    process.exit(1);
  }

  try {
    // This is our governance contract.
    const vote = await sdk.getContract("INSERT_VOTE_ADDRESS", "vote");
    // This is our ERC-20 contract.
    const token = await sdk.getContract("INSERT_TOKEN_ADDRESS", "token");
    // Create proposal to transfer ourselves 6,900 tokens for being awesome.
    const amount = 6_900;
    const description = "Should the DAO transfer " + amount + " tokens from the treasury to " +
      process.env.WALLET_ADDRESS + " for being awesome?";
    const executions = [
      {
        // Again, we're sending ourselves 0 ETH. Just sending our own token.
        nativeTokenValue: 0,
        transactionData: token.encoder.encode(
          // We're doing a transfer from the treasury to our wallet.
          "transfer",
          [
            process.env.WALLET_ADDRESS,
            ethers.utils.parseUnits(amount.toString(), 18),
          ]
        ),
        toAddress: token.getAddress(),
      },
    ];

    await vote.propose(description, executions);

    console.log(
      "✅ Successfully created proposal to reward ourselves from the treasury, let's hope people vote for it!"
    );
  } catch (error) {
    console.error("failed to create second proposal", error);
  }
})();
```

- mint: Nous créons une proposition qui permet au Trésor de frapper 420 000 nouveaux jetons
- transfer: Nous créons une proposition qui transfère 6 900 jetons vers notre portefeuille depuis le Trésor
  
nativeTokenValue : récompenser les gens avec des jetons ETH et des jetons de gouvernance. Ce 0,1 ETH devrait être dans notre trésorerie si nous voulions l'envoyer
  
```
node scripts/10-create-vote-proposals.js
```
  
Les nouvelles propositions sont intégrées.
  
Si `proposal_token_threshold > 0`, le code peut générer une erreur. Dans ce cas, vous devez déléguer vos jetons au contrat de vote pour qu'il fonctionne avant de déployer les propositions.

## Laissez les utilisateurs voter sur les propositions du tableau de bord
  
nous voulons que nos utilisateurs puissent facilement les voir et voter
  
Dans App.jsx:
- ajouter :
  
```
const { contract: vote } = useContract("INSERT_VOTE_ADDRESS", "vote");
```
 
- sous `shortenAddress`, ajouter :
  
```
const [proposals, setProposals] = useState([]);
const [isVoting, setIsVoting] = useState(false);
const [hasVoted, setHasVoted] = useState(false);

// Retrieve all our existing proposals from the contract.
useEffect(() => {
  if (!hasClaimedNFT) {
    return;
  }

  // A simple call to vote.getAll() to grab the proposals.
  const getAllProposals = async () => {
    try {
      const proposals = await vote.getAll();
      setProposals(proposals);
      console.log("🌈 Proposals:", proposals);
    } catch (error) {
      console.log("failed to get proposals", error);
    }
  };
  getAllProposals();
}, [hasClaimedNFT, vote]);

// We also need to check if the user already voted.
useEffect(() => {
  if (!hasClaimedNFT) {
    return;
  }

  // If we haven't finished retrieving the proposals from the useEffect above
  // then we can't check if the user voted yet!
  if (!proposals.length) {
    return;
  }

  const checkIfUserHasVoted = async () => {
    try {
      const hasVoted = await vote.hasVoted(proposals[0].proposalId, address);
      setHasVoted(hasVoted);
      if (hasVoted) {
        console.log("🥵 User has already voted");
      } else {
        console.log("🙂 User has not voted yet");
      }
    } catch (error) {
      console.error("Failed to check if wallet has voted", error);
    }
  };
  checkIfUserHasVoted();

}, [hasClaimedNFT, proposals, address, vote]);
```
  
Dans le premier `useEffect`:
- `vote.getAll()` récupère les propositions du contrat de gouvernance
- `setProposals` permet de les restituer.

Dans le deuxième `useEffect`:
- `vote.hasVoted(proposals[0].proposalId, address)` vérifie si cette adresse a voté sur la première proposition.
- Si c'est le cas, nous le faisons `setHasVoted` pour que l'utilisateur ne puisse plus voter. Même si nous n'avions pas cela, notre contrat rejetterait la transaction si un utilisateur tentait de doubler son vote.

`thirdweb` facilite le déploiement de contrats intelligents et l'interaction avec eux depuis notre client.
  
Les propositions sont affichées dans la console.
  
Maintenant, nous voulons rendre les propositions que nous venons de récupérer ici afin que les utilisateurs puissent avoir trois options pour voter :
- Pour
- Contre
- Abstention

Ajouter :
  
```
import { AddressZero } from "@ethersproject/constants";
```

Sous le bloc `<div>` de MemberList, ajouter :
  
```
          <div>
            <h2>Active Proposals</h2>
            <form
              onSubmit={async (e) => {
                e.preventDefault();
                e.stopPropagation();

                //before we do async things, we want to disable the button to prevent double clicks
                setIsVoting(true);

                // lets get the votes from the form for the values
                const votes = proposals.map((proposal) => {
                  const voteResult = {
                    proposalId: proposal.proposalId,
                    //abstain by default
                    vote: 2,
                  };
                  proposal.votes.forEach((vote) => {
                    const elem = document.getElementById(
                      proposal.proposalId + '-' + vote.type,
                    );

                    if (elem.checked) {
                      voteResult.vote = vote.type;
                      return;
                    }
                  });
                  return voteResult;
                });

                // first we need to make sure the user delegates their token to vote
                try {
                  //we'll check if the wallet still needs to delegate their tokens before they can vote
                  const delegation = await token.getDelegationOf(address);
                  // if the delegation is the 0x0 address that means they have not delegated their governance tokens yet
                  if (delegation === AddressZero) {
                    //if they haven't delegated their tokens yet, we'll have them delegate them before voting
                    await token.delegateTo(address);
                  }
                  // then we need to vote on the proposals
                  try {
                    await Promise.all(
                      votes.map(async ({ proposalId, vote: _vote }) => {
                        // before voting we first need to check whether the proposal is open for voting
                        // we first need to get the latest state of the proposal
                        const proposal = await vote.get(proposalId);
                        // then we check if the proposal is open for voting (state === 1 means it is open)
                        if (proposal.state === 1) {
                          // if it is open for voting, we'll vote on it
                          return vote.vote(proposalId, _vote);
                        }
                        // if the proposal is not open for voting we just return nothing, letting us continue
                        return;
                      }),
                    );
                    try {
                      // if any of the propsals are ready to be executed we'll need to execute them
                      // a proposal is ready to be executed if it is in state 4
                      await Promise.all(
                        votes.map(async ({ proposalId }) => {
                          // we'll first get the latest state of the proposal again, since we may have just voted before
                          const proposal = await vote.get(proposalId);

                          //if the state is in state 4 (meaning that it is ready to be executed), we'll execute the proposal
                          if (proposal.state === 4) {
                            return vote.execute(proposalId);
                          }
                        }),
                      );
                      // if we get here that means we successfully voted, so let's set the "hasVoted" state to true
                      setHasVoted(true);
                      // and log out a success message
                      console.log('successfully voted');
                    } catch (err) {
                      console.error('failed to execute votes', err);
                    }
                  } catch (err) {
                    console.error('failed to vote', err);
                  }
                } catch (err) {
                  console.error('failed to delegate tokens');
                } finally {
                  // in *either* case we need to set the isVoting state to false to enable the button again
                  setIsVoting(false);
                }
              }}
            >
              {proposals.map((proposal) => (
                <div key={proposal.proposalId} className="card">
                  <h5>{proposal.description}</h5>
                  <div>
                    {proposal.votes.map(({ type, label }) => (
                      <div key={type}>
                        <input
                          type="radio"
                          id={proposal.proposalId + '-' + type}
                          name={proposal.proposalId}
                          value={type}
                          //default the "abstain" vote to checked
                          defaultChecked={type === 2}
                        />
                        <label htmlFor={proposal.proposalId + '-' + type}>
                          {label}
                        </label>
                      </div>
                    ))}
                  </div>
                </div>
              ))}
              <button disabled={isVoting || hasVoted} type="submit">
                {isVoting
                  ? 'Voting...'
                  : hasVoted
                  ? 'You Already Voted'
                  : 'Submit Votes'}
              </button>
              {!hasVoted && (
                <small>
                  This will trigger multiple transactions that you will need to
                  sign.
                </small>
              )}
            </form>
          </div>
```
  
Le contrat de gouvernance s'arrête de voter après 24 heures, c'est-à-dire si `votes "for" proposal > votes "against" proposal`.
  
N'importe quel membre pourrait exécuter la proposition via notre contrat de gouvernance. Les propositions ne peuvent pas être exécutées automatiquement. Mais, une fois qu'une proposition est acceptée, tout membre du DAO peut déclencher la proposition acceptée.

# Supprimez vos pouvoirs d'administrateur et gérez les erreurs de base
  
## Révoquer les rôles

  
  
  
  
  
  

  

  
  
  
  


  
  
  
  
  
  
