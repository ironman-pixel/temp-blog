---
date: 2026-08-15
tags:
  - cs
  - algorithm
  - time-complexity
  - space-complexity
  - data-structure
---
# What to analize
1. time

## Time
- from a basic level:
  we just suppose that each `statement` takes one unit of time.

## Space
- how many variables


---
## Frequency count method
 
```
Algorithm Add(A,B,n)
{
	for(i=0;i<n;i++)                    // n+1
	{
		for (j=0;j<n;j++)               // n * (n+1)
		{
			c[i, j]=A[i, j] + B[i, j];  // n * n
		}
	}
}
```

### Time Complexity
`(n+1) + n(n+1) + n² = 2n² + 2n + 1`
=> `O(n^2)`

### Space Complexity
`A`  = `n^2`
`B`  = `n^2`
`C`  = `n^2`
`n`  = `1`
`i`  = `1`
`j`  = `1`
total = `3n^2 + 3`
=> `O(n^2)`