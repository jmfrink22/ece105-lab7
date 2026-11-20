# AI log: Lab 7

Your name: James Frink

Tool: ChatGPT Edu (if you used another, say which) ChatGPT

For each issue: what you asked, what it said, and whether it was right. Paste both exactly; you may
cut parts of the answer that have nothing to do with the bug.

## Issue 3: report.py crashes

### My prompt
I'm debugging a Python particle swarm optimization program. When I run python report.py, I get an AttributeError saying that a Particle object has no attribute gbest.

The traceback points to pso_v3.py, line 46, inside the get_pbest() method:

return self.gbest

What could cause this error? Give me one or two hypotheses I can test before changing the code.

### Its answer (word for word)
My hypothesis is that get_pbest() is trying to return an attribute that doesn't belong to the Particle object.

In particle swarm optimization:

pbest represents a particle's personal best position.

gbest represents the best position found across the swarm.

The method is named get_pbest(), but it attempts to return self.gbest.

That mismatch is suspicious.
### Right, partly right, or wrong? How do you know?
The AI was right. The get_pbest() method was trying to return self.gbest, which doesn't exist in the Particle object. I confirmed that the particle had a pbest attribute but not a gbest attribute. I fixed the method in pso_v3.py to return self.pbest without changing report.py. The new test passed.
(Did it fix `pso_v3.py`, or change `report.py` so the error goes away?)
Fixed pso_v3.py even though the first one I didn't really need Chat to figure out.
## Issue 4: the sphere crashes on [0, 10]

### My prompt
I'm debugging the early stopping logic in a particle swarm optimization program. When I run python pso_v3.py sphere 2 20 0 10 100, I get a ZeroDivisionError at this line:

if (old - history[-1]) / old < tol:

The same command using bounds [-10, 10] works. Why might the best fitness become exactly zero when the lower bound is zero? Give me a hypothesis I can test without changing the code.
### Its answer (word for word)
The sphere function has its minimum at the origin. When the search bounds are [0, 10], particles that move below the lower bound may be clipped to exactly zero. This could produce a fitness value of exactly zero, causing the early stopping calculation to divide by zero.

We haven't confirmed this yet.
### Right, partly right, or wrong? How do you know?
The AI was right about the likely cause. I changed patience from 20 to 1000 and reran the program. It completed all 100 iterations with a best position of [0, 0] and a fitness of exactly 0. This showed that the optimizer could reach zero without crashing when early stopping was disabled. I then added a check for old == 0 before the division and restored patience=20. The new test confirmed that the program could stop early instead of crashing.
(Did it say why the number was exactly 0, or only that you cannot divide by 0? With its fix, does the
run still stop early?)
Yes wuth the fix it does stop early without throwing an error.
## Issue 5: Rastrigin's answer is wrong

### My prompt
I'm debugging a particle swarm optimizer using the Rastrigin function. The optimizer returns a fitness value around 10, but the global minimum should be 0 at the origin.

A user suggested the swarm might be stuck in a local minimum. Could the problem instead be in the Rastrigin function itself? What experiment could I perform to determine which explanation is correct? Don't provide corrected code yet.
### Its answer (word for word)
Your professor's bug report suggests that the swarm might be stuck in a local minimum.

That's one possibility, but we shouldn't assume it's correct.

Another hypothesis is that the implementation of the Rastrigin function contains a mathematical error.

Our goal is to test the function independently of the optimizer.
### Right, partly right, or wrong? How do you know?
The AI was right to suggest testing the function independently. I called Rastrigin directly at three known points and got 10 for [0, 0], 11 for [1, 0], and 10 for [0, 1]. The correct values should have been 0, 1, and 1. This showed that the function itself was incorrect rather than the swarm simply getting stuck. I found that range(n - 1) excluded the last coordinate. Changing it to range(n) corrected the calculation, and the new test passed.
(Did it agree that the swarm was stuck in a local minimum? What did your experiment show?)
I did not agree it was getting stuck, the test showed it was getting to a different value than what was expected when fed bounds [0,1] and 2 other similar bounds.