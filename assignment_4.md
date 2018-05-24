# Assignment 4 -- Optimisations

*due on 27 June 2018*

## Task 1

- Implement the generation of control-flow graphs (CFGs) based on your TAC --- one graph per function.
- Provide a mechanism to print a CFG in the DOT format.

## Task 2

- Implement *global redundancy elimination* as work-list algorithm.
- Add a switch `-O` (no argument) to your `mCc` executable, which enables optimisation.
- Show the result using the example from the lecture:
    - Present the mC input program.
    - Show the corresponding CFG.
    - Show the relevant TAC before optimisation.
    - Show the relevant TAC after optimisation.
    - (optional) Show a table of the calucation, similar to the one from the lecture.
    - Measure the impact of this optimisation.
        - For a single run, the improvement will be quite hard to notice.
          Consider putting the lecture example in a loop with a fixed number of iterations.
        - Present the measurement results.
        - Tell which hardware was used for the measuring process.
- Do the same for another example, containing a loop -- so that there is a backward edge in the CFG processed by the optimisation logic.

Put all this into `doc/optimisation.md`, images go into `doc/images`.

## Submission

Follow the steps stated in the first assignment, using the following command to build the archive and the link below.

    $ git archive --prefix=team_XX_assignment_4/ --format=zip HEAD > team_XX_assignment_4.zip

:email: [send email](mailto:alexander.hirsch@uibk.ac.at?subject=703602%20-%20Assignment%204)
