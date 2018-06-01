# Current Problems
1. Is there a way to parallize OMP_GaussianBlur?
Now the problem is that each step is dependent on the previous one so don't know how to parallize.
Parallize inner loop only? Seems no overhead of creating threads, see this: https://stackoverflow.com/questions/4906983/parallelize-inner-loop-using-openmp
** It seems parallizing inner loop didn't speed up the program. Instead it even slowed down it.
And since function blur is called 20 times, it seems not a good idea to parallize anything inside of it.


# Commands 

### parallel
(1) Everything declared outside the **#pragma omp parallel** will be shared be default.
(2) #pragama omp parallel will parallize the block after it if no "{ }" specified.

If the load is not well balanced, the performance will decrease drastically.


### Cache
Change the order of loop to z, y, x to improve space locality


# Data
xMax = yMax = zMax = 128;

# logs
(1) reduce function calls to addition(1 and 3): 1.28
(2) reduce ....... (1,2,3): 1.29

# Important tips!!
1. Since floating point multiplication is not associative, the following two codes may not necessarily equivalent:
last_val = u * (index * index) and last_val = u * (index * index). This is important when the amount of calculation is huge.
2. loop unrolling
for (i = 128; i >= 1; i -= 2) {
    a[i] ...
    a[i-1] ...
}
if (i >= 0) {
    ...
}

3. I found that using tiling here indeed can speed up, but is still slower than making inner-most loop row-major
index = z * xyMax + y * xMax + x
```c
		for(z = 0; z < zMax; z += B)
		{
			for(x = 0; x < xMax; x += B)
			{
				for(y = yMax - 2; y >= 0; y -= B)
				{
					for (z1 = z; z1 < zMax && z1 < (z+B); z1++) {
						for (x1 = x; x1 < xMax && x1 < (x+B); x1++) {
							for (y1 = y; y1 >= 0 && y1 > (y-B); y1--) {
								u[Index(x1, y1, z1)] += u[Index(x1, y1 + 1, z1)] * nu;
							}
						}
					}
				}
			}
		}
    is faster than 
    for (z)
        for (x)
            for (y)        
    but is slower than 
    for (z)
        for (y)
            for (x)
    this is the same as
    for (y) 
        for (z)
            for (x)
```
