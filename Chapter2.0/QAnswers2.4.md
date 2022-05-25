Q1. Q2 and Q3 (its all in the same code block)

```javascript
pub contract structExample {

    pub var favoriteGames: {Int: VideoGame}

    pub struct VideoGame {
        pub let index: Int          // An simple unique index to identify entries
        pub let title: String       // The title of the game
        pub let company: String     // Development studio
        pub let pubYear: Int        // Publication year
        pub var completed: Bool     // Have I completed it?

        init(_index: Int, _title: String, _company: String, _pubYear: Int, _completed: Bool) {
            self.index = _index
            self.title = _title
            self.company = _company
            self.pubYear = _pubYear
            self.completed = _completed
        }
    }

    pub fun addNewGame(index: Int, title: String, company: String, pubYear: Int, completed: Bool) {
        let newVideoGame = VideoGame(_index: index, _title: title, _company: company, _pubYear: pubYear, _completed: completed)
        self.favoriteGames[index] = newVideoGame
    }

    init() {
        self.favoriteGames = {}
    }
}
```
    
Q4.
```javascript
import structExample from 0x01

transaction(index: Int, title: String, company: String, pubYear: Int, completed: Bool) {
    prepare(signer: AuthAccount) {}

    execute{
        structExample.addNewGame(index: index, title: title, company: company, pubYear: pubYear, completed: completed)

        log("Added '".concat(title).concat("' to my favourite games!"))
    }
}
```

Q5.
```javascript
import structExample from 0x01

pub fun main(index: Int): structExample.VideoGame {
    return structExample.favoriteGames[index]!
}
```
