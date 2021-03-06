# general_purpose_fixedpoint

> A "practical proof" and implementation of the recursion theorem.

This implements a "practical proof" to [Roger's fixed-point theorem](https://en.wikipedia.org/wiki/Kleene%27s_recursion_theorem#Rogers's_fixed-point_theorem), sometimes also known as the quine theorem or (falsely) recursion theorem.

## Table of Contents

- [Usage](#usage)
- [Background](#background)
- [TODOs](#todos)
- [NOTDOs](#notdos)
- [Contribute](#contribute)
- [License](#license)

## Usage

Just run it!  (I was actually too lazy to implement argument parsing.)

Here's what a session can look like:

```
$ ./general_purpose_proof.py
Write your transformation on a single line, then press ENTER.
The input will be given as the *variable* `input`.
Write the output into the variable `output`, do not print it!
Note that it has to be a *transformation*, so both input and output are programs.
>>> output = 'output = "Hello!"'

# The fixed-point code `e` is:
def e(c, i):
    l = dict()
    exec(c, dict(input=i), l)
    return l.get('output') or ''
magic = 'def e(c, i):\n    l = dict()\n    exec(c, dict(input=i), l)\n    return l.get(\'output\') or \'\'\nmagic = %\nown_code = magic.replace(chr(37), repr(magic))\noutput = e(e(\'output = \\\'output = "Hello!"\\\'\', own_code), input)'
own_code = magic.replace(chr(37), repr(magic))
output = e(e('output = \'output = "Hello!"\'', own_code), input)

# In order to show the theorem, this should have the same effect as the code `F(e)`,
# i.e., the code after running it *once* through your transformation:
output = "Hello!"


# If you want to run it, enter anything (not necessarily code)
# and press ENTER. If you want to exit, just press ENTER:
>>> Not greeting nope absolutely not.


# First, run it with the (second) transformed code:
Hello!

# Finally, run it with the (first) "fixed-point" code:
Hello!

# They are equal!  Told ya so! :D
```

That's a bunch!  But don't worry, there is actually very little going on.

First, you need to enter a code transformation.  In this example, we chose `output = 'output = "Hello!"'`.  Note that this means that the output must be a program.  And since we are interested in achieving a fixed-point in the observable behavior of that program, having an output helps.

Any python code is legal, but you should keep it a "total function of `input`" in the mathematical sense. This includes constant functions like in this example.  More on that in the [background section](#background).

Then, the program responds with what it thinks is a fixed-point, let's call it program `e`:

```
def e(c, i):
    l = dict()
    exec(c, dict(input=i), l)
    return l.get('output') or ''
magic = 'def e(c, i):\n    l = dict()\n    exec(c, dict(input=i), l)\n    return l.get(\'output\') or \'\'\nmagic = %\nown_code = magic.replace(chr(37), repr(magic))\noutput = e(e(\'output = \\\'output = "Hello!"\\\'\', own_code), input)'
own_code = magic.replace(chr(37), repr(magic))
output = e(e('output = \'output = "Hello!"\'', own_code), input)
```

See the [background](#background) section on how this works, and why it always works.  Sure, in this toy example a much simpler fixed-point exists, but I didn't attempt to find the *simplest*, only *any* fixed-point.

```
output = "Hello!"
```

This is what the fixed-point code `e` looks like *after* being run through your provided earlier.  Let's call this snipped `F(e)`.  The code is very different!  But don't worry, that's not what the fixed-point theorem is about.  Instead, it guarantees that the *behavior* of `e` and `F(e)` are identical, even though their code is not.  Note that "behavior" includes whether or not it diverges.

You can convince yourself on your own that `e` and `F(e)` exhibit the same behavior.  (Note that a proof may not always be so easy, or even possible, without going through the fixed-point theorem!)

In order to make it easier to convince yourself, you can also enter an input, and it runs both `e` and `F(e)` on the input.  Note that this is impractical if either of them diverge – or, as I claim, both of them or neither of them.

And that's it!  What we have "demonstrated" in a very non-rigorous way is the theorem:

> If `F` is a total computable function, it has a fixed point.

Or more specifically:

> Given a total computable function `F`, this script spits out a fixed-point `e` such that `\phi_e = \phi_{F(e)}`, meaning that the partial function computed by `e` and `F(e)` are identical.

## Background

In this section, I try to shed light on some of the inner workings and details.

### Total function

This term is rather central in the theorem.  Specifically, it is important that:

- `make_fixedpoint` must be total, i.e., always halt (because otherwise it is difficult to make a statement)
- `F` must be total, i.e. always halt (because otherwise it is not much of a transformation code)
- The code generated by `make_fixedpoint` is allowed to diverge (because otherwise the transformation could just spit out an infinite loop like `while (true) {}`, and this behavior could not be imitated then)

Home exercise: What if the code generated by `make_fixedpoint` is not allowed to diverge?

### Guts of the fixed-point

We continue with the example from the [usage section](#usage), because it illustrates nicely where the transformation function is involved:

```
def e(c, i):
    l = dict()
    exec(c, dict(input=i), l)
    return l.get('output') or ''
magic = 'def e(c, i):\n    l = dict()\n    exec(c, dict(input=i), l)\n    return l.get(\'output\') or \'\'\nmagic = %\nown_code = magic.replace(chr(37), repr(magic))\noutput = e(e(\'output = \\\'output = "Hello!"\\\'\', own_code), input)'
own_code = magic.replace(chr(37), repr(magic))
output = e(e('output = \'output = "Hello!"\'', own_code), input)
```

This code first defines a function that enables sandboxed code execution.
"Sandboxed" not in a security sense, but in that global variables will not be
overwritten.  In particular, note that if the code (stored in `c`) makes its
own call to `exec`, there is no chance to accidentally overwrite the
global-ish variables `input` or `output`.

The code then constructs its own source code in variable `own_code`, making it
a quine.  The "magic" for the quine is stored in the aptly named variable
`magic`.

Next, the code applies the user-supplied transformation code `F` to it's own source code.
This means that *this* execution of `F` has exactly the same worldview as if it was called from the outside.  Therefore, the output code of the computation `F(e)` must be the same in both cases.

Finally, all `e` has left to do at this point is execute `F(e)`, and store it in `output`.

Of course, this might take a bit more time and space than just running `F(e)` in the first place, but this is Theoretical Computer Science, specifically Computability: We don't care about that.

You might think that it's cheating to give `e` access to the code of `F`.  However, it is not!  We want to show that `F` has a fixed-point, so we are allowed to construct a fixed-point *in dependence of `F`*.

You might think that it's cheating to give `e` the ability to execute `F`.  However, it is not!  `F` is executable in *some* form, and `e` might very well just include a fully-fledged interpreter to deal with `F`'s source code.  (In this case, we already are in an interpreter, so this burden is taken from us, thankfully.)

You might think something along the lines of "Obviously, then I just construct my `F` by looking at `e`!"
And that is even reasonable: `F` doesn't know `e` per-se, but `F` does receive `e` as input rather often.
But *executing* `e` is impossible, or at least not as straight-forward.  This is where the word "total" becomes important: The code implementing `F` *must* guarantee that `F` terminates.  If `F` were to call `e`, in the above construction this would diverge (well, cause a stack overflow, but whatever).  Diverging means that `F` does not halt, therefore is not a total function.  But the fixed-point theorem is *only* about total functions, therefore there is nothing to show in this case!

I hope this convinces you that the fixed-point theorem makes sense.
Have fun playing around with it! :)

## TODOs

* Other languages?
* Maybe allow multi-line or external transformations? (Kinda possible already, just write: `import foo; output = foo.process(input)`)

## NOTDOs

Here are some things this project will definitely not support:
* Support libraries or any other interaction.  The theorem is about *functions*.  As soon as you interact with the rest of the system, that's not a *function* in the input (i.e. `sim_input`) anymore, in the strict mathematical sense.
* Care about memory or time issues – Yes, your computer is not a Turing machine, therefore the transformation might be able to distinguish the phases by allocating a bunch of memory.  But that's kinda cheating.

## Contribute

The previous two sections are just cornerstones.
If you have a suggestion, feel free to [open an issue](https://github.com/BenWiederhake/subint/issues/new), submit PRs, or even fork the project! :D

## License

I don't think this work is copyrightable, but just in case: Do whatever the fuck you want, it's yours now! :D

This project is licensed under The Unlicense (see the file `LICENSE`).
