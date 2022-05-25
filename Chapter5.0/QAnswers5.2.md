Q1.

Contract standards, especially if formally imposed by an Interface for instance, create a new security layer over 3rd party development. By "forcing" developers that wish to publish their work in the Flow ecosystem (or any other for that matter), Flow can ensure that any smart contract deployed and active in its blockchain contains a minimum of functionality. This disables windows of opportunity for certain hacks while solving other avoidable problems, just by ensuring that certain parameters and/or functions need to be implemented for a certain contract or resource - that must also conform to a specific interface - are present. Also, users that interact with a smart contract/resource that requires conformity with a certain known interface can execute certain functions and access certain parameters without having to read the actual contract code. Consulting the required interface should provide them with that knowledge, at least at a high level.

Q2.

Depends on the season really and on much "tired" of it I am. I used to love sushi until I ate so much of it that I went a whole year almost gagging if I saw a rice ball. I'm back on sushi again, but it was never the same.
Right now? I'm in a mood for a salmon penne pasta with cream cheese and rocket salad. I'm also going through a "curry" phase, so I'm probably end up getting a nice squid or tofu, mango and spinach curry over the next days.

Q3.

Fixed contract code (with commentaries)
```javascript
// The contract interface was not being imported 
import ITest from 0x01

// The contract definition did not specify the Interface, therefore the contract was not
// abiding by it. Fixed now
pub contract Test:ITest {
  pub var number: Int
  
  // As it is, this function is going to trigger the post condition for newNumber != 5
  // Also, though technically correct from the Interface standpoint, this function was not
  // doing what it was supposed to. Fixed it too
  pub fun updateNumber(newNumber: Int) {
    self.number = newNumber
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  // As it was, the resource 'Stuff' was abiding by the local 'IStuff' interface. Though this is allowed and correct from
  // a code development standpoint, it does not conform with the best practices. Fixed it so that the function abides
  // by the Interface defined in the contract interface instead
  pub resource Stuff: ITest.IStuff {
    pub var favouriteActivity: String

    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }

  init() {
    self.number = 0
  }
}
```