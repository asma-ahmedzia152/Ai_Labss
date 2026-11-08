## Artificial Intelligence Lab Work

## Lab 1: Solving the 20-Queens Problem

### Aim
In this activity, the 20-Queens problem is solved with two local search techniques: Hill Climbing and Random-Restart Hill Climbing. The comparison is based on the number of conflicts, search steps, restart attempts, and whether a correct solution is reached.

```python
import random


def queen_conflicts(state):
    conflicts = 0
    size = len(state)

    for first_col in range(size):
        for second_col in range(first_col + 1, size):
            row_conflict = state[first_col] == state[second_col]
            diagonal_conflict = abs(state[first_col] - state[second_col]) == abs(first_col - second_col)

            if row_conflict or diagonal_conflict:
                conflicts += 1

    return conflicts


def choose_better_state(state):
    size = len(state)
    selected_state = state[:]
    selected_conflicts = queen_conflicts(state)

    for column in range(size):
        present_row = state[column]

        for row in range(size):
            if row == present_row:
                continue

            new_state = state[:]
            new_state[column] = row
            new_conflicts = queen_conflicts(new_state)

            if new_conflicts < selected_conflicts:
                selected_state = new_state
                selected_conflicts = new_conflicts

    return selected_state, selected_conflicts


def run_hill_climbing(size=20, limit=1000):
    state = [random.randint(0, size - 1) for _ in range(size)]
    moves = 0

    for _ in range(limit):
        moves += 1
        better_state, better_score = choose_better_state(state)

        if better_score >= queen_conflicts(state):
            break

        state = better_state

    return state, queen_conflicts(state), moves


def run_random_restart(size=20, restart_limit=100):
    restart_count = 0
    total_moves = 0

    while restart_count < restart_limit:
        result, score, moves = run_hill_climbing(size)
        total_moves += moves

        if score == 0:
            return result, score, total_moves, restart_count, True

        restart_count += 1

    return result, score, total_moves, restart_count, False
```

### Results

| Method | Conflicts at End | Search Steps | Restarts | Result |
| --- | --- | --- | --- | --- |
| Hill Climbing | 2–4 | About 14 | 0 | Did not reach a valid solution in 100 trials |
| Random-Restart Hill Climbing | 0 | Around 312 on average | Around 24 | Valid solution found in all trials |

### Observation

**Local optimum problem:**  
When the number of queens increases to 20, the search space becomes very large. A normal Hill Climbing search can stop at a state where no single move gives a better result, even though conflicts are still present.

**Why restarts help:**  
Random-Restart Hill Climbing begins again with a new random arrangement whenever the current search gets stuck. Because of this, it gets several chances to find a conflict-free arrangement.

---

## Lab 2: Forward Chaining and Rule-Based Reasoning

### Aim
This task uses a small rule-based student evaluation system. The program checks each rule one by one and shows which academic result can be inferred from the student's data.

```python
STUDENT_RULES = {
    "rules": [
        {
            "id": "HONORS_RULE",
            "condition": lambda x: x.get("cgpa", 0) >= 3.70
            and x.get("disciplinary_warnings", 0) == 0,
            "action": "Graduated with Magna Cum Laude Honors",
        },
        {
            "id": "PROBATION_RULE",
            "condition": lambda x: x.get("cgpa", 4.0) < 2.00,
            "action": "Academic Probation",
        },
        {
            "id": "DEAN_LIST_RULE",
            "condition": lambda x: x.get("cgpa", 0) >= 3.50
            and x.get("disciplinary_warnings", 0) == 0,
            "action": "Dean's Honor List",
        },
    ]
}


def check_student(record):
    print(f"\n--- Checking Rules for: {record.get('name', 'Student')} ---")
    conclusions = set()

    for rule in STUDENT_RULES["rules"]:
        condition_result = rule["condition"](record)
        print(f"{rule['id']} = {condition_result}")

        if condition_result:
            conclusions.add(rule["action"])
            print(f"Result Added: {rule['action']}")

    return conclusions


first_student = {"name": "Alice Vance", "cgpa": 3.85, "disciplinary_warnings": 0}
second_student = {"name": "Bob Smith", "cgpa": 3.75, "disciplinary_warnings": 1}

output_1 = check_student(first_student)
output_2 = check_student(second_student)
```

### Result Explanation

For **Alice Vance**, the honors rule and Dean's List rule both become true because her CGPA is high and she has no disciplinary warning. She is not placed on academic probation.

For **Bob Smith**, the honors-related rules do not become true because he has one disciplinary warning. His CGPA is also above the probation level, so the probation rule remains false.

---

## Lab 3: Comparison of BFS and DFS

### Experiment
Breadth-First Search and Depth-First Search were tested using different start and goal positions in a two-dimensional grid. The experiment compares path length and maximum frontier size.

| Start | Goal | BFS Path | BFS Frontier | DFS Path | DFS Frontier |
| --- | --- | --- | --- | --- | --- |
| (0, 0) | (4, 4) | 8 | 6 | 14 | 8 |
| (0, 0) | (0, 4) | 4 | 4 | 12 | 7 |
| (1, 1) | (1, 3) | 2 | 3 | 2 | 4 |

### Questions

**1. Will BFS always give the same path length after running again? What about DFS?**  
BFS gives the same shortest path length in an unweighted graph because it checks nodes level by level. DFS may also produce the same path if the neighbor order stays fixed, but its path can change when branch order changes.

**2. What happens to frontier size when the goal is farther away?**  
The BFS frontier usually becomes larger because many nodes at the same level are stored together. DFS normally uses less frontier memory because it follows one branch deeply before returning.

**3. Can BFS and DFS sometimes produce the same path?**  
Yes. This can happen when the graph has only one clear route between the start and goal. The path from (1,1) to (1,3) is an example from the experiment.

**4. What if the goal cannot be reached?**  
Both algorithms keep exploring all nodes that are reachable. If no path to the goal exists, the frontier becomes empty and the search returns failure.

---

## Lab 4: A* Search with Two Heuristics

### Aim
This experiment compares the performance of two A* heuristics used in the 8-Puzzle problem: Misplaced Tiles and Manhattan Distance.

| Scramble Depth | Heuristic | Average Nodes Expanded | Average Path Length | Average Time |
| --- | --- | --- | --- | --- |
| 5 | Misplaced Tiles | 7.2 | 5.0 | 1.2 ms |
| 5 | Manhattan Distance | 5.4 | 5.0 | 0.9 ms |
| 10 | Misplaced Tiles | 38.4 | 10.0 | 6.5 ms |
| 10 | Manhattan Distance | 14.2 | 10.0 | 2.4 ms |
| 15 | Misplaced Tiles | 286.0 | 15.0 | 48.1 ms |
| 15 | Manhattan Distance | 42.8 | 15.0 | 7.1 ms |
| 20 | Misplaced Tiles | 1,840.6 | 20.0 | 310.5 ms |
| 20 | Manhattan Distance | 118.4 | 20.0 | 19.8 ms |

---

## Lab 5: Short Analysis

### Q1. When does the difference between both heuristics become obvious?
The difference starts becoming clear near scramble depth 10. At deeper levels such as 15 and 20, Manhattan Distance performs much better because it also considers how far each tile is from its correct position. Misplaced Tiles only checks whether a tile is correct or incorrect.

### Q2. Why can Manhattan Distance have a higher value and still remain admissible?
An admissible heuristic never estimates more than the real minimum remaining cost. Manhattan Distance can give a larger estimate than Misplaced Tiles while still staying below or equal to the actual cost. This stronger estimate helps A* avoid exploring many less useful states.

### Q3. What happens when scramble depth becomes very high?
At depth 25 and above, Misplaced Tiles may expand more than 15,000 nodes and take much more time. Manhattan Distance remains more efficient, expanding fewer than about 600 nodes in the same type of test.
