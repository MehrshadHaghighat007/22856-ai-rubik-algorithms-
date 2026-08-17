# Rubik's Cube Solver — A Comparative Study of Search Algorithms

An implementation and empirical comparison of ten classical search algorithms — from uninformed search (BFS, DFS, UCS, IDS) to informed/heuristic search (Greedy Best-First, A*, Weighted A*, IDA*, RBFS) — all applied to solving a generalized N×N×N Rubik's Cube, with a custom admissible heuristic and full performance benchmarking (time, nodes expanded, solution depth) across algorithms.

## Overview

Solving a Rubik's Cube is a classic search problem with an enormous state space (a standard 3×3×3 cube has over 4×10¹⁹ possible states), making it an effective testbed for comparing how different search strategies handle scale. This project implements a generalized cube model that supports arbitrary dimensions (not just 3×3×3), a shared move-generation and search infrastructure, and ten distinct search algorithms layered on top of a single `Best_First_Search` core — allowing direct, controlled comparison of each algorithm's efficiency on identical problem instances.

## Key Technical Components

- **Generalized N×N×N cube model** — the `Rubik` class represents a cube of arbitrary dimensions as a 3D grid of `Cube` objects, each tracking the color of its six potential faces (only the outward-facing ones are meaningful). This generalizes beyond the standard 3×3×3 cube, allowing the same algorithms to be benchmarked across different state-space sizes.
- **Custom equality and hashing over cube *appearance*, not internal structure** — `Rubik.__eq__` and `__hash__` compare only the colors visible on each of the six outer faces, rather than the full internal piece arrangement. This is a deliberate simplification of the true cube state space (real cubes distinguish states that look identical but have different piece orientations at the cubie level), which correctly and substantially reduces the effective search space while still producing a genuinely solved-looking cube.
- **A single move primitive (`row_reverse`) generating all cube moves** — every possible face/slice rotation is expressed as reversing a row of cubes along one axis and swapping the two faces perpendicular to that axis. This keeps the move-generation logic (`expand`) compact and uniform across all three axes.
- **Admissible, consistent heuristic function** — for each of the 6 faces, the heuristic counts how many stickers differ from that face's majority color, sums this across all faces, and divides by 2 — since any single move can correct at most 2 mismatched stickers into place at once, dividing by 2 guarantees the heuristic never overestimates the true remaining cost, which is the formal requirement for A*-family algorithms to guarantee optimal solutions.
- **Ten search algorithms implemented, most sharing one core routine** — BFS, DFS, and Iterative Deepening Search are implemented directly; UCS, Greedy Best-First, A*, and Weighted A* are all implemented as thin `f`-function variants passed into a single shared `Best_First_Search` priority-queue routine, demonstrating that these algorithms differ only in their node-ordering function, not their underlying search structure. IDA* and Recursive Best-First Search (RBFS) are implemented separately, since their memory-bounded recursive structure doesn't fit the shared frontier-based routine.
- **Empirical benchmarking harness** — every algorithm is run against the same shuffled cube instance, recording wall-clock time, number of nodes expanded, and solution depth into a pandas DataFrame for direct side-by-side comparison.

## Results & Analysis

Benchmarking on a shuffled cube (solution depth 3) showed a clear efficiency ordering, consistent with search theory:

| Algorithm | Time (s) | Nodes Expanded |
|---|---|---|
| UCS | 13.04 | 18,128 |
| IDS | 4.56 | 4,323 |
| BFS | 0.99 | 1,237 |
| RBFS | 0.023 | 79 |
| GBFS | 0.028 | 78 |
| Weighted A* | 0.026 | 78 |
| A* | 0.044 | 128 |
| IDA* | 0.024 | 20 |

The uninformed algorithms (UCS, IDS, BFS) expand orders of magnitude more nodes than any heuristic-guided algorithm, confirming the practical necessity of a good heuristic for a state space this large. Among the informed algorithms, IDA* expanded the fewest nodes of all, combining the low memory footprint of iterative deepening with heuristic guidance, while still guaranteeing an optimal-depth solution — consistent with its reputation as a strong general-purpose choice when memory is constrained. Greedy Best-First Search was fast but is not guaranteed optimal, since it ignores path cost entirely; A* and RBFS both guarantee optimality at some additional node-expansion cost relative to Greedy and IDA*.

## Tech Stack

- **Python** — `numpy` (grid-based cube representation), `pandas` (results benchmarking table), `heapq` (priority queue for Best-First Search variants), `collections.deque` / `Counter`, `copy`, `time`

## My Contribution

Designed and implemented the full project independently for the university AI course: the generalized cube model and move representation, the admissibility proof and implementation of the heuristic function, all ten search algorithm implementations, and the benchmarking/analysis comparing their behavior on identical problem instances.
