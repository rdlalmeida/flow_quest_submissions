Q1.

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
    
Q2.

    import messingWithReferences from 0x03

    pub fun main(): String {
        let ref = messingWithReferences.getBookReference(key: 1)

        return ref.title.concat(", by ").concat(ref.author).concat(ref.haveRead ? " was finished already" : " was not touched yet...")
    }
    
Q3.

References are a handy way to keep the operational overhead on the Flow blockchain to a minimum. Data flow in a blockchain is conditioned by the resources made available by the supporting network, both for read and write operations, since these need to be executed by the virtual machine. Writes are more costly  due to the consensus operation that it involves, but read operations also consume resources from the virtual machine that runs the underlining contracts/scripts, therefore it is in the interest of the blockchain to keep the associated effort to a minimum. When dealing with Resources, because of the added security that these require, read operations require effort, especially if the Resource associated is complex. References abstract that by making the relevant fields of the Resource available for read but without actually doing anything to the Resource itself.
