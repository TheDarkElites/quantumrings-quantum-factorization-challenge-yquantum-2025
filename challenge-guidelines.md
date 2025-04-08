## Guidelines for a Genuine (Yet Fun) Shor’s Implementation

Below are some friendly guidelines to help you build a **truly** quantum solution, without accidentally relying on trivial gcd checks or small-scope hacks.

1. **Separate Counting vs. Working Registers**  
   - We encourage you to structure your circuit using **at least two distinct registers**:
     1. A **counting (phase) register** for your QFT-based phase estimation.  
     2. A **working register** (often called the computational register) to actually hold or build \((a^x \bmod N)\).
   - This is how Shor’s algorithm typically tracks modular exponentiation. Even if you experiment with partial or approximate methods, **please** keep these registers conceptually separate to avoid “one-phase-fits-all” shortcuts.

2. **Modular Arithmetic with a Quantum Adder**  
   - **Yes**, adders can be **ripple-carry**, **carry-lookahead**, **QFT-based**, or other creative designs.
   - Partial or approximate adders are fine, as long as you’re genuinely handling **addition** or **multiplication** on the working register in a reversible way.
   - The essential point: we want a *bona fide* quantum subroutine for `(x * a) mod N` (or `(a^x) mod N`), **not** a mere global phase shift or toggling.

3. **Direct Multiplication or Repeated Squaring**  
   - Standard Shor’s does **repeated squaring**: for each bit \(j\) of the counting register, multiply by `(a^{2^j} mod N)` under control of that bit.
   - Alternatively, you might do one big exponent sub-block—**as long** as it’s genuinely computing `(a^x) mod N`.
   - Bottom line: we’d like to see actual *modular exponentiation logic*, not just a single “phase gate” representing \(a^{2^j}\).

4. **Beware the GCD Trick**  
   - It’s perfectly fine to do a quick check if `gcd(a,N) != 1` before running your circuit, but if your solution **solely** tries random `a` until `gcd(a,N) > 1`, that’s effectively classical.
   - We want to see genuine quantum advantage from the phase-estimation step. Don’t let random gcd checks become your entire factoring strategy!

5. **Measure a Phase, Then Do Classical Fraction**  
   - After applying the (inverse) QFT on the counting register, measure it. Use a rational approximation (e.g., continued fractions) to interpret `m / 2^n ≈ k / r`.
   - We’re not requiring a specific approach, but **some** classical step to get \(r\) is crucial. Skipping or faking the fraction step loses the essence of Shor’s.

6. **We Provide Semiprimes up to ~100 Bits**  
   - Even if you can’t reach the largest, aim to handle numbers bigger than just 15 or 21.
   - A solution hard-coded to a single small semiprime or performing a single “phase shift” is **not** considered universal.

7. **Have Fun, but Keep It Real**  
   - Experimenting with partial carry or skipping small angles in the QFT is awesome—**go for it**!
   - Just ensure your approach is truly building `(a^x) mod N` in that working register (rather than a single-phase shift or random gcd tries).
   - If your code never has a second register or never does quantum addition for each `2^j`, we might not judge it as a “Shor’s” solution.

**In Summary**: We’re excited for innovative quantum circuits, but please ensure the **core** Shor’s logic is present:  
1) Counting (phase) register + QFT  
2) A working register for `(a^x mod N)`  
3) Some classical fraction approach to extract `r` from measurement  
4) Final `gcd(a^{r/2} ± 1, N)` to find a factor.

That way, we can all see a genuine demonstration of quantum period finding in action!
