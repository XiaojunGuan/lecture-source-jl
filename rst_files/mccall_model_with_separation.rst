.. _mccall_with_sep:

.. include:: /_static/includes/lecture_howto_jl.raw

.. highlight:: julia


********************************************
Job Search II: Search and Separation
********************************************

.. index::
    single: An Introduction to Job Search

.. contents:: :depth: 2

Overview
===============


Previously :doc:`we looked <mccall_model>` at the McCall job search model :cite:`McCall1970` as a way of understanding unemployment and worker decisions

One unrealistic feature of the model is that every job is permanent

In this lecture we extend the McCall model by introducing job separation

Once separation enters the picture, the agent comes to view

*  the loss of a job as a capital loss, and

*  a spell of unemployment as an *investment* in searching for an acceptable job



The Model
============

The model concerns the life of an infinitely lived worker and

* the opportunities he or she (let's say he to save one character) has to work at different wages

* exogenous events that destroy his current job

* his decision making process while unemployed

The worker can be in one of two states: employed or unemployed

He wants to maximize

.. math::
    :label: objective

    {\mathbb E} \sum_{t=0}^\infty \beta^t u(Y_t)

The only difference from the :doc:`baseline model <mccall_model>` is that
we've added some flexibility over preferences by introducing a utility function :math:`u`

It satisfies :math:`u'> 0` and :math:`u'' < 0`



Timing and Decisions
-----------------------

Here's what happens at the start of a given period in our model with search and separation

If currently *employed*, the worker consumes his wage :math:`w`, receiving utility :math:`u(w)`

If currently *unemployed*, he

* receives and consumes unemployment compensation :math:`c`

* receives an offer to start work *next period* at a wage :math:`w'` drawn from a known distribution :math:`p_1, \ldots, p_n`

He can either accept or reject the offer

If he accepts the offer, he enters next period employed with wage :math:`w'`

If he rejects the offer, he enters next period unemployed

When employed, the agent faces a constant probability :math:`\alpha` of becoming unemployed at the end of the period


(Note: we do not allow for job search while employed---this topic is taken
up in a :doc:`later lecture <jv>`)


Solving the Model using Dynamic Programming
============================================

Let

* :math:`V(w)` be the total lifetime value accruing to a worker who enters the current period *employed* with wage :math:`w`

* :math:`U` be the total lifetime value accruing to a worker who is *unemployed* this period

Here *value* means the value of the objective function :eq:`objective` when the worker makes optimal decisions at all future points in time

Suppose for now that the worker can calculate the function :math:`V` and the constant :math:`U` and use them in his decision making

Then :math:`V` and :math:`U`  should satisfy

.. math::
    :label: bell1_mccall

    V(w) = u(w) + \beta [(1-\alpha)V(w) + \alpha U ]


and

.. math::
    :label: bell2_mccall

    U = u(c) + \beta \sum_i \max \left\{ U, V(w_i) \right\} p_i


Let's interpret these two equations in light of the fact that today's tomorrow is tomorrow's today

* The left hand sides of equations :eq:`bell1_mccall` and :eq:`bell2_mccall` are the values of a worker in a particular situation *today*

* The right hand sides of the equations are the discounted (by :math:`\beta`) expected values of the possible situations that worker can be in *tomorrow*

*  But *tomorrow* the worker can be in only one of the situations whose values *today* are on the left sides of our two equations

Equation :eq:`bell2_mccall` incorporates the fact that a currently unemployed worker will maximize his own welfare

In particular, if his next period wage offer is :math:`w'`, he will choose to remain unemployed unless  :math:`U < V(w')`

Equations :eq:`bell1_mccall` and :eq:`bell2_mccall` are the Bellman equations
for this model

Equations :eq:`bell1_mccall` and :eq:`bell2_mccall` provide enough information to solve out for both :math:`V` and :math:`U`

Before discussing this, however, let's make a small extension to the model


Stochastic Offers
-----------------

Let's suppose now that unemployed workers don't always receive job offers

Instead, let's suppose that unemployed workers only receive an offer with probability :math:`\gamma`

If our worker does receive an offer, the wage offer is drawn from :math:`p` as before

He either accepts or rejects the offer

Otherwise the model is the same


With some thought, you  will be able to convince yourself that :math:`V` and :math:`U`  should now satisfy

.. math::
    :label: bell01_mccall

    V(w) = u(w) + \beta [(1-\alpha)V(w) + \alpha U ]


and

.. math::
    :label: bell02_mccall

    U = u(c) +
      \beta (1 - \gamma) U
          + \beta \gamma \sum_i \max \left\{ U, V(w_i) \right\} p_i


Solving the Bellman Equations
-------------------------------


We'll use the same iterative approach to solving the Bellman equations that we
adopted in the :doc:`first job search lecture <mccall_model>`

Here this amounts to

#. make guesses for :math:`U` and :math:`V`

#. plug these guesses into the right hand sides of :eq:`bell01_mccall` and :eq:`bell02_mccall`

#. update the left hand sides from this rule and then repeat


In other words, we are iterating using the rules


.. math::
    :label: bell1001

    V_{n+1} (w_i) = u(w_i) + \beta [(1-\alpha)V_n (w_i) + \alpha U_n ]


and

.. math::
    :label: bell2001

    U_{n+1} = u(c) +
        \beta (1 - \gamma) U_n
         + \beta \gamma \sum_i \max \{ U_n, V_n(w_i) \} p_i


starting from some initial conditions :math:`U_0, V_0`

As before, the system always converges to the true solutions---in this case,
the :math:`V` and :math:`U` that solve :eq:`bell01_mccall` and :eq:`bell02_mccall`

A proof can be obtained via the Banach contraction mapping theorem


Implementation
================

Let's implement this iterative process

In the code you'll see that we use a type to store the various parameters and other
objects associated with a given model

This helps to tidy up the code and provides an object that's easy to pass to functions

The default utility function is a CRRA utility function

Activate the project environment, ensuring that ``Project.toml`` and ``Manifest.toml`` are in the same location as your notebook

.. code-block:: julia

    using Pkg; Pkg.activate(@__DIR__); #activate environment in the notebook's location

.. code-block:: julia
    :class: test

    using Test

.. literalinclude:: /_static/code/mccall/mccall_bellman_iteration.jl

The approach is to iterate until successive iterates are closer together than some small tolerance level

We then return the current iterate as an approximate solution

Let's plot the approximate solutions :math:`U` and :math:`V` to see what they look like

We'll use the default parameterizations found in the code above


.. code-block:: julia

    using Plots, LaTeXStrings


    mcm = McCallModel()
    V, U = solve_mccall_model(mcm)
    U_vec = U .* ones(length(mcm.w_vec))

    plot(mcm.w_vec,
        [V U_vec],
        lw = 2,
        α = 0.7,
        label = [L"$V$" L"$U$"])

.. code-block:: julia
    :class: test

    @testset "First Plot Tests" begin
        @test U ≈ 45.623263135173644 # U value
        @test V[3] ≈ 45.58068793055933 # Arbitrary V

    end


The value :math:`V` is increasing because higher :math:`w` generates a higher wage flow conditional on staying employed


The Reservation Wage
=======================

Once :math:`V` and :math:`U` are known, the agent can use them to make decisions in the face of a given wage offer

If :math:`V(w) > U`, then working at wage :math:`w` is preferred to unemployment

If :math:`V(w) < U`, then remaining unemployed will generate greater lifetime value

Suppose in particular that :math:`V` crosses :math:`U` (as it does in the preceding figure)

Then, since :math:`V` is increasing, there is a unique smallest :math:`w` in the set of possible wages such that :math:`V(w) \geq U`

We denote this wage :math:`\bar w` and call it the reservation wage

Optimal behavior for the worker is characterized by :math:`\bar w`

*  if the  wage offer :math:`w` in hand is greater than or equal to :math:`\bar w`, then the worker accepts

*  if the  wage offer :math:`w` in hand is less than :math:`\bar w`, then the worker rejects

Here's a function called `compute_reservation_wage` that takes an instance of a McCall model and returns the reservation wage associated with a given model

It uses `np.searchsorted <https://docs.scipy.org/doc/numpy/reference/generated/numpy.searchsorted.html>`__ to obtain the first :math:`w` in the set of possible wages such that :math:`V(w) > U`

If :math:`V(w) < U` for all :math:`w`, then the function returns `np.inf`

.. literalinclude:: /_static/code/mccall/compute_reservation_wage.jl

Let's use it to look at how the reservation wage varies with parameters

In each instance below we'll show you a figure and then ask you to reproduce it in the exercises


The Reservation Wage and Unemployment Compensation
----------------------------------------------------

First, let's look at how :math:`\bar w` varies with unemployment compensation

In the figure below, we use the default parameters in the `McCallModel` type, apart from
`c` (which takes the values given on the horizontal axis)

.. figure:: /_static/figures/mccall_resw_c.png
    :scale: 100%

As expected, higher unemployment compensation causes the worker to hold out for higher wages

In effect, the cost of continuing job search is reduced


The Reservation Wage and Discounting
----------------------------------------------------

Next let's investigate how :math:`\bar w` varies with the discount rate

The next figure plots the reservation wage associated with different values of
:math:`\beta`

.. figure:: /_static/figures/mccall_resw_beta.png
    :scale: 100%

Again, the results are intuitive: More patient workers will hold out for higher wages


The Reservation Wage and Job Destruction
----------------------------------------------------

Finally, let's look at how :math:`\bar w` varies with the job separation rate :math:`\alpha`

Higher :math:`\alpha` translates to a greater chance that a worker will face termination in each period once employed


.. figure:: /_static/figures/mccall_resw_alpha.png
    :scale: 100%

Once more, the results are in line with our intuition

If the separation rate is high, then the benefit of holding out for a higher wage falls

Hence the reservation wage is lower


Exercises
=============


Exercise 1
----------------

Reproduce all the reservation wage figures shown above

Exercise 2
------------

Plot the reservation wage against the job offer rate :math:`\gamma`

Use

.. code-block:: julia

	grid_size = 25
	γ_vals = range(0.05, stop = 0.95, length = grid_size)


Interpret your results


Solutions
==========

Exercise 1
----------

Using the `compute_reservation_wage` function mentioned earlier in the lecture,
we can create an array for reservation wages for different values of :math:`c`,
:math:`\beta` and :math:`\alpha` and plot the results like so

.. code-block:: julia

    grid_size = 25
    c_vals = range(2, stop = 12, length = grid_size)
    w_bar_vals = similar(c_vals)

    mcm = McCallModel()

    for (i, c) in enumerate(c_vals)
        mcm.c = c
        w_bar = compute_reservation_wage(mcm)
        w_bar_vals[i] = w_bar
    end

    plot(c_vals,
         w_bar_vals,
         lw = 2,
         α = 0.7,
         xlabel = "unemployment compensation",
         ylabel = "reservation wage",
         label = L"$\bar w$ as a function of $c$")

.. code-block:: julia
    :class: test

    @testset "Solutions 1 Tests" begin
        @test w_bar_vals[10] ≈ 11.35593220338983
        @test c_vals[1] == 2 && c_vals[end] == 12 && length(c_vals) == 25
        @test w_bar_vals[17] - c_vals[17] ≈ 4.72316384180791 # Just a sanity check on how these things relate.
    end


Exercise 2
----------

Similar to above, we can plot :math:`\bar w` against :math:`\gamma` as follows

.. code-block:: julia

    grid_size = 25
    γ_vals = range(0.05, stop = 0.95, length = grid_size)
    w_bar_vals = similar(γ_vals)

    mcm = McCallModel()

    for (i, γ) in enumerate(γ_vals)
        mcm.γ = γ
        w_bar = compute_reservation_wage(mcm)
        w_bar_vals[i] = w_bar
    end

    plot(γ_vals,
         w_bar_vals,
         lw = 2,
         α = 0.7,
         xlabel = "job offer rate",
         ylabel = "reservation wage",
         label = L"$\bar w$ as a function of $\gamma$")

.. code-block:: julia
    :class: test

    @testset "Solutions 2 Tests" begin
        @test w_bar_vals[17] ≈ 11.35593220338983 # same as w_bar_vals[10] before.
        @test γ_vals[1] == 0.05 && γ_vals[end] == 0.95 && length(γ_vals) == grid_size == 25
    end

As expected, the reservation wage increases in :math:`\gamma`

This is because higher :math:`\gamma` translates to a more favorable job
search environment

Hence workers are less willing to accept lower offers
