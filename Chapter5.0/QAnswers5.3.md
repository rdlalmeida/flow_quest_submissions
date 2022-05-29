Q1.

The <code>as!</code> operator downcasts types from more generic ones, that can be enforced through an Interface for instance, to more specific types, if compatible. If not, the <code>!</code> operator ensures that no additional computations take place, since there's a type mismatch. This can, and it is, used for contract security purposes: a standard can impose a given Interface to be implemented (as a contract or resource interface), which establishes basic functionalities to the element in question, but typically in a broader fashion, thus acting as a basic gateway to interact with a given contract that imposes an Interface (imposing a standard, Flow NFT Resource Interface in a contract ensures that any Resource that gets represented in the contract's context possesses the minimum elements and functionalities), but this can be limiting. To extend the representation (adding new functions and fields to a Resource for example) we can define another Interface bounded to the contract's context, exposing the fields and functions we need, and then downcast the Resource received to a more specific, localized interface. If the downcast is successful, the Resource should have the additional functionalities required.

Q2.

There is a similar process as the one described in Q1 for dealing with References, since these also have an associated type and can be required to conform to Interface specifications. But in the case of References, before attempting to downcast these, essentially by the same reasons as before, we need to get an authorized Reference first. To do that, we use the <code>auth</code> keyword when retrieving the initial, broader (in principle) Reference and only after do we attempt to downcast it (still as a Reference) to a more specific Reference type using the <code>as!</code> operator as before. <code>auth</code> is only used when dealing with References that need to be downcast later on.


Q3.

