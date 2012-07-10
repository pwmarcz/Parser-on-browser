==============================
Parser on browser - code guide
==============================

This is a high-level description of the Javascript code in ``eparse.js``.

Operators
=========

The user supplies an operator table (see ``eparse.loadOps(code)``). For each operator, we define *associativity* as left or right (``2+2+2`` is ``(2+2)+2`` with left associativity, but ``2+(2+2)`` with right associativity), and *priority* (e.g. ``*`` should have higher priority than ``+``).

Lexer
=====

We first process the input with lexer (also called tokenizer). The input string (for example, ``2 + 2``) is split into objects called *tokens*: ``lparen()``, ``number(2)``, ``op(+)``, ``number(2)``, ``rparen()``. The tokens are implemented as JavaScript objects with ``type`` and ``val`` keys, e.g. ``{type: 'number', val: 2}``.

The lexer is in the ``eparse.tokenize(str)`` function. It tries to read a number, then one of the operators, then left/right paren.

Parser
======

The parser is a `recursive descent parser <http://en.wikipedia.org/wiki/Recursive_descent_parser>`_. Each function tries to consume a constituent from the input, and returns when it has read a full one.

We have three types of constituents:

- ``operator()``,
- ``simpleExpr()``, which is either a number (e.g. ``1``) or parenthesized expression (e.g. ``(2+3)``),
- ``expr(priority)``, which is a complex expression with operators (e.g. ``1+(2+3)+4``).

The ``priority`` parameter allows us handle operators of different priority -- if we encounter an operator with a low priority, we need to exit and let the outer function handle it.

Example
=======

An example run of the algorithm for ``2*3+4``:

::

    tokenize('2*3+4'):
        return [numer(2), op(*), number(3), op(+), number(4)]

    expr(0):
        simpleExpr() reads number(2)
        operator() reads op(*), operator priority = 2
        expr(2):
            simpleExpr() reads number(3)
            operator() reads op(+), operator priority = 1
            priority is lower than 2, we un-read the operator and exit
            return '3'
        operator() reads op(+), operator priority = 1
        expr(1) reads number(4)
        return '(2*3)+4'
