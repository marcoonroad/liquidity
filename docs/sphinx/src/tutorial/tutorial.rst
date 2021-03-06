
Tutorial for Liquidity
======================

TODO


Tuples
------

Tuples in Liquidity are compiled to pairs in Michelson::

 (x, y, z) <=> Pair x (Pair y z)

Tuples can be accessed using the field access notation of Liquidity::

 let t = (x,y,z) in
 let should_be_true = t.(2) = z in


A new tuple can be created from another one using the field access update
notation of Liquidity::

 let t = (1,2,3) in
 let z = t.(2) <- 4 in

Tuples can be deconstructed::

 (* t : (int * (bool * nat) * int) *)
 let _, (b, _), i = t in
 ...
 (* b : bool
    i : int *)


Records
-------

Record types can be declared and used inside a liquidity contract::

 type storage = {
   x : string;
   y : int;
 }

Such types can be created and used inside programs::

 let r = { x = "foo"; y = 3 } in
 r.x

Records are compiled as tuples.

Deep record creation is possible using the notation::

 let r1 = { x = 1; y = { z = 3 } } in
 let r2 = r1.y.z <- 4 in
 ...

Variants
--------

Variants should be defined before use, before the contract
declaration::

 type t =
 | X
 | Y of int
 | Z of string * nat

Variants can be created using::

 let x = X 3 in
 let y = Z s in
 ...

The ``match`` construct can be used to pattern-match on them, but only
on the first constructor::

 match x with
 | X -> ...
 | Y i -> ...
 | Z s -> ...

where ``i`` and ``s`` are variables that are bound by the construct to the
parameter of the variant.

Parameters of variants can also be deconstructed when they are tuples,
so one can write::

 match x with
 | X -> ...
 | Y i -> ...
 | Z (s, n) -> ...



A special case of variants is the ``Left | Right`` predefined variant,
called ``variant``::

 type (``left, ``right) variant =
 | Left of ``left
 | Right of ``right


All occurrences of these variants should be constrained with type
annotations::

 let x = (Left 3 : (int, string) variant) in
 match x with
 | Left left  -> ...
 | Right right -> ...

Another special variant is the ``Source`` variant: it is used to refer to
the contract that called the current contract::

 let s = (Source : (unit, unit) contract) in
 ...

As for ``Left`` and ``Right``, ``Source`` occurrences should be constrained by
type annotations.

Functions and Closures
----------------------

Unlike Michelson, functions in Liquidity can also be closures. They can take
multiple arguments and are curryfied. Because closures are lambda-lifted, it is
however recommended to use a single tuple argument when possible.  Arguments
must be annotated with their (monomorphic) type, while the return type
is inferred.

Function applications are often done using the ``Lambda.pipe`` function
or the ``|>`` operator::

  let succ = fun (x : int) -> x + 1 in
  let one = 0 |> succ in
  ...

but they can also be done directly::

  ...
  let succ (x : int) = x + 1 in
  let one = succ 0 in
  ...

A toplevel function can also be defined before the main entry point::

 [%%version 0.2]
 
 let succ (x : int) = x + 1
 
 let%entry main ... =
   ...
   let one = succ 0 in
   ...

Closures can be created with the same syntax::

 let p = 10 in
 let sum_and_add_p (x : int) (y : int) = x + y + p in
 let r = add_p 3 4 in
 ...

This is equivalent to::

 let p = 10 in
 let sum_and_add_p =
   fun (x : int) ->
     fun (y : int) ->
       x + y + p
 in
 let r = 4 |> (3 |> add_p) in
 ...


Functions with multiple arguments should take a tuple as argument because
curried versions will generate larger code and should be avoided
unless partial application is important. The previous function should
be written as::

 let sum_and_add_p ((x : int), (y : int)) =
   let p = 10 in
   x + y + p
 in
 let r = add_p (3, 4) in
 ...


Loops
-----

Loops in liquidity share some syntax with functions, but the body of
the loop is not a function, so it can access the environment, as would
a closure do::

 let end_loop = 5 in
 let x = Loop.loop (fun x ->
     ...
     (x < end_loop, x')
   ) x_init
 in
 ...

As shown in this example, the body of the loop returns a pair, whose first
part is the condition to remain in the loop, and the second part is the
accumulator.

