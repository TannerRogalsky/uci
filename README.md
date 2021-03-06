UCI
===

UCI is a thin wrapper on a [uci
interface](http://en.wikipedia.org/wiki/Universal_Chess_Interface) chess engine.
It also runs a chess clock and a [chess.js](https://github.com/jhlywa/chess.js)
instance internally so that the user can quickly play a game of chess with time
controls. Currently only [stockfish](http://stockfishchess.org/) is bundled with
UCI but other engines can be used as well. See installing engines section below
for how to do that. UCI also supports polyglot books. See installing books
section for that.

## Installation
Make sure you have [node.js](http://nodejs.org/) installed. Then do:

    $ npm install uci

## Example
```js
var UCI = require('uci').UCI;
var uci = new UCI();
var os = require('os');
var Chess = require('chess.js').Chess;
var game = new Chess();

console.log('Type exit or quit to exit.');
uci.on('ready', function () {
    //Start a new 10 minute game with engine as black, use the first found
    //engine and the first found polyglot book
    uci.startNewGame(uci.getAvailableEngines()[0], 'black', 10,
        uci.getAvailableBooks()[0]);
}).on('newgame', function () {
    console.log("A new 10 minute game has started.");
    console.log("Enter your moves in algebraic notation. E.g. e2e4<Enter>");
    console.log(game.ascii());
    var stdin = process.openStdin();
    stdin.on('data', function (move) {
        if (move == 'exit' + os.EOL || move == 'quit' + os.EOL) {
            uci.shutdown();
            process.exit();
            return;
        }
        function convertToMoveObject(move) {
            if (typeof move == 'object') {
                return move;
            }
            var result = {};
            result.from = move.substring(0, 2);
            result.to = move.substring(2, 4);
            if (move.length > 4) {
                result.promotion = move.substring(4);
            }
            return result;
        }
	move = convertToMoveObject(move.toString().replace(os.EOL, ''));
	game.move(move);
	console.log(game.ascii());
	console.log("Engine thinking...");
        uci.move(move);
    });
}).on('moved', function (move) {
    game.move(move);
    console.log(move.from + move.to + (move.promotion ? move.promotion : ''));
    console.log(game.ascii());
}).on('error', function (message) {
    console.log('Error:' + message);
}).on('exit', function (message) {
    console.log('Exiting:' + message);
}).on('gameends', function (result, reason) {
    console.log('Game ends with result ' + result + ' because ' + reason);
    uci.shutdown();
    process.exit();
});
```
## API

### Events
UCI is an [EventEmitter](http://nodejs.org/api/events.html) and it raises
following events -

#### ready
This event is raised as soon as uci has completed enumerating all the engines.

#### newgame
This event is raised once UCI has started a new game.

#### moved
This event is raised when the engine has made a move. The only argument _move_
is an object with properties _from_, _to_ and _promotion_. _from_ and _to_ are
the algebraic notation square names for the from square and to square
respectively. _promotion_ property contains the piece to which the pawn is being
promoted otherwise it is null. E.g. following is a valid move object
```js
{from:'h7', to:'h8', promotion:'q'}
```

#### error
This event is raised when UCI detects an error. E.g. if the move function is
passed an invalid move object. The argument _message_ contains a string with
details about the error.

#### exit
This event is raised when the UCI engine process terminates. The argument
_message_ contains a string with the reason for exiting.

#### gameends
This event is raised when the game ends either in a draw or with one of the
players (engine or the other player) winning. The two arguments are _result_
which contains the result in a string (e.g. '1-0', '1/2-1/2' or '0-1') and
_reason_ which describes the reason for the result.

### Functions
UCI exposes following functions -

#### move(move)
This function takes an argument _move_ and tries to move this on the internal
board. See the format of the _move_ object above.  If the move object is invalid
in any way the _error_ event is raised.
```js
var move = {from:'h7', to:'h8', promotion:'q'};
uci.move(move);
```

#### getAvailableEngines()
When a new uci instance is created it detects all the uci engines placed inside
the *uci/engines* directory. This function returns a list of these engines.

#### getAvailableBooks()
When a new uci instance is created it enumerates all the files in the
*uci/books* directory. This function returns a list of these books.

#### startNewGame(engine, engineSide, gameLength, [bookFile])
This function starts a new game. _engine_ is the path of the engine executable,
_engineSide_ is the side which the engine will play. It should be either 'white'
or 'black'. _gameLength_ is the game length in minutes. The optional bookFile is
the path to the polyglot book which should be used to lookup moves.
```js
uci.startNewGame('path/to/engine-executable', 'white', 10,
    'path/to/polyglot-book);
```

#### shutdown()
This function should be called once at the end for cleanup. It will send the
quit uci command to any running engine process.

## Installing engines
Place your own uci engines inside the *uci/engines* directory. UCI will detect
all the uci engines inside the engines directory by visiting all the files
recursively. The detected engines can be retrieved by calling the
*getAvailableEngines* function.

## Installing books
Place any polyglot format books under the *uci/books* directory. You can also
create sub directories within the books directory for better organization of
your books. UCI will treat all files found recursively under the *uci/books*
directory as polyglot books so only put valid books files there.

## Contributing
Fork, pick an issue to fix from [issues](https://github.com/imor/uci/issues) or
add a missing feature and send a pull request.

## Credits
The excellent [chess.js](https://github.com/jhlywa/chess.js) library by [Jeff
Hlywa](https://github.com/jhlywa) and [Stockfish](http://stockfishchess.org/)
the chess engine bundled with UCI.

## License
UCI is released under the MIT License. See the bundled LICENSE file for details.
