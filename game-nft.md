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




