Avec les jeux, un NFT peut par exemple donner le droit de jouer au jeu.

les joueurs pourraient vendre leurs Mario NFT sur un marché.

L'éditeur pourrait prendre une commission.

à mesure que Mario gagne en popularité, le NFT nécessaire pour jouer aux jeux Mario prend également de la valeur

Mais si la base de joueurs augmente beaucoup rapidement, le NFT devient très cher, trop cher.

les développeurs de jeux aléatoires pourraient créer des jeux qui obligeraient les joueurs à connecter leur portefeuille et à vérifier qu'ils ont le Mario NFT

à mesure que ces jeux créés par des développeurs aléatoires deviennent plus populaires, cela signifie que la valeur des NFT originaux de Mario augmenterait, ce qui est bon pour Nintendo

chaque fois que le Mario NFT est utilisé dans un nouveau jeu, demandez au joueur de payer 10 $ de son portefeuille Ethereum : 5 $ iraient directement dans le portefeuille de Nintendo. 5 $ iraient directement dans le portefeuille du développeur.

# Obtenez env local tous mis en place

Hardhat permet de compiler rapidement des contrats intelligents et de les tester localement

```
mkdir epic-game
cd epic-game
npm init -y
npm install --save-dev hardhat@latest
npx hardhat
```

Choisir `Create a JavaScript project`

```
npm install --save-dev chai @nomiclabs/hardhat-ethers ethers @nomicfoundation/hardhat-toolbox @nomicfoundation/hardhat-chai-matchers
```

Installer `OpenZeppelin` :

```
npm install @openzeppelin/contracts
npx hardhat run scripts/deploy.js
```

- Hardhat compile votre contrat intelligent Solidity en bytecode.
- Hardhat crée une "blockchain locale".
- Hardhat déploie le contrat compilé sur votre blockchain locale. C'est l'adresse que vous voyez à la fin.

`Lock.sol` est le contrat intelligent et `deploy.js` exécute le contrat.

```
rm test/Lock.js
rm scripts/deploy.js
rm contracts/Lock.sol
```

Créer contracts/MyEpicGame.sol :

```
// SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.17;

import "hardhat/console.sol";

contract MyEpicGame {
  constructor() {
    console.log("THIS IS MY GAME CONTRACT. NICE.");
  }
}
```

Créer scripts/run.js :

```
const main = async () => {
  const gameContractFactory = await hre.ethers.getContractFactory('MyEpicGame');
  const gameContract = await gameContractFactory.deploy();
  await gameContract.deployed();
  console.log("Contract deployed to:", gameContract.address);
};

const runMain = async () => {
  try {
    await main();
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};

runMain();
```

Déployer le contrat :

```
npx hardhat run scripts/run.js
```

L'environnement d'exécution Hardhat (Hardhat Runtime Environment), ou HRE en abrégé, est un objet contenant toutes les fonctionnalités exposées par Hardhat lors de l'exécution d'une tâche, d'un test ou d'un script. En réalité, Hardhat est le HRE.

# Données de configuration pour les NFT de caractères

Le but de notre jeu sera de détruire un boss. Disons que ce boss a 1 000 000 HP. 

Quand les joueurs commencent le jeu, ils créent un personnage NFT qui a une certaine quantité de dégâts d'attaque et de HP

Les joueurs peuvent ordonner à leur personnage NFT d'attaquer le boss et de lui infliger des dégâts.

Chaque fois qu'un joueur frappe le boss, le boss frappe le joueur en retour.

Si le HP du NFT passe en dessous de 0, le NFT du joueur meurt et il ne peut plus frapper le boss. Les joueurs ne peuvent avoir qu'un seul personnage NFT dans leur portefeuille. Une fois que le NFT du personnage meurt, la partie est terminée. Cela signifie que de nombreux joueurs doivent unir leurs forces pour attaquer le boss et le tuer.

les personnages eux-mêmes sont des NFT

lorsqu'un joueur va jouer au jeu :
- Ils connecteront leur portefeuille.
- Notre jeu détectera qu'ils n'ont pas de personnage NFT dans leur portefeuille.
- Nous les laisserons choisir un personnage et créer leur propre personnage NFT pour jouer au jeu. Chaque personnage NFT a ses propres attributs stockés sur le NFT directement comme : HP, Attack Damage, l'image du personnage, etc. Ainsi, lorsque le HP du personnage atteint 0, il le dit hp: 0sur le NFT lui-même.

## Configurez les données pour vos NFT

Chaque personnage aura quelques attributs : une image, un nom, une valeur HP et une valeur de dégâts d'attaque

il n'y aura qu'un ensemble de caractères (ex. 3). Mais, un nombre illimité de NFT de chaque personnage peut être frappé.

Cela signifie donc que si cinq personnes frappent le caractère n ° 1, cela signifie que les cinq personnes auront exactement le même caractère, mais chaque personne aura un NFT unique et chaque NFT aura son propre état. Par exemple, si le NFT du joueur #245 est touché et perd des HP, seul son NFT devrait perdre des HP !

initialiser les attributs par défaut d'un personnage:

Modifier MyEpicGame.sol :

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "hardhat/console.sol";

contract MyEpicGame {
  // We'll hold our character's attributes in a struct. Feel free to add
  // whatever you'd like as an attribute! (ex. defense, crit chance, etc).
  struct CharacterAttributes {
    uint characterIndex;
    string name;
    string imageURI;        
    uint hp;
    uint maxHp;
    uint attackDamage;
  }
  // A lil array to help us hold the default data for our characters.
  // This will be helpful when we mint new characters and need to know
  // things like their HP, AD, etc.
  CharacterAttributes[] defaultCharacters;

  // Data passed in to the contract when it's first created initializing the characters.
  // We're going to actually pass these values in from run.js.
  constructor(
    string[] memory characterNames,
    string[] memory characterImageURIs,
    uint[] memory characterHp,
    uint[] memory characterAttackDmg
  )
  {
    // Loop through all the characters, and save their values in our contract so
    // we can use them later when we mint our NFTs.
    for(uint i = 0; i < characterNames.length; i += 1) {
      defaultCharacters.push(CharacterAttributes({
        characterIndex: i,
        name: characterNames[i],
        imageURI: characterImageURIs[i],
        hp: characterHp[i],
        maxHp: characterHp[i],
        attackDamage: characterAttackDmg[i]
      }));

      CharacterAttributes memory c = defaultCharacters[i];
      console.log("Done initializing %s w/ HP %s, img %s", c.name, c.hp, c.imageURI);
    }
  }
}
```

mettre à jour run.js:

```
const main = async () => {
  const gameContractFactory = await hre.ethers.getContractFactory('MyEpicGame');
  const gameContract = await gameContractFactory.deploy(
    ["Leo", "Aang", "Pikachu"],       // Names
    ["https://i.imgur.com/pKd5Sdk.png", // Images
    "https://i.imgur.com/xVu4vFL.png", 
    "https://i.imgur.com/WMB6g9u.png"],
    [100, 200, 300],                    // HP values
    [100, 50, 25]                       // Attack damage values
  );
  await gameContract.deployed();
  console.log("Contract deployed to:", gameContract.address);
};

const runMain = async () => {
  try {
    await main();
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};

runMain();
```

```
npx hardhat run scripts/run.js
```

# En fait, frappez vos NFT localement

## Mint les NFT

MyEpicGame.sol :

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

// NFT contract to inherit from.
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

// Helper functions OpenZeppelin provides.
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/utils/Strings.sol";


import "hardhat/console.sol";

// Our contract inherits from ERC721, which is the standard NFT contract!
contract MyEpicGame is ERC721 {

  struct CharacterAttributes {
    uint characterIndex;
    string name;
    string imageURI;        
    uint hp;
    uint maxHp;
    uint attackDamage;
  }

  // The tokenId is the NFTs unique identifier, it's just a number that goes
  // 0, 1, 2, 3, etc.
  using Counters for Counters.Counter;
  Counters.Counter private _tokenIds;

  CharacterAttributes[] defaultCharacters;

  // We create a mapping from the nft's tokenId => that NFTs attributes.
  mapping(uint256 => CharacterAttributes) public nftHolderAttributes;

  // A mapping from an address => the NFTs tokenId. Gives me an ez way
  // to store the owner of the NFT and reference it later.
  mapping(address => uint256) public nftHolders;

  constructor(
    string[] memory characterNames,
    string[] memory characterImageURIs,
    uint[] memory characterHp,
    uint[] memory characterAttackDmg
    // Below, you can also see I added some special identifier symbols for our NFT.
    // This is the name and symbol for our token, ex Ethereum and ETH. I just call mine
    // Heroes and HERO. Remember, an NFT is just a token!
  )
    ERC721("Heroes", "HERO")
  {
    for(uint i = 0; i < characterNames.length; i += 1) {
      defaultCharacters.push(CharacterAttributes({
        characterIndex: i,
        name: characterNames[i],
        imageURI: characterImageURIs[i],
        hp: characterHp[i],
        maxHp: characterHp[i],
        attackDamage: characterAttackDmg[i]
      }));

      CharacterAttributes memory c = defaultCharacters[i];
      
      // Hardhat's use of console.log() allows up to 4 parameters in any order of following types: uint, string, bool, address
      console.log("Done initializing %s w/ HP %s, img %s", c.name, c.hp, c.imageURI);
    }

    // I increment _tokenIds here so that my first NFT has an ID of 1.
    // More on this in the lesson!
    _tokenIds.increment();
  }

  // Users would be able to hit this function and get their NFT based on the
  // characterId they send in!
  function mintCharacterNFT(uint _characterIndex) external {
    // Get current tokenId (starts at 1 since we incremented in the constructor).
    uint256 newItemId = _tokenIds.current();

    // The magical function! Assigns the tokenId to the caller's wallet address.
    _safeMint(msg.sender, newItemId);

    // We map the tokenId => their character attributes. More on this in
    // the lesson below.
    nftHolderAttributes[newItemId] = CharacterAttributes({
      characterIndex: _characterIndex,
      name: defaultCharacters[_characterIndex].name,
      imageURI: defaultCharacters[_characterIndex].imageURI,
      hp: defaultCharacters[_characterIndex].hp,
      maxHp: defaultCharacters[_characterIndex].maxHp,
      attackDamage: defaultCharacters[_characterIndex].attackDamage
    });

    console.log("Minted NFT w/ tokenId %s and characterIndex %s", newItemId, _characterIndex);
    
    // Keep an easy way to see who owns what NFT.
    nftHolders[msg.sender] = newItemId;

    // Increment the tokenId for the next person that uses it.
    _tokenIds.increment();
  }
}
```

`nftHolderAttributes` stocke l'état des NFT du joueur. Nous mappons l'identifiant du NFT à une CharacterAttributesstructure.

`nftHolders` permet de mapper facilement l'adresse d'un utilisateur à l'ID du NFT qu'il possède.

`nftHolders[INSERT_PUBLIC_ADDRESS_HERE]` permet de savoir quel NFT possède cette adresse.

## ERC 721

`mintCharacterNFT(0)` crée le personnage avec les statistiques de defaultCharacters[0].

`newItemId` est l'identifiant du NFT

`_tokenIds` permet de garder une trace de l'identifiant unique des NFT

`_tokenIds` est une variable d' état  ce qui signifie que si nous la modifions, la valeur est stockée directement sur le contrat comme une variable globale qui reste en permanence en mémoire.

`_safeMint(msg.sender, newItemId)` permet de mint le NFT avec l'id `newItemId` à l'utilisateur avec l'adresse `msg.sender`.

`msg.sender` correspond à l'adresse publique  de la personne qui appelle le contrat.

## Maintien de données dynamiques sur un NFT

Stocker les données des personnages par joueur :

```
nftHolderAttributes[newItemId] = CharacterAttributes({
  characterIndex: _characterIndex,
  name: defaultCharacters[_characterIndex].name,
  imageURI: defaultCharacters[_characterIndex].imageURI,
  hp: defaultCharacters[_characterIndex].hp,
  maxHp:defaultCharacters[_characterIndex].maxHp,
  attackDamage: defaultCharacters[_characterIndex].attackDamage
});
```

notre NFT contient des données relatives au NFT de notre joueur. Mais ces données sont dynamiques

Exemple :

```
{
  characterIndex: 1,
  name: "Aang",
  imageURI: "https://i.imgur.com/xVu4vFL.png",
  hp: 200,
  maxHp: 200,
  attackDamage: 50
} 
```

les métadonnées NFT peuvent changer : elles sont au créateur.

Les NFT maintiennent l'état du personnage spécifique de notre joueur.

`nftHolderAttributes` mappe le `tokenId` du NFT à une structure de `CharacterAttributes` pour permettre de mettre à jour les valeurs liées au NFT du joueur.

`nftHolders[msg.sender] = newItemId;` permet de savoir facilement qui possède quels NFT.

## Exécuter localement

Dans scripts/run.js, ajouter à la fin de la fonction `main()` :

```
let txn;
// We only have three characters.
// an NFT w/ the character at index 2 of our array.
txn = await gameContract.mintCharacterNFT(2);
await txn.wait();

// Get the value of the NFT's URI.
let returnedTokenUri = await gameContract.tokenURI(1);
console.log("Token URI:", returnedTokenUri);
```

`mintCharacterNFT(2)` est appelé avec un portefeuille par défaut configuré localement.
Donc `msg.sender` est l'adresse publique de ce portefeuille local.

`tokenURIest` renvoie les données réelles attachées au NFT

```
npx hardhat run scripts/run.js
```

`nftHolderAttributes` n'est pas attaché au NFT. Nous allons maintenant l'attacher au `tokenURI`.

## Configurer l'URI du jeton

```
mkdir contracts/libraries
```

Créer `contracts/librairies/Base64.sol` avec :

```
/**
 *Submitted for verification at Etherscan.io on 2021-09-05
 */

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

/// [MIT License]
/// @title Base64
/// @notice Provides a function for encoding some bytes in base64
/// @author Brecht Devos <brecht@loopring.org>
library Base64 {
    bytes internal constant TABLE =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

    /// @notice Encodes some bytes to the base64 representation
    function encode(bytes memory data) internal pure returns (string memory) {
        uint256 len = data.length;
        if (len == 0) return "";

        // multiply by 4/3 rounded up
        uint256 encodedLen = 4 * ((len + 2) / 3);

        // Add some extra buffer at the end
        bytes memory result = new bytes(encodedLen + 32);

        bytes memory table = TABLE;

        assembly {
            let tablePtr := add(table, 1)
            let resultPtr := add(result, 32)

            for {
                let i := 0
            } lt(i, len) {

            } {
                i := add(i, 3)
                let input := and(mload(add(data, i)), 0xffffff)

                let out := mload(add(tablePtr, and(shr(18, input), 0x3F)))
                out := shl(8, out)
                out := add(
                    out,
                    and(mload(add(tablePtr, and(shr(12, input), 0x3F))), 0xFF)
                )
                out := shl(8, out)
                out := add(
                    out,
                    and(mload(add(tablePtr, and(shr(6, input), 0x3F))), 0xFF)
                )
                out := shl(8, out)
                out := add(
                    out,
                    and(mload(add(tablePtr, and(input, 0x3F))), 0xFF)
                )
                out := shl(224, out)

                mstore(resultPtr, out)

                resultPtr := add(resultPtr, 4)
            }

            switch mod(len, 3)
            case 1 {
                mstore(sub(resultPtr, 2), shl(240, 0x3d3d))
            }
            case 2 {
                mstore(sub(resultPtr, 1), shl(248, 0x3d))
            }

            mstore(result, encodedLen)
        }

        return string(result);
    }
}
```

Ce script permet d'encoder n'importe quelle donnée dans une chaîne Base64

Dans le contrat `MyEpicGame.sol` :
- importez :

```
import "./libraries/Base64.sol";
```

- ajouter :

```
function tokenURI(uint256 _tokenId) public view override returns (string memory) {
  CharacterAttributes memory charAttributes = nftHolderAttributes[_tokenId];

  string memory strHp = Strings.toString(charAttributes.hp);
  string memory strMaxHp = Strings.toString(charAttributes.maxHp);
  string memory strAttackDamage = Strings.toString(charAttributes.attackDamage);

  string memory json = Base64.encode(
    abi.encodePacked(
      '{"name": "',
      charAttributes.name,
      ' -- NFT #: ',
      Strings.toString(_tokenId),
      '", "description": "This is an NFT that lets people play in the game Metaverse Slayer!", "image": "',
      charAttributes.imageURI,
      '", "attributes": [ { "trait_type": "Health Points", "value": ',strHp,', "max_value":',strMaxHp,'}, { "trait_type": "Attack Damage", "value": ',
      strAttackDamage,'} ]}'
    )
  );

  string memory output = string(
    abi.encodePacked("data:application/json;base64,", json)
  );
  
  return output;
}
```

`CharacterAttributes memory charAttributes = nftHolderAttributes[_tokenId];` récupère les données NFT spécifiques à l'aide de `_tokenId`.

On emballe ces données dans une variable nommée `json` :

```
string memory json = Base64.encode(
  abi.encodePacked(
        '{"name": "',
        charAttributes.name,
        ' -- NFT #: ',
        Strings.toString(_tokenId),
        '", "description": "This is an NFT that lets people play in the game Metaverse Slayer!", "image": "',
        charAttributes.imageURI,
        '", "attributes": [ { "trait_type": "Health Points", "value": ',strHp,', "max_value":',strMaxHp,'}, { "trait_type": "Attack Damage", "value": ',
        strAttackDamage,'} ]}'
  )
);
```

`abi.encodePacked` combine des chaînes

Cette norme de métadonnées est suivie par les sites Web NFT populaires comme OpenSea

`abi.encodePacked("data:application/json;base64,", json)` transforme le fichier JSON en une chaîne encodée lisible par le navigateur

```
npx hardhat run scripts/run.js
```

Copier la valeur de `Token URI`, affichée dans la console, dans un navigateur. Les données JSON sont affichée dans la fenêtre du navigateur.

# Déployer à Rinkeby, voir sur OpenSea

QuickNode nous aide essentiellement à diffuser notre transaction de création de contrat afin qu'elle puisse être récupérée par les mineurs le plus rapidement possible. Une fois la transaction extraite, elle est ensuite diffusée sur la blockchain en tant que transaction légitime. À partir de là, chacun met à jour sa copie de la blockchain.

Scénario :
Diffusez notre transaction
Attendez qu'il soit récupéré par de vrais mineurs
Attendez qu'il soit miné
Attendez qu'il soit rediffusé sur la blockchain en disant à tous les autres mineurs de mettre à jour leurs copies

Obtenir des faucets :
- Chainlink : https://faucets.chain.link/goerli	0,1	Aucun
- Goerli :	https://goerlifaucet.com	0,25	24 heures
- MonCrypto : https://app.mycrypto.com/faucet	0,01	Aucun

## Configurer un fichier deploy.js

scripts/deploy.js :

```
const main = async () => {
  const gameContractFactory = await hre.ethers.getContractFactory('MyEpicGame');
  const gameContract = await gameContractFactory.deploy(                     
    ["Leo", "Aang", "Pikachu"],       
    ["https://i.imgur.com/pKd5Sdk.png", 
    "https://i.imgur.com/xVu4vFL.png", 
    "https://i.imgur.com/u7T87A6.png"],
    [100, 200, 300],                    
    [100, 50, 25]                       
  );
  await gameContract.deployed();
  console.log("Contract deployed to:", gameContract.address);

  
  let txn;
  txn = await gameContract.mintCharacterNFT(0);
  await txn.wait();
  console.log("Minted NFT #1");

  txn = await gameContract.mintCharacterNFT(1);
  await txn.wait();
  console.log("Minted NFT #2");

  txn = await gameContract.mintCharacterNFT(2);
  await txn.wait();
  console.log("Minted NFT #3");

  txn = await gameContract.mintCharacterNFT(1);
  await txn.wait();
  console.log("Minted NFT #4");

  console.log("Done deploying and minting!");

};

const runMain = async () => {
  try {
    await main();
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};

runMain();
```

## Déployer sur Goerli testnet

Ajouter à hardhat.config.js :

```
  networks: {
    goerli: {
      url: 'YOUR_QUICKNODE_API_URL',
      accounts: ['YOUR_PRIVATE_GOERLI_ACCOUNT_KEY'],
    },
  },
```

```
npx hardhat run scripts/deploy.js --network goerli
```

Sur https://goerli.etherscan.io/, vérifier qu'il y a 5 nouvelles transactions.

Les NFT que vous venez de créer seront sur le site Testnet de Pixxiti :

Aller sur https://goerli.pixxiti.com

Créez cette URL : https://goerli.pixxiti.com/nfts/INSERT_DEPLOY_CONTRACT_ADDRESS_HERE/INSERT_TOKEN_ID_HERE

attendez au moins 15 minutes pour que Pixxiti mette à jour votre NFT.

??? cliquez sur "Collections" puis sur "Heroes". on voit les nfts

Cliquez sur l'un des NFTs

Cliquez sur "Levels" à gauche

Les attributs sont affichés

si nous modifions la valeur HP du NFT de ce lecteur 150 ou quoi que ce soit, il serait en fait mis à jour sur Pixiti

les joueurs pourraient emmener leur personnage NFT vers d'autres jeux qui le prennent en charge.

les dégâts d'attaque pourraient même partagées entre les jeux.

# SHIP THE GAME LOGIC

# Construisez un boss + une logique d'attaque

## Construire notre boss

Le boss aura essentiellement un nom, une image, des dégâts d'attaque et des HP. Le patron ne sera pas un NFT. Les données du patron vivront simplement sur notre contrat intelligent.

```
struct BigBoss {
  string name;
  string imageURI;
  uint hp;
  uint maxHp;
  uint attackDamage;
}

BigBoss public bigBoss;
```

Initialiser le boss :

```
constructor(
  string[] memory characterNames,
  string[] memory characterImageURIs,
  uint[] memory characterHp,
  uint[] memory characterAttackDmg,
  string memory bossName, // These new variables would be passed in via run.js or deploy.js.
  string memory bossImageURI,
  uint bossHp,
  uint bossAttackDamage
)
  ERC721("Heroes", "HERO")
{
  // Initialize the boss. Save it to our global "bigBoss" state variable.
  bigBoss = BigBoss({
    name: bossName,
    imageURI: bossImageURI,
    hp: bossHp,
    maxHp: bossHp,
    attackDamage: bossAttackDamage
  });

  console.log("Done initializing boss %s w/ HP %s, img %s", bigBoss.name, bigBoss.hp, bigBoss.imageURI);

  // All the other character code is below here is the same as before, just not showing it to keep things short!
```

Modifier `run.js` et `deploy.js` pour passer en params pour notre boss. Dans `const gameContract = await gameContractFactory.deploy`, ajouter les arguments :

```
  "Elon Musk", // Boss name
  "https://i.imgur.com/AksR0tt.png", // Boss image
  10000, // Boss hp
  50 // Boss attack damage
```

## Récupérer les attributs NFT du joueur

Dans `MyEpicGame.sol`, créer une fonction attackBoss :

```
function attackBoss() public {
  // récupérer l'état NFT du personnage du joueur
  // Ces données sont détenues dans nftHolderAttributesce qui nécessite tokenIdle NFT
  uint256 nftTokenIdOfPlayer = nftHolders[msg.sender];
  // lorsque nous le faisons storage, puis le faisons player.hp = 0, cela changerait la valeur de santé sur le NFT lui-même en 0.
  // si nous devions utiliser à la memoryplace storage, cela créerait une copie locale de la variable dans le cadre de la fonction
  CharacterAttributes storage player = nftHolderAttributes[nftTokenIdOfPlayer];
  console.log("\nPlayer w/ character %s about to attack. Has %s HP and %s AD", player.name, player.hp, player.attackDamage);
  console.log("Boss %s has %s HP and %s AD", bigBoss.name, bigBoss.hp, bigBoss.attackDamage);
}
```

Dans `scripts/run.js`, ajouter sous `gameContract.deployed();` :

```
let txn;
txn = await gameContract.mintCharacterNFT(2);
await txn.wait();

txn = await gameContract.attackBoss();
await txn.wait();
```

ATTENTION, VOIR SI CE CODE N'EST PAS DOUBLE :

```
let txn;
txn = await gameContract.mintCharacterNFT(2);
await txn.wait();
```

```
npx hardhat run scripts/run.js
```

## Faites quelques vérifications avant d'attaquer

vérifier que le personnage a des HP

Dans la fonction `attackBoss()`, ajouter :

```
// Make sure the player has more than 0 HP.
  require (
    player.hp > 0,
    "Error: character must have HP to attack boss."
  );

  // Make sure the boss has more than 0 HP.
  require (
    bigBoss.hp > 0,
    "Error: boss must have HP to attack character."
  );
```

## Attaquez le boss

On travail avec `uint` or le HP pourrait devenir négatif. On évite d'utiliser `int` car les bibliothèques comme OpenZeppelin ou Hardhat gèrent mal `int`.

La solution est d'ajouter dans la fonction `attackBoss()` :

```
// Allow player to attack boss.
  if (bigBoss.hp < player.attackDamage) {
    bigBoss.hp = 0;
  } else {
    bigBoss.hp = bigBoss.hp - player.attackDamage;
  }
```

## Ajoutez une logique pour que le boss attaque le joueur

S'assurer que les PV du joueur ne se transforment pas en un nombre négatif également, car les PV du joueur sont aussi des `uint`.

Dans la fonction `attackBoss()`, ajouter :

```
// Allow boss to attack player.
  if (player.hp < bigBoss.attackDamage) {
    player.hp = 0;
  } else {
    player.hp = player.hp - bigBoss.attackDamage;
  }
  
  // Console for ease.
  console.log("Player attacked boss. New boss hp: %s", bigBoss.hp);
  console.log("Boss attacked player. New player hp: %s\n", player.hp);
```

Dans le fichier `scripts/run.js`, ajouter avant `tokenURI` :

```
txn = await gameContract.attackBoss();
await txn.wait();
```

```
npx hardhat run scripts/run.js
```

On va générer des nombres pseudo-aléatoires sur la chaîne pour introduire des chances qu'une attaque échoue.

l'utilisation de nombres pseudo-aléatoires ne devrait être faite que dans des situations à faible enjeu

Pour les nombres aléatoires, vous devriez utiliser https://docs.chain.link/docs/vrf/v2/introduction/?utm_source=buildspace.so&utm_medium=buildspace_project.

Dans le fichier `contracts/MyEpicGame.sol`, ajouter :
- la déclaration de variable :

```
uint randNonce = 0; // this is used to help ensure that the algorithm has different inputs every time
```

- la fonction :

```
function randomInt(uint _modulus) internal returns(uint) {
   randNonce++;                                                     // increase nonce
   return uint(keccak256(abi.encodePacked(block.timestamp,                      // an alias for 'block.timestamp'
                                          msg.sender,               // your address
                                          randNonce))) % _modulus;  // modulo using the _modulus argument
 }
```

`internal` signifie que la fonction ne peut être appelée que depuis le contrat lui-même.

Dans la fonction `randomInt()`, on envoie un flux de données dans la fonction keccak256, qui nous renvoie un hachage (pensez à quelque chose qui ressemble à l'adresse publique de votre portefeuille), puis nous ignorons tout dans ce hachage sauf le dernier chiffre 

Remplacer le bloc `if (bigBoss.hp < player.attackDamage) {` par :

```
console.log("%s swings at %s...", player.name, bigBoss.name);        
        if (bigBoss.hp < player.attackDamage) {
            bigBoss.hp = 0;
            console.log("The boss is dead!");
        } else {
            if (randomInt(10) > 5) {                                 // by passing 10 as the mod, we elect to only grab the last digit (0-9) of the hash!
                bigBoss.hp = bigBoss.hp - player.attackDamage;
                console.log("%s attacked boss. New boss hp: %s", player.name, bigBoss.hp);
            } else {
                console.log("%s missed!\n", player.name);
            }
        }
```
 
```
npx hardhat run scripts/run.js
```

# Déployez et voyez les NFT évoluer en prod

## Déployez à nouveau et voyez les valeurs NFT changer

Dans le fichier `scripts/run.js`, supprimer la section :

```
    // Get the value of the NFT's URI.
    let returnedTokenUri = await gameContract.tokenURI(1);
    console.log("Token URI:", returnedTokenUri);
```

Puis copier le contenu restant dans le fichier `scripts/deploy.js`.

```
npx hardhat run scripts/deploy.js --network goerli
```

Aller sur `https://goerli.pixxiti.com/nfts/INSERT_DEPLOY_CONTRACT_ADDRESS_HERE/INSERT_TOKEN_ID_HERE`.

Pour voir les valeurs actualisées, vous pouvez devoir cliquer sur le bouton `Refresh`.






