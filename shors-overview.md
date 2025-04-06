# Shor's Overview

Your [challenge](./README.md) is to factor the biggest semiprime number of the hackathon by developing your own Shor's factoring algorithm.

As a Reminder, we do not want to use brute force or classical methods like:
* Pollard’s Rho
* Pollard’s p-1
* Quadratic Sieve
* Number Field Sieve (NFS) (the fastest known classical method for large 𝑁)

## Shor's Factoring Algorithm

Shor's was among the first algorithms demonstrating the benefits of quantum computing.  Here's a video of Dr Shor himself giving a quick description of the algorithm: [Shor's Idea](https://youtu.be/hOlOY7NyMfs)  

### 1) Pick **a** to Evaluate the Modular Exponentiation

The first step is to create a list of numbers generated from a cyclic function, we want to find the **order** of **a**.   In practice, this means we are guessing values of **a** and trying them in our algorithm.  These guesses generate values of the period **r**.  A good value of **r** is even, so we can use it to look for a solution that *fits* the perodicity of the problem.  Our guess of **a** is between 1 and N, and it's always worth checking to see if the guess is actually a solution.

### 2) Build Quantum Circuit

The next step is to build your circuit to exponentiate the function, generate the function, measure the output, and find the period.

    a) Counting Register (for phase estimation)

    b) Work Register (for modular exponentiation)

    c) Controlled Modular Exponentiation (this is the hard part)

    d) Quantum Fourier Transformation (to find the period)

    e) Measure

### 3) Post Processing

If you get a good value of **r** then check to see if you found an answer:

    If r is **even**, compute:

    gcd(a^(r/2) - 1, N) and gcd(a^(r/2) + 1, N).

    If either gcd is > 1 and < N, you have a factor of N.

## Learning Example of Shor's

Let's drill down with a specific example, factoring 15.  This is the smallest semiprime number that is interesting to factor.  A semiprime number is the product of two prime numbers.  You can find a list of semiprime numbers to use in this challenge [HERE](./semiprimes.py).

This example is small enough that we can directly build and understand the circuit. [HERE](shors.ipynb)
The notebook describes the steps involved in the algorithm.  Take a look to understand how the pieces fit and what we're trying to do.

This is not a complete universal example.  It is hard-coded to factor 15, but it shows how you get from theory to code.  The rest will be part of the hackathon challenge.

Qubit set [0, 1, 2] is our *counting register* that gets initialized with a Hadamard to explore all of possible values for the exponent **x** in superposition.  Qubit set [3, 4, 5] is for our *modular exponentiation*.  This is a teaching example that is hard-coded for **15**, rather than a universial approach that lets you choose a semiprime.  

To do a universal Shor's you need to 

* construct a controlled subscript, **j**, for the *counting register*:

        > a^(2j) mod N

        *where j is the subscript for the counting register*.

* Reuse a general pattern for modular multiplication.

After the QFT we measure to find the period.  

At the end of the notebooks you get a graph.  What is this?  Everything was going fine, now what did we just measure?  Were you expecting a solution?  These are **m** values that are the *phase* result of the QFT.  

Convert the binary result to a number, like 0010 -> 2 or 0110 -> 4.  So from our graph we get the value of m split between 0, 2, 4, 6.  A ‘real’ or ‘universal’ Shor’s circuit would have more gates, multiple controlled exponent operations, and a classical fraction step to interpret the outcome.  All of that would give us a strong single peak for **m**.

In our toy example we can **check each value of m** (excluding 0).

What you want is **r**.   We can find it using the ratio:

    m / 2^n ≈ k / r

So let's skip 0, which is out of range, and try m = 2.  We have three qubits in our register, so 2^n = 8.

    2 / 8 ≈ 1 / r

gives r = 4.  This is a known solution so we'll use it.  But in a real Shor's you should just have one best value for m and wouldn't need to try each value.

Next is the classical processing. Recall that we are factoring N =15 and our guess is a = 7. 

    gcd(a^(r/2) + 1, N) and gcd(a^(r/2) - 1, N)

gives

    gcd(7^(r/2) + 1, 15) = gcd(49 + 1, 15) = gcd(50, 15) = 5
    
    and 

    gcd(7^(r/2) - 1, 15) = gcd(49 - 1, 15) = gcd(48, 15) = 3

There you have it.  The factors are 3 and 5.

What next?


## Using Quantum Rings

Goal: Implement a universal Shor’s algorithm that can factor some large semiprime from the hackathon’s list. You get 200 qubits, so push it as far as you can without overshooting the time limit!

Most quantum circuit simulators are unable to use provide more than 30 or so qubits to use in your algorithm.  
* Quantum Rings will provide you **64 qubits** for free!  
* If you are a student, you get **128 qubits**.  
* But wait, there's more... 
* If you are attending the YQuantum 2025 hackathon, you get **200 qubits** free for a year.  

See the full documentation on [Quantum Rings SDK](https://quantumrings.dev/docs).  We also have ChatGPT base online [LLM Assistant](https://chatgpt.com/g/g-67d47e3159f88191b20c3aec22410021-quantum-rings-code-help).

In the above example we used core Quantum Rings functionality.  This provides direct access to the Quantum Rings SDK, but in the documentation you will find integration with Qiskit, that lets you use it's built in Qiskit functions like QFT.  You can also use other quantum computing languages like Pennylane, Cirq, and Q# by converting their circuit to QASM as an intermediate representation that can be understood by the Quantum Rings SDK. 

#### Build a fully universal Shor's

The notebook demo of Shor's is not universal and also does not provide a final solution.  You will want to have a fully functional algorithm.  To get see a full description you can look in [Nielsen & Chuang, *Quantum Computation and Quantum Informaiton*](http://almuhammadi.com/sultan/books_2020/Nielsen_Chuang.pdf), where it is described as *phase estimation* (chapter 5.2).

Hand coding a solution gives you the most control and understand of your code.  But popular LLMs understand both the algorithm and the programming language.  Using LLM tools like ChatGPT, Perplexity, Bard, Claude, *etc.* can help you quickly prototype your code and explore ideas.

You will need to balance between trying to factor the largest semiprime, from the list, with consideration of the limited time.  If you code takes 24 hours to run, it doesn't leave time for testing, iterations, and modification.  Develop with smaller numbers and when you're satisfied, try bigger numbers to see how the time scales.

### Brainstorming Ideas

#### Implementing the Controlled Modular Exponentiation

* A simple approach is **Direct Multiplication**, which you can try step-by-step
* The **Repeated Squaring** is more efficient and more often used.

#### Quantum Adder/Arithmetic Approach

* Ripple-Carry Adder (depth O(n))
* Carry Lookahead or Parallel Adders (depth O(log(n)))
* QFT-based Adders (lower depth, more complex)
* Partially Reversible Adders (reduce ancilla)

#### QFT Optimization

* Skip small angles in the inverse QFT (reduces fidelity)
* Mid-circuit measure with qubit reuse (fewer qubits needed)

#### Partial vs Full Exponentiation

* Full exponentiation is about O(n) wide and O(n) deep so total O(n^2)
* Partial exponentiation can produce smaller blocks, for example n/2.  Faster but lower probability.

#### Hackathon Musing

* Pick strategic "a" for easier exponentiation (power of 2) or (close to a prime).  No guarantees.
* Minimizing the Counting Register, rather than n use log2(n)+margin.
