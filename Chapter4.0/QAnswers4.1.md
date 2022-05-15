Q1. 

In Flow, an account is used as a storage element to hold smart contract data (source code) and also account related data, which can be saved in different areas (directories) separated by access permissions that allow a user to split his data into public or private regarding external access.

Q2.

Account storage is split into three different areas/paths according to its access permissions:
* <code>/storage/</code> can only be accessed by the owner/controller of the underlying account
* <code>/public/</code> is available for everyone
* <code>/private/</code> is a mix between the previous two. By default it can only be accessed by the account's owner but he can grant access permissions to external users (not possible in the <code>/storage/</code> path)

Q3.

* <code>.save()</code> function receives two arguments and stores the data indicated in its first argument into the path provided with the second argument
* <code>.load()</code> function works inversely from the previous one. This one takes a single argument, the location of the resource to retrieve, and return an optional (because there are no guarantees that the required resource exists in the provided path) element from the types indicated in the command (inside the '<>' part), or nil if none exist.
* <code>.borrow()</code> is akin to the .load() function but this one is "safer" in the sense that it returns a Reference to the Resource instead of the Resource itself, i.e., the Resource never leaves the original storage path and only its readable data is returned instead.

Q4. 

Our account storage belongs to the Flow blockchain, therefore any save operations change its state. As such we need to necessarily use a transaction since scripts can only read data from the blockchain.

Q5.

Besides the obvious security problems of allowing an external user to modify the account contents of another, the only way to save something into my account is to create a transaction and, somehow, be able to sign it with the account that (presumably) only I have access to, so creating the transaction itself is a problem unless somehow you have hacked it and can control it. 
From the external user point of view, unless Cadence allows for a different signer other than <code>AuthAccount</code>, there is no mechanism to store data into someone else's account unless you have control of that person's account.

Q6.

Contract:

    pub contract Aviary {
        pub resource Bird {
            pub var name: String
            pub var flying: Bool

            init() {
                self.name = "Woodpecker"
                self.flying = true
            }
        }

        pub fun createBird(): @Bird {
            return <- create Bird()
        }
    }


i.

    import Aviary from 0x02

    transaction() {
        prepare(signer: AuthAccount) {

            // Create the Resource first
            let someBird <- Aviary.createBird()

            // And save it to storage
            signer.save(<- someBird, to: /storage/myBirdCage)
            log("Saved another bird into the cage!")

            // Now to retrieve the stored Resource, carefully protecting from potential nils with a panic
            let storedBird <- signer.load<@Aviary.Bird>(from: /storage/myBirdCage) ?? panic("Couldn't find any birds in the storage cage!")

            // Log the bird that was retrieved
            log("Got a ".concat(storedBird.name).concat(storedBird.flying ? " flying around the place" : " quiet in a perch"))

            // Done with the bird. Kill it with a slingshot
            destroy storedBird
        }

        execute {
        }
    }

ii.


    import Aviary from 0x02

    transaction() {
        prepare(signer: AuthAccount) {

            // Create the Resource first
            let someBird <- Aviary.createBird()

            // And save it to storage
            signer.save(<- someBird, to: /storage/myBirdCage)
            log("Saved another bird into the cage!")

            // Retrieve the stored bird data as a Reference instead
            let birdRef = signer.borrow<&Aviary.Bird>(from: /storage/myBirdCage) ?? panic("Couldn't find any birds to reference here...")

            // Log the retrieved data
            log("There's a ".concat(birdRef.name).concat(birdRef.flying ? " flying over my head!" : " but it seems to be dead..."))
        }

        execute {
        }
    }