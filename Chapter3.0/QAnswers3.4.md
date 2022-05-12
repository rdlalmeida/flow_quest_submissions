Q1.

Interfaces can be used to establish a framework for Resource creation, defining which fields and functions can be defined in a future implementation, which "locks" any Resource defined with the Interface to specific fields and functions. This is also used to establish access control to the fields implementable via Interface, i.e., to control what can be seen by whom.

Q2. 

    pub contract interfaceExample {

        // Interface definition with 3 fields
        pub resource interface IFruit {
            pub let name: String
            pub let seeds: Int
        }

        // Resource implementation using the Interface
        pub resource Apple: IFruit {
            pub let name: String
            pub let seeds: Int
            // Extra field outside of the Interface definition
            pub let color: String

            init() {
                self.name = "Apple"
                self.seeds = 4
                self.color = "Red"
            }
        }

        // Resource created without an Interface
        pub fun getNoInterFruit() {
            let apple: @Apple <- create Apple()

            // Attempt to access field outside of the Interface
            log(apple.color)            // "Red"

            destroy apple
        }

        // Resource created with an Interface
        pub fun getInterFruit() {
            let apple: @Apple{IFruit} <- create Apple()

            // Attempt to access field outside of the Interface
            log(apple.color)            // "member of restricted type is not accessible:color"

            destroy apple
        }
    }
    
Q3.

The function definition needs to be added to the Interface setup.
Add the line <code>pub fun changeGreeting(newGreeting: String): String</code> inside in the struct interface block.
