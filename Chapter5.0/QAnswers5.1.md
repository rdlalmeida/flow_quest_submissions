Q1. 

In order to retrieve information from a Blockchain, one has to either:
1. Check the data in a specific block (using a block explorer for example)
2. Check the data stored in a given smart contract (very limited since it requires access to the storage area of it, i.e., can only be done by the contract's owner)
3. Check the transactional log of the blockchain operations, a text based interface to the outside world

Of all these methods, the simplest and fastest one to consult operational data in a blockchain is #3. <code>Events</code> operate in this context. When an <code>event</code> is triggered (using the <code>emit</code>) instruction), the information included in its definition is written into the blockchain's transactional log, which can then be consulted to determine the outcome of an operation without needing to search for the block where the result may have been written or, in the case of smart contract operation, consult the smart contract's storage area. This also allows for clients, i.e., people listening for changes in a contract, to react automatically to changes by "listening" for events emitted from that contract.


Q2.

Contract code:

```javascript
pub contract TryingEvents{
	// A simple event to be triggered upon calling another function later on
	pub event deadMonster(name: String)
	pub event monsterStillAlive(name: String)


	pub fun createMonstrosity(name: String): @Frankenstein {
		return <- create Frankenstein(name: name)
	}

	pub resource Frankenstein{
		pub var name: String
		pub var isAlive: Bool

		init(name: String) {
			self.name = name
			self.isAlive = true
		}

		pub fun killTheAbomination() {
			self.isAlive = false
		}

		pub fun checkPulse(): Bool {
			if (self.isAlive) {
				log("The abomination is still alive and kicking!")
				// Emit the respective event
				emit monsterStillAlive(name: self.name)
			}
			else {
				log("The freak is dead already!")
				// Same here
				emit deadMonster(name: self.name)
			}

			return self.isAlive
		}
	}
}
```

Transaction code used to trigger the events:

```javascript
import TryingEvents from 0x01

transaction() {
prepare(signer: AuthAccount) {
}

execute{
	// Create Frank, the Frankenstein
	log("Assembling a monster out of spare parts from the local cemetery...")
	let frankTheFreak <- TryingEvents.createMonstrosity(name: "Frank")

	// Check if its alive, triggering one of the events and setting the internal boolean to a different value to trigger the next event
	if (frankTheFreak.checkPulse()) {
		log("It's alive! I'm bored now... Going to kill it...")
		// If it is, kill it
		frankTheFreak.killTheAbomination()

		// Check the pulse once again to trigger the other event
		if (frankTheFreak.checkPulse()) {
			log("Unable to kill the freak.. somehow. Burn it to a crisp then!")
		}
		else {
			log(frankTheFreak.name.concat(" is dead. Going to set fire to the corpse..."))
		}
		}
		else {
			log("The monster is already dead. All its left is to get rid of the cadaver...")
		}
		
		// In either case, the Resource needs to be destroyed
		destroy frankTheFreak
		log("Experiment complete!")
	}
}
```


Q3.

Contract code with <code>pre</code>, <code>post</code> and <code>before</code> usage:

```javascript
pub contract TryingEvents{
	// A simple event to be triggered once a Resource gets destroyed
	pub event deadMonster(name: String)
	pub event monsterStillAlive(name: String)
	priv let maxName: UInt64


	pub fun createMonstrosity(name: String): @Frankenstein {

		pre{
			name.length > 0 && name.length < 10: 
				"The monster cannot hold long names. Please don't use more than ".concat(self.maxName.toString()).concat(" characters")
		}

		post {
			result.isAlive: "Oops... something went very wrong there... the monster did not survive the operation..."
		}
		return <- create Frankenstein(name: name)
	}

	init() {
		self.maxName = 10
	}

	pub resource Frankenstein{
		pub var name: String
		pub var isAlive: Bool

		init(name: String) {
			self.name = name
			self.isAlive = true
		}

		pub fun killTheAbomination() {
			self.isAlive = false
		}

		// Simple renaming function to implement 'before'
		pub fun changeName(newName: String) {
			pre {
				newName.length > 0: "The monster cannot remain nameless!"
			}

			post {
				before(self.name) != self.name: "The new name is identical to the previous one! The monster is getting confused..."
			}

			self.name = newName
		}

		pub fun checkPulse(): Bool {
			if (self.isAlive) {
				log("The abomination is still alive and kicking!")
				// Emit the respective event
				emit monsterStillAlive(name: self.name)
			}
			else {
				log("The freak is dead already!")
				// Same here
				emit deadMonster(name: self.name)
			}

			return self.isAlive
		}
	}
}
```

Q4.

Contract code with commentaries answering the questions posed:

```javascript
pub contract Test {
	pub fun numberOne(name: String) {
	pre {
		// The pre-condition succeeds for "Jacob" since "Jacob".length == 5, therefore the pre-condition evaluates to 'true'.
		// The execution goes through and the log instruction occurs
		name.length == 5: "This name is not cool enough."
	}
	log(name)
}

pub fun numberTwo(name: String): String {
	pre {
		name.length >= 0: "You must input a valid name."
	}
	post {
		result == "Jacob Tucker": result.concat(" :I don't like this name at all")
	}

	/* 
	* Both pre and post conditions evaluate to 'true':
	* "Jacob".length == 5 (> 0) so the pre-condition is true
	* The result is going to be "Jacob Tucker", therefore evaluating the post condition to true also.
	* Both conditions are true, so there is no flow interruption. The function returns "Jacob Tucker" back  
	*/
	return name.concat(" Tucker")
}

pub resource TestResource {
	pub var number: Int

	pub fun numberThree(): Int {
		post {
			// In this case, the result is the original self.number plus one, therefore this post condition also evaluates to true,
			// which allows for the function to execute completely
			before(self.number) == result + 1
		}
			self.number = self.number + 1

			// The self.number being returned is the initial value plus 1. Assuming a single execution right after the resource
			// creation (which sets self.number = 0), the self.number should be 1 after this run.
			return self.number
		}

		init() {
			self.number = 0
		}

	}

}
```