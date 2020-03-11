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

Oh god FIXME

## Background

FIXME

## TODOs

* Write rest
* Clean up code
  * Nicer inclusion of safe_eval() in the quine
  * Remove old artifacts (but leave them in git; hi future people!)
* Show to other people
* Enjoy even more! :D
* Other languages?

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
