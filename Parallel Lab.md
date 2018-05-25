# Current Problems
1. Is there a way to parallize OMP_GaussianBlur?
Now the problem is that each step is dependent on the previous one so don't know how to parallize.
Parallize inner loop only? Seems no overhead of creating threads, see this: https://stackoverflow.com/questions/4906983/parallelize-inner-loop-using-openmp
** It seems parallizing inner loop didn't speed up the program. Instead it even slowed down it.

# Commands 

### parallel
(1) Everything declared outside the **#pragma omp parallel** will be shared be default.
(2) #pragama omp parallel will parallize the block after it if no "{ }" specified.
