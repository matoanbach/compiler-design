# Lexical Analysis
- Lexical analysis is the process of converting a sequence of characters into a sequence of tokens.
- A program or function that performs lexical analysis is called a lexical analyzer, a lexer, or a scanner.
- A scanner often exists as a single function which is called by the parser, whose functionality is to extract the next token from the source code.
- The lexical specification of a programming language is defined by a set of rules that define the scanner, which are understood by a lexical analyzer generator such as lex or flex. These are most often expressed as regular expressions.
- The lexical analyzer (either generated automatically by a tool like lex, or hand-crafted) reads the source code as a stream of characters, identifies the lexemes in the stream, categorizes them into tokens, and outputs a token stream.
- This is called "tokeninzing"
- If the scanner finds an invalid token, it will report a lexical error.

## Roles of the scanner
- Removal of comments
    - Comments are not part of the program's meaning
        - Multiple-line comments?
        - Nested comments?
- Case conversion
    - Is the lexical definition case sensitive?
        - For identifiers
        - For keywords
- Removal of white spaces
    - Blanks, tabulars, carriage returns
    - Is it possible to identify tokens in a program without spaces?
- Convert the input file to a token stream (tokenization)
    - Input file is a character stream
    - Lexical specifications: literals, operators, keywords, punctuation

### Lexical specifications: tokens and lexems
- Token: An element of the lexical definition of the language
- Lexeme: A sequence of characters identified as a token

| Token    | Lexeme                 |
|----------|------------------------|
| id       | distance,rate,time,a,x |
| relop    | >=,<,==                |
| openpar  | `(`                    |
| if       | if                     |
| then     | then                   |
| assignop | =                      |
| semi     | ;                      |

### Design of a lexical analyser
- Procedure
    1. Construct a set of regular expressions (REs) that define the form of any valid token
    2. Derive an NFA from the REs (e.g. Thompson's construction)
    3. Derive a DFA from the NFA (e.g Rabin-Scott powerset construction)
    4. Translate the DFA to a state transition table
    5. Implement the table
    6. Implement the algorithm to intepret the table
- This is exactly the procedure that a scanner generator is implementing.
- Scanner generators include:
    - Lex, flex
    - Jlex
    - Alex
    - Lexgen
    - re2c

## Regular expressions
- `ε`: represents the empty string (no characters)
- `s`: represents the set containing the string s itself
- `a`: represents the set containing the single character a
- `r | s`: means either r or s (union)
- `s*`: means zero or more repetitions of s
- `s⁺`: means one or more repetitions of s

```text
ε       : {}
s       : {s | s in s^}
a       : {a}
r | s   : {r | r in r^} or {s | s in s^}
s*      : {s^n | s in s^ and n >= 0}
s⁺      : {s^n | s in s^ and n >= 1}
id      ::= letter(letter|digit)*
```

## Thompson's construction
### REs to NDFA: Thompson's construction
- `id ::= letter(letter|digit)`
- It is used to convert a regular expression into a Non-deterministic Finite Automaton (NFA).
    - The algorithm recursively splits a regular expression into subexpressions.
    - Each subexpression is represented as a small NFA subgraph (N(s), N(t)), and these subgraphs are then connected depending on the operator:
        - s -> a single transition for an atomatic symbol
        - st -> concatenation: connect N(s)'s final state to N(t)'s start
        - s | t -> union: use ε-transitions (empty moves) to branch to either N(s) or N(t)
        - s* -> kleene star: allows zero or more repetitions using ε-loops to re-enter N(s) or skip it entirely
- This construciton gurantees an NFA that accepts exactly the same language as the original regular expression

### Implementation Concerns
- Backtracking
    - Principle: A token is normally recognized only when the next character is read
    - Problem: Maybe this character is part of the next token
    - Example:  `x < 1`, `<` is recognized only when `1` is read. In this case, we have to backtrack one character to continue token recognition without skipping the first character of the next token
    - Solution: include the occurrence of these cases in the state transition table.
- Ambiguity
    - Problem: Some tokens' lexemes are subsets of other tokens
    - Example:
        - `n-1`. Is it `<n><-><1>` or `<n><-1>`?
    - Solutions:
        - Postpone the decision to the syntactic analyzer
        - Do not allow sign prefix to numbers in the lexical specification
        - Interact with the syntactic analyzer to find a solution. (Includes coupling)

### Table-driven scanner - algorithm
```c
nextToken()
    state = 1
    token = null
    do
        lookup = nextChar()
        state = table(state, lookup)
        if (isFinalState(state))
            token = createToken(state)
            if (table(state, "backup") == yes)
                backupChar()
    until(token != null)
    return (token)
```
- `nextToken()`
    - Extract the next token in the program (called by syntactic analyzer)
- `nextChar()`
    - Read the next character in the input program
- `backupChar()`
    - Back up one character in the input file in case we have just read the next character in order to resolve an ambiguity
- `isFinalState(state)`
    - Returns TRUE if state is a final state
- `table(state, column)`
    - Returns the value corresponding to [state, column] in the state transition table
- `createToken(state)`
    - Creates and returns a structure that contains the token type, its location in the source code, and its value (for literals), for the token kind corresponding to a state, as found in the state transition table.

### Hand-written scanner
```c
nextToken()
    c = nextChar()
    case (c) of
        "[a..z][A..Z]":
            c = nextChar()
            while (c in [a..z],[A..Z],[0..9]) do
                s = makeUpString()
                c = nextChar()
            if (isReservedWord(s)) then
                token = createToken(RESWORD, null)
            else
                token = createToken(ID, s)
            backupChar()
        "[0..9]":
            c = nextChar()
            while (c in [0..9]) do
                v = makeUpValue()
                c = nextChar()
            token = createToken(NUM, v)
            backupChar()
        "{":
            c = nextChar()
            while (c != "}") do
                c = nextChar()
        "(":
            c = nextChar()
            if (c == "*") then
                c = nextChar()
                repeat
                    while (c != "*") do
                        c = nextChar()
                    c = nextChar()
                until (c != ")")
            else
                token = createToken(LPAR, null)
        ":":
            c = nextChar()
            if (c == "=") then
                token = createToken(ASSIGNOP, null)
            else
                token = createToken(COLON, NULL)
                backupChar()
        ...
```

## Possible Lexical Errors
- Depends on the accepted conventions:
    - Invalid character
    - Letter not allowed to terminate a number
    - numerical overflow
    - identifier too long
    - end of line before end of string
    - Are these lexical errors?

```
123 - <Error> or <num><id>
21234256312434333434334 - <Error> related to machine's limitations
"Hello <CR> world" - either <CR> is skipped or <Error>
ThisIsAVeryLongVariableNameThatIsMeantToConveyMeaning = 1
Limit idenfiifer length?
```

### Lexical error recovery techniques
- Finding only the first error and stopping the compilation is not acceptable
- Panic Mode:
    - Output an error and resume tokenization. For example: report/skip invalid characters until a valid character is read
- Guess Mode:
    - Do pattern matching between erroneous strings and valid strings. Example: (beggin vs begin)
    - Rarely implemented, leads to confusion

## Conclusions
### Possible implementations
- Lexical Analyzer Generator (e.g. Lex)
    - Safe, quick
    - Must learn software, often unable to handle unusual situations
- Table-driven lexical analyzer
    - General and adaptable method, same function can be used for all table-driven lexical analyzers
    - Building transition table can be tedious and error-prone - should use tools to generate the table from regular expressions or DFA
- Hand-written
    - Can be optimized, can handle any unusual situation, easy to build for most languages
    - Error-prone, not adaptable or maintainable

### Lexical analyzer's modularity
- Why should the Lexical Analyzer and the Syntactic Analyzer be separated?
    - Modularity/Maintainability: system is more modular, thus more maintainable
    - Efficiency: Modularity = task specialization = easier optimization
    - Reusability: Can change the whole lexical analyzer without changing other parts