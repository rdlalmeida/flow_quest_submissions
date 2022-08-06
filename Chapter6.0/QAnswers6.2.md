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

* Q5 - Script to read the totalSupply from the GoatedGoats:

```cadence
// readGoatedGoatsTotalSupply.cdc

import GoatedGoats from 0x2068315349bdfce5

pub fun main(): UInt64 {
    return GoatedGoats.totalSupply
}
```

* Command to execute this script in mainnet:
<code>flow scripts execute ./flow/cadence/00/scripts/readGoatedGoatsTotalSupply.cdc --network mainnet</code>

* Execution result:
![image](https://user-images.githubusercontent.com/39467168/183252239-2f46aab1-28d1-4a31-a0be-f3f8e1cce9ff.png)

* Q6 - Script to retrieve the metadata from a TopShot NFT (because I have a bunch of those already)

```cadence
/*
    Since I don't own any Goated Goats but I do have a ton of TopShot moments, I'm going to stuck my nose in this contracts instead. The important
    elements to retrieve here is the address of the main contract, or in the case of Flow, the account address where the main contracts for
    TopShot are deployed. A simple Google search, alongside with a confirmation using https://flow-view-source.com revealed that the main
    TopShot contracts are deployed in account 0xc1e4f4f4c4257510.
    The other important element to get is the address of my TopShot account, where all my TopShot NFTs are stored. To get this I've logged in
    the Dapper Wallet that I've been using for this purpose (https://accounts.meetdapper.com), selected 'Inventory' in the left side navigation
    pane, clicked on the '+' button on the top right side of the page (used to receive NFTs into this account) which revealed to be
    0x37f3f5b3e0eaf6ca
*/

// Note this import is local for code correction purposes. The flow.json that regulates this session has this contract properly set to its
// remote location in Flow's mainnet
import TopShot from "../contracts/TopShot.cdc"

pub fun main(collectionAddress: Address): TopShot.TopShotMomentMetadataView {
    // Begin by attempting to retrieve a capability to the Collection resource of the account provided as argument. According to the TopShot
    // contract, a capability can be retrieved from /public/MomentCollection
    let optionalReference = getAccount(collectionAddress).getCapability<&{TopShot.MomentCollectionPublic}>(/public/MomentCollection).borrow()

    // As always, test if a nil was returned and panic if it is the case
    if (optionalReference == nil) {
        panic(
            "Unable to retrieve a Collection capabilty from /public/MomentCollection for account "
            .concat(collectionAddress.toString())
        )
    }

    // The rest of this logic operates on the assumption that a valid collection reference was retrieved. This one is still in an optional
    // format. Force cast it to the proper format first
    let collectionReference: &{TopShot.MomentCollectionPublic} = (optionalReference)!

    // Cool. I have a proper Collection reference. Lets start by checking how many moments are stored in it
    let collectionIDs: [UInt64] = collectionReference.getIDs()

    let sizeMessage: String = 
        "Account's "
        .concat(collectionAddress.toString())
        .concat(" contains a collection with ")
        .concat(collectionIDs.length.toString())
        .concat(" NFTs in it")

    // Retrieve the metadata from the last NFT added to that collection (using their IDs as organizing element). Is it going to be the last
    // moment that I've purchased? Let's find out...
    let lastNftID: UInt64 = collectionIDs[collectionIDs.length - 1]

    let idMessage: String =
        "Account's "
        .concat(collectionAddress.toString())
        .concat(" last acquired NFT ID = ")
        .concat(lastNftID.toString())

    // Use the 'borrowMoment' function to get a reference to the TopShot.NFT resource with that id
    let lastMomentOptionalReference: &TopShot.NFT? = collectionReference.borrowMoment(id: lastNftID)

    // Same as before. Test if a proper NFT reference was returned. Panic if not
    if (lastMomentOptionalReference == nil) {
        panic(
            "Unable to retrieve metadata for NFT with ID"
            .concat(lastNftID.toString())
            .concat(" from account's ")
            .concat(collectionAddress.toString())
            .concat(" collection!")
        )
    }

    // All is good then. Force cast the reference to remove the optional and proceed to print out the NFT's metadata
    let lastMomentReference: &TopShot.NFT = lastMomentOptionalReference!

    // The TopShot.NFT resource has a very useful function called 'description', which produces a formatted string with the data
    // from the NFT in cause. Let's see if this works
    let NFTdescription: String = lastMomentReference.description()

    // All good. Return all the Strings created so far concatenated with each other
    log(sizeMessage)
    log(idMessage)
    log(NFTdescription)
    
    // Get the TopShotMetadataView resource for this NFT, which has a much more detailed information about it
    let nftViews: [Type] = lastMomentReference.getViews()

    log(
        "Retrieved the following views from NFT with ID"
        .concat(lastNftID.toString())
    )

    // Resolve the TopShotMomentMetadataView, which should be the second element of the last array
    let optionalResolvedView = lastMomentReference.resolveView(nftViews[nftViews.length - 1])
    
    if (optionalResolvedView == nil) {
        panic(
            "Unable to resolve a proper View for an NFT with id "
            .concat(lastNftID.toString())
            .concat(" from account ")
            .concat(collectionAddress.toString())
        )
    }

    // The View appears to be valid. It should be a TopShot.TopShotMomentMetadataView. Try to force cast the strut received to this type
    let resolvedView = optionalResolvedView!

    // At this point, the resolvedView is still a AnyStrut. Cast it to the desired Type before returning it
    let topShotView: TopShot.TopShotMomentMetadataView = resolvedView as! TopShot.TopShotMomentMetadataView

    // Return the NFT's metadata as a structure. This gets automatically printed 
    return topShotView
}
```

* Command to execute this script on the mainnet
<code>flow scripts execute ./flow/cadence/00/scripts/readMyTopShotMoments.cdc 0x37f3f5b3e0eaf6ca --network mainnet</code>

* Output returned from this script:
  