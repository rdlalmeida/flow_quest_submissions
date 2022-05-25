Q1.
```javascript
pub contract messingWithResources{
	pub var petArray: @[Pet]
	pub var petDict: @{String: Pet}

	// Resource definition
	pub resource Pet {
	pub let species: String
	pub let name: String
	pub let age: Int

	// My cat for sake of example
	init() {
		self.species = "cat"
		self.name = "Boris"
		self.age = 5
	}
	}

	// Array operations
	// Add a new Pet to the array
	pub fun addPetToArray(pet: @Pet) {
		self.petArray.append(<- pet)
	}

	// Remove the Pet at index from the array and return it
	pub fun removePetFromArray(index: Int): @Pet {
		return <- self.petArray.remove(at: index)
	}

	// Dictionary operations
	// Add a new Pet to the dictionary
	pub fun addPetToDict(pet: @Pet) {
		// Extract the key to use from the pet's name
		let key = pet.name

		// Store any existing old pet referenced with the key to a new constant and store the provided Pet in that position instead
		let anyExistingPet <- self.petDict[key] <- pet
		// Destroy any old pet retrieved
		destroy anyExistingPet
	}

	// Remove and return a Pet from the dictionary
	pub fun removePetFromDict(petName: String): @Pet {
		// Attempt to extract the Pet with the provided name from the dictionary, throwing an alarm if there is none referenced
		let petToReturn <- self.petDict.remove(key: petName) ?? panic("There is no pet named '".concat(petName).concat("' in the dictionary"))
		// Return the resource
		return <- petToReturn
	}

	// Initialization function
	init() {
	self.petArray <- []
	self.petDict <- {}
	}
}
```