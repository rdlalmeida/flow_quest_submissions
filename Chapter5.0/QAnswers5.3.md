Q1.

The <code>as!</code> operator downcasts types from more generic ones, that can be enforced through an Interface for instance, to more specific types, if compatible. If not, the <code>!</code> operator ensures that no additional computations take place, since there's a type mismatch. This can, and it is, used for contract security purposes: a standard can impose a given Interface to be implemented (as a contract or resource interface), which establishes basic functionalities to the element in question, but typically in a broader fashion, thus acting as a basic gateway to interact with a given contract that imposes an Interface (imposing a standard, Flow NFT Resource Interface in a contract ensures that any Resource that gets represented in the contract's context possesses the minimum elements and functionalities), but this can be limiting. To extend the representation (adding new functions and fields to a Resource for example) we can define another Interface bounded to the contract's context, exposing the fields and functions we need, and then downcast the Resource received to a more specific, localized interface. If the downcast is successful, the Resource should have the additional functionalities required.

Q2.

There is a similar process as the one described in Q1 for dealing with References, since these also have an associated type and can be required to conform to Interface specifications. But in the case of References, before attempting to downcast these, essentially by the same reasons as before, we need to get an authorized Reference first. To do that, we use the <code>auth</code> keyword when retrieving the initial, broader (in principle) Reference and only after do we attempt to downcast it (still as a Reference) to a more specific Reference type using the <code>as!</code> operator as before. <code>auth</code> is only used when dealing with References that need to be downcast later on.


Q3.

* BorrowAuthNFT function added inside the CryptoPoops contract (as part of the CryptoPoops.Collection Resource implementation)
NOTE: Omitted the rest of the unaltered contract code for simplicity sake

```javascript
// Auth version of the borrow function that returns a downcast reference to the
// local, more specific 'CyptoPoops.NFT' Resource instead of the more generic 'NonFungibleToken.NFT' one
// NOTE: Apparently I can set the return type for this function to '&CryptoPoops.NFT', as well as to just '&NFT' 
// but in the implementation, regarding the reference return itself, it needs to be '&NFT'. Functionally I can't
// detect any differences but this syntax inconsistency may lead to confusion
// The algorithm works in both cases, so yeah...
pub fun borrowAuthNFT(id: UInt64): &CryptoPoops.NFT {
    let reference = &self.ownedNFTs[id] as auth &NonFungibleToken.NFT

    return reference as! &NFT
}
```

* Transaction code to create and link an empty CryptoPoops.Collection (to expose the borrowAuthNFT function from above, which is only bounded
to the CryptoPoops contract).

```javascript
// create and link EmptyCollection
import CryptoPoops from 0x01
import NonFungibleToken from 0x02

transaction() {
  prepare(signer: AuthAccount) {
    // Start by creating an empty collection from the general NonFungibleToken.Collection type and downcast it
    // to CryptoPoops.Collection before saving it, to expose the borrowAuthNFT function
    let generalCollection: @CryptoPoops.Collection <- CryptoPoops.createEmptyCollection() as! @CryptoPoops.Collection

    // Save the empty collection into the transaction signer's storage
    // NOTE: I could've done this in a single instruction but its more clear this way
    signer.save(<- generalCollection, to: /storage/MyCollection)

    // Link it to expose the deposit function to allow NFT drops from other user's accounts
    signer.link<&CryptoPoops.Collection>(/public/MyCollection, target: /storage/MyCollection)
  }

  execute {
    log("Created and linked an empty CryptoPoops.Collection successfully!")
  }
}
```

* Transaction to mint and store NFTs into an input address, which has to be the signer of the previous transaction, since it is the only account with an empty CyptoPoops.Collection linked to the user's public storage path (I've used 0x03 for this purpose). This transaction has to be signed by the user that owns the NFT Minter, i.e., the account where the CryptoPoops contract is deployed

```javascript
// Mint and Deposit an NFT into the Collection previously created
import CryptoPoops from 0x01
import NonFungibleToken from 0x02

transaction(depositAccount: Address, name: String, favouriteFood: String, luckyNumber: Int) {
    prepare(signer: AuthAccount) {
        // Borrow the NFT Minter from account 0x01 (where the CryptoPoops contract was deployed. Its initiation
        // created a Minter resource in /storage/Minter)
        let minterRef = signer.borrow<&CryptoPoops.Minter>(from: /storage/Minter) ?? panic("There's no Minter resource in storage!")

        // Create the NFT to deposit into account 0x03 (where I've create a Collection with the previous transaction)
        let nftToDeposit <- minterRef.createNFT(name: name, favouriteFood: favouriteFood, luckyNumber: luckyNumber)

        // Get the Collection capability for depositAccount address (It was publicly linked in the creation transaction)
        let collectionCap: Capability<&CryptoPoops.Collection> = getAccount(depositAccount).getCapability<&CryptoPoops.Collection>(/public/MyCollection)

        // Get a reference to the deposit Collection from the Capability
        let collectionRef = collectionCap.borrow() ?? panic("Unable to retrieve a Collection Reference from the Capability!")

        // Use the Collection reference to access the deposit function and drop the NFT into the other user's collection
        collectionRef.deposit(token: <- nftToDeposit)
    }

    execute{
        log("Sent a new CryptoPoops NFT into "
        .concat(depositAccount.toString())
        .concat("'s Collection!")
        )
    }
}
```

* Created a pair of test NFTs into account 0x03:

![image](https://user-images.githubusercontent.com/39467168/170889690-cea6a342-b22d-4198-bf16-b192943f0371.png)

* Script code to borrow an auth reference to a CryptoPoops.NFT using the borrowAuthNFT function:

```javascript
// Script to retrieve a downcast NFT from account 0x03 and print out the internal parameters
// exposed only in the specific CryptoPoops.NFT type

import CryptoPoops from 0x01
import NonFungibleToken from 0x02

pub fun main(depositAddress: Address) {
  // Get the Collection reference from account 0x03 using a Capability. This should be a '&CryptoPoops.Collection' type instead of
  // '&NonFungibleToken.Collection' because the resource was already downcast after creation
  let collectionReference: &CryptoPoops.Collection = getAccount(depositAddress).getCapability<&CryptoPoops.Collection>(/public/MyCollection).borrow() 
          ?? panic("Unable to borrow a &CryptoPoops.Collection from address ".concat(depositAddress.toString()))
  
  // Use the collection reference to retrive the array of NFT ids, check if it is not empty and retrieve the id of the last element to borrow its auth reference
  let nftIds = collectionReference.getIDs()

  // Panic if the collection retrieved is still empty
  if (nftIds.length == 0) {
    panic(
      "Got a valid but empty Collection from address "
      .concat(depositAddress.toString())
      .concat(". Cannot proceed!")
    )
  }
  else {
    // In this case, print out how many NFTs exist in the collection retrieved
    log(
      "Got a reference to a Collection with "
      .concat(nftIds.length.toString())
      .concat(" NFT in it")
      )
  }
  // Use the Collection reference to access the borrowAuthNFT function to retrieve a downcast reference to the last NFT in the Collection
  let poopNFTRef = collectionReference.borrowAuthNFT(id: nftIds[nftIds.length - 1])

  // All good. Use the NFT reference to log the internal fields exposed only in the specific NFT type
  log("Got an NFT named '"
  .concat(poopNFTRef.name)
  .concat("', whose favorite food is ")
  .concat(poopNFTRef.favouriteFood)
  .concat(" and feels lucky around number ")
  .concat(poopNFTRef.luckyNumber.toString())
  )

}
```

* Running this script returns:

![image](https://user-images.githubusercontent.com/39467168/170889653-e46c32da-6136-4e69-83b9-45a34031bd38.png)

As intended. My job here is done!


