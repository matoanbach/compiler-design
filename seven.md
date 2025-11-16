# Generating an Abstract Syntax Tree using Syntax-Directed Translation
## Abstract Syntax Tree: Definition
- AST is a tree representation of the abstract syntactic structure of source code.
- Each node of the tree denotes a syntactic construct occurring in the source code.
- The syntax is "abstract". It does not represent every detail appearing in the concrete syntax used in the source code, or some of the non-terminals in the grammar that do not directly represent syntactical/semantic constructs.
- Such details are removed because they do not convey any form of meaning and are thus superfluous for further processing.
- For instance:
    - punctuation such as commas, semicolons, and grouping parentheses removed
    - syntactic construct like an `if-then-else` may be denoted by means of a single node with three branches
    - non-terminals that were introduced as accessory to grammer transformations are removed.
- This distinguishes abstract syntax tree from concrete syntax trees, which are traditionally designated as parse trees.
- Once built, additional information is added to the AST by means of subsequent processing steps such as semantic analysis and code generation.

## Abstract Syntax Tree: Goals
- Goals:
    1. to aggregate information gathered during the parse in order to get a broader understanding of the meaning of whole syntatic constructs in a single subtree.
    2. to represent the entire program in a data structure that can later be repeatedly traversed for further analysis and translation steps.

- At the leaves of the tree is fine-grained syntactical concepts/information.
- Intermediate nodes represents higher-level constructs created by aggregation of the information conveyed by its branches' subtrees.
- The root node has direct access to all the information for an entire syntactical construct and its composing constructs.

## AST DATA STRUCTURE: requirements, design, implementation
### Data structure requirements
- The AST structure is constructed bottom-up:
    - A set of sibling nodes is generated and each is pushed on a sematic stack through the operation of a semantic action.
    - The elements are later popped from the semantic stack and adopted by a parent node, which is then pushed onto the stack through the operation of another semantic action.
- Some AST nodes require a fixed number of children, e.g:
    - Arithmetic operators
    - `if-then-else` statement
- Some AST nodes require zero or more number of children
    - Parameter lists, array dimension lists
    - Statements in a statement block
    - Members of a class
- In order to be generally applicable, an AST node data structure should allow for any number of children.

### Data structure requirements and design
- According to depth-first-search tree traversal
- Each node needs connetion to: 
    - `Parent`: to migrate information upwards in the tree
        - Link to parent
    - `Siblings`: to iterate through (1) a list of operands or (2) members of a group, e.g members of a class, or statement block.
        - Link to right siblings (thus creating a linked list of siblings)
        - Link to leftmost sibling (in case on needs to traverse the list as a sibling is being processed)
    - `Children`: to generate / traverse the tree
        - Link to leftmost child (most represents the head of the linked list of children, which are each other's siblings).

### Data structure implementation 
- A factory method that creates / returns a node whose members are adapted to the type of the parameters `t`. For example:
    - `makeNode(intNum i)`: instantiates a node that represents a numeric literal value. Offers a get method to get the value it represents.
    - `makeNode(id n)`: instantiates a node that represents an identifier. Offers get/set methods to get/set the symbol table entry it represents, which stores information such as its type/protection/scope.
    - `makeNode(composite c)`: instantiates a node that represents composite structures such as operators, statements, or blocks. There should be one for each such possible different nodes for each different kind of composite structures in the language. Each offers get/set methods appropriate to what they represent.
    - `makeNode()`: instantiates a null node in order to represent, e.g. the end of sibling list.
    - `makeFamily(op, kid1, kid2, ...m kidn)`: generates a family with n children under a parent `op`.
        - One such function exists to create each kind of sub-tree, or one single variadic function
        - Some programming languages do not allow variadic functions.

### Data structure implementation (example)
- `x.makeSiblings(y)`: inserts a new sibling node `y` in the list of siblings of node `x`.
- `x.adoptChildren(y)`: adopts node `y` and all its siblings under the parent `x`.

### Insert semantic actions in the grammar / parser
- Example grammar with semantic actions added.
- AST leaf nodes are created when the parse reaches leaves in the parse tree (23, 24) (`makeNode`)
- Sibling lists are constructred as lists are processed inside a structure (19) (`makeSiblings`)
- Subtrees are crated when an entire structure has been parsed (14, 15, 16, 17, 18, 21) (`makeFamily`)
- Some sematic actions are only migrating information across the tree (20, 22)

```txt
1 Start -> Stmt $
    return (ast)
    # the program root is the single statement/block AST built.
2 Stmt -> id assign E
        result <- makeFamily(assign, var, expr)
        # Create an assign node; left child is the id leaf, right child is the expression subtree.
    3 if lparen Ep rparen Stmt fi
        result <- makeFamily(if, p, s, makeNode())
        # build an if with condition p, then-stmt s, and empty else (represented by a NIL node)
    4 if lparen Ep rparen Stmt1 else Stmt2 fi
        result <- makeFamily(if, p, s1, s2)
        # Same, but with an else subtree
    5 while lparen Ep rparan do Stmt od
        result <- makeFamily(while, p, s)
    6 begin Stmts end
        result <- makeFamily(block, list)
        # Wrap the siblings list into a block
7 Stmts -> Stmts_sofar semi Stmt next
    result <- sofar.makeSibling(next)
    # append the newly parsed Stmt to the existing list
    8 Stmt_first
        result <- first
        # start the list with a single statement (no new node)
9 E_result -> E_e1 plus T_e2
    result <- makeFamily(plus, e1, e2)
    # build binary operator nodes as you reduce
    10 E->T
        result <- e
        # migrate
11 T_result -> id_var
    result <- makeNode(var)
    12 t -> num_val
        result <- makeNode(var)
```

### AST Generation using Syntax-Directed Translation
- A language's `Semantic Concept` is a building block of the meaning of a program
    - literal value, variable, function definition, class and/or data structure, statement, expression, etc.
- In an AST, each concept is represented by a `node` and possibly a `subtree`.
- Atomic concepts (`Ca`) are represented by `AST leaf nodes (CaN)`
    - Literal value, identifier, etc.
- Composite concepts (`Cc`) represent higher-level concepts that aggregate n subordinate concepts (`Cs`)
- Composite concepts are represented by an `AST subtree (CcN)` with n AST subtree as children.
    - class, function definition, statement, expression, etc.