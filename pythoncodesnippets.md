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
# Graphs
Dada uma representação simples (lista de adjacência) de um grafo
```py
	graph = {'A': ['B', 'C'],
             'B': ['C', 'D'],
             'C': ['D'],
             'D': ['C'],
             'E': ['F'],
             'F': ['C']
            }
```
Para usar pesos, podemos fazer cada entrada na lista de adjacência ser uma tupla (e.g `'A':[('B',10)]`), ou ter um segundo dicionário indexado por tuplas `(u,v)`. **Dependendo de como isso for feito, é necessário mudar várias implementações abaixo.** Por questão de simplicidade, estou assumindo que `weights` é um dicionário diferente.

Uma implementação de nó que pode ser usada na estrutura acima (é hasheável):
```py
class Node:
    def __init__(self, index, data):
        self.index = index
        self.data = data
    def __hash__(self):
        return self.index
```
É fácil adicionar mais variáveis se necessário. Pode ser uma boa ideia trocar `__hash__` para o seguinte dependendo da situação.
```py
	def __repr__(self):
		return str(self.index)
```
Funções básicas
```py
def vertices(g):
    return list(k for k in g.keys())
def edges(g):
    return list((a,b) for a,v in g.items() for b in v)
# OU, SE O INTUITO FOR ITERAR:
def vertices_iter(g):
    return g.keys()
def edges_iter(g):
    return ((a,b) for a,v in g.items() for b in v)
```
Seguem implementações básicas de BFS e DFS, que devem ser modificadas dependendo do uso, especialmente pra ficarem mais limpas (i.e, dependendo se quisermos caminhos, distâncias, etc)
```py
def bfs(g, start):
    from collections import deque
    d = {start:0}
    p = {start:None}
    queue = deque(start)
    while queue:
        curr = queue.popleft()
        for v in (adj for adj in g[curr] if adj not in distance):
            d[v] = distance[curr] + 1
            p[v] = curr
            queue.append(v)
    return d, p

# segue somente uma transcrição básica do dfs-visit do cormen,
# que é mais fácil de generalizar pra outros algoritmos
# muitos dos parametros podem ser tirados dependendo do escopo
def dfs_visit(g, start, d, p, f, visited=set()):
    # assume-se que existe um 'time' no escopo superior
    time = time + 1
    d[start] = time
    visited.add(start)
    for v in (adj for adj in g[start] if adj not in visited):
        p[v] = start
        dfs_visit(g, v, d, p, f, visited=visited)
    time = time + 1
    f[start] = time
    return d, p, f
```
## Disjoint set / Connected Components

```py
def connected_components(g):
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
    sets = [make_set(v)  for v in vertices_iter(g)]
    set_lookup = { s:sets[i] for i,s in enumerate(vertices_iter(g)) }
    for (u,v) in edges_iter(g):
        if find_set(u) != find_set(v):
            union(u,v)
    return sets
```
## Kruskal (minimum spanning tree)
Kruskal depende de conjuntos disjuntos, então usa as mesmas três funções base
```py
def kruskal(g, weights):
    from heapq import heapify, heapop
    ## make_set, find_set, union ##
    sets = [make_set(v)  for v in vertices_iter(g)]
    set_lookup = { s:sets[i] for i,s in enumerate(vertices_iter(g)) }
    # queremos um priority queue das edges ordenados pelo maior peso
    # então precisamos inverter os pesos (heapq é ordenado pelo menor).
    # para fazer uma maximum spanning tree, trocar `-w` por só `w`
    edgeq = list((-w,e) for e,w in weights.items())
    heapify(edgeq)
    tree = []
    while edgeq:
        (w, (u,v)) = heappop(edgeq)
        if find_set(u) != find_set(v):
            tree.append((u,v))
            union(u,v)
```
# Shortest Paths
Os algoritmos a seguir geralmente usam a função `relax` apresentada aqui. Ela foi feita para ser declarada dentro da função do algoritmo em si (assume um, `weight`, `d` (distância, delta) e `p` (predecessor) do escopo externo).
```py
def relax(u,v):
    if d[v] > d[u] + weight[(u,v)]:
        d[v] = d[u] + weight[(u,v)]
        p[v] = u
```
O uso de `p` é espelhando a estrutura apresentada no Cormen, pode ser mais eficiente trocar por outra coisa dependendo do caso.
## Comparações

|      | BFS    | Dijkstra    | Bellman-Ford |
|------|--------|-------------|--------------|
|Big-O | O(V+E) | O((V+E)logV)| O(VE)   |
|Sem Peso|Melhor | Bom | Ruim |
|Com Peso|Só árvore e DAG| Melhor | Bom |
|Pesos negativos | Erra | Erra | Melhor|
|Ciclos negativos | Não Detecta | Não Detecta | Detecta

## Dags com peso (baseado em ordenação topológica)
A implementação abaixo não checa se o grafo realmente é um DAG ou não, porque isso torna a implementação da ordenação topológica bem mais simples. Alterar se checagem for necessária.
```py
def dag_sssp(g, weight, start):
    import math
    def topological(g, s, visited=None):
	    if not visited:
	        visited = set()
        yield s
        visited.add(s)
        for v in (adj for adj in g[s] if adj not in visited):
            yield from topological(g, v, visited)
    d = { v:math.inf for v in vertices_iter(g) }
    d[start] = 0
    p = { start:None}
    for u in topological(g, start):
        for v in g[u]:
            relax(u,v,weight)
```

## Bellman-Ford

```py
def bellman_ford(g, start, weight):
    ## def relax
    from itertools import repeat
    import math
    d = { v:math.inf for v in vertices_iter(g) }
    d[start] = 0
    p = { start:None}
    for (u,v) in repeat(edges_iter(g), len(vertices(g))-1):
        relax(u,v)
    if any(d[v] > d[u] + weight[(u,v)] for (u,v) in edges_iter(g)):
        raise ValueError("Ciclo Negativo")
    return d, p
```
## Dijkstra
 
```py
def dijkstra(g, start, weight):
    ## def relax
    import math
    d = { v:math.inf for v in vertices_iter(g) }
    d[start] = 0
    p = { start:None}
    verts = set(vertices_iter(g))
    visited = set()
    while visited != verts:
        curr = min(set(d), key=d.get)
        visited.add(curr)
        for v in g[curr]:
            relax(v,curr)
    return d, p
```
## Floyd-Warshal
All-pairs shortest paths, O(V³), usável pra casos pequenos e fácil de implementar. O código abaixo assume a mesma representação dos anteriores, mas dá para cortar parte do overhead se o grafo já for ldio como matriz de adjacencia.
```py
def floydwarshal(g, weight):
    from itertools import product
    from collections import defaultdict
    import math
    d = { v:defaultdict(math.inf) for v in vertices_iter(g) }
    p = { v:defaultdict(None) for v in vertices_iter(g) }
    for (u,v) in product(vertices_iter(g), repeat=2):
        if u == v:
            d[u][v] = 0
        elif (u,v) in weight:
            d[u][v] = weight[(u,v)]
            p[u][v] = u
    ################
    for (k,i,j) in product(vertices_iter(g), repeat=3):
        new = d[i][k] + d[k][j]
        if new < d[i][j]:
            d[i][j] = new
            p[i][j] = p[k][j]
    return d, p
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
**Nota** Números tem default de right-align, string left-align
```py
'{:5}'.format(10) #-> '___10'
'{:>5}'.format('asd') #-> '__asd' (force right-alignment
'{:<5}'.format(10) #-> '10___' (force left alignment
'{:^10}'.format(10) #-> '____10____' (center, pends to left
'{:a>5}'.format(10) #-> 'aaa10' (pads left with a, can be used with any char
```
# General Math 
## Sieve of Eratosthenes

```py
def sieve(n):
    non_primes = set()
    primes = []
    for i in range(2, n+1):
        if i not in non_prime:
            #yield i
            primes.append(i)
            non_primes.update(range(i*i, n+1, i))
    return primes
```

## Prime Factorization

```py
def prime_factors(n):
    primes = sieve(n)
    if n in primes:
        return {n:1}
    factors = defaultdict(int)
    primes = iter(primes)
    pf = next(primes)
    while (n != 1):
        print(n, pf)
        while n % pf == 0:
            n = n // pf
            factors[pf] += 1
        pf = next(primes)
    return factors
```

## Matrix Multiplication

```py
# naive
def mat_mult(A, B):
    return [[sum(a*b for a,b in zip(Arow,Bcol)) for Bcol in zip(*B)] for Arow in A]
```
## Gaussian Elimination

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
area = (base * height)/2
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
