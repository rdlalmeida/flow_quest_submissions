Q1.
```javascript
pub contract messingWithReferences {
	pub var resourceDict: @{Int: Book }

	pub resource Book{
		pub let index: Int
		pub let title: String
		pub let author: String
		pub let haveRead: Bool

	init(_index: Int, _title: String, _author: String, _haveRead: Bool) {
		self.index = _index
		self.title = _title
		self.author = _author
		self.haveRead = _haveRead
		}
	}

	pub fun getBookReference(key: Int): &Book {
		return &self.resourceDict[key] as &Book
	}

	init() {
		let index1: Int = 1
		let index2: Int = 2
		self.resourceDict <- { 
		index1: <- create Book(_index: index1, _title: "Lord of the Rings", _author: "J.R.R.Tolkien", _haveRead: true),
		index2: <- create Book(_index: index2, _title: "The Shining", _author: "Stephen King", _haveRead: true)
		}
	}
}
```

Q2.
```javascript
import messingWithReferences from 0x03

pub fun main(): String {
	let ref = messingWithReferences.getBookReference(key: 1)

	return ref.title.concat(", by ").concat(ref.author).concat(ref.haveRead ? " was finished already" : " was not touched yet...")
}
```

Q3.

References are a handy way to keep the operational overhead on the Flow blockchain to a minimum while also adding a layer of security that allows access to information contained in a Resource without compromising (moving or copying) the Resource itself. References get all the information contained in a Resource but without providing access to it.
