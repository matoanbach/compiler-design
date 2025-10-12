## Syntax error handling in top-down predictive parsing
### Syntax error handling
- A syntax error happens when the stream of tokens coming from the lexical analyzer does not comply with the grammatical rules defining the programming language.
- A syntax error is found when the next token in input is not expected according to the syntactic definition of the language
- One of the main roles of a compiler is to identify all programming errors and give meaningful indications about the location and nature of errors in the input program.

### Goals of syntax error handling
- Detect all syntax errors.
- Report the presenc of syntax errors clearly and accurately.
- Recover from syntax errors to be able to detect all subsequent errors.
- Should not slow down the processing of correct programs.
- Avoid incorrect reporting of errors, e.g. reporting spurious errors that are just consequences of an earlier error.

### Reporting errors
- Give the position of the error in the source file, maybe print the offending line and point at the error location.
- If the nature of the error is easily identifiable, give a meaningful error message.
- The compiler should not provide errorneous information about the nature of errors.

### Error recovery
- When a syntax error is encountered, the parser should be able to continue the parse in order to report all errors
- Good error recovery highly depends on how quickly the error is detected.
- Often, an error will be detected only after the faulty token has passed.
- If so, it will then be more difficult to achieve good error reporting, as well as good error recovery.
- Bottom-up parsers generally detect errors quicker than top-down parsers.
- Should recover from each error quickly enough to be able to detect subsequent errors. Error recovery should skip as less tokens as possible
- Should not identify more errors than there really is. Cascades of errors that result from token skipping should be avoided.
- Should not induce processing overhead when errors are not encountered
- Should aboid to report other errors that are consequences of the application of error recovery, e.g. semantic error.

# Error Recovery Strategies
- There are many different stratefies that a parser can employ to recover from syntatic errors.
- Although some are better than others, none of these methods provide a universally optimal solution.
- Counter-examples:
    - Error productions
    - Phrase level correction
    - Global correction
- Most often used technique:
    - Panic mode, or don't panic (Nicklaus Wirth)

# Error Recovery Strategies: counter-examples
- Error productions:
    - The grammar is augmented with `error productions`. For each possible error, an error productin is added. An error is trapped when an error production is successfully used.
    - Assumes that all specific errors are known in advance
    - One error production is needed fork each possible error.
    - Error productions are specific to the rules in the grammar. A change in the grammar implies a change of the corresponding error productions.
    - Extremely hard/costly to maintain.

- Phrase-Level Correction
    - On discovering an error, the parser performs a local correction on the remaining input, e.g. replace a comma by a semicolon, delete an extraneous semicolon, insert a missing semicolon, etc.
    - Corrections are done in specific contexts. THere are myriads of different such contexts.
    - Cannot cope with errors that occured before the point of detection.
    - Can enter an infinite loop, e.g. insertion of an expected token.
    - The corrected program is not guaranteed to have the originally intended meaning.

- Global Correction
    - The entire erroenous input token stream is transformed step by step until it becomes valid
    - Ideally, as few chagnes as possible are made in processing an incorrect token stream
    - Global correction is about choosing the minimal sequence of changes to obtain a least-cost correction
    - Given an incorrect input token stream X, global correction will find a parse tree for a related token stream Y, such that the number of insertions, deletions, and changes of token required to transform X into Y is as reduced as possible
    - Can be very computationally-intensive
    - The closest correct program is unlikely to carry the original meaning intended by the programmer.
    - Interesting problem, but not very useful in practice.

- Panic Mode
    - On discovering an error, the parser discards input tokens until an element of a designated set of synchronizing tokens is found. Synchronizing tokens are typically delimiters such as semicolons or end of block delimiters
    - A systemic and general approach is to use the FIRST and FOLLOW sets as synchronizing tokens.
    - Skipping tokens often has a side-effect of skipping other errors. Choosing the right set of synchronizing tokens is of prime importance.
    - Simplest method to implement
    - Can be integrated in most parsing methods.
    - Cannot enter an infinite loop

## Panic mode error recovery: variations
- Variation 1:
    - Given a non-terminal A on top of the stack, skip input tokens until an element of FOLLOW(A) appears in the token stream
    - Pop A from the stack and resume parsing
    - Report on the error found and where the parsing was resumed
- Variation 2:
    - Given a non-terminal A on top of the stack, skip input tokens until an element of FIRST(A) appears in the token stream
    - Report on the error found and where the parsing was resumed.
- Variation 3:
    - If we combine variation 1 and 2, when there is a parse error and non-terminal A on top of the stack, we skip input tokens until we see either
        - A token in FIRST(A), in which case resume parsing
        - A token in FOLLOW(A), in which case we pop A off the stack and resume parsing
    - Report on the error found and where the parsing was resumed.

## Error Recovery in Recursive Descent Predictive Parsers
- In what was presented before, the parser would stop at the first error, i.e. it cannot recover from errors.
- There possible cases of error detection:
    - The lookahead symbol is not in FIRST(LHS) and there is no epsilon production for the currently parsed non-terminal
    - If there is an epsilon production for the currently parsed non-terminal, the lookahead symbol is not in FOLLOW(LHS), and neither in FIRST(LHS).
    - The `match()` function is called in a no-match situation
- Solution:
    - Create a `skipErrors()` function that skips tokens until an element of FIRST(LHS) or FOLLOW(LHS) is encountered
    - Upon entering any parsing function, call `skipErrors()`

## Error Recovery in Table-Driven Predictive Parsers
- All empty cells in the parsing table represent the occurrence of a syntax error
- Each case represents a specific kind of error
- Task when an empty (error) cell is read:
    - Recover from the error
        - Either pop the stsack, or skip tokens (often called `scan`)
    - Report the error.