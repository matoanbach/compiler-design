# Introduction to code generation
- Front end:
    - Lexical Analysis
    - Syntatic Analysis
    - Intermediate Code/Representation Generation
    - Intermediate Code/Representation Optimization
    - Sematic Analysis

- Back end:
    - Object Code Generation
    - Object Code Optimization

- The front end is `machine-independent`, i.e. the decisions made in its processing do not need on the target machine on which the translated program will be executed.
- A well-designed front end can be reused to build compilers for different target machines.
- The back end is `machine-dependenet`, i.e. these steps are related to the nature of the assembly or machine language of the target architecture.

# Intermediate representation
- `Intermediate representations` synthetize the syntatic information gathered during the parse, generally in the form of a tree or directed graph.
- Intermediate representations enable high-level code optimization.
- In our project, we use an `Abstract Syntax Tree (AST)` as an intermediate representation.
- `Intermediate code` is a low-level coded (text) representation of the program, directly translatable to object code.
- Intermediate code enables low-level, architecture-dependent optimizations.
- We don't have intermediate code in our project.

## Abstract Syntax Tree
- Each node represents the application of a rule in the grammar.
- A subtree is created only after the complete parsing of a right-hand side.
- References to subtrees are sent up and grafted as upper subtrees are completed.
- `Parse trees` (concrete syntax trees) emphasize the grammatical structure of the program, based on the exact concrete syntax of the grammar.
- `Abstract Syntax Trees` emphasize the actual computations to be performed. They do not refer to the actual non-terminals defined in the grammar, nor to tokens that play no role in defining the translated program, hence their name.
- When implementing an LL parser, there is generally substantial differences between the parse tree and the corresponding abstract syntax tree.
- A parse tree contains grammar-specific nodes, e.g. non-terminal symbols, punctuation, etc. These are not neccessary for further processing. Abstract syntax trees do not include them, hence the name abtract.

## Directed acyclic graph
- It is a relative of syntax trees: they are used to show the syntax structure of valid programs in the form of a graph with the following restrictions:
    - Edges between nodes can only `unidirectional`
    - There cannot be `cycles` in a DAG, i.e. no path can ever lead twice to the same node.
    - Presence of a `root node`
- For example, in a DAG representation, the nodes for repeated variables and expressions are merged into a single node.
- DAGs are more complex to build and use than syntax trees, but easily allow the implementation of a variety of `optimization` techniques by avoiding redundant operations.

# Postfix notation

# Three-address code
-  TAC or 3AC is an intermediate language that maps directly to `assembly pseudo-code`, i.e. architecture-independent assembly code.
- It breaks the program into short uniform statements requiring no more than three variables (hence its name) and no more than one operator.
- As it is an intermediate (abstract) language, its `addresses` represent symbolic addresses (i.e. variables), as opposed to either registers or memory addresses that would be used bu the target machine code.
- These characteristics allows 3AC to:
    - be more abstract than assembly language, enabling optimizations at the higher abstract level.
    - have high resemblance to assembly language, enabling easy translation to assembly language.

# Semantic actions
- It is about giving a meaning to the compiled program.
- it has two counterparts:
    - `Semantic Checking`: check if the compiled program can have a meaning, e.g identifiers are declared and properly used, operatios and functions have the right parameter types and number of parameters upon calling.
    - `Semantic Translate`: translate declarations, expressions, statements and functions to target code.
- Semantic translation is conditional to semantic checking, i.e. if a program is semantically invalid, the generated code would be invalid.
- There is no such thing as code generation errors detection/reporting.
- Semantic actions are associated with:
    - Declarations:
        - variable declarations
        - type declarations
        - function declarations
    - Control structures
        - conditional statements
        - loop statements
        - function calls
    - Assignments and expressions:
        - assignment operations
        - arithmetic and logical expressions
    
- This applies to strongly typed procedural programming.
- Other programming language paradigms often have to be analyzed/translated differently, either at compile time and/or run time.

