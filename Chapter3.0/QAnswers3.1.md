Q1. 

R1 - Resource creation is limited in scope, Structs are not.

R2 - Structs can be copied as many times as desired while a Resource is a unique data element that can only exist at one place at a time.

R3 - Structs operation set is more flexible than the comparable Resource operation set, even though both elements are structurally similar. Resources require language to be much more explicit when operated upon.

Q2. The handling of NFTs is the ideal example where a resource should be used instead of a struct. Using a Resource to internally represent a NFT guarantees that the scarcity and uniqueness of it are going to be respected and its inherent limitations and security requirements are imposed by default through the Resource's operational boundaries.

Q3. The keyword to use when defining a resource, as opposed to a struct, is just the lowercase string for it: <code>resource</code>. To construct the actual resource from the definition in a contract, the keyword <code>create</code> needs to be used instead.

Q4. No. As stated above, resources can only be created using the <code>create</code> keyword and this keyword's scope of utilization is limited to contracts, as per compiler imposition (I'm assuming at this point), therefore resources are limited to be created only inside contracts.

Q5. The resource represented is from the <code>Jacob</code> type.

Q6. Here's the corrected code with comments explaining what was wrong with the original one:

    pub contract Test {
        pub resource Jacob {
            pub let rocks: Bool
            init() {
                self.rocks = true
            }
        }

        pub fun createJacob(): @Jacob {     // Resources require the '@' operator when referenced within a function to denote that the target object is indeed a Resource
            let myJacob <- create Jacob()   // Was missing the '<-' operator required for Resource creation, as well as the 'create' keyword while invoking the Resource's constructor
            return <- myJacob               // Missed the '<-' operator required to return the Resource created with the previous instruction. 
        }
    }
