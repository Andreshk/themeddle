---
category: Functional Programming
tags:
  - y
  - recursion
  - combinators
  - anonymous functions
  - elixir
  - erlang
---

# Defining Recursion

This is the English version of an article, I wrote for [elixir-lang.bg](https://elixir-lang.bg/posts).
You can find it in [Bulgarian on this blog too](http://themeddle.com/bg/posts/y).

It is an interesting exercise - we are going to use a very limited subset of a language.
A subset which doesn't support recursion and will try to define a recursive function.
By language I mean a few languages. I added interesting new feature to my blog engine's front-end [BlogtWeb](https://github.com/meddle0x53/blogit_web),
which allows me to write examples in more than one programming language and give the reader the feeedom to select the one he/she likes.

This article will use [elixir]() {: .multi-code} and [erlang]() {:.multi-code}.
Let's begin.

## Prerequisites

For our language of choice, we are going to use only its interpreter and won't use named functions.
We will forget that [elixir]() {: .multi-code} and [erlang]() {:.multi-code} support named functions.

So let's run our interpreter:

```elixir
iex
```
```erlang
erl
```
{: .multi-code}

Now we are going to limit ourselves by using only basic types:

```elixir
23
24.5
"Some String"
["a", "list"]
{"a", "tupple"}
....
```

```erlang
23.
24.5.
"Some String".
["a", "list"].
{"a", "tupple"}.
....
```
{: .multi-code}

In fact we are going to try to define and use a [Factorial function](https://en.wikipedia.org/wiki/Factorial).
So we are going to use just integer numbers and arithmetic operations with them.
I assume we all know how to subtract and multiply numbers in our language of choice. Let's see how we can
define a simple anonymous function. Let's define an `add` function, which adds its two arguments and returns the result:

```elixir
fn (a, b) -> a + b end
```
```erlang
fun (A, B) -> A + B end.
```
{: .multi-code}

Now we can call this function like this:
```elixir
(fn (a, b) -> a + b end).(4, 5)
# 9
```
```erlang
(fun (A, B) -> A + B end)(4, 5).
% 9
```
{: .multi-code}

Or assign it to variable and call it:

```elixir
add = fn (a, b) -> a + b end
add.(9, 10)
# 19
```
```erlang
Add = fun (A, B) -> A + B end.
Add(9, 10).
% 19
```
{: .multi-code}

Good. Now we know everything we need to know in order to define the Factorial function! Or do we?

## Factorial - The Beginning

With our current knowledge we can define a function and assign it to a variable.
If we had factorial defined we would expect from it to behave like this:

```elixir
factorial.(0)
# 1

factorial.(1)
# 1 * factorial.(0) => 1

factorial.(2)
# 2 * factorial.(1) => 2

..........

factorial.(n)
# n * factorial.(n - 1) => n * n-1 * n-2 ... 1
```
```erlang
Factorial(0).
% 1

Factorial(1).
% 1 * Factorial(0) => 1

Factorial(2).
% 2 * Factorial(1) => 2

..........

Factorial(N).
% N * Factorial(N - 1) => N * N-1 * N-2 ... 1
```
{: .multi-code}

That's wonderful! Now we can just define the function and we can finish reading this post!
Let's define it:

```elixir
iex> factorial = fn
...>   0 -> 1
...>   n -> n * factorial.(n - 1)
...> end
  ** (CompileError) iex:4: undefined function factorial/0
```
```erlang
> Factorial = fun
>   (0) -> 1;
>   (N) -> N * Factorial(N - 1)
> end.
* 3: variable 'Factorial' is unbound
```
{: .multi-code}

Since [OTP 17](http://www.erlang.org/downloads/17.0) for [erlang]() {: .multi-code} it is possible to define a name of the anonymous function
and it will work. But we will forget about the existence of this syntax in this article.

What's happening in the example above is normal. We are defining *factorial* and assigning it to a variable,
but this variable doesn't exist in the body of the function we define and we get an error.

But it is possible to write the factorial function in a way we can pass it to itself as a variable, so the variable exists in its definition:

```elixir
iex> factorial = fn (me) ->
...>   fn
...>     0 -> 1
...>     n -> n * me.(n - 1)
...>   end
...> end
#Function<6.52032458/1 in :erl_eval.expr/5>
```
```erlang
> Factorial = fun (Me) ->
>   fun
>     (0) -> 1;
>     (N) -> N * Me(N - 1)
>   end
> end.
#Fun<erl_eval.6.99386804>
```
{: .multi-code}

That's working. No errors. But what should we pass as its `me` `Me`*multi-code* argument?
The one thing that comes to mind is to pass `factorial` `Factorial`*multi-code* to itself:

```elixir
iex> factorial.(factorial)
#Function<6.52032458/1 in :erl_eval.expr/5>
```
```erlang
> Factorial(Factorial).
#Fun<erl_eval.6.99386804>
```
{: .multi-code}

And that works too! Maybe this function implements factorial? It seems like that.
Let's try it out:

```elixir
iex> factorial.(factorial).(0)
1
iex> factorial.(factorial).(1)
** (ArithmeticError) bad argument in arithmetic expression
```
```erlang
> (Factorial(Factorial))(0).
1
> (Factorial(Factorial))(1).
** exception error: an error occurred when evaluating an arithmetic expression
     in operator  */2
             called as 1 * #Fun<erl_eval.6.99386804>
```
{: .multi-code}

OK, it works only when we pass `0`. Why? I think the easiest way to understand what is happening is
to write the actual function we get when we call our `factorial` `Factorial`*multi-code* function with itself as an argument:

```elixir
(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(n - 1)
  end
end).(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(n - 1)
  end
end)

=>

factorial_factorial = fn
  0 -> 1
  n -> n * (fn (me) ->
    fn
      0 -> 1
      n -> n * me.(n - 1)
    end
  end).(n - 1)
end

=>

factorial_factorial.(0) # It is simple -> the 0 -> 1 is evaluated and it is success:
1

factorial_factorial.(1) # Here we hit the n -> n * me.(n - 1) case

=>

1 * (fn (me) ->
      fn
        0 -> 1
        n -> n * me.(n - 1)
      end
    end).(0)

=>

1 * (fn
      0 -> 1
      n -> n * 0.(n - 1)
    end)

# This one can't be reduced any more so we get
# a number multiplied by an anonymous function - ArithmeticError
```

```erlang
(fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * Me(N - 1)
  end
end)(fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * Me(N - 1)
  end
end).

=>

FactorialFactorial = fun
  (0) -> 1;
  (N) -> N * (fun (Me) ->
    fun
      (0) -> 1;
      (N) -> N * Me(N - 1)
    end
  end)(N - 1)
end.

=>

FactorialFactorial(0). % It is simple -> the (0) -> 1 is evaluated and it is success:
1

FactorialFactorial(1). % Here we hit the (N) -> N * Me(N - 1) case

=>

1 * (fun (Me) ->
      fun
        (0) -> 1;
        (N) -> N * Me(N - 1)
      end
    end)(0).

=>

1 * (fun
      (0) -> 1;
      (N) -> N * 0(N - 1)
    end).

% This one can't be reduced any more so we get
% a number multiplied by an anonymous function - ArithmeticError
```
{: .multi-code}

That's good. We now know why it worked when we passed `0` to it and why it didn't when we passed `1`.
Now that we know what's the problem, we can start thinking about how we could solve it.

## Factorial II - Forever

OK, how should we augment the `factorial` `Factorial`*multi-code* function, we have in the moment, so it could work when it is passed `1` as argument?
Let's isolate only this case. When we pass `0`, it works. Let's try making it work with `1` and not think about all the other infinite to count cases.

It is time to return to the reduction, we called `factorial_factorial` `FactorialFactorial`*multi-code*:

```elixir
factorial_factorial = fn
  0 -> 1
  n -> n * (fn (me) ->
    fn
      0 -> 1
      n -> n * me.(n - 1)
    end
  end).(n - 1)
end

# And when we pass 1, we have this case:

1 * (fn (me) ->
      fn
        0 -> 1
        n -> n * me.(n - 1)
      end
    end).(0)

```
```erlang
FactorialFactorial = fun
  (0) -> 1;
  (N) -> N * (fun (Me) ->
    fun
      (0) -> 1;
      (N) -> N * Me(N - 1)
    end
  end)(N - 1)
end.

1 * (fun (Me) ->
      fun
        (0) -> 1;
        (N) -> N * Me(N - 1)
      end
    end)(0).
```
{: .multi-code}

This looks like `1 * factorial.(0)` `1 * Factorial(0).`*multi-code*. And we know that `factorial.(0)` `Factorial(0).`*multi-code* returns a function.
The case working with `0` is actually `factorial.(factorial).(0)` `(Factorial(Factorial))(0).`*multi-code*.
So if we think of a way to get `1 * factorial.(factorial).(0)` `1 * (Factorial(Factorial))(0).`*multi-code* instead of `1 * factorial.(0)` `Factorial(0).`*multi-code* in the
above code, the `1` case would work.

Let's look at how we defined the factorial function:

```elixir
factorial = fn (me) ->
  fn
    0 -> 1
    n -> n * me.(n - 1)
  end
end
```
```erlang
Factorial = fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * Me(N - 1)
  end
end.
```
{: .multi-code}

When we try to compute factorial of `1`, the problem is that `me.(0)` `Me(0).`*multi-code* returns a function and not
the factorial of `0`. And we thought that `factorial.(factorial).(0)` `(Factorial(Factorial))(0).`*multi-code*, which returns the factorial of `0`
will solve the problem.

This means that if we replace `me.(n - 1)` `Me(N - 1).`*multi-code* with `me.(me).(n - 1)` `(Me(Me))(N - 1).`*multi-code* when we call `factorial.(factorial).(1)` `(Factorial(Factorial))(1).`*multi-code*,
we will get `1 * factorial.(factorial).(0)` `1 * (Factorial(Factorial))(0).`*multi-code* which is `1 * 0!` or `1!`.
Let's try it:

```elixir
factorial = fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end

factorial.(factorial).(0) # 0!
# 1
factorial.(factorial).(1) # 1!
# 1
```
```erlang
Factorial = fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end.

(Factorial(Factorial))(0). % 0!
% 1
(Factorial(Factorial))(1). % 1!
% 1
```
{: .multi-code}

And that's a success. It works when we pass `1`! So now we can think of a way to implement `n!`.
Let's try with `2!`, see the possible error and analyse how we got it in the same way we did it with
`factorial.(factorial).(1)` `(Factorial(Factorial))(1).`*multi-code*:

```elixir
factorial.(factorial).(2)
# 2
# Hmm... That's like 2 * 1 * 0! - 2!

factorial.(factorial).(3)
# 6 or 3!
factorial.(factorial).(5)
# 120 or 5!
factorial.(factorial).(10)
# 3628800 or 10!
```
```erlang
(Factorial(Factorial))(2).
% 2
% Hmm... That's like 2 * 1 * 0! - 2!

(Factorial(Factorial))(3).
% 6 or 3!
(Factorial(Factorial))(5).
% 120 or 5!
(Factorial(Factorial))(10).
% 3628800 or 10!
```
{: .multi-code}

That's a working factorial and we wrote it only using anonymous functions.
That's recursion we implemented ourselves. But how we achieved that?
Actually we can write it without using variables at all, like this:

```elixir
(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end).(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end).(5)
# 120
```
```erlang
((fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end)(fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end))(5).
% 120
```
{: .multi-code}

But how is this working?

## Factorial III - Infinity

Let's see what's happening when we invoke `factorial.(factorial).(3)` `(Factorial(Factorial))(3).`*multi-code*, for example:

```elixir
(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end).(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end).(3)

=>

(fn
  0 -> 1
  n -> n * (fn (me) ->
              fn
                0 -> 1
                n -> n * me.(me).(n - 1)
              end
            end).(fn (me) ->
              fn
                0 -> 1
                n -> n * me.(me).(n - 1)
              end
            end).(n - 1)
end).(3)

=>

3 * (fn (me) ->
      fn
        0 -> 1
        n -> n * me.(me).(n - 1)
      end
    end).(fn (me) ->
      fn
        0 -> 1
        n -> n * me.(me).(n - 1)
      end
    end).(2)

=>

3 * (fn
      0 -> 1
      n -> n * (fn (me) ->
                  fn
                    0 -> 1
                    n -> n * me.(me).(n - 1)
                  end
                  end).(fn (me) ->
                    fn
                      0 -> 1
                      n -> n * me.(me).(n - 1)
                    end
                  end).(n - 1)
      end).(2)

# Notice that this is the same function, we invoke with 3 from above.
# If we call it 'f', this is 3 * f.(2)

=>

3 * 2 * (fn
          0 -> 1
          n -> n * (fn (me) ->
                      fn
                        0 -> 1
                        n -> n * me.(me).(n - 1)
                      end
                      end).(fn (me) ->
                        fn
                          0 -> 1
                          n -> n * me.(me).(n - 1)
                        end
                      end).(n - 1)
        end).(1)
# 3 * 2 * f.(1)

=>

3 * 2 * 1 * (fn
              0 -> 1
              n -> n * (fn (me) ->
                      fn
                        0 -> 1
                        n -> n * me.(me).(n - 1)
                      end
                      end).(fn (me) ->
                        fn
                          0 -> 1
                          n -> n * me.(me).(n - 1)
                        end
                      end).(n - 1)
            end).(0)

# 3 * 2 * 1 * f.(0), but here 0 matches 0 -> 1 and we get:

=>

3 * 2 * 1 * 1 = 6 # 3!
```
```erlang
((fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end)(fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end))(3).

=>

(fun
  (0) -> 1;
  (N) -> N * ((fun (Me) ->
              fun
                (0) -> 1;
                (N) -> N * (Me(Me))(N - 1)
              end
            end)(fun (Me) ->
              fun
                (0) -> 1;
                (N) -> N * (Me(Me))(N - 1)
              end
            end))(N - 1)
end)(3).

=>

3 * ((fun (Me) ->
       fun
         (0) -> 1;
         (N) -> N * (Me(Me))(N - 1)
       end
     end)(fun (Me) ->
       fun
         (0) -> 1;
         (N) -> N * (Me(Me))(N - 1)
       end
     end))(2).

=>

3 * (fun
       (0) -> 1;
       (N) -> N * ((fun (Me) ->
                   fun
                     (0) -> 1;
                     (N) -> N * (Me(Me))(N - 1)
                   end
                 end)(fun (Me) ->
                   fun
                     (0) -> 1;
                     (N) -> N * (Me(Me))(N - 1)
                   end
                 end))(N - 1)
     end)(2).

% Notice that this is the same function, we invoke with 3 from above.
% If we call it 'F', this is 3 * F(2).

=>

3 * 2 * (fun
          (0) -> 1;
          (N) -> N * ((fun (Me) ->
                      fun
                        (0) -> 1;
                        (N) -> N * (Me(Me))(N - 1)
                      end
                    end)(fun (Me) ->
                      fun
                        (0) -> 1;
                        (N) -> N * (Me(Me))(N - 1)
                      end
                    end))(N - 1)
        end)(1).

% 3 * 2 * F(1).

=>

3 * 2 * 1 * (fun
               (0) -> 1;
               (N) -> N * ((fun (Me) ->
                           fun
                             (0) -> 1;
                             (N) -> N * (Me(Me))(N - 1)
                           end
                         end)(fun (Me) ->
                           fun
                             (0) -> 1;
                             (N) -> N * (Me(Me))(N - 1)
                           end
                         end))(N - 1)
             end)(0).

% 3 * 2 * 1 * F(0).
% But here 0 matches (0) -> 1 and we get:

=>

3 * 2 * 1 * 1 = 6. % 3!
```
{: .multi-code}

In other words we have a repeating function, which behaves like factorial.
And let's look at the function we called `factorial` `Factorial`*multi-code* again:

```elixir
factorial = fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end
```
```erlang
Factorial = fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end.
```
{: .multi-code}

The name is not right for this function, right? It is a higher-order function which takes itself to produce the factorial function.
This is the real definition of the factorial function, using only anonymous functions:

```elixir
factorial = (fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end).(fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end)

iex> factorial.(5)
120
```
```erlang
Factorial = (fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end)(fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end).

> Factorial(5).
120
```
{: .multi-code}

And with this definition, we did what we wanted to do.

But let's play a bit more. Let's generalise our recursive function in a way it
could implement another one-argument recursive function. For example, the function which computes the Nth
[Fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number).
We can do it like this:

```elixir
fib = (fn (f) ->
  fn
    0 -> 1
    1 -> 2
    n -> f.(f).(n - 1) + f.(f).(n - 2)
  end
end).(fn (f) ->
  fn
    0 -> 1
    1 -> 2
    n -> f.(f).(n - 1) + f.(f).(n - 2)
  end
end)

iex> fib.(3)
5
```
```erlang
Fib = (fun (F) ->
  fun
    (0) -> 1;
    (1) -> 2;
    (N) -> (F(F))(N - 1) + (F(F))(N - 2)
  end
end)(fun (F) ->
  fun
    (0) -> 1;
    (1) -> 2;
    (N) -> (F(F))(N - 1) + (F(F))(N - 2)
  end
end).

> Fib(3).
5
```
{: .multi-code}

## Factorial IV - Generalisations

Abstraction is a very important concept in programming.
* It is good to try to not repeat code if it is possible.
* It is great to reuse as much code as we can.

We are going to create a function, which will help us define arbitrary recursive functions of one argument.
Let's start with two definitions:

1. A function, which has only bound variables is called *combinator*.
These are functions which depend only on variables, declared as their arguments or in their bodies:

```elixir
id = fn (x) -> x end # This is a combinator - The I combinator or the identity combinator

y = 5
fn (x) -> x + y end
# This is not a combinator.
# It depends on some variable which is not declared in its body or as its arguments.
```
```erlang
I = fun (X) -> X end. % This is a combinator - The I combinator or the identity combinator

Y = 5.
fun (X) -> X + Y end.
% This is not a combinator.
% It depends on some variable which is not declared in its body or as its arguments.
```
{: .multi-code}

2. The *Omega combinator* or the looping combinator is defined like this:
```elixir
o = fn (f) -> f.(f) end

o.(id) == id
# true
o.(id) == id.(id)
# true
```
```erlang
O = fun (F) -> F(F) end.

O(I) == I.
% true
O(I) == I(I).
% true
```
{: .multi-code}

And now we can return to our definition of the factorial function and try to reuse a piece of code repeated twice:

```elixir
factorial = (fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end).(fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end)

=>

factorial = fn (g) ->
  g.(g)
end.(fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end)
```
```erlang
Factorial = (fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end)(fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end).

=>

Factorial = (fun (G) ->
  G(G)
end)(fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end).
```
{: .multi-code}

Yes, we can define the factorial function as just an invocation of the *proto-factorial* function with
argument - itself. In other words passing the *proto-factorial function* to the *Omega combinator*:

```elixir
factorial = o.(fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end)
```
```erlang
Factorial = O(fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end).
```
{: .multi-code}

That's easier to read.
Now, let's extract the logic, specific for the factorial function into a separate function:

```elixir
factorial = o.(fn (f) ->
  proto_factorial = fn g ->
    fn
      0 -> 1
      n -> n * g.(n - 1)
    end
  end

  proto_factorial.(fn (x) -> f.(f).(x) end)
end)
```
```erlang
Factorial = O(fun (F) ->
  ProtoFactorial = fun (G) ->
    fun
      (0) -> 1;
      (N) -> N * (G)(N - 1)
    end
  end,

  ProtoFactorial(fun (X) -> (F(F))(X) end)
end).
```
{: .multi-code}

Now it seems more complex than before, but actually, it is not.
We extracted the essence of the factorial function in a special *proto* factorial function.
But why we've done it?
Because it is a step in the right direction - we want to extract the essence of the factorial from the code which allows us to define recursive anonymous functions.
In fact, we are very close to achieving this. The next step is to extract the *proto* factorial from the body of the `factorial` `Factorial`*multi-code* function:

```elixir
proto_factorial = fn g ->
  fn
    0 -> 1
    n -> n * g.(n - 1)
  end
end

factorial = (fn (h) ->
  o.(fn (f) ->
    h.(fn (x) -> f.(f).(x) end)
  end)
end).(proto_factorial)
```
```erlang
ProtoFactorial = fun (G) ->
  fun
    (0) -> 1;
    (N) -> N * (G)(N - 1)
  end
end.

Factorial = (fun (H) ->
  O(fun (F) ->
    H(fun (X) -> (F(F))(X) end)
  end)
end)(ProtoFactorial).
```
{: .multi-code}

It is easy to extract a function definition from the body of another function.
We just move its definition outside the defining function, wrap the defining function in another function,
so it can receive the extracted function as an argument and pass the extracted function.

Now we have the piece of code we wanted : a function which lets us define recursive anonymous functions of one argument:

```elixir
fn (h) ->
  o.(fn (f) ->
    h.(fn (x) -> f.(f).(x) end)
  end)
end

=>

y = fn (h) ->
  (fn (f) ->
    f.(f)
  end).(fn (g) ->
    h.(fn (x) -> g.(g).(x) end)
  end)
end
```
```erlang
fun (H) ->
  O(fun (F) ->
    H(fun (X) -> (F(F))(X) end)
  end)
end.

=>

Y = fun (H) ->
  (fun (F) ->
    F(F)
  end)(fun (G) ->
    H(fun (X) -> (G(G))(X) end)
  end)
end.
```
{: .multi-code}

We can define the *Nth Fibonacci number function* like this:

```elixir
proto_fib = fn (f) ->
  fn
    0 -> 1
    1 -> 2
    n -> f.(n - 1) + f.(n - 2)
  end
end

fib = y.(proto_fib)
```
```erlang
ProtoFib = fun (F) ->
  fun
    (0) -> 1;
    (1) -> 2;
    (N) -> F(N - 1) + F(N - 2)
  end
end.

Fib = Y(ProtoFib).
```
{: .multi-code}

And that's how we defined the famous **Y combinator**. It's a higher-order function which allows
defining recursion in places where the recursion is not part of the language. We limited our
language to a subset which doesn't support recursion to check that. There are infinitely many *Y combinators*, the
one we defined above works only with functions of one argument. It is a kind of *Y combinator*, known as **Z combinator**, because
[elixir]() {: .multi-code} and [erlang]() {:.multi-code} are not lazy languages and the arguments of a function are evaluated before passing them to the function.
In lazy languages there are even simpler *Y combinators*.

Another name of the *Y combinator* is _the fixed point combinator_. This means that the combinator returns a fixed point of the function passed to it.
A fixed point `x` for a function `f` is a value for which `f(x) = x`. And we can check that:

```elixir
fib = y.(proto_fib)

proto_fib.(fib).(5)
# 13
fib.(5)
# 13
proto_fib.(proto_fib.(fib)).(5)
# 13
```
```erlang
Fib = Y(ProtoFib).

(ProtoFib(Fib))(5).
% 13
Fib(5).
% 13
(ProtoFib(ProtoFib(Fib)))(5).
% 13
```
{: .multi-code}

Let's create an alias of `proto_fib` `ProtoFib`*multi-code* - `f` and another one of `y.(proto_fib)` `Y(ProtoFib)`*multi-code* - `x`.
We can say that `x` is fixed point of `f`. That's a fact, because the following equation is valid: `f(x) = x = f(f(x)) = f(..(f(..(f(x))..))..)`.

The *Y combinator* won't be of much practical use to you. All the languages, we work with, implement recursion.
But it is a very beautiful construct. It works a bit magically. Its definition and understanding are not meaningless in the end.
We saw how we can abstract and generalise concepts using ideas and terms from the **lambda calculus**, the grandfather of all the functional programming languages.
And speaking of the *lambda calculus*, I'm thinking of writing more articles related to it and its concepts in the future, so stay tuned.

Bye for now.
