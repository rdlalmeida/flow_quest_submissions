* Q1 - Transaction used to create and link a new Collection resource into the signer's account:
```cadence
// createAndLinkEmptyCollection.cdc

import CryptoPoops from "../contracts/CryptoPoops.cdc"
import NonFungibleToken from "../contracts/NonFungibleToken.cdc"

transaction() {
    prepare(signer: AuthAccount) {
        // Start by checking if a Collection already exists in storage. Try to borrow a reference to it and test if it is a nil or not
        var collectionReference = signer.borrow<&CryptoPoops.Collection>(from: CryptoPoops.CollectionStoragePath)

        // Test if the reference is nil and move from this result
        if (collectionReference == nil) {
            // Nothing was found in storage. Create an empty collection and save it to storage.
            let newCollection: @CryptoPoops.Collection <- CryptoPoops.createEmptyCollection() as! @CryptoPoops.Collection

            // Save the new Collection to storage
            signer.save(<- newCollection, to: CryptoPoops.CollectionStoragePath)

            // Inform the user
            log(
                "No collection was found in storage yet for account "
                .concat(signer.address.toString())
                .concat(". Creating and saving one...")
            )
        }
        else {
            // If the code gets here, the assumption is that there is something in storage and that something is the collection I'm looking for. 
            // But right now, that variable is still optional. Force-cast it to the desired type
            collectionReference = collectionReference as! &CryptoPoops.Collection

            // If the code gets here, the last instruction was successful, which implies that the type of the resource is the one I'm looking for
            // Log this to the user and move on
            log(
                "Found a CryptoPoops.Collection resource in account "
                .concat(signer.address.toString())
                .concat(" storage. Nothing else to do")
            )
        }

        // Next one: recreate the public link to the the resource, which at this point I have ensured that it is in storage already
        // Begin by removing any existing links (if there is no link yet, this instruction does nothing. Its safe to use this way I guess)
        signer.unlink(CryptoPoops.CollectionPublicPath)

        // Re-create the link to the public storage
        signer.link<&CryptoPoops.Collection>(CryptoPoops.CollectionPublicPath, target: CryptoPoops.CollectionStoragePath)

        log(
            "Created a public link to"
            .concat(CryptoPoops.CollectionPublicPath.toString())
            .concat(" for account ")
            .concat(signer.address.toString())
        )
    }

    execute {

    }
}
```

* Transaction to mint and store a new NFT into an existing collection (assumed created with the last transaction)
```cadence
// mintAndDepositNFT.cdc

import CryptoPoops from "../contracts/CryptoPoops.cdc"
import NonFungibleToken from "../contracts/NonFungibleToken.cdc"

transaction(depositAddress: Address, name: String, favouriteFood: String, luckyNumber: Int) {
    prepare(signer: AuthAccount) {
        // Borrow the NFT minter from the signer's account (the one that deployed the CryptoPoops contract, which is initialized
        // by creating a resource of this type and saving it into storage)
        let minterRef = signer.borrow<&CryptoPoops.Minter>(from: CryptoPoops.MinterStoragePath) 
            ?? panic("There's no Minter resource in storage!")

        // Create the NFT to deposit into the deposit address (assuming that the transaction that creates the Collection was already executed)
        let nftToDeposit <- minterRef.createNFT(name: name, favouriteFood: favouriteFood, luckyNumber: luckyNumber)

        // Get the Collection capability from deposit address provided
        let collectionCap: Capability<&CryptoPoops.Collection> = getAccount(depositAddress).getCapability<&CryptoPoops.Collection>(CryptoPoops.CollectionPublicPath)

        // Get a reference to the deposit Collection from the Capability
        let collectionRef = collectionCap.borrow() ?? panic("Unable to retrieve a Collection Reference from the Capability")

        // Use the Collection reference to access the deposit function and drop the NFT into the other user's collection
        collectionRef.deposit(token: <- nftToDeposit)
    }

    execute {
        log(
            "Sent a new CryptoPoops NFT into "
            .concat(depositAddress.toString())
            .concat("'s Collection!")
        )
    }
}
```

* Command used to create a new Collection into the testnet account:
<code>flow transactions send ./flow/cadence/00/transactions/createAndLinkEmptyCollection.cdc --signer testnet-account --network testnet</code>

* Command used to create a new NFT and deposit it in the Collection just created:
* <code>flow transactions send ./flow/cadence/00/transactions/mintAndDepositNFT.cdc 0xb7fb1e0ae6485cf6 "testNFT'1" "Pork Rinds" 56 --signer testnet-account --network testnet</code>

* Q2 - Script to read the 'totalSupply' from the CryptoPoops contract:

```cadence
// readTotalSupply.cdc

import CryptoPoops from "../contracts/CryptoPoops.cdc"

pub fun main(): UInt64 {
    return CryptoPoops.totalSupply
}
```

* Command to run this script:
<code>flow scripts execute ./flow/cadence/00/scripts/readTotalSupply.cdc --network emulator</code>

* Q3 - Script to return an array of UInt64 with the NFTs IDs found in the CryptoPoops.Collection:

```cadence
// readCollectionNFTs.cdc

import CryptoPoops from "../contracts/CryptoPoops.cdc"
import NonFungibleToken from "../contracts/NonFungibleToken.cdc"

transaction(depositAddress: Address, name: String, favouriteFood: String, luckyNumber: Int) {
    prepare(signer: AuthAccount) {
        // Borrow the NFT minter from the signer's account (the one that deployed the CryptoPoops contract, which is initialized
        // by creating a resource of this type and saving it into storage)
        let minterRef = signer.borrow<&CryptoPoops.Minter>(from: CryptoPoops.MinterStoragePath) 
            ?? panic("There's no Minter resource in storage!")

        // Create the NFT to deposit into the deposit address (assuming that the transaction that creates the Collection was already executed)
        let nftToDeposit <- minterRef.createNFT(name: name, favouriteFood: favouriteFood, luckyNumber: luckyNumber)

        // Get the Collection capability from deposit address provided
        let collectionCap: Capability<&CryptoPoops.Collection> = getAccount(depositAddress).getCapability<&CryptoPoops.Collection>(CryptoPoops.CollectionPublicPath)

        // Get a reference to the deposit Collection from the Capability
        let collectionRef = collectionCap.borrow() ?? panic("Unable to retrieve a Collection Reference from the Capability")

        // Use the Collection reference to access the deposit function and drop the NFT into the other user's collection
        collectionRef.deposit(token: <- nftToDeposit)
    }

    execute {
        log(
            "Sent a new CryptoPoops NFT into "
            .concat(depositAddress.toString())
            .concat("'s Collection!")
        )
    }
}
```

* Command to execute this script
<code>flow scripts execute ./flow/cadence/00/scripts/readCollectionNFTs.cdc 0xb7fb1e0ae6485cf6 --network testnet</code>

* Q4 - Script to read the metadata from a NFT in an account's collection (since NFT ids are generated using the uuid() function, this script simply reads the metadata of the last NFT in the list)

```cadence
// retrieveNFTfromCollection.cdc

import CryptoPoops from "../contracts/CryptoPoops.cdc"
import NonFungibleToken from "../contracts/NonFungibleToken.cdc"

pub fun main(depositAddress: Address): String {
    // Get the Collection reference from the depositAddress provided using a Capability. This should be a '&CryptoPoops.Collection' type instead of
    // '&NonFungibleToken.Collection' because the resource was already downcast after creation
    let collectionReference: &CryptoPoops.Collection = getAccount(depositAddress).getCapability<&CryptoPoops.Collection>(CryptoPoops.CollectionPublicPath).borrow()
        ?? panic(
            "Unable to borrow a &CryptoPoops.Collection from address "
            .concat(depositAddress.toString())
        )
    
    // Use the collection reference to retrieve the array of NFT ids, check if it is not empty and retrieve the id of the last element to borrow its auth reference
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
        // In this case print out how many NFTs exist in the collection retrieved
        log(
            "Got a reference to a Collection with "
            .concat(nftIds.length.toString())
            .concat(" NFTs in it")
        )
    }

    // Use the Collection reference to access the borrowAuthNFT function to retrive a downcast reference to the last NFT in the Collection
    let poopNFTRef = collectionReference.borrowAuthNFT(id: nftIds[nftIds.length - 1])

    // All good. Use the NFT reference to log the internal fields exposed only in the specific NFT type
    return "Got a NFT named '".concat(poopNFTRef.name).concat("', whose favourite food is ").concat(poopNFTRef.favouriteFood).concat(" and feels lucky around number ").concat(poopNFTRef.luckyNumber.toString())
}
```

* Command to run this script
<code>flow scripts execute ./flow/cadence/00/scripts/retrieveNFTfromCollection.cdc --network testnet</code>