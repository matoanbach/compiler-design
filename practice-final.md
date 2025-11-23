## Question 1: Describe the advantage of using an explicit semantic stack in a top-down predictive parser rather than relying solely on the function call stack. How does an explicit stack help debug and manage attribute migration?

- Function-call stack:
    - Hidden: it's manage/runtime, not by you
    - About control flow, not about data. It remembers which function called which, but not how your AST pieces should be combined in a clean, visible way.
    - Harder to inspect: you need a debugger and it's messy to see what values belong to which part of the parse
    - You'd pass tons of parameters up and down (left child, right child, types, etc.).
    - You'd need many local variables in each funtion to temporarily store parts of expressions.
    - It quickly becomes confusing to track "who owns which piece" of the tree.

- Semantic stack:
    - a single, visible place for tree pieces
    - the stack always contains "the partial results we've build so far", which makes it very clear where information lives while parsing.
    - we can print the stack at any point, and then see exactly which nodes are there, in which order.
    - With function-call stack, this is much harder:
        - You'd have to inspect locals in multiple nested functions
        - You have no single snapshot of "all semantic pieces we've accumulated so far."

- Summary:
    - An explicit semantic stack gives you a clear, debuggable, central place to store and combine the pieces of your syntax tree and other information, instead of hiding them across nested function calls - making it much easier to see what's going wrong and to control how information moves around during parsing.

## Question 2. Describe how semantic actions differ in a recursive descent predictive parser versus a table-driven predictive parser. How would you implement semantic actions for a function call in each type of parser?

### What's different?
- Recursive-descent predictive parse
    - Your grammar rules are literally functions (E(), T(), FACTOR(), ...)
    - Semantic actions = normal code you write inside those functions.
    - You can call helpers, push on the semantic stack, build AST nodes, etc. whenever you want.
- Table-driven predictive parse
    - There's one generic parse loop driven by a parse table + stacks.
    - It doesn't call grammar functions, it just:
        - look at the table
        - replaces symbols on the stack with the right-hand side of a rule
    - Semantic actions must be attached to productions, not to functions.
        - Typically: when you appply a production, you also call a semantic routine and manipulate a separate semantic stack.
- So recursive-descent, actions live in the code
- table-driven, actions live next to the grammar rules and are triggered by the generic loop.

### Implement semantic actions for a function call
- Recursive-descent parser
    - Think of your parser as a bunch of small functions, one per `kind of phrase` in the language (expression, term, factor, etc.).
    - Inside each of these functions, you:
        - read the next tokens
        - and immediately run the extra code that builds your tree / AST nodes.
    - For a function call like f(a, b):
        - the function that handles `factor` (FACTOR())
            - remembers the function's name (f)
            - calls another routine to read the arguments (a, b).
            - then creates a `function-call node` and puts it on the semantic stack

- Table-driven parser
    - Here you mostly have one main loop that uses a table to decide what to do next.
    - This loop itself doesn't know about `factor` or `expression` directly; it just follows the table.
    - You extra code (semantic actions) is attached to rows of that table.
    - For a function call:
        - When the main loop sees the pattern `id '(' ARGS ')'` like `f(a, b)`.
        - it takes the function name off the semantic stack
        - creates a new `function-call node` from them,
        - and pushes that new node back on the semantic stack.

## Question 3.Consider a language that allows variable shadowing (where an inner scope can redefine a variable with the same name as one in an outer scope). Describe how a symbol table visitor should handle this situation, and explain the potential issues if this behavior is not managed correctly.

### To handle shadowing safely, it should:
1. When entering a new region (inner block / function / class)
    - Create a new symbol table for that region.
    - Remember who the `parent` table is (the region outside).
2. When it sees a variable declaration in that region
    - It inserts a new entry with that name into the current table.
    - If that name is already in the current table, that's a real error: you're declaring the same name twice in the same place.
    - If the name is only in a parent table, that's okay: this is shadowing. The new entry just temporarily hides the outer one.
3. When it later sees a use of a name
    - It first looks in the current table
    - If it finds x there, that's the one it uses
    - If notm it moves up to the parent table, then the parent of that, and so on.
4. When leaving the inner region
    - It goes back to the parent table
    - The inner table can be thrown away or just not used anymore.

### What goes wrong if we don't handle shadowing correctly?
1. Using the wrong variable
    - If you ignore shadowing and always use the outer x, then inside the block you think you're talking about the inner x but the checker uses the outer one.
    - This can cause wrong type checks (e.g, outer x is a float and inner x is an int) and wrong code generation.

2. `Variable not declared` errors when it is declared
    - If you search function only looks in the gloval table, or you forget to link child tables to their parent, you might not find the inner x at all and falsely say `x is not declared`.

3. False `redeclaration` errors
    - If you treat `same name in inner region` as always illegal, you'll reject prefectly valid programs that rely on shadowing.

4. Using dead variables after leaving a region
    - If you don't switch back to the parent table after leaving the block, you might still think the inner x exists when it shouldn't, and type checking / code generation will be wrong.