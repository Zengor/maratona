# Read matrix
3 exemploes de leitura pra ilustrar leitura curta e efetiva

Quando a matriz está separada por espaços
```py
grid = [map(int, input().split()) for i in range(n)]
```
Para ler arquivos de caracteres (não números), com membros separados por espaço, como acima.
```py
with open(filename, 'r') as f:
	grid = [line.split() for line in f]
```
Se não for separado por espaço, usar algo como
```py
with open(filename, 'r') as f:
	grid = [[c for c in line if c != '\n'] for line in f]
```

# Range of chars
Ver as constantes do módulo `string`:
```
string.ascii_lowercase
string.ascii_uppercase
string.ascii_letters
string.hex_digits
```

Ou defina um gerador:
```py
def char_range(c1, c2):
    for c in xrange(ord(c1), ord(c2)+1):
        yield chr(c)
```
Usar simplesmente como `for c in char_range('c', 'k')`

# Disjoint set / Connected Components
Segue uma implementação, incluindo exemplo de uso:
```py
def connected_components(edges, vertices):
    ##############
    def make_set(x):
        return [x]
    def find_set(x):
        return set_lookup[x]
    def union(a, b):
        a_set, b_set = find_set(a), find_set(b)
        a_set.extend(b_set)
        set_lookup.update( { s:a_set for s in b_set } )
        sets.remove(b_set)
    #############
    sets = [make_set(v)  for v in vertices]
    set_lookup = { s:sets[i] for i,s in enumerate(vertices) }
	# assume-se que edges são tuplas
    for (u,v) in edges:
        if find_set(u) != find_set(v):
            union(u,v)
    return sets
# Uso abaixo ####	
import string
vertices = string.ascii_lowercase[:10]
edges = [('b', 'd'), ('e', 'g'), ('a', 'c'), ('h', 'i'), ('a', 'b'), ('e', 'f'), ('b', 'c')]
print(connected_components(edges, vertices))

```

# Format strings
## Basic

```py
'{1} {0}'.format('a', 'b') #-> 'b a'
'{0!r} {0!s}'.format(a, b) #-> repr(a) str(b)
'{:+d}'.format(10) #-> '+10' (sempre mostra sinal)
'{: d}'.format((-10)) #-> ' 10' (mostra o sinal somente quando negativo)
```
## Padding
Espaços vazios representados como `_`  nos comentários
*Nota* Números tem default de right-align, string left-align
```py
'{:5}'.format(10) #-> '___10'
'{:>5}'.format('asd') #-> '__asd' (force right-alignment
'{:<5}'.format(10) #-> '10___' (force left alignment
'{:^10}'.format(10) #-> '____10____' (center, pends to left
'{:a>5}.format(10) #-> 'aaa10' (pads left with a, can be used with any char
```

# Matrix Multiplication

```py
# naive
def mat_mult(A, B):
    return [[sum(a*b for a,b in zip(Arow,Bcol)) for Bcol in zip(*B)] for Arow in A]
```
# Gaussian Elimination

```py
def gauss(A):
    n = len(A)
    for i in range(n):
        max_row = max(range(i,n), key=lambda r: abs(A[r][i]))
        A[i], A[max_row] = A[max_row], A[i]
        for j in range(i,n):
            c = A[j][i]/A[i][i]
            A[j] = [A[j][k] - c for k in range(n+1)]
    if A[n-1][n-1] == 0:
        return None
    x = [0 for i in range(n)]
    for i in range(n-1, -1, -1):
        s = sum(A[i][j] * x[j] for j in range(i, n))
        x[i] = (A[i][n] - s) / A[i][i]
    return x

```

# Geometry 
## Line and segment
Linha: `y = mx + c` (tratar linha vertical como caso especial) ou `y - y1 = m * (x-x1)` 
Interesting properties:
- Slopes of parallel lines are equal
- Slopes of perpendicular lines are opposite reciprocal (2 <-> -1/2)
```py
# slope formula (given 2 points)
m = (y2 - y1) / (x2 - x1)
# midpoint (given 2 points)
midpoint = lambda p1,p2: (p1+p2)/2
x, y = midpoint(x1,x2), midpoint(y1,y2)
# distance between two points
d = math.sqrt((x2-x1)**2 + (y2-y1)**2)
```
## Triangle
Area
```py
area = (base * height)/2`
# ibid, usando só lados:
semiper = (a + b + c)/2 #semiperimeter
area = math.sqrt(semiper * (semiper - a) * (semiper - b) * (semiper - c))
# ibid, de um triângulo equilátero:
area = ((lado ** 2) * sqrt(3)) / 4
```

## Circle
Defined as `(x - a)**2 + (y - b)**2 = r**2`
```py
# Circumference
circ = math.pi * 2  * r # ou `circ = math.pi * d` se dado diametro
# Arc length (degrees) given arc_angle
arc_length = (arc_degree/360) * circ
# Area
area = math.pi * (r**2)
# Sector area
sec_area = (sec_degree/360) * area
# Length of chord
chord_len = 2 * (r**2) * (1 - math.cos(chord_degree))
# Area of segment defined by a chord
seg_area = sec_area - triangle_area(r,r,chord_len)
```
## General polygons (n-sided)

```py
# Sum of interior angles 
sum_angles = (n - 2) * 180
num_diagonals = (n * (n - 3))/2
```
## 3D objects (just in case)

```py
# Flat-tops
vol = base_area * height
surface_area = 2 * base_area + side_areas
# Cones and pyramids
vol = (1/3) * base_area * height
surface_area = base_area + side_areas #that is, area of each lateral side
# Sphere
vol = (4/3) * math.pi * (r ** 3)
surface_area = 4 * math.pi * (r ** 2)
```
# python std table of contents shortlist

```
The Python Standard Library
1. Introduction
2. Built-in Functions
3. Built-in Constants
4. Built-in Types
   4.5. Iterator Types
	   4.5.1. Generator Types
   4.6. Sequence Types — list, tuple, range
   4.7. Text Sequence Type — str
	   4.7.1. String Methods
   4.9. Set Types — set, frozenset
   4.10. Mapping Types — dict
5. Built-in Exceptions
6. Text Processing Services
   6.1. string — Common string operations
   6.2. re — Regular expression operations
7. Binary Data Services
8. Data Types
   8.1. datetime — Basic date and time types
   8.2. calendar — General calendar-related functions
   8.3. collections — Container datatypes
	   8.3.1. ChainMap objects
	   8.3.2. Counter objects
	   8.3.3. deque objects
	   8.3.4. defaultdict objects
	   8.3.5. namedtuple() Factory Function for Tuples with Named Fields
	   8.3.6. OrderedDict objects
   8.5. heapq — Heap queue algorithm
	   8.5.2. Priority Queue Implementation Notes
   8.6. bisect — Array bisection algorithm
	   8.6.1. Searching Sorted Lists
   8.7. array — Efficient arrays of numeric values
   8.8. weakref — Weak references
   8.10. copy — Shallow and deep copy operations
   8.13. enum — Support for enumerations
9. Numeric and Mathematical Modules
   9.1. numbers — Numeric abstract base classes
   9.2. math — Mathematical functions
   9.3. cmath — Mathematical functions for complex numbers
   9.4. decimal — Decimal fixed point and floating point arithmetic
   9.5. fractions — Rational numbers
   9.6. random — Generate pseudo-random numbers
   9.7. statistics — Mathematical statistics functions
10. Functional Programming Modules
	10.1. itertools — Functions creating iterators for efficient looping
	10.2. functools — Higher-order functions and operations on callable objects
	10.3. operator — Standard operators as functions
17. Concurrent Execution
	17.1. threading — Thread-based parallelism
	17.3. The concurrent package
	17.4. concurrent.futures — Launching parallel tasks
```

# To print
