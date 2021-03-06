![build status](https://travis-ci.org/meiersi/strict-base-types.svg?branch=master)


Strict variants of the types provided by base
=============================================

Motivation
----------

It is common knowledge that lazy datastructures can lead to space-leaks.
This problem is particularly prominent, when using lazy datastructures to
store the state of a long-running application in memory. The easiest
solution to this problem is to use fully strict types to store such state
values. By "fully strict types" we mean types for whose values it holds
that, if they are in weak-head normal form, then they are also in normal
form. Intuitively, this means that values of fully strict types cannot
contain unevaluated thunks.

To define a fully strict datatype, one typically uses the following recipe.
 
1. Make all fields of every constructor strict; i.e., add a bang to
   all fields.
 
2. Use only strict types for the fields of the constructors. 
 
The second requirement is problematic as it rules out the use of
the standard Haskell `Maybe`, `Either`, and pair types. This library
solves this problem by providing strict variants of these types and their
corresponding standard support functions and type-class instances. 
 
Note that this library does currently not provide fully strict lists.
They can be added if they are really required. However, in many cases one
probably wants to use unboxed or strict boxed vectors from the `vector`
library (http://hackage.haskell.org/package/vector) instead of strict
lists.  Moreover, instead of `String`s one probably wants to use strict
`Text` values from the `text` library
(http://hackage.haskell.org/package/text).
 
Example
-------

This library comes with batteries included; i.e., missing instances
for type-classes from the `deepseq`, `binary`, `aeson`, `QuickCheck`, and
`lens` packages are included. Of particluar interest is the `Strict`
type-class provided by the lens library
(http://hackage.haskell.org/packages/archive/lens/3.9.0.2/doc/html/Control-Lens-Iso.html#t:Strict).
It is used in the following example to simplify the modification of
strict fields.
 
~~~~{.haskell}
(-# LANGUAGE TemplateHaskell #-)   -- replace with curly braces, 
(-# LANGUAGE OverloadedStrings #-) -- the Haddock prologues are a P.I.T.A!

import           Control.Lens ( (.=), Strict(strict), from, Iso', makeLenses)
import           Control.Monad.State.Strict (State)
import qualified Data.Map                   as M
import qualified Data.Maybe.Strict          as S
import qualified Data.Text                  as T

-- | An example of a state record as it could be used in a (very minimal)
-- role-playing game.
data GameState = GameState
    ( _gsCooldown :: !(S.Maybe Int)
    , _gsHealth   :: !Int
    )  -- replace with curly braces, *grmbl*

makeLenses ''GameState

-- The isomorphism, which converts a strict field to its lazy variant
lazy :: Strict lazy strict => Iso' strict lazy
lazy = from strict

type Game = State GameState

cast :: T.Text -> Game ()
cast spell =
    gsCooldown.lazy .= M.lookup spell spellDuration
    -- ... implement remainder of spell-casting ...
  where
    spellDuration = M.fromList [("fireball", 5)]
~~~~

See
http://www.haskellforall.com/2013/05/program-imperatively-using-haskell.html
for a gentle introduction to lenses and state manipulation.
 
Related work
------------
 
Note that this package uses the types provided by the `strict` package
(http://hackage.haskell.org/package/strict), but organizes them a bit
differently. More precisely, the @strict-base-types@ package
 
- only provides the fully strict variants of types from `base`,
 
- is in-sync with the current base library (base-4.6), 
 
- provides the missing instances for (future) Haskell platform packages, and
 
- conforms to the standard policy that strictness variants of an existing
  datatype are identified by suffixing \'Strict\' or \'Lazy\' in the
  module hierarchy.
