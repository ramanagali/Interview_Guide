# Data Structures and Algorithams 

<hr />

## Algorithams 
**An algorithm** -  Is a set of well-defined instructions to solve a particular problem. 
It takes a set of input and produces a desired output

- Input
- Output
- Definiteness 	- It must be clear
- Finiteness 	- It must stop
- Effectiveness - some purpose

**How to Analyse Alogithm**

1. Time  -  must be Time efficient
2. Space - must be memory effficient
3. N/W 	 -  
4. Power - 
5. CPU Registers - 

Every statement/line is 1 unit of time

**Frequency Count Method**
- Means, the number of times the statement is executed in the program

- To calculate the time, consider the degree
    ```
	- sum(a,b) 		    => f(n) = 2n +3                  => O(n)
	- add(a,b,n)        => f(n) = 2n2 + 2n + 1           => O(n2)
	- multiply(a,b,n)   => f(n) = 3n3 + 3n2 + 2n + 1    => O(n3)
    ```

- To Calculate space complexity, consider all variables
    ```
	- sum(a,b)		    => s(n) = n+3        => O(n)
 	- add(a,b,n)		=> s(n) = 3n2 + 3    => O(n2)
 	- multiply(a,b,n)	=> s(n) = 3n2 +4     => O(n2)
    ```

**Time Complexity Examples**

```
- for(i=0;i<n;i++) 		        => O(n)
- for(i=n;i>0;i--) 		        => O(n)
- for(i=0;i<n;i=i+2)		n/2 => O(n)
- for(i=0;i<n;i++)	
    for(j=0;j<i;j++)	        => O(n^2)
- for(i=1;i<n; i=i++) 
  for(j=0;j<n;j++)	            =>O(n^2)
- p=0
  for(i=0;p<=n;i++) 		    =>O(root N)

- for(i=1;i<n; i=i*2)	        => O(log n base 2)
- for(i=1;i<n; i=i*3)           => O(log n base 3)
- for(i=n;i>1; i=i/2)           => O(log n base 2 )

- for(i=0;i*i<n;i++) 	        => O(root n)

- for(i=0;i<n;i++) 	
    stmt
  for(j=0;j<i;j++)	
    stmt                        => O(n)
- p=0
  for(i=0;i<n;i*2) 	
    stmt
  for(j=0;j<p;j*2)	
	stmt                        => O(log log n)

- for(i=0;i<n;i++) 	
    stmt
  for(j=0;j<i;j=n*2)	
	stmt                        =>O(nlogn)
```
**Big O noation is for...**
- Time Complexity (how much runtime it will take)
- Space Complexity (how much memory it will take)

**Types of Time Complexities (Classes of Funcitons)**
- Constant Time Complexity 		-  O(1)
- Linear Time Complexity 		-  O(n)
- Logarithmic Time  Complexity 	-  O(log n)
- Quadratic Time Complexity 	- O(n ** 2)
- Cubic Time Complexity			- O(n ** 3)
- Exponential Complexity		- O(2 power n)

**How to calculate Time Complexities**
- Mathematical way
- Programatic or Practical way
  1. Find the Fast increasing term
  2. Remove the cofficient in that term
   
**Compare Class of Functions**
```
1 < log n < /n < n < n log n < n2 < n3 ... 2n < 3n < Nn < O(1) < O(logN) < O(N) < O(NlogN) < O(N^2) < O(2^N)< O(N!)
```

**Log Values**
* Logn n = 1
* logn 1 = 0

```  
log1=0 	1	1	2
log2=1	2 	4	4
log4=2 	4 	16 	16
log8=3 	8 	64 	256
log16=4 16 	256 65536
```

**Asymptotic Notations - Big Oh**
1. BigO  	Upper bound		(f(n) <= BigO(n) )
2. Omega  	Lower bound 	(f(n) >= Omega(g(n) )
3. Theta 	Average bound 	(f(n) = Theta(n) )

* BigO or Omega is when we are not sure
* Theta gives best 
* IF Theta can't find, go for BigO or Omega

<hr />

## Data Structures
Data Structures are way storing and organizing data in a computer for efficient access and modification
