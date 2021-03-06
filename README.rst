Fn.py: enjoy FP in Python
=========================

.. image:: https://badges.gitter.im/fnpy/fn.py.svg
   :alt: Join the chat at https://gitter.im/fnpy/fn.py
   :target: https://gitter.im/fnpy/fn.py?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge

.. image:: https://travis-ci.org/fnpy/fn.py.svg?branch=master
    :target: https://travis-ci.org/fnpy/fn.py

.. image:: https://img.shields.io/pypi/v/fn.py
    :alt: PyPI
    :target: https://pypi.org/project/fn.py

.. image:: https://img.shields.io/pypi/dm/fn.py
    :alt: PyPI - Downloads
    :target: https://pypi.org/project/fn.py

Despite the fact that Python is not a pure-functional programming language, it
is multi-paradigm and gives you enough freedom to take advantage of a functional
approach.  There are theoretical and practical advantages to the functional
style:

-  Formal provability
-  Modularity
-  Composability
-  Ease of debugging and testing

``Fn.py`` library provides you with the missing "batteries" to get the maximum
from a functional approach, even in mostly-imperative softwares.

You can find more about the functional approach from my Pycon UA 2012 talks:
`Functional Programming with Python
<https://kachayev.github.com/talks/uapycon2012/index.html>`_.

Scala-style lambdas definition
------------------------------

::

    >>> from fn import _
    >>> from fn.op import zipwith
    >>> from itertools import repeat

    >>> list(map(_ * 2, range(5)))
    [0, 2, 4, 6, 8]
    >>> list(filter(_ < 10, [9,10,11]))
    [9]
    >>> list(zipwith(_ + _)([0,1,2], repeat(10)))
    [10, 11, 12]

You can find more examples of ``_`` in `test cases <tests/test_underscore.py>`_
(attributes resolving, method calling, slicing).

**Attention!** If you work in an interactive python shell, ``_`` can be assigned
to the latest output and you'll get unpredictable results.  In this case, you
can use ``X`` instead with ``from fn import _ as X``.

If you are not sure what your function is going to do, print it::

    >>> print(_ + 2)
    (x1) => (x1 + 2)
    >>> print(_ + _ * _)
    (x1, x2, x3) => (x1 + (x2 * x3))

Note that ``_`` will fail with ``ArityError`` (``TypeError`` subclass) when
called with the wrong number of arguments, so as to avoid errors::

    >>> (_ + _)(1)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "fn/underscore.py", line 82, in __call__
        raise ArityError(self, self._arity, len(args))
    fn.underscore.ArityError: (_ + _) expected 2 arguments, got 1


Persistent data structures
--------------------------

**Attention:** Persistent data structures are under active development.

A persistent data structure always preserves its previous version when it is
modified (more on `Wikipedia <https://goo.gl/8VveOH>`_).  Each operation thus
yields a new updated structure instead of performing in-place modifications (all
previous versions are potentially available or GC-ed when possible).

::

    >>> from fn.immutable import SkewHeap
    >>> s1 = SkewHeap(10)
    >>> s2 = s1.insert(20)
    >>> s2
    <fn.immutable.heap.SkewHeap object at 0x...>
    >>> s3 = s2.insert(30)
    >>> s3
    <fn.immutable.heap.SkewHeap object at 0x...>
    >>> id(s3) != id(s2)
    True
    >>> s3.extract()
    (10, <fn.immutable.heap.SkewHeap object at 0x...>)
    >>> s3.extract() # <-- s3 isn't changed
    (10, <fn.immutable.heap.SkewHeap object at 0x...>)

If you think I'm totally crazy and it will work despairingly slow, just give it
5 minutes.  Relax, take a deep breath and read about a few techniques that make
persistent data structures fast and efficient: `structural sharing
<https://en.wikipedia.org/wiki/Persistent_data_structure#Examples_of_persistent_data_structures>`_
and `path copying
<https://en.wikipedia.org/wiki/Persistent_data_structure#Path_Copying>`_.  To
see how it works in "pictures", you can check the great slides from Zach
Allaun's talk (StrangeLoop 2013): `"Functional Vectors, Maps And Sets In Julia"
<https://goo.gl/Cp1Qsq>`_.  And, if you are brave enough, go and read:

- `Chris Okasaki, "Purely Functional Data Structures" <https://goo.gl/c7ptkk>`_
- `Fethi Rabhi and Guy Lapalme, "Algorithms: A Functional Programming Approach" <https://goo.gl/00BxTO>`_

Immutable data structures available in ``fn.immutable``:

- ``LinkedList``: the most "obvious" persistent data structure, used as building
  block for other list-based structures (stack, queue)
- ``Stack``: wraps linked list implementation with well-known pop/push API
- ``Queue``: uses two linked lists and lazy copy to provide O(1) enqueue and
  dequeue operations
- ``Deque`` (in progress): `"Confluently Persistent Deques via Data
  Structural Bootstrapping" <https://goo.gl/vVTzx3>`_
- ``Deque`` based on ``FingerTree`` data structure (see more information below)
- ``Vector``: O(log32(n)) access to elements by index (which is near-O(1) for
  reasonable vector size), implementation is based on ``BitmappedTrie``, almost
  drop-in replacement for built-in Python ``list``
- ``SkewHeap``: self-adjusting heap implemented as a binary tree with specific
  branching model, uses heap merge as basic operation, more information -
  `"Self-adjusting heaps" <https://goo.gl/R1PZME>`_
- ``PairingHeap``: `"The Pairing-Heap: A New Form of Self-Adjusting Heap"
  <https://goo.gl/aiVtPH>`_
- ``Dict`` (in progress): persistent hash map implementation based on
  ``BitmappedTrie``
- ``FingerTree`` (in progress): `"Finger Trees: A Simple General-purpose Data
  Structure" <https://goo.gl/Bzo0df>`_

To get a clearer vision of how persistent heaps work (``SkewHeap`` and
``PairingHeap``), you can look at slides from my talk `"Union-based heaps"
<https://goo.gl/VMgdG2>`_ (with analyzed data structures definitions in Python
and Haskell).

**Note.** Most functional languages use persistent data structures as basic
building blocks, well-known examples are Clojure, Haskell and Scala.  Clojure
community puts much effort to popularize programming based on the idea of data
immutability.  There are few amazing talk given by Rich Hickey (creator of
Clojure), you can check them to find answers on both questions "How?" and
"Why?":

- `"The Value of Values" <https://goo.gl/137UG5>`_
- `"Persistent Data Structures and Managed References" <https://goo.gl/M3vZ7E>`_

Streams and infinite sequences declaration
------------------------------------------

Lazy-evaluated Scala-style streams.  Basic idea: evaluate each new element "on
demand" and share consumed elements between all created iterators.  A ``Stream``
instance supports ``<<`` to push new elements.

::

    >>> from fn import Stream

    >>> s = Stream() << [1,2,3,4,5]
    >>> list(s)
    [1, 2, 3, 4, 5]
    >>> s[1]
    2
    >>> list(s[0:2])
    [1, 2]

    >>> s = Stream() << range(6) << [6,7]
    >>> list(s)
    [0, 1, 2, 3, 4, 5, 6, 7]

    >>> def gen():
    ...     yield 1
    ...     yield 2
    ...     yield 3

    >>> s = Stream() << gen << (4,5)
    >>> list(s)
    [1, 2, 3, 4, 5]

Lazy-evaluated streams are useful for infinite sequences, e.g. the fibonacci
sequence can be computed as::

    >>> from fn.iters import take, drop, map
    >>> from operator import add

    >>> f = Stream()
    >>> fib = f << [0, 1] << map(add, f, drop(1, f))

    >>> list(take(10, fib))
    [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
    >>> fib[20]
    6765
    >>> list(fib[30:35])
    [832040, 1346269, 2178309, 3524578, 5702887]

Trampolines decorator
---------------------

``fn.recur.tco`` simulates TCO (tail call optimization).  Let's start with a
simple example of recursive factorial computation::

    >>> def fact(n):
    ...    if n == 0: return 1
    ...    return n * fact(n-1)

This variant works, but it's really inefficient.  Why?  It will consume too much
memory, linear in the depth of the recursion: if you execute this function
with a big ``n`` (more than ``sys.getrecursionlimit()``) CPython will fail::

    >>> import sys
    >>> fact(sys.getrecursionlimit() * 2)
    Traceback (most recent call last):
        ...
    RecursionError: maximum recursion depth exceeded ...

Which is good, as it prevents you from terrible mistakes in your code.

How can we optimize this function?  Easy, let's transform it to use a tail
call::

    def fact(n, acc=1):
        if n == 0: return acc
        return fact(n-1, acc*n)

Is this variant better?  Yes, because you don't need to remember previous values
(local variables) to get the final result.  More about `tail call optimization
<https://en.wikipedia.org/wiki/Tail_call>`_ on Wikipedia.  But... the Python
interpreter will execute this function the same way as the previous one, so you
won't win anything here.

``fn.recur.tco`` allows you to optimize a bit this tail call recursion (using a
"trampoline" approach)::

    >>> from fn import recur

    >>> @recur.tco
    ... def fact(n, acc=1):
    ...    if n == 0: return False, acc
    ...    return True, (n-1, acc*n)

``@recur.tco`` executes your function in a ``while`` loop and checks the output:

- ``(False, result)`` means that we finished,
- ``(True, args, kwargs)`` means that we need to recurse,
- ``(func, args, kwargs)`` switches the function executed inside the while loop.

Example for the third case, to recursively check the parity of a number::

    >>> @recur.tco
    ... def even(x):
    ...     if x == 0: return False, True
    ...     return odd, (x-1,)
    ...
    >>> @recur.tco
    ... def odd(x):
    ...     if x == 0: return False, False
    ...     return even, (x-1,)
    ...
    >>> even(100000)
    True

**Attention:** be careful with mutable/immutable data structures processing.

Itertools recipes
-----------------

``fn.uniform`` unifies generator functions between Python versions (use
generator versions of ``map, filter, reduce, zip, range, filterfalse,
zip_longest``, backported ``accumulate``).

``fn.iters`` offers high-level recipes for working with iterators, most of them
are from the `itertools docs
<https://docs.python.org/3/library/itertools.html#itertools-recipes>`_ and
adapted for Python 2+/3+.

-  ``take``, ``drop``
-  ``takelast``, ``droplast``
-  ``head`` (alias: ``first``), ``tail`` (alias: ``rest``)
-  ``second``, ``ffirst``
-  ``compact``, ``reject``
-  ``every``, ``some``
-  ``iterate``
-  ``consume``
-  ``nth``
-  ``padnone``, ``ncycles``
-  ``repeatfunc``
-  ``grouper``, ``powerset``, ``pairwise``
-  ``roundrobin``
-  ``partition``, ``splitat``, ``splitby``
-  ``flatten``
-  ``iter_except``
-  ``first_true``

See the `docstrings <fn/iters.py>`_ and `tests <tests/test_iterators.py>`_ for
more information.

High-level operations with functions
------------------------------------

``fn.F`` wraps functions for easy-to-use partial application and composition::

    >>> from fn import F
    >>> from operator import add, mul

    # F(f, *args) means partial application
    # same as functools.partial but returns fn.F instance
    >>> F(add, 1)(10)
    11

    # F << F means functions composition,
    # so (F(f) << g)(x) == f(g(x))
    >>> f = F(add, 1) << F(mul, 100)
    >>> list(map(f, [0, 1, 2]))
    [1, 101, 201]
    >>> list(map(F() << str << (_ ** 2) << (_ + 1), range(3)))
    ['1', '4', '9']

You can also pipe functions with ``>>``::

    >>> from fn.iters import filter, range

    >>> func = F() >> (filter, _ < 6) >> sum
    >>> func(range(10))
    15

You can find more examples in the ``fn._`` `implementation <fn/underscore.py>`_.

``fn.op.apply`` executes a function with given args and kwargs.  ``fn.op.flip``
wraps a function, flipping the order of its arguments.

::

    >>> from fn.op import apply, flip
    >>> from operator import add, sub

    >>> apply(add, [1, 2])
    3
    >>> flip(sub)(20, 10)
    -10
    >>> list(map(apply, [add, mul], [(1 ,2), (10, 20)]))
    [3, 200]

``fn.op.foldl`` and ``fn.op.foldr`` create a reducer from a function of two
arguments (think of it as curried ``reduce``).

::

    >>> from fn import op
    >>> op.foldl(_ + _)([1,2,3])
    6
    >>> mult = op.foldr(_ * _, 1)
    >>> mult([1,2,3])
    6
    >>> op.foldr(op.call, 0 )([_ ** 2, _ + 10])
    100
    >>> op.foldr(op.call, 10)([_ ** 2, _ + 10])
    400


Function currying
-----------------

::

    >>> from fn.func import curried
    >>> @curried
    ... def sum5(a, b, c, d, e):
    ...     return a + b + c + d + e
    ...
    >>> sum5(1)(2)(3)(4)(5)
    15
    >>> sum5(1, 2, 3)(4, 5)
    15


Functional style error-handling
-----------------------------------

``fn.monad.Option`` represents optional values, each instance of ``Option`` can
be either ``Full`` or ``Empty``.  It provides you with a simple way to write
long computation sequences and get rid of many ``if/else`` blocks.  See usage
examples below.

Assume that you have a ``Request`` class that gives you a parameter value by its
name, and you have to convert it to uppercase and strip it::

    >>> class Request(dict):
    ...     def parameter(self, name):
    ...         return self.get(name, None)

    >>> r = Request(testing="Fixed", empty="   ")
    >>> param = r.parameter("testing")
    >>> if param is None:
    ...     fixed = ""
    ... else:
    ...     param = param.strip()
    ...     if len(param) == 0:
    ...         fixed = ""
    ...     else:
    ...         fixed = param.upper()
    >>> fixed
    'FIXED'


Hmm, looks ugly..  But now with ``fn.monad.Option``::

    >>> from operator import methodcaller
    >>> from fn.monad import optionable

    >>> class Request(dict):
    ...     @optionable
    ...     def parameter(self, name):
    ...         return self.get(name, None)

    >>> r = Request(testing="Fixed", empty="   ")
    >>> (r
    ...     .parameter("testing")
    ...     .map(methodcaller("strip"))
    ...     .filter(len)
    ...     .map(methodcaller("upper"))
    ...     .get_or("")
    ... )
    'FIXED'

Use ``or_call`` for more complex alternatives, for example::

    from fn.monad import Option

    request = dict(url="face.png", mimetype="PNG")
    tp = (Option 
        .from_value(request.get("type", None))  # check "type" key first
        .or_call(from_mimetype, request)  # or.. check "mimetype" key
        .or_call(from_extension, request)  # or... get "url" and check extension
        .get_or("application/undefined")
    )


Installation
------------

To install ``fn.py``, simply::

    $ pip install fn.py

You can also build library from source

::

    $ git clone https://github.com/fnpy/fn.py.git
    $ pip install -e fn.py

Work in progress
----------------

"Roadmap":

- ``fn.monad.Either`` to deal with error logging
-  C-accelerator for most modules

Ideas to think about:

-  Scala-style for-yield loop to simplify long map/filter blocks

Contribute
----------

If you find a bug:

1. Check for open issues or open a fresh issue to start a discussion
   around a feature idea or a bug.
2. Fork the repository on Github to start making your changes to the
   master branch (or branch off of it).
3. Write a test which shows that the bug was fixed or that the feature
   works as expected.

If you like fixing bugs:

1. Check for open issues with the label "Help Wanted" and either claim
   it or collaborate with those who have claimed it.
2. Fork the repository on Github to start making your changes to the
   master branch (or branch off of it).
3. Write a test which shows that the bug was fixed or that the feature
   works as expected.

How to contact the maintainers
------------------------------

- Gitter: https://gitter.im/fnpy/fn.py
- Jacob's (Organization Owner) Email: him <at> jacobandkate143.com
- Alex's (Original Project Owner) Email: kachayev <at> gmail.com
