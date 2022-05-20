    pub contract CryptoPoops {
      pub var totalSupply: UInt64

      // This is an NFT resource that contains a name,
      // favouriteFood, and luckyNumber
      pub resource NFT {
        pub let id: UInt64

        pub let name: String
        pub let favouriteFood: String
        pub let luckyNumber: Int

        // Resource constructor. Requires 3 input arguments that are to be set as the Resource's default parameters.
        init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
          self.id = self.uuid

          self.name = _name
          self.favouriteFood = _favouriteFood
          self.luckyNumber = _luckyNumber
        }
      }

      // Resource Interface to establish the framework for Collections
      pub resource interface CollectionPublic {
        // Collection Resource needs to implement these three functions
        pub fun deposit(token: @NFT)
        pub fun getIDs(): [UInt64]
        pub fun borrowNFT(id: UInt64): &NFT
      }

        // Implement the Collection Resource according to the specifications in the CollectionPublic interface
      pub resource Collection: CollectionPublic {
            // The dictionary that holds the NFTs is not exposed by the Interface above
        pub var ownedNFTs: @{UInt64: NFT}

            // The deposit function is exposed by the Interface, therefore it needs to be implemented in the Resource
        pub fun deposit(token: @NFT) {
                // Save the input token using its id field as key to the dictionary
                self.ownedNFTs[token.id] <-! token
        }

            // The withdraw function is also hidden (not exposed by the Collection Interface)
        pub fun withdraw(withdrawID: UInt64): @NFT {
                // Use the proper 'remove' function in this case
          let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
          return <- nft
        }

            // The getIDs function, also exposed in the Interface
        pub fun getIDs(): [UInt64] {
                // Return an array with all the stored keys so far, which are the IDs of the stored NFTs
          return self.ownedNFTs.keys
        }

            // The last of the Interface exposed function
        pub fun borrowNFT(id: UInt64): &NFT {
                // Returns a Reference to the NFT identified by the input ID, with the required cast before returning it
          return &self.ownedNFTs[id] as &NFT
        }

            // Collection constructor
        init() {
                // Set an empty dictionary to store Resources
          self.ownedNFTs <- {}
        }

            // And the required destructor for nested Resources, which is this case
        destroy() {
          destroy self.ownedNFTs
        }
      }

        // Function to create an empty Collection according to the Resource specification above
      pub fun createEmptyCollection(): @Collection {
            // Create and return a new Collection Resource that, due to its constructor, has an empty dictionary as 'ownedNFTs'
        return <- create Collection()
      }

        // Define a NFT Minter also as a Resource
      pub resource Minter {

            // The NFT Miner is the only one capable of minting brand new NFTs, therefore, only its owner is able to do so also
            pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
                // The only place in this contract where NFTs are created from scratch, using the input arguments to populate its metadata
                return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
            }

            // And the function that creates the Minter Resource
            pub fun createMinter(): @Minter {
                return <- create Minter()
            }

      }

        // Contract initializer/constructor
      init() {
            // Set the total number of NFTs to 0 upon contract initialization: it starts empty
        self.totalSupply = 0

            // Create and save a Minter resource under the '/storage/' path, thus only accessible by the contract owner
        self.account.save(<- create Minter(), to: /storage/Minter)
      }
    }
