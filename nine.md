# Concepts of Bottom-up parsing
## Top-down vs. bottom-up parsing
- `Top-down` parsers construct a derivation from the root to the leaves of a parse tree.
- `Bottom-up` parsers construct a reverse derivation from the leaves up to the root of the parse tree.
    - Can handler a larger class of grammars
    - most parser generators use bottom-up methods
    - requires less grammar transformations (generally no transformation at all)

## LR parsers
- Often called `shift-reduce` parsers because they implement two main operatons:
    - `shift`: pushes the lookahead symbol onto the stack and reads another token
    - `reduce`: matches a group of adjacent symbol `β` on the stack with the right-hand-side of a production and replaces `β` with the left-hand-side of the production (reverse application of a production). We reduce only if `β` is a handle
    - `handle`: a handle of a sentential form is a portion of this sentential form that matches the right-hand-side of a production, and whose reduction to the non-terminal on the left-hand-side of the production represents one step along a reverse rightmost derivation.

- LR parsers can be constructed to recognize virtually all programming language constructs for which a context-free grammar can be written
- The LR method is the most general non-backtracking shift-reduce parsing method known, yet it can be implemented as efficiently as other shift-reduce methods
- The class of grammars that can be parsed using LR method is a proper superset of the class of grammars that can be parsed with LL predictive parsers.
- An LR parser can detect a syntactic error as soon as it is posible to do so on a left-to-right scan of the input 
- Three types of LR parsers:
    - `SLR`: simple LR
        - Easiest to implement, but the least powerful of LR methods. May fail to produce a parsing table for some grammars
    - `CLR`: canonical LR - or LR(1)
        - most powerful, general and expensive LR method.
    - `LALR`: lookahead LR
        - intermediate in power and cost. Will work for most programming language constructs.

## LL vs LR parsing
- `LR(k)` stands for:
    - scan left to right
    - compute rightmost derivation
    - using k lookahead tokens

- `LL(k)` stands for:
    - scan left to right
    - compute rightmost derivation
    - using k lookahead tokens

## LR parsing algorithm

```txt
push(0)
x = top()
a = nextToken()
repeate forever:    
    if (action[x, a] == shift s')
        push(a)
        push(s')
    else if (action[x, a] == reduce A→β)
        multiPop(2*|β|)
        s' = top()
        push(A)
        push(goto[s', A])
        write(A→β)
    else if (action[x, a] == accept)
        return true
    else
        return false
```

## LR parsing table: example

# Construction of an LR Parsing table
## LR parsing tables: concepts
- The parsing table represents a DFA
- The states:
    - represent what has been parsed in the recent past on the way to recognize a handle
    - Control how the DFA will respond to the next symbol on top of the stack
    - a stack is used to keep track of the yet unresolved part of the path taken by the parse
- The transitions:
    - Represent the fact that we successfully go past a certain symbol during the derivation
- The goal is to find and reduce handles;
- To keep track of how far we have gotten through the growth of a handle, we use the notion of items.

## Constructing LR parsing tables: items
- An item is a production with a place marker inserted somewhere in its right-hand-side (in other words, a potential handle), indicating the current parsing state in the production, e.g. E→E+T has 4 items:
```
[E → •E+T]
[E → E•+T]
[E → E+•T]
[E → E+T•]
```

- Symbols to the left of the dot are already on the stack, those to the right are expected to be successfully parsed in the future
- A `shift action` taken by the parser represents the moving of the dot within a right-hand-side, transitioning to a new state in the DFA
- A `reduce action` taken by the parser represents the inverse application of a grammar rule once we have come to the end of its  right-hand-side (i.e. handle). Once a handle has been identified, the elements of the right hand side are removed from the stack and replace by the left hand side symbol. Then, we jump to the state that corresponds to the next handle to be parsed, as indicated by the `goto` part of the table.
- `initial item`: an item beginign with a dot. This represents the fact that we are starting to recognized an entirely new handle.
- `completed item`: an item ending with a dot. These are candiates for reduction. When we have a completed item, a handle has been recognized. We can then reduce this handle to its corresponding left hand side non-terminal.
- Moving the dot one symbol further in the right hand side corresponds to a state transition in the DFA. 
- Starting from state 0 (starting symbol rule), and computing the `closure` (i.e. all different possible immediate expansions using the grammar rules) we find all possible state transitions from this state using items (i.e. the `item set of a state`)
- Then for each group of items with the same symbol to the right of the dot in the item set of a state, we apply a transition to another state (i.e. moving the dot one step further). If the state is new (i.e. the sentential form to the left of the dot has not been encountered yet), we compute its closure and further transitions.
- We repeat this process until no more new states can be generated and all handlers have been processed.

## Constructing LR item sets: CLOSURE and GOTO

## Constructing the item sets

## Constructing the LR table

## Filling the table