---
layout:     post
title:      Playing Tic-Tac-Toe using purescript-signal
date:       2015-10-07 12:15:00
summary:    Elm in PureScript
categories: tutorial
---

*Working Title: Elm in PureScript*

<span class="absolute-center"> ![Tic Tac Toe](http://ecx.images-amazon.com/images/I/51Ak%2BBWmrSL.jpg) </span>

Last time I left you hanging with a still Tic-Tac-Toe board. Time to make it come to life! We'll be using the [purescript-signal](https://github.com/bodil/purescript-signal) library from Bodil Stokke(@bodil), which ports the underlying abstraction of the [Elm programming language](http://elm-lang.org/), *Signals*, to PureScript.

Signals and you!
----------------

I'll give an incomprehensive definition that should nonetheless suffice for our needs.

> A Signal denotes a value that changes over time - not in the sense of a mutable variable - but as a value that incorporates the concept of time.
> -- <cite>kRITZCREEK</cite>

This allows us to declaratively describe our UI as a value that changes over time aswell as the user input which also changes over time.

Because Events are more occurences than values that change over time they are rather hard to describe with signals.

purescript-signal helps us out by providing the Channel datatype which we can use to send events around and turn these streams of events into signals with the `subscribe :: forall a. Channel a -> Signal a` function once we want to use them as signals.

Let's look at the function at the core of the purescript-signal library.

``` haskell
  foldp :: forall a b. (a -> b -> b) -> b -> Signal a -> Signal b
```

Where foldp stands for "fold the past". We substitute these generic types to a more concrete type so it's easier to talk about them.

``` haskell
  foldp :: (Action -> State -> State) -> State -> Signal Action -> Signal State
```

`foldp` takes a "stepper" function which takes a State, an Action and returns a new State that takes that Action into account. It takes an initial State and a Signal of Actions and uses the stepper function to fold the initial State and all the Actions that occur into a Signal of States.

Using these building blocks, Elm defines "The Elm Architecture" which consists of three pieces:
-   Model
-   Update
-   View

Let's use these concepts to give life our Tic-Tac-Toe game!

Our Datatypes(Model)
--------------------

We set up two types. GameState holds the board and the current player while an Environment additionally holds a Channel we use to send the Actions back from the view.

PureScript's expressive typesystem allows us express the fact that these two types share the board and the current player through extensible rows.

Additionally we'll define a helper that takes a GameState and a Channel and constructs an Environment from the two.

``` haskell
  type State a = { board :: Board, currentPlayer :: Token | a }
  type GameState = State ()
  type Environment = State (channel :: C.Channel Action)

  mkEnv :: C.Channel Action -> GameState -> Environment
  mkEnv channel gameState = {
    board: gameState.board,
    currentPlayer: gameState.currentPlayer,
    channel: channel
  }
```

The "stepper" function (Update)
-------------------------------

The actions we'll be supporting is starting a new game and putting a token onto the board.

``` haskell
  data Action = NewGame | Click Int Int
```

Our step function takes an Action and the current GameState and applys that to the GameState. We'll pattern match against the type of Action and act accordingly.

``` haskell
  step :: Action -> GameState -> GameState
```

In the case of the NewGame Action we want to discard the old state and just replace it with a new(empty) one.

``` haskell
  step NewGame _ = newGameState
```

In the case of a Click Action we first determine whether the clicked field was actually empty...

``` haskell
  step (Click x y) gameState =
    case get x y gameState.board of
```

... and if it was we put the Token of the current player onto the clicked field and make it the next players turn.

``` haskell
  E -> gameState {
    board = set x y gameState.currentPlayer gameState.board,
    currentPlayer = nextPlayer gameState.currentPlayer
  }
```

If the field already holds a token, we'll just ignore the Action and return the old GameState.

``` haskell
  _ -> gameState
```

`nextPlayer` alternates in between `X` and `O` and is shown here for completion.

``` haskell
  nextPlayer :: Token -> Token
  nextPlayer X = O
  nextPlayer O = X
  nextPlayer E = E
```

User Interaction (View)
-----------------------

Next we need to generate the Actions for our game from the view.

Because we'll be using the board, that we built in [the last post](https://kritzcreek.github.io/example/2015/10/03/tic-tac-toe-with-purescript/), we'll have to apply some changes to our existing codebase and I'll walk you through the important changes in the form of a git diff.

Since we'll start to put tokens onto the board we need to add the companion of our get method.

``` haskell
  +set :: Int -> Int -> Token -> Board -> Board
  +set x y token board = fromJust (updateAt (3 * x + y) token board)
```

In contrast to a static board an ongoing game of Tic-Tac-Toe also has a currently active player.

``` haskell
  +newGameState = {currentPlayer: X, board: replicate 9 E}
```

We add a little wrapper around our existing grid which contains a button to restart the game and a textfield showing the currently active player. On clicking the newGameButton we send a NewGame action.

``` haskell
  +newGameButton c =
  +  D.button [P.onClick (\_ -> C.send c NewGame)] [D.text "New Game"]

  +game :: Environment -> ReactElement
  +game env = D.div'
  +  [newGameButton env.channel
  +  , grid env
  +  , D.text (show (env.currentPlayer) ++ "'s turn.")]
```

When the user clicks one of the cells we'll send a Click action which contains the x and y coordinates of the clicked cell.

``` haskell
  -cell :: Board -> Int -> Int -> ReactElement
  -cell board x y  = D.td
  -  [P.className (classForToken token)]
  +cell :: Environment -> Int -> Int -> ReactElement
  +cell env x y = D.td
  +  [P.className (classForToken token)
  +  , P.onClick (\_ -> C.send env.channel (Click x y))]
```

Putting it together
-------------------

Let's look at the main function *Literate Programming Style!*

We've got an initial State...

``` haskell

  main = do
    body' <- getBody

    channel <- C.channel NewGame
```

... and a Signal of Actions from the view.

``` haskell

  let actions = C.subscribe channel
```

Using the `foldp` function we can then fold an initial GameState and our Signal of Actions into a Signal of GameStates...

``` haskell
  let gameState = S.foldp step newGameState actions
```

... over which we then map(~&gt;) a function...

``` haskell
  let game = gameState S.~>
```

that adds the channel to our GameState so that the view can send new Actions...

``` haskell
  mkEnv channel >>>
```

... and renders it to the screen using our render function.

``` haskell
  (\env -> render (ui env) body') >>> void
```

Now all that is left is to turn that Signal of UI into an effectful computation...

``` haskell
  S.runSignal game
```

... and we're done!

Conclusion
----------

1.  PureScript's typesystem is expressive enough to port the entirety of **The Elm Architecture**.
2.  Using Signals and the declarative style of React, we can write clear, declarative and typesafe UIs.
3.  PureScript is lots of fun!

Here is a gist with the final solution: [https://gist.github.com/kRITZCREEK/cdf9f403b55b345cdc51](https://gist.github.com/kRITZCREEK/cdf9f403b55b345cdc51)

Links
----------

"Controlling Time and Space: understanding the many formulations of FRP" by Evan Czaplicki - [https://www.youtube.com/watch?v=Agu6jipKfYw](https://www.youtube.com/watch?v=Agu6jipKfYw)

The Elm Architecture - [https://github.com/evancz/elm-architecture-tutorial/](https://github.com/evancz/elm-architecture-tutorial/)
