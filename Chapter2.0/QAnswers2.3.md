Q1. My favourite people:

```javascript
pub fun main() {
    let favouritePeople: [String] = ["Thom", "David", "Jimmy"]
    log(favouritePeople)
}
```

Q2. Social media ranking:

```javascript
pub fun main() {
    let socialMediaStuff: {String: UInt64} = {
    "Facebook": 5, 
    "Instagram": 2,
    "Twitter": 0,
    "YouTube": 1,
    "Reddit": 3,
    "LinkedIn": 4 
    }
    log(socialMediaStuff)
}
```

Q3. The "!" operator prevents the referencing of nil type values by ensuring the code flow only continues if the result obtained in the variable affected by "!" is different than nil. If this condition is violated, an exception (or whatever error structure Cadence use. I haven't figure it out yet...) is thrown to signal this, preventing the following instruction to try and access properties and functions of nil values. The example illustrates a situation when a function expects a specific type (Int) returned, but because of the data structure used to access data (dictionary) does not impose that values cannot be 'nil' and thus, its default return is Int? (it can return an Integer or a nil. Who knows...). Accessing the dictionary with "!" removes the "nil" return possibility and synchronizes the return types. I'm assuming that "!" is particularly useful to ensure that objects or contracts are properly initialized before attempting any accesses (just based on my experience as developer), so something of this nature should be possible:

Consider the following simple contract:
```javascript
pub contract randomizedContract {
    pub var someVar: String?

    init() {
        if (unsafeRandom() % 2 == 0) {
            self.someVar = nil
        }
        else {
            self.someVar = "Ricardo"
        }
    }
}
```

(I was playing around with the emulator's autocomplete and saw that handy randomizer function. Perfect for my example)
I've set the initialization of its internal value to nil, if the random Integer returned is even, or to my name if odd. So in this case, I really don't know what is going to be set in that variable. If I deploy a script like this:

```javascript
import randomizedContract from 0x01

pub fun main(): String {
    return randomizedContract.someVar!
}
```

That return instruction can run into problems because I'm setting the return type of the main function to a "hard" String, when I don't know what has been set in "someVar" during the contract deployment. To prevent returning a nil, I can use the "!" operator to crash the script instead.
PS: I tried to test this code in the emulator but... to no avail. I'm unable to deploy any contracts without running into the infamous "GraphQL" error.

Q4.1 Mismatch types Errors are thrown when there are function calls that don't have synchronized return types. In this particular case, the return type of the function does not match exactly the return type of the dictionary that is accessed in it, and that is enough to raise that error.

Q4.2 There is a mismatch in return types between the main() function, which is set to a String, and the value returned from the dictionary, which is, by default, String? (or whatever the dictionary's value type is + '?'). The function is required to return a String at any times and reading a value from a dictionary does not guarantee that, hence the error.

Q4.3 Force an unwarp at the dictionary's return, thus "forcing" its return type from 'String?' to 'String' and a match between the function calls: 

```javascript
pub fun main(): String {
    let thing ; {Address:String} = (...)
    return thing[0x01]! 
}
```