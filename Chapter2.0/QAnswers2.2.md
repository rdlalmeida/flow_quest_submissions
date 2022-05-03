Q1. changeGreeting is a function that changes the data stored in a contract variable. Once the contract is deployed, that means that this function changes data in the blockchain that is housing said contract. Scripts should only be used to consult data. To change blockchain data, a transaction is a more apt option (for curiosity sake, I did try to call that function in a script to see what would happen, and the compiler, apparentely, is OK with it. I'm guessing this convention is not imposed at the compiler level but it is still a nice rule to follow that can save a lot of time later on).

Q2. The AuthAccount is an abstraction, an alias to the account that signs the transaction to execute (hence why this happens only in the transaction context... I think). In playground mode, all training accounts have easy to remember addresses (0x01, 0x02,...) but in the real world these are long strings of 256 bits, often represented in base64 encoding, of seemingly random characters, which are a nightmare to deal with and almost impossible to memorize. Even if one uses the same account for years, it is a tall order to require the memorization - and constant recall - of such large but important value, hence why the "AuthAccount" alias is such an handy thing to have.

Q3. The prepare phase is (should) be used for instructions that read data from the blockchain. The section is used to gather all the information required for the transaction in order to let it proceed only of the necessary requirements are met (I'm assuming a similar effect to the "require" statement in Solidity). The execute phase should contain the instructions that do change data in the blockchain, i.e., the "consequential" part of the transaction. Because of blockchain's data immutability, data changes in it should take as much heed as possible, since its quite complicated to revert any errors once the transaction has executed.

Q4. Variable "myNumber" and function "updateMyNumber" added to the contract:

![image](https://user-images.githubusercontent.com/39467168/166564757-b44ac112-2dcd-4a30-97e5-75f21b3590be.png)

Script to read "myNumber"

![image](https://user-images.githubusercontent.com/39467168/166565188-a478ed47-d848-4f33-8968-e43040353304.png)

Transaction to update "myNumber":

![image](https://user-images.githubusercontent.com/39467168/166565485-a41351f8-4d53-46fa-a108-056e881fa974.png)

Script output:

![image](https://user-images.githubusercontent.com/39467168/166565654-5ee88c84-3f55-4a96-8b4c-8bc226f9bb52.png)
