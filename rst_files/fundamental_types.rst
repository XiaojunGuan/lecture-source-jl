.. _fundamental_types:

.. include:: /_static/includes/lecture_howto_jl.raw

**********************************************
Arrays, Tuples, and Ranges
**********************************************

.. contents:: :depth: 2

.. epigraph::

    "Let's be clear: the work of science has nothing whatever to do with consensus.
    Consensus is the business of politics. Science, on the contrary, requires only
    one investigator who happens to be right, which means that he or she has
    results that are verifiable by reference to the real world. In science
    consensus is irrelevant. What is relevant is reproducible results." -- Michael Crichton

Overview
============================

In Julia, arrays and tuples are the most important data type for working with numerical data

In this lecture we give more details on

* declaring types

* creating and manipulating Julia arrays

* fundamental array processing operations

* basic matrix algebra

* tuples and named tuples

* ranges

Array Basics
================


Shape and Dimension
----------------------

Activate the project environment, ensuring that ``Project.toml`` and ``Manifest.toml`` are in the same location as your notebook

.. code-block:: julia

    using Pkg; Pkg.activate(@__DIR__); #activate environment in the notebook's location
    using LinearAlgebra, Statistics


We've already seen some Julia arrays in action


.. code-block:: julia

    a = [10, 20, 30]


.. code-block:: julia

    a = [1.0, 2.0, 3.0]


The REPL tells us that the arrays are of types ``Array{Int64,1}`` and ``Array{Float64,1}`` respectively

Here ``Int64`` and ``Float64`` are types for the elements inferred by the compiler

We'll talk more about types later on

The ``1`` in ``Array{Int64,1}`` and ``Array{Any,1}`` indicates that the array is
one dimensional (i.e., a ``Vector``)

This is the default for many Julia functions that create arrays


.. code-block:: julia

    typeof(randn(100))

In Julia, one dimensional vectors are best interpreted as column vectors, which we will see when we take transposes

We can check the dimensions of ``a`` using ``size()`` and ``ndims()``
functions

.. code-block:: julia

    ndims(a)


.. code-block:: julia

    size(a)


The syntax ``(3,)`` displays a tuple containing one element --- the size along the one dimension that exists



Array vs Vector vs Matrix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In Julia, ``Vector`` and ``Matrix`` are just aliases for one- and two-dimensional arrays
respectively

.. code-block:: julia

    Array{Int64, 1} == Vector{Int64}
    Array{Int64, 2} == Matrix{Int64}


Vector construction with ``,`` is then interpreted as a column vector

To see this, we can create a column vector and row vector more directly

.. code-block:: julia

    [1, 2, 3] == [1; 2; 3] #both column vectors

.. code-block:: julia

    [1 2 3] #a row vector is 2-dimensional

As we've seen, in Julia we have both

* one-dimensional arrays (i.e., flat arrays)

* arrays of size ``(1, n)`` or ``(n, 1)`` that represent row and column vectors respectively

Why do we need both?

On one hand, dimension matters when we come to matrix algebra

* Multiplying by a row vector is different to multiplication by a column vector

On the other, we use arrays in many settings that don't involve matrix algebra

In such cases, we don't care about the distinction between row and column vectors

This is why many Julia functions return flat arrays by default


.. _creating_arrays:

Creating Arrays
------------------


Functions that Create Arrays
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We've already seen some functions for creating a vector filled with ``0.0``

.. code-block:: julia

    zeros(3)

This generalizes to matrices and higher dimensional arrays

.. code-block:: julia

    zeros(2, 2)

To return an array filled with a single value, use ``fill``

.. code-block:: julia

    fill(5.0, 2, 2)

Finally, you can create an empty array using the ``Array()`` constructor

.. code-block:: julia

    x = Array{Float64}(undef, 2, 2)


The printed values you see here are just garbage values

(the existing contents of the allocated memory slots being interpreted as 64 bit floats)

If you need more control over the types, fill with a non-floating point

.. code-block:: julia

    fill(0, 2, 2) # fills with 0, not 0.0
    
Or fill with a boolean type

.. code-block:: julia

    fill(false, 2, 2) # produces a boolean matrix



Creating Arrays from Existing Arrays
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For the most part, we will avoid directly specifying the types of arrays, and let the compiler deduce the optimal types on its own

The reasons for this, discussed in more detail in :doc:`this lecture <generic_functional_programming>`, are to ensure both clarity and generality

One place this can be inconvenient is when we need to create an array based on an existing array

First, note that assignment in Julia binds a name to a value, but does not make a copy of that type

.. code-block:: julia

    x = [1, 2, 3]
    y = x
    y[1] = 2
    x

In the above, the ``y = x`` simply create a new named binding called ``y`` which refers to whatever ``x`` currently binds to

To copy the data, you need to be more explicit

.. code-block:: julia

    x = [1, 2, 3]
    y = copy(x)
    y[1] = 2
    x

However, rather than making a copy of ``x``, you may want to just have a similarly sized array

.. code-block:: julia

    x = [1, 2, 3]
    y = similar(x)
    y


Similar can also be used to pre-allocate a vector with a different size, but the same shape

.. code-block:: julia

    x = [1, 2, 3]
    y = similar(x, 4) # make a vector of length 4

Which generalized to higher dimensions

.. code-block:: julia

    x = [1, 2, 3]
    y = similar(x, 2, 2) # make 2x2 matrix

Manual Array Definitions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As we've seen, you can create one dimensional arrays from manually specified data like so

.. code-block:: julia

    a = [10, 20, 30, 40]

In two dimensions we can proceed as follows

.. code-block:: julia

    a = [10 20 30 40]  # two dimensional, shape is 1 x n


.. code-block:: julia

    ndims(a)


.. code-block:: julia

    a = [10 20; 30 40]  # 2 x 2


You might then assume that ``a = [10; 20; 30; 40]`` creates a two dimensional column vector but this isn't the case

.. code-block:: julia

    a = [10; 20; 30; 40]


.. code-block:: julia

    ndims(a)


Instead transpose the matrix (or adjoint if complex)

.. code-block:: julia

    a = [10 20 30 40]'


.. code-block:: julia

    ndims(a)


Array Indexing
-----------------

We've already seen the basics of array indexing

.. code-block:: julia

    a = [10 20 30 40]
    a[end-1]


.. code-block:: julia

    a[1:3]


For 2D arrays the index syntax is straightforward

.. code-block:: julia

    a = randn(2, 2)
    a[1, 1]


.. code-block:: julia

    a[1, :]  # First row


.. code-block:: julia

    a[:, 1]  # First column


Booleans can be used to extract elements

.. code-block:: julia

    a = randn(2, 2)


.. code-block:: julia

    b = [true false; false true]


.. code-block:: julia

    a[b]


This is useful for conditional extraction, as we'll see below

An aside: some or all elements of an array can be set equal to one number using slice notation


.. code-block:: julia

    a = zeros(4)


.. code-block:: julia

    a[2:end] .= 42


.. code-block:: julia

    a

Views and Slices
-----------------------------

Using the ``:`` notation provides a slice of an array, copying the sub-array to a new array with a similar type 

.. code-block:: julia

    a = [1 2; 3 4]
    b = a[:, 2] 
    @show b
    a[:, 2] = [4, 5] # modify a
    @show a
    @show b;

A ``view`` on the other hand does not copy the value

.. code-block:: julia

    a = [1 2; 3 4]
    @views b = a[:, 2] 
    @show b
    a[:, 2] = [4, 5]
    @show a
    @show b;

Note that the only difference is the ``@views`` macro, which will replace any slices with views in the expression

An alternative is to call the ``view`` function directly--though it is generally discouraged since it is a step away from the math

.. code-block:: julia

    @views b = a[:, 2] 
    view(a, :, 2) == b

As with most programming in Julia, it is best to avoid prematurely assuming that ``@views`` will have a significant impact on performance, and stress code clarity above all else

Another important lesson about views is that they **are not** concrete arrays types 

.. code-block:: julia

    a = [1 2; 3 4]
    b_slice = a[:, 2]
    @show typeof(b_slice)
    @show typeof(a)
    @views b = a[:, 2] 
    @show typeof(b);

The type of ``b`` is a good example of how types are not as they may sometimes

Similarly

.. code-block:: julia

    a = [1 2; 3 4]
    b = a' # transpose
    typeof(b)


To copy into a concrete array

.. code-block:: julia

    a = [1 2; 3 4]
    b = a' # transpose
    c = Matrix(b) # convert to matrix
    d = collect(b) # also `collect` works on any iterable
    c == d

Special Matrices
-----------------

As we saw with the ``transpose``, sometimes types that look like matrices are not stored as a dense array

As an example, consider creating a diagonal matrix 

.. code-block:: julia

    using LinearAlgebra
    d = [1.0, 2.0]
    a = Diagonal(d)

As you can see, the type is ``2×2 Diagonal{Float64,Array{Float64,1}}``, which is not a 2-dimensional array

The reasons for this are both efficiency in storage, as well as efficiency in arithmetic and matrix operations

For example, this both behaves (and is, in an important sense) like any other Matrix

.. code-block:: julia

    @show 2a
    b = rand(2,2)
    @show b * a;

Another example is in the construction of an identity matrix, where a naive implementation is

.. code-block:: julia

    b = [1.0 2.0; 3.0 4.0]
    b - Diagonal([1.0, 1.0]) # poor style, inefficient code

Whereas you should instead use

.. code-block:: julia

    b = [1.0 2.0; 3.0 4.0]
    b - I # good style, and note the lack of dimensions of I

While the implementation of ``I`` is a little abstract to go into at this point, a hint is that 

.. code-block:: julia

    typeof(I)

So that this is a ``UniformScaling`` type rather than an identity matrix, making it much more powerful and general

Assignment and Passing Arrays
------------------------------

As discussed above, in Julia, the left hand side of an assignment is a "binding" to a name

.. code-block:: julia

    x = [1 2 3]
    y = x # name y binds to whatever `x` bound to

The consequence of this, is that you can re-bind that name 

.. code-block:: julia

    x = [1 2 3]
    y = x # name y binds to whatever `x` bound to
    z = [2 3 4]
    y = z # just changes name binding, not value!
    @show (x, y, z);

What this means is that if ``a`` is an array and we set ``b = a`` then ``a`` and ``b`` point to exactly the same data

In the above, suppose you had meant to change the value of ``x`` to the values of ``y``, you need to assign the values rather than the name

.. code-block:: julia

    x = [1 2 3]
    y = x # name y binds to whatever `x` bound to
    z = [2 3 4]
    y .= z # Now dispatches the assignment of each element
    @show (x, y, z);    

Alternatively, you could have used ``y[:] = z``

This applies to in-place functions as well

First, define a simple function for a linear map

.. code-block:: julia

    function f(x)
        return [1 2; 3 4] * x # matrix * column vector
    end
    val = [1, 2]
    f(val)

In general, these "out-of-place" functions are preferred to "in-place" functions, which modify the arguments

.. code-block:: julia

    function f(x)
        return [1 2; 3 4] * x # matrix * column vector
    end
    val = [1, 2]
    y = similar(val)
    function f!(out, x)
        out .= [1 2; 3 4] * x
    end
    f!(y, val)
    y
    
This demonstrates a key convention in Julia: functions which modify any of the arguments have the name ending with ``!`` (e.g. ``push!``)

We can also see a common mistake, where instead of modifying the arguments, the name binding is swapped

.. code-block:: julia

    function f(x)
        return [1 2; 3 4] * x # matrix * column vector
    end
    val = [1, 2]
    y = similar(val)

    function f!(out, x)
        out = [1 2; 3 4] * x # MISTAKE! should be .= or [:]
    end
    f!(y, val)
    y
    
The frequency of making this mistake is one of the reasons to avoid in-place functions, unless proven to be necessary by benchmarking 

Operations on Arrays
================================


Array Methods
------------------

Julia provides standard functions for acting on arrays, some of which we've
already seen

.. code-block:: julia

    a = [-1, 0, 1]


    @show length(a)
    @show sum(a)
    @show mean(a)
    @show std(a) #standard deviation
    @show var(a) # variance
    @show maximum(a)
    @show minimum(a)
    @show extrema(a) # (mimimum(a), maximum(a))


To sort an array

.. code-block:: julia

    b = sort(a, rev = true)  # returns new array, original not modified


.. code-block:: julia

    b = sort!(a, rev = true)  # returns *modified original* array


.. code-block:: julia

    b == a  # tests if have the same values


.. code-block:: julia

    b === a  # tests if arrays are identical (i.e share same memory)


Matrix Algebra
---------------------

For two dimensional arrays, ``*`` means matrix multiplication

.. code-block:: julia

    a = ones(1, 2)


.. code-block:: julia

    b = ones(2, 2)


.. code-block:: julia

    a * b


.. code-block:: julia

    b * a'


To solve the linear system ``A X = B`` for ``X`` use ``A \ B``

.. code-block:: julia

    A = [1 2; 2 3]


.. code-block:: julia

    B = ones(2, 2)


.. code-block:: julia

    A \ B


.. code-block:: julia

    inv(A) * B


Although the last two operations give the same result, the first one is numerically more stable and should be preferred in most cases

Multiplying two **one** dimensional vectors gives an error --- which is reasonable since the meaning is ambiguous

.. code-block:: julia
    :class: no-execute

    ones(2) * ones(2)


If you want an inner product in this setting use ``dot()`` or the `` ``unicode ``\dot<TAB>``

.. code-block:: julia

    dot(ones(2), ones(2))


Matrix multiplication using one dimensional vectors is a bit inconsistent --- pre-multiplication by the matrix is OK, but post-multiplication gives an error


.. code-block:: julia

    b = ones(2, 2)


.. code-block:: julia

    b * ones(2)


.. code-block:: julia
    :class: no-execute

    ones(2) * b


Elementwise Operations
------------------------

Algebraic Operations
^^^^^^^^^^^^^^^^^^^^^^^^

Suppose that we wish to multiply every element of matrix ``A`` with the corresponding element of matrix ``B``

In that case we need to replace ``*`` (matrix multiplication) with ``.*`` (elementwise multiplication)

For example, compare

.. code-block:: julia

    ones(2, 2) * ones(2, 2)   # Matrix multiplication


.. code-block:: julia

    ones(2, 2) .* ones(2, 2)   # Element by element multiplication


This is a general principle: ``.x`` means apply operator ``x`` elementwise


.. code-block:: julia

    A = -ones(2, 2)


.. code-block:: julia

    A.^2  # Square every element

However in practice some operations are mathematically valid without broadcasting, and hence the ``.`` can be omitted

.. code-block:: julia

    ones(2, 2) + ones(2, 2)  # same as ones(2, 2) .+ ones(2, 2)


Scalar multiplication is similar

.. code-block:: julia

    A = ones(2, 2)


.. code-block:: julia

    2 * A  # same as 2 .* A


In fact you can omit the ``*`` altogether and just write ``2A``

Unlike matlab and other languages, scalar addition requires the ``.+`` in order to correctly broadcast

.. code-block:: julia

    x = [1, 2]
    x .+ 1 # i.e. not x + 1 
    x .- 1 # i.e. not x - 1


Elementwise Comparisons
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Elementwise comparisons also use the ``.x`` style notation

.. code-block:: julia

    a = [10, 20, 30]


.. code-block:: julia

    b = [-100, 0, 100]


.. code-block:: julia

    b .> a


.. code-block:: julia

    a .== b


We can also do comparisons against scalars with parallel syntax

.. code-block:: julia

    b


.. code-block:: julia

    b .> 1



This is particularly useful for *conditional extraction* --- extracting the elements of an array that satisfy a condition

.. code-block:: julia

    a = randn(4)


.. code-block:: julia

    a .< 0


.. code-block:: julia

    a[a .< 0]


Changing Dimensions
^^^^^^^^^^^^^^^^^^^^^^^^

The primary function for changing the dimension of an array is ``reshape()``


.. code-block:: julia

    a = [10, 20, 30, 40]


.. code-block:: julia

    b = reshape(a, 2, 2)


.. code-block:: julia

    b


Notice that this function returns a "view" on the existing array

This means that changing the data in the new array will modify the data in the
old one:

.. code-block:: julia

    b[1, 1] = 100  # Continuing the previous example


.. code-block:: julia

    b


.. code-block:: julia

    a


To collapse an array along one dimension you can use ``dropdims()``

.. code-block:: julia

    a = [1 2 3 4]  # Two dimensional


.. code-block:: julia

    dropdims(a, dims = 1)


The return value is an array with the specified dimension "flattened"

Broadcasting Functions
--------------------------

Julia provides standard mathematical functions such as ``log``, ``exp``, ``sin``, etc.

.. code-block:: julia

    log(1.0)


By default, these functions act *elementwise* on arrays

.. code-block:: julia

    log.(1:4)

Note that we can get the same result as with a comprehension or more explicit loop


.. code-block:: julia

    [ log(x) for x in 1:4 ]


.. ACTUALLY, kind of the opposite, as in most languages.  "for" loops typically fastest when done correctly.  In Julia loops are typically fast and hence the need for vectorized functions is less intense than for some other high level languages

Nonetheless the syntax is convenient

Linear Algebra
-------------------


Julia provides some a great deal of additional functionality related to linear operations


.. code-block:: julia

    A = [1 2; 3 4]


.. code-block:: julia

    det(A)


.. code-block:: julia

    tr(A)


.. code-block:: julia

    eigvals(A)


.. code-block:: julia

    rank(A)


For more details see the `linear algebra section <https://docs.julialang.org/en/stable/manual/linear-algebra/>`_ of the standard library



Introduction to Types
======================

We will discuss this in detail in :doc:`this lecture <generic_functional_programming>`, but much of its performance gains and generality of notation comes from Julia's type system

For example, compare

.. code-block:: julia

    x = [1, 2, 3]

Gives ``Array{Int64,1}`` as the type whereas

.. code-block:: julia

    x = [1.0, 2.0, 3.0]

These return ``Array{Int64,1}`` and ``Array{Float64,1}`` respectively, which the compiler is able to infer from the right hand side of the expressions

Given the information on the type, the compiler can work through the sequence of expressions to infer other types

.. code-block:: julia

    # define some function
    f(y) = 2y

    # call with an integer array
    x = [1, 2, 3]
    z = f(x) # compiler deduces type

Good Practices for Functions and Variables
--------------------------------------------

In order to keep many of the benefits of Julia, you will sometimes want to help the compiler ensure that it can always deduce a single type from any function or expression

As an example of bad practice, is to use an array to hold unrelated types

.. code-block:: julia

    x = [1.0, "test"] # poor style

The type of this is ``Array{Any,1}``, where the ``Any`` means the compiler has determined that any valid Julia types can be added to the array

While occasionally useful, this is to be avoided whenever possible in performance sensitive code

The other place this can come up is in the declaration of functions,

As an example, consider a function which returns different types depending on the arguments

.. code-block:: julia

    function f(x)
        if x > 0
            return 1.0
        else 
            return 0 # Probably meant 0.0
        end
    end
    @show f(1)
    @show f(-1)

The issue here is relatively subtle:  ``1.0`` is a floating point, while ``0`` is an integer

Consequently, given the type of ``x``, the compiler cannot in general determine what type the function will return

This issue, called **type stability** is at the heart of most Julia performance considerations

Luckily, the practice of trying to ensure that functions return the same types is also the most consistent with simple, clear code


Manually Declaring Types
-------------------------

While we keep talking about types, you will notice that we have never declared any types in the underlying code

This is intentional for exposition and "user" code of packages, rather than the writing of those packages themselves

It is also in contrast to some of the sample code you will see

To give an example of the declaration of types, the following are equivalent

.. code-block:: julia

    function f(x, A)
        b = [5.0; 6.0]
        return A * x .+ b
    end
    val = f([0.1, 2.0], [1.0 2.0; 3.0 4.0])

.. code-block:: julia

    function f2(x::Vector{Float64}, A::Matrix{Float64})::Vector{Float64} # argument and return types
        b::Vector{Float64} = [5.0; 6.0]
        return A * x .+ b
    end
    val = f2([0.1; 2.0], [1.0 2.0; 3.0 4.0])

While declaring the types may be verbose, would it ever generate faster code?

The answer is: almost never

Furthermore, it can lead to confusion and inefficiencies since many things that behave like vectors and matrices are not ``Matrix{Float64}`` and ``Vector{Float64}``

To see a few examples where the first works and the second fails

.. code-block:: julia

    @show f([0.1; 2.0], [1 2; 3 4])
    @show f([0.1; 2.0], Diagonal([1.0, 2.0]))

    #f2([0.1; 2.0], [1 2; 3 4]) # not a Float64
    #f2([0.1; 2.0], Diagonal([1.0, 2.0])) # not a Matrix{Float64}


Some Other Types
==================


Ranges
-----------

As with many other types, a ``Range`` can act as a vector

.. code-block:: julia

    a = 10:12 # a range, equivalent to 10:1:12
    @show Vector(a) # can convert, but shouldn't

    b = Diagonal([1.0, 2.0, 3.0])
    b * a .- [1.0; 2.0; 3.0]

Ranges can also be created with floating point numbers using the same notation

.. code-block:: julia

    a = 0.0:0.1:1.0 # 0.0, 0.1, 0.2, ... 1.0

But care should be taken if the terminal node is not a multiple of the set sizes

.. code-block:: julia

    maxval = 1.0
    minval = 0.0
    stepsize = 0.15
    a = minval:stepsize:maxval # 0.0, 0.15, 0.3, ... ??? Not 1.0
    maximum(a) == maxval


To evenly space points where the maximum value is important, i.e., ``linspace`` in other languages 

.. code-block:: julia

    maxval = 1.0
    minval = 0.0
    numpoints = 10
    a = range(minval, stop=maxval, length=numpoints)
    # a = range(minval, maxval, length=numpoints) # supported soon...
    maximum(a) == maxval

Tuples and Named Tuples
---------------------------------

We were introduced to Tuples earlier, which provide high-performance but immutable sets of distinct types

.. code-block:: julia

    t = (1.0, "test")
    t[1] # access by index
    a, b = t # unpack
    # t[1] = 3.0 # would fail as tuples are immutable
    println("a = $a and b = $b")

As well as named tuples, which are tuples extended to have a list of names 

.. code-block:: julia

    t = (val1 = 1.0, val2 = "test")
    t.val1 # access by index
    a, b = t # unpack
    println("a = $a and b = $b") # access by index

While immutable, it is possible to manipulate tuples and generate new ones

.. code-block:: julia

    t2 = (val3 = 4, val4 = "test!!")
    t3 = merge(t, t2) # new tuple


Named tuples are a convenient and high-performance way to manage and unpack sets of parameters


.. code-block:: julia

    function f(parameters)
        α, β = parameters.α, parameters.β # poor style, error prone if adding parameters
        return α + β
    end 
    parameters = (α = 0.1, β = 0.2)
    f(parameters)


This functionality is aided by the ``Parameters.jl`` package and the `@unpack` macro

.. code-block:: julia

    using Parameters
    function f(parameters)
        @unpack α, β = parameters # good style, less sensitive to errors
        return α + β
    end 
    parameters = (α = 0.1, β = 0.2)
    f(parameters)

In order to manage default values, use the ``@with_kw`` macro

.. code-block:: julia

    using Parameters
    paramgen = @with_kw (α = 0.1, β = 0.2) # creates named tuples with defaults

    # creates named tuples, replacing defaults
    @show paramgen()
    @show paramgen(α = 0.2)
    @show paramgen(α = 0.2, β = 0.5);

Exercises
=============


.. _np_ex1:

Exercise 1
----------------

This exercise is on some matrix operations that arise in certain problems, including when dealing with linear stochastic difference equations

If you aren't familiar with all the terminology don't be concerned --- you can skim read the background discussion and focus purely on the matrix exercise

With that said, consider the stochastic difference equation

.. math::
    :label: ja_sde

    X_{t+1} = A X_t + b + \Sigma W_{t+1}


Here

* :math:`X_t, b` and :math:`X_{t+1}` ar :math:`n \times 1`

* :math:`A` is :math:`n \times n`

* :math:`\Sigma` is :math:`n \times k`

* :math:`W_t` is :math:`k \times 1` and :math:`\{W_t\}` is iid with zero mean and variance-covariance matrix equal to the identity matrix

Let :math:`S_t` denote the :math:`n \times n` variance-covariance matrix of :math:`X_t`

Using the rules for computing variances in matrix expressions, it can be shown from :eq:`ja_sde` that :math:`\{S_t\}` obeys

.. math::
    :label: ja_sde_v

    S_{t+1} = A S_t A' + \Sigma \Sigma'


It can be shown that, provided all eigenvalues of :math:`A` lie within the unit circle, the sequence :math:`\{S_t\}` converges to a unique limit :math:`S`

This is the **unconditional variance** or **asymptotic variance** of the stochastic difference equation

As an exercise, try writing a simple function that solves for the limit :math:`S` by iterating on :eq:`ja_sde_v` given :math:`A` and :math:`\Sigma`

To test your solution, observe that the limit :math:`S` is a solution to the matrix equation

.. math::
    :label: ja_dle

    S = A S A' + Q
    \quad \text{where} \quad Q := \Sigma \Sigma'


This kind of equation is known as a **discrete time Lyapunov equation**

The `QuantEcon package <http://quantecon.org/julia_index.html>`_
provides a function called ``solve_discrete_lyapunov`` that implements a fast
"doubling" algorithm to solve this equation

Test your iterative method against ``solve_discrete_lyapunov`` using matrices

.. math::

    A =
    \begin{bmatrix}
        0.8 & -0.2  \\
        -0.1 & 0.7
    \end{bmatrix}
    \qquad
    \Sigma =
    \begin{bmatrix}
        0.5 & 0.4 \\
        0.4 & 0.6
    \end{bmatrix}


Solutions
==================

Exercise 1
----------

Here's the iterative approach

.. code-block:: julia

    function compute_asymptotic_var(A, Sigma;
                                    S0 = Sigma * Sigma',
                                    tolerance = 1e-6,
                                    maxiter = 500)
        V = Sigma * Sigma'
        S = S0
        err = tolerance + 1
        i = 1
        while err > tolerance && i ≤ maxiter
            next_S = A * S * A' + V
            err = norm(S - next_S)
            S = next_S
            i += 1
        end
        return S
    end


.. code-block:: julia

    A =     [0.8 -0.2;
            -0.1 0.7]
    Sigma = [0.5 0.4;
             0.4 0.6]


Note that all eigenvalues of :math:`A` lie inside the unit disc:


.. code-block:: julia

    maximum(abs, eigvals(A))

Let's compute the asymptotic variance:


.. code-block:: julia

    our_solution = compute_asymptotic_var(A, Sigma)


Now let's do the same thing using QuantEcon's `solve_discrete_lyapunov()` function and check we get the same result


.. code-block:: julia

    using QuantEcon
    norm(our_solution - solve_discrete_lyapunov(A, Sigma * Sigma'))

