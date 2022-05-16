Q1.

The <code>.link()</code> function effectively returns a Reference to a Resouce previously saved in <code>/storage/</code>. Conceptually this consists in creating a pointer, or capability, from a Resource in <code>/storage/</code> to the <code>/public/</code> or <code>/private/</code> areas.

Q2.

Resource Interfaces can be used to establish an access framework to the Resource in question. Since the exposure of Resources in the <code>/public/</code> path is done via the <code>.link()</code> function and this one accepts a Resource Interface as well as a simple Resource as target, Interfaces can/should be used to control the access to the Resource fields. 

Q3.

Contract code:


    pub contract CandyStore {
        pub resource interface ICandy {
            // Name of the candy
            pub var name: String

            // Is it hard or soft candy?
            pub var isSoft: Bool
        }

        // Create the Resource from the Interface implementation
        pub resource Candy: ICandy {
            pub var name: String
            pub var isSoft: Bool
            // And a non-exposed field here just to make sure this works
            pub var weight: Int

            // A couple of non-exposed functions to change the softness of the candy
            pub fun Soften() {
                self.isSoft = true
            }

            pub fun Harden() {
                self.isSoft = false
            }

            init(name: String, isSoft: Bool, weight: Int) {
                self.name = name
                self.isSoft = isSoft
                self.weight = weight
            }

        }

        pub fun createCandy(_name: String, _isSoft: Bool, _weight: Int): @Candy {
            return <- create Candy(name: _name, isSoft: _isSoft, weight: _weight)
        }
    }    
 
i.
Transaction:

    import CandyStore from 0x02

    transaction(name: String, isSoft: Bool, weight: Int) {
      prepare(signer: AuthAccount) {
        // Create the Resource from the exposed Interface and save it to /storage/
        signer.save(<- CandyStore.createCandy(_name: name, _isSoft: isSoft, _weight: weight), to: /storage/myCandyStore)

        // And now to link it to the public area
        signer.link<&CandyStore.Candy{CandyStore.ICandy}>(/public/myCandyStore, target: /storage/myCandyStore)
      }

      execute {
      }
    }

ii.

![image](https://user-images.githubusercontent.com/39467168/168671530-15ab1ecc-e048-4643-b1ce-6fba0423a8c0.png)


iii.

![image](https://user-images.githubusercontent.com/39467168/168671712-7a3f0fde-b1b2-4c9b-adea-322ebcfcbcc8.png)

