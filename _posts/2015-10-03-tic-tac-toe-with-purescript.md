---
layout:     post
title:      Rendering a Tic-Tac-Toe board with purescript-react
date:       2015-10-03 12:15:00
summary:    Playing around with the purescript-react bindings
categories: example
---

In web development there is nothing quite as satisfying as getting something on the screen. Today we'll use the [purescript-react](https://github.com/purescript-contrib/purescript-react) library to do just that. We'll think of a model for the Tic-Tac-Toe game and render it using React.

Setting up our project
----------------------

We'll start of by creating a new project with [pulp](https://github.com/bodil/pulp):

``` bash
  mkdir tic-tac-toe
  cd tic-tac-toe
  pulp init
```

We'll also install some needed dependencies:

``` bash
  pulp dep install --save purescript-react purescript-arrays
```

Now let's put these snippets in the root folder of your project and we're good to go.

{% gist d1fce315c8f52b6fc76e %}

Running `pulp server` in the project root should leave you with a website that greets you in your browsers console at <http://localhost:1337>.

Ready, Steady, Go
-----------------

The remaining blogpost is a literate sourcefile which should go into your src/Main.purs file. Don't worry about copying it bit by bit. I'll link to a gist with the complete source at the end.

### Get the imports out of the way first

``` haskell
  module Main where

  import           Control.Monad.Eff
  import           Data.Array (replicate, (!!))
  import           Data.Maybe (Maybe(..))
  import           Data.Maybe.Unsafe (fromJust)
  import           Data.Nullable (toMaybe)
  import           Prelude

  import           DOM (DOM())
  import           DOM.HTML (window)
  import           DOM.HTML.Document (body)
  import           DOM.HTML.Types (htmlElementToElement)
  import           DOM.HTML.Window (document)
  import           DOM.Node.Types (Element())

  import           React
  import qualified React.DOM as D
  import qualified React.DOM.Props as P
```

### Datatypes and Accessors

In Tic-Tac-Toe there are three different tokens a field could hold. It could hold

``` haskell
  data Token = X | O | E
```

where `E` denotes an empty field and `X` and `O` the two players.

We'll also define a `Show` instance for our Token datatype, so that we can display them as Strings in our UI.

``` haskell
  instance showToken :: Show Token where
    show X = "X"
    show O = "O"
    show E = ""
```

Lastly we'll write a method that generates some CSS classes for the different Tokens, so we can later style them on the webpage.

``` haskell
  classForToken X = "cell x"
  classForToken O = "cell o"
  classForToken E = "cell"
```

The Tic-Tac-Toe board consists out of 9 fields, of which each can hold a token. We'll model this as a simple Array and define an accessor function which resolves the x and y coordinates for us.

``` haskell
  type Board = Array Token

  get :: Int -> Int -> Board -> Maybe Token
  get x y board = board !! (3 * x + y)
```

In order to have a board that we can render we'll construct the following board:

``` bash
  |---+---+---|
  | O | O | O |
  |---+---+---|
  | X | X | X |
  |---+---+---|
  | O | O | O |
  |---+---+---|
```

``` haskell
  newBoard = replicate 3 O ++ replicate 3 X ++ replicate 3 O
```

### Describing our View

Because we are using React as our rendering engine, we'll need some boilerplate to construct valid React components. We'll create a new `ReactClass` using the spec method, which takes an initial componentstate and a function that produces a `ReactElement`. Because our BoardComponent is actually a pure function from a `Board` to an HTML representation, we don't need any state and just pass `unit` as the first argument to spec.

``` haskell
  boardComponent :: Board -> ReactClass Unit
  boardComponent board = createClass $ spec unit \_ -> return (grid board)
```

We'll render our Board into an HTML table element. The tick denotes that we don't want to pass any properties (for example classes) to the table element. We then nest an array of rows inside our table.

``` haskell
  grid :: Board -> ReactElement
  grid board = D.table' (map (row board) [0,1,2])
```

A row takes the boardstate and an x coordinate. We partially apply the row constructor to the boardstate and map the resulting function over the possible coordinates. Inside of each row we nest 3 cells.

``` haskell
  row :: Board -> Int -> ReactElement
  row board x = D.tr' (map (cell board x) [0,1,2])
```

For the cell we extract the token at the corresponding field inside our boardstate and then render a table cell. `className` corresponds to HTML's class. Strings are not ReactElements by themselves so we need to wrap our serialised token with the `text` wrapper from `React.DOM`.

``` haskell
  cell :: Board -> Int -> Int -> ReactElement
  cell board x y = D.td
    [P.className (classForToken token)]
    [D.text (show token)]
    where
      token = fromJust (get x y board)
```

### Putting it on the page

Our main method handles placing our boardComponent inside the body element of our page. The most interesting part is the argument to the `ui` function, which is the boardstate that we want to display. You can play around with it and change `newBoard` to something like \[X,X,X,E,X,X,X,X,O\].

``` haskell
  main = do
    body' <- getBody
    render (ui newBoard) body'
    where
      ui board = D.div' [ createFactory (boardComponent board) unit ]

      getBody = do
        win <- window
        doc <- document win
        elm <- fromJust <$> toMaybe <$> body doc
        return $ htmlElementToElement elm
```

This is all the code we need to render our Tic-Tac-Toe board. If you see a nice little playing field at <http://localhost:1337> everything went well.

<span class="absolute-center">![tic-tac-toe-board](/images/tic-tac-toe-board.png)</span>

I've put the resulting Main.purs file here: <https://gist.github.com/kRITZCREEK/607577be749f44eec600>

### Wrapping up

The purescript-react library is a nice example of how PureScript allows one to write low-level bindings to existing JavaScript libraries and utilize them in a seemless manner while using PureScripts type system to model the problem domain. Once we start adding functionality and interactivity into our UI's these benefits extend their reach into our business logic.

Happy hacking

Christoph
