# Lab 7: answers

Your name: James Frink

For each issue, fill in Reproduce to Confirm **before** you change any code. Then fix it and test it.

## Issue 3: report.py crashes

- Reproduce (the command, and the last line it printed):   File "D:\Coding\ECE105\Code\Lab7_Code\report.py", line 18, in <module>
    print(f"{rank}. f = {p.get_fitness():.3g} at {np.round(p.get_pbest(), 4)}")
                                                           ~~~~~~~~~~~^^
  File "D:\Coding\ECE105\Code\Lab7_Code\pso_v3.py", line 46, in get_pbest
    return self.gbest
- Where (the lowest frame: file, line, and the code on that line): pso_v3.py, line 46, inside get_pbest, gbest is not attribute of the object
- Hypothesis (one sentence):The get_pbest() method tries to return self.gbest instead of self.pbest, but gbest is not an attribute of the Particle object.
- Confirm (an experiment of 1 to 3 lines, and what it printed):
- Fix (the line before, and after; which file?): changed in pso_v3.py,    def get_pbest(self):
        """The particle's best position so far."""
        return self.pbest
- Test (its name; it failed before the fix, and passes after): pso_v3.py:46: AttributeError
============================ short test summary info =============================
FAILED test_pso_v3.py::test_get_pbest - AttributeError: 'Particle' object has no attribute 'gbest'. Did you mean: 'pbe...
========================== 1 failed, 5 passed in 0.61s ===========================
test_pso_v3.py ......                                                       [100%]

=============================== 6 passed in 0.71s ================================

## Issue 4: the sphere crashes on [0, 10]

- Reproduce (the command, and the last line it printed):
(ece105) PS D:\Coding\ECE105\Code\Lab7_Code> python pso_v3.py sphere 2 20 0 10 100
Traceback (most recent call last):
  File "D:\Coding\ECE105\Code\Lab7_Code\pso_v3.py", line 149, in <module>
    x, value, history = run_pso(function, dim, size, xmin, xmax, max_iter)
                        ~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\Coding\ECE105\Code\Lab7_Code\pso_v3.py", line 139, in run_pso
    gidx, history = pso_solve(function, swarm, max_iter)
                    ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\Coding\ECE105\Code\Lab7_Code\pso_v3.py", line 126, in pso_solve
    if (old - history[-1]) / old < tol:
       ~~~~~~~~~~~~~~~~~~~~^~~~~
ZeroDivisionError: division by zero
- Where (the lowest frame: file, line, and the code on that line):
File "D:\Coding\ECE105\Code\Lab7_Code\pso_v3.py", line 126, in pso_solve
    if (old - history[-1]) / old < tol:
- Hypothesis (one sentence): why is the number you divide by exactly 0?
The sphere function has a minimum of zero at the origin.
When particles move outside the lower bound of zero,
np.clip may force their coordinates to exactly zero.
This could make the best fitness exactly zero, causing
the early stopping calculation to divide by zero.
- Confirm (an experiment of 1 to 3 lines, and what it printed):
Changing to patience = 1000: 
(ece105) PS D:\Coding\ECE105\Code\Lab7_Code> python pso_v3.py sphere 2 20 0 10 100
sphere: best x = [0. 0.], f(x) = 0, iterations run: 100
- What should the early stop do when the best value is exactly 0, and stays 0?
The early stop should stop the optimization after 20 iterations without improvement, even when the best fitness value is exactly zero.
- Fix (the line before, and after):
   if (old - history[-1]) / old < tol:
                break
After: 
            if old == 0 or (old - history[-1]) / old < tol:
                break
- Test (its name; it failed before the fix, and passes after):
test_early_stop_zero
============================ short test summary info =============================
FAILED test_pso_v3.py::test_early_stop_zero - RuntimeError: Error creating swarm
========================== 1 failed, 6 passed in 0.44s ===========================
After:
platform win32 -- Python 3.14.7, pytest-9.1.1, pluggy-1.6.0
rootdir: D:\Coding\ECE105\Code\Lab7_Code
collected 7 items                                                                 

test_pso_v3.py .......                                                      [100%]

=============================== 7 passed in 0.27s ================================
## Issue 5: Rastrigin's answer is wrong

- Reproduce (the command, and what it printed):
(ece105) PS D:\Coding\ECE105\Code\Lab7_Code> python pso_v3.py rastrigin 2 20 -5.12 5.12 100
rastrigin: best x = [0.     1.6587], f(x) = 10, iterations run: 45
- Expected (what, and why):
I expected the Rastrigin function to return a fitness value close to 0 at a position near [0, 0], because its global minimum is 0 at the origin. Instead, the optimizer returned a fitness value around 10, suggesting either an error in the function or that the swarm became stuck in a local minimum.
- Hypothesis (one sentence): is the swarm stuck, or is something else wrong?
I suspect the swarm is not stuck in a local minimum, but that there is an error in the Rastrigin function's implementation causing it to calculate incorrect fitness values.
- Confirm (an experiment of 1 to 3 lines, and what it printed): >>> rastrigin(np.array([0.0, 0.0]))
np.float64(10.0)
>>> rastrigin(np.array([1.0, 0.0]))
np.float64(11.0)
>>> rastrigin(np.array([0.0, 1.0]))
np.float64(10.0)
- Fix (the line before, and after):
Not running last coordinate of the input array Before:
return 10 * n + sum(x[i] ** 2 - 10 * np.cos(2 * np.pi * x[i]) for i in range(n - 1))
After:
return 10 * n + sum(x[i] ** 2 - 10 * np.cos(2 * np.pi * x[i]) for i in range(n))
- Test (its name; it failed before the fix, and passes after):
test_rastrigin()
Before:
    def test_rastrigin():
        """Issue 5: Rastrigin must include every coordinate."""
>       assert np.isclose(rastrigin(np.array([0.0, 0.0])), 0)
E       assert np.False_
E        +  where np.False_ = <function isclose at 0x00000217780F7330>(np.float64(10.0), 0)
E        +    where <function isclose at 0x00000217780F7330> = np.isclose
E        +    and   np.float64(10.0) = rastrigin(array([0., 0.]))
E        +      where array([0., 0.]) = <built-in function array>([0.0, 0.0])
E        +        where <built-in function array> = np.array

test_pso_v3.py:56: AssertionError
========================================= short test summary info ==========================================
FAILED test_pso_v3.py::test_rastrigin - assert np.False_
======================================= 1 failed, 7 passed in 0.45s ========================================
After:
=========================================== test session starts ============================================
platform win32 -- Python 3.14.7, pytest-9.1.1, pluggy-1.6.0
rootdir: D:\Coding\ECE105\Code\Lab7_Code
collected 8 items                                                                                           

test_pso_v3.py ........                                                                               [100%]

============================================ 8 passed in 0.27s =============================================

## Three questions

1. Issue 3's bug was in `pso_v3.py` from the start, yet every run and all five of Wednesday's tests
   passed. Why?
   The original tests never called the get_pbest() method, so the incorrect reference to self.gbest was never executed. The optimizer could still run normally because it accessed the particle's best position through other parts of the code.
2. Issue 4: how can the best value be exactly 0.0? Why does the same run on [-10, 10] never crash?
   When particles move outside the bounds [0, 10], np.clip will force their coordinates to exactly 0. The sphere function is the sum of the squared coordinates, a particle at [0, 0] has a fitness of exactly 0. On [-10, 10], zero is inside the search range rather than at the boundary, so clipping does not force particles to the origin. The optimizer typically gets very close to zero without reaching exactly 0, avoiding the division by zero.
3. Issue 5: the user guessed a local minimum. What showed that the swarm was not stuck?
   I called the Rastrigin function directly at points where I knew the expected values. The function returned 10 instead of 0 at [0, 0], and 10 instead of 1 at [0, 1]. This showed that the function itself was calculating incorrect values. The loop used range(n - 1), which ignored the last coordinate, so the problem was in the function rather than the swarm getting stuck.