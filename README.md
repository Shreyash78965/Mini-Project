# Mini-Project
# Browser Navigation (Stack) and Snake and Ladder (BFS)

Two Python mini projects that show how classic data structures and algorithms solve everyday problems:

| Project | Problem | Technique | File |
|---------|---------|-----------|------|
| Browser Back and Forward Buttons | Return to previously visited pages in reverse order | Stack (LIFO) | `browser_navigation_full.py` |
| Snake and Ladder | Fewest dice throws needed to finish the board | Breadth First Search (BFS) | `snake_and_ladder_full.py` |

Short versions of the same programs are in `browser_back_button.py` and `snake_and_ladder.py`.

**Requirements:** Python 3.8 or newer. No external packages.

```bash
python browser_navigation_full.py
python snake_and_ladder_full.py
```

---

## 1. Browser Back and Forward Buttons (Stack)

### The problem

When you browse the web, the back button takes you to the page you were on before the current one, then the one before that, and so on. Pages must come back in exactly the reverse order of the visit. This is Last-In-First-Out (LIFO) behaviour, which is what a stack provides.

### The idea

The browser keeps two stacks and one variable:

- `current`: the page being shown.
- `back_stack`: pages that were left behind, with the most recent on top.
- `forward_stack`: pages that were left by pressing back, so that forward can return to them.

| Action | What happens |
|--------|--------------|
| `visit(url)` | Push `current` on `back_stack`, make `url` the current page, and **clear** `forward_stack` |
| `back()` | Push `current` on `forward_stack`, pop `back_stack` into `current` |
| `forward()` | Push `current` on `back_stack`, pop `forward_stack` into `current` |

If the stack needed by `back()` or `forward()` is empty, the program prints a message and nothing changes (stack underflow is handled instead of crashing).

Clearing the forward history on a new visit matches real browsers. If you go back and then open a new page, the old forward pages are gone.

### Worked example

Starting on `google.com`:

| Step | `back_stack` (bottom to top) | `current` | `forward_stack` (bottom to top) |
|------|------------------------------|-----------|---------------------------------|
| start | [ ] | google.com | [ ] |
| visit youtube.com | [google.com] | youtube.com | [ ] |
| visit github.com | [google.com, youtube.com] | github.com | [ ] |
| back() | [google.com] | youtube.com | [github.com] |
| forward() | [google.com, youtube.com] | github.com | [ ] |
| back() twice | [ ] | google.com | [github.com, youtube.com] |
| visit d.com | [google.com] | d.com | [ ] (cleared) |

### What is in the code

- `Stack`: own implementation with `push`, `pop`, `peek`, `is_empty`, `size`, `clear`, `to_list`. Built on a Python list, used only through `append` and `pop`.
- `Browser`: `visit`, `back`, `forward`, `back_many(n)`, `can_go_back`, `can_go_forward`, `history`, `show_history`, `status`.
- Optional **history limit**: when set, the oldest pages are dropped once the limit is exceeded.
- Repeat visits to the page you are already on are ignored.
- `run_session(commands)`: runs a list of text commands such as `"visit a.com"`, `"back 2"`, `"forward"`, `"history"`, so test sessions are easy to write.

### Complexity

| Operation | Time |
|-----------|------|
| `visit`, `back`, `forward` | O(1) (amortized for `visit`, because a list occasionally resizes) |
| `show_history` | O(n) |
| Space | O(n) for n stored pages |

The history limit uses `pop(0)` on the oldest entry, which is O(n). A `collections.deque` would make that O(1).

### Sample output

```
=== Session 1: basic navigation ===
Visited : youtube.com
Visited : github.com
Visited : stackoverflow.com
History : google.com > youtube.com > github.com > [stackoverflow.com]
Back    : github.com
Back    : youtube.com
History : google.com > [youtube.com] > github.com > stackoverflow.com
Forward : github.com
```

The page in square brackets is the current page.

### Real-world uses of the same idea

Browser history, undo and redo in editors, the back stack of screens in mobile apps, function call stacks, and bracket matching in compilers.

---

## 2. Snake and Ladder (BFS)

### The problem

On a board of N squares you start at square 1 and want to reach square N. A throw moves you forward 1 to 6 squares. Landing on the foot of a ladder lifts you to its top, and landing on the mouth of a snake drops you to its tail. **What is the fewest number of throws needed?**

The program assumes you can choose the number shown on the dice. This gives the best possible case, which is the shortest path.

### Why BFS

The board becomes an unweighted graph:

- every square is a **node**,
- the squares 1 to 6 steps ahead are its **neighbours** (after applying any snake or ladder),
- every throw costs the same, so every edge has the same weight.

BFS visits nodes level by level. Level k holds all squares reachable in exactly k throws and not earlier. So the first time BFS reaches square N, it has found the minimum number of throws. Trying every sequence of throws with recursion would take exponential time and could loop forever on snakes.

### The algorithm

1. Put square 1 in a queue with 0 throws, and mark it visited.
2. Remove the square at the front of the queue.
3. If it is square N, stop. The stored throw count is the answer.
4. For each dice value 1 to 6, find `landed = square + dice`. Skip it if it is beyond N.
5. If `landed` is the start of a snake or ladder, move to its end.
6. If that square is not visited yet, mark it visited, remember where it came from, and add it to the queue with one more throw.
7. If the queue becomes empty before N is reached, the answer is **-1**.

The visited set is what makes it fast and safe: each square is processed once, and snakes can never cause an endless loop.

Remembering where each square came from lets the program rebuild the route by walking back from square N.

### What is in the code

- `Board(size, jumps)`: stores the board and **validates** it. It rejects boards that are too small, jumps that start or end outside the board, jumps that go nowhere, and a snake or ladder on the last square.
- `Board` helpers: `kind`, `land`, `ladders`, `snakes`, `describe`, and `draw`, which prints the board in zig-zag order with `L` and `S` markers.
- `min_throws(board, dice_faces=6)`: BFS that returns the minimum throws and the full route. The dice size can be changed.
- `explore_levels(board)`: lists the squares first reached at each throw count.
- `print_route`: prints each throw, including the snake or ladder that was used.

### Sample board

```
 25   26   27S  28   29   30
 24   23   22   21S  20L  19S
 13   14   15   16   17S  18
 12   11L  10    9    8    7
  1    2    3L   4    5L   6
```

Ladders: 3→22, 5→8, 11→26, 20→29. Snakes: 17→4, 19→7, 21→9, 27→1.

### Sample output

```
Result     : minimum throws = 3
  Throw 1: square  1 + dice 2 =  3  (ladder 3 -> 22)
  Throw 2: square 22 + dice 2 = 24
  Throw 3: square 24 + dice 6 = 30

Squares reached at each level (throw count):
  0: [1]
  1: [2, 4, 6, 7, 8, 22]
  2: [9, 10, 12, 13, 14, 23, 24, 25, 26, 28]
  3: [15, 16, 18, 29, 30]
```

Square 30 first appears at level 3, so 3 throws is the minimum. Squares such as 3, 5, 11 and 27 never appear in the levels, because landing on them immediately sends you elsewhere.

### Test cases included

| Case | Board | Result |
|------|-------|--------|
| 1 | Sample board above (30 squares) | 3 throws |
| 2 | 30 squares, no snakes or ladders | 5 throws |
| 3 | Classic 100-square board | 5 throws |
| 4 | Snakes on squares 2 to 7, all back to square 1 | -1 (unreachable) |
| 5 | Invalid boards (jump on last square, off the board, or to itself; size 1) | Rejected with an error message |

### Complexity

| | Cost |
|---|------|
| Time | O(N): each square is queued once and tries 6 dice values, at most 6N steps |
| Space | O(N) for the queue, visited set and route information |

### Real-world uses of the same idea

Fewest-move puzzles (word ladder, sliding puzzles), shortest routes in grids and mazes, fewest hops in computer networks, and degrees of separation in social networks.

---

## Ideas for extending the projects

**Browser:** separate history for every tab, saving history to a file, jumping to any page in the history list, a graphical Back and Forward interface.

**Snake and Ladder:** reading the board from a file or user input, several players, dice with a different number of faces (already supported by `dice_faces`), the expected number of throws with a random dice (Markov chain), and an animated board.

## What you learn from these projects

- How the stack's LIFO rule matches real behaviour such as back and forward.
- How to handle underflow (empty stack) and other edge cases without crashing.
- How to turn a game into a graph.
- How BFS, a queue and a visited set give shortest paths in unweighted graphs.
- How to reason about time and space complexity.
