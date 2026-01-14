# ConfigX List Support: Complete Syntax Specification

This document defines the full programmatic specification for **LIST** support in ConfigX, covering syntax, semantics, and all supported operations.

---

## 1. Core Design Principles

| Principle | Description |
|:---|:---|
| **Homogeneous by Default** | Lists should ideally contain elements of the same type (e.g., all `INT`, all `STR`) |
| **Heterogeneous Allowed** | Mixed-type lists are permitted: `[1, "hello", true, 3.14]` |
| **Nested Support** | Lists can contain other lists: `[[1, 2], [3, 4]]` |
| **Immutable Path** | A list stored at `key.path` is ONE atomic value—you cannot add children to it |
| **Type Inference** | Type is inferred as `"LIST"` when value is a Python `list` |

---

## 2. Complete Syntax Reference

### 2.1 List Creation (SET)

```python
# Basic list creation
c.resolve('items=[1, 2, 3, 4]')              # Integer list
c.resolve('names=["alice", "bob", "carol"]') # String list
c.resolve('flags=[true, false, true]')       # Boolean list
c.resolve('scores=[9.5, 8.7, 10.0]')         # Float list

# Empty list
c.resolve('emptyList=[]')

# Mixed-type list (heterogeneous)
c.resolve('mixed=[1, "two", 3.0, true]')

# Nested lists
c.resolve('matrix=[[1, 2], [3, 4], [5, 6]]')
c.resolve('deep=[[[1]], [[2]]]')
```

---

### 2.2 List Retrieval (GET)

| Syntax | Description | Return Value |
|:---|:---|:---|
| `items` | Get entire list | `[1, 2, 3, 4]` |
| `items!` | Safe get (None if missing) | `[1, 2, 3, 4]` or `None` |

```python
c.resolve('items')   # Returns [1, 2, 3, 4]
c.resolve('items!')  # Returns [1, 2, 3, 4] or None if not found
```

---

### 2.3 List Indexing (Element Access)

| Syntax | Description | Example |
|:---|:---|:---|
| `path[N]` | Get element at index N (0-based) | `items[0]` → `1` |
| `path[-N]` | Negative indexing from end | `items[-1]` → `4` |
| `path[N]!` | Safe index access | `items[99]!` → `None` |

```python
# Standard indexing
c.resolve('items[0]')     # First element: 1
c.resolve('items[2]')     # Third element: 3
c.resolve('items[-1]')    # Last element: 4
c.resolve('items[-2]')    # Second-to-last: 3

# Safe indexing (returns None if out of bounds)
c.resolve('items[100]!')  # Returns None instead of error

# Nested list indexing
c.resolve('matrix[0]')    # Returns [1, 2]
c.resolve('matrix[0][1]') # Returns 2
c.resolve('matrix[1][-1]')# Returns 4
```

---

### 2.4 List Slicing

| Syntax | Description | Example Result |
|:---|:---|:---|
| `path[start:end]` | Slice from start to end (exclusive) | `items[0:2]` → `[1, 2]` |
| `path[:end]` | Slice from beginning | `items[:2]` → `[1, 2]` |
| `path[start:]` | Slice to end | `items[2:]` → `[3, 4]` |
| `path[:]` | Copy entire list | `items[:]` → `[1, 2, 3, 4]` |
| `path[::step]` | Slice with step | `items[::2]` → `[1, 3]` |
| `path[start:end:step]` | Full slice syntax | `items[0:4:2]` → `[1, 3]` |

```python
c.resolve('items[1:3]')    # [2, 3]
c.resolve('items[:2]')     # [1, 2]
c.resolve('items[2:]')     # [3, 4]
c.resolve('items[:-1]')    # [1, 2, 3]
c.resolve('items[::-1]')   # [4, 3, 2, 1] (reversed)
```

---

### 2.5 List Modification Operations

#### 2.5.1 Element Update (Index Assignment)

```python
# Update single element at index
c.resolve('items[0]=99')      # items becomes [99, 2, 3, 4]
c.resolve('items[-1]=100')    # items becomes [99, 2, 3, 100]

# Update nested list element
c.resolve('matrix[0][0]=10')  # matrix becomes [[10, 2], [3, 4], [5, 6]]
```

#### 2.5.2 Append Operation

| Syntax | Description |
|:---|:---|
| `path+=value` | Append single value to end |
| `path+=[...]` | Extend with multiple values |

```python
c.resolve('items+=5')             # [1, 2, 3, 4, 5]
c.resolve('items+=[6, 7]')        # [1, 2, 3, 4, 5, 6, 7] (extend)
c.resolve('names+="dave"')        # ["alice", "bob", "carol", "dave"]
```

#### 2.5.3 Insert Operation

| Syntax | Description |
|:---|:---|
| `path+N=value` | Insert value at index N, shift others right |

```python
c.resolve('items+0=0')      # [0, 1, 2, 3, 4] - insert at beginning
c.resolve('items+2=99')     # [0, 1, 99, 2, 3, 4] - insert at index 2
```

#### 2.5.4 Remove Operations

| Syntax | Description |
|:---|:---|
| `path[N]-` | Remove element at index N |
| `path[-]=value` | Remove first occurrence of value |

```python
c.resolve('items[0]-')       # Remove first element
c.resolve('items[-1]-')      # Remove last element
c.resolve('items[-]=2')      # Remove first occurrence of value 2
```

---

### 2.6 List Deletion

```python
# Delete entire list
c.resolve('items-')
```

---

### 2.7 List Queries / Built-in Functions

> [!NOTE]
> These are **optional advanced features** that can be implemented in a later phase.

| Syntax | Description | Example |
|:---|:---|:---|
| `path!len` | Get list length | `items!len` → `4` |
| `path!contains=value` | Check if value exists | `items!contains=2` → `true` |
| `path!index=value` | Get index of first occurrence | `items!index=3` → `2` |
| `path!sum` | Sum of numeric list | `items!sum` → `10` |
| `path!min` | Minimum value | `items!min` → `1` |
| `path!max` | Maximum value | `items!max` → `4` |
| `path!reverse` | Get reversed copy | `items!reverse` → `[4, 3, 2, 1]` |

```python
c.resolve('items!len')          # 4
c.resolve('items!contains=2')   # True
c.resolve('items!index=3')      # 2 (0-indexed position)
c.resolve('scores!sum')         # Sum of all scores
c.resolve('scores!max')         # Maximum score
```

---

### 2.8 Wildcard Operations

#### 2.8.1 Select All: `[*]`

| Syntax | Description | Example |
|:---|:---|:---|
| `path[*]` | Get all elements | `items[*]` → `[1,2,3,4]` |
| `path[*]=val` | Set all elements | `items[*]=0` → `[0,0,0,0]` |
| `path[*]-` | Clear list (keep empty) | `items[*]-` → `[]` |

```python
c.resolve('items[*]')           # [1, 2, 3, 4]
c.resolve('items[*]=0')         # Set all to 0
c.resolve('items[*]-')          # Clear list
```

#### 2.8.2 Filter by Condition: `[?condition]`

| Syntax | Description |
|:---|:---|
| `path[?>N]` | Elements greater than N |
| `path[?<N]` | Elements less than N |
| `path[?==val]` | Elements equal to val |
| `path[?!=val]` | Elements not equal to val |

```python
c.resolve('scores[?>90]')       # Get scores above 90
c.resolve('scores[?<50]=50')    # Set failing scores to 50
c.resolve('flags[?==true]-')    # Remove all true values
```

#### 2.8.3 Range Selection: `[N..M]`

```python
c.resolve('items[0..2]')        # Elements 0, 1, 2
c.resolve('items[0..2]=0')      # Set range to 0
```

#### 2.8.4 Field Projection (Lists of Objects)

```python
# If users is a list of branches with name/age fields:
c.resolve('users[*].name')            # ["Alice", "Bob"]
c.resolve('users[*].active=true')     # Set active for all
c.resolve('users[?.age>25].name')     # Names where age > 25
```

---

## 3. Grammar Changes Required

Update `configx/qlang/configxql.lark`:

```lark
// ----------------------
// Values (STRICT)
// ----------------------

?value: STRING -> string
      | SIGNED_INT -> int
      | SIGNED_FLOAT -> float
      | BOOL -> bool
      | list -> list           // NEW

list: "[" [value ("," value)*] "]"

// ----------------------
// Path (Extended for List Access)
// ----------------------

path: IDENT ("." IDENT)* index_access*

index_access: "[" index_expr "]"
index_expr: SIGNED_INT                           // items[0], items[-1]
          | SIGNED_INT? ":" SIGNED_INT? (":" SIGNED_INT)?  // Slicing
          | "-"                                  // Remove marker
          | "*"                                  // Wildcard all: items[*]
          | SIGNED_INT ".." SIGNED_INT           // Range: items[0..2]
          | "?" COMPARISON_OP value              // Filter: items[?>5]

COMPARISON_OP: ">" | "<" | ">=" | "<=" | "==" | "!="

// Statement extensions for list operations
set_stmt: path "=" value
        | path index_access "=" value            // items[0]=99
        | path "+=" value                        // items+=5 (append)
        | path "+" SIGNED_INT "=" value          // items+2=99 (insert at index)
        | path "[*]" "=" value                   // items[*]=0 (set all)

delete_stmt: path "-"
           | path index_access "-"               // items[0]- or items[*]-

// Remove by value
remove_by_value_stmt: path "[-]" "=" value       // items[-]=2
```

---

## 4. Type System Integration

### 4.1 Node Type Inference (`node.py`)

```python
@staticmethod
def infer_type(value) -> str:
    if isinstance(value, bool):
        return "BOOL"
    if isinstance(value, int) and not isinstance(value, bool):
        return "INT"
    if isinstance(value, float):
        return "FLOAT"
    if isinstance(value, str):
        return "STR"
    if isinstance(value, list):      # NEW
        return "LIST"
    return "JSON"
```

### 4.2 Binary Snapshot Format (`snapshot.py`)

| Tag | Type | Format |
|:---|:---|:---|
| `b'L'` | LIST | `[Tag:L][Count:4-byte int][Item1][Item2]...` |

Each item is recursively serialized using the existing value serialization (with type tags).

---

## 5. Recommended Implementation Phases

### Phase 1: MVP (Minimum Viable Product)
- [x] List literal parsing: `[1, 2, 3]`
- [x] List storage as node value
- [x] Binary persistence with `b'L'` tag
- [x] Basic get/set: `items=[1,2,3]`, `items`

### Phase 2: Index Access
- [ ] Single index access: `items[0]`
- [ ] Negative indexing: `items[-1]`
- [ ] Safe index access: `items[0]!`
- [ ] Nested index access: `matrix[0][1]`

### Phase 3: Modification Operations
- [ ] Element update: `items[0]=99`
- [ ] Append: `items+=5`
- [ ] Insert: `items+2=99`
- [ ] Remove by index: `items[0]-`

### Phase 4: Slicing
- [ ] Basic slicing: `items[1:3]`
- [ ] Step slicing: `items[::2]`

### Phase 5: Wildcards
- [ ] Select all: `items[*]`
- [ ] Set all: `items[*]=0`
- [ ] Clear all: `items[*]-`
- [ ] Range selection: `items[0..2]`
- [ ] Filter by condition: `items[?>5]`
- [ ] Field projection: `users[*].name`

### Phase 6: Built-in Functions (Optional)
- [ ] `!len`, `!contains`, `!sum`, `!min`, `!max`

---

## 6. Error Handling

| Error | Condition | Message |
|:---|:---|:---|
| `ConfigPathNotFoundError` | List doesn't exist | `"Path 'items' not found"` |
| `ConfigIndexOutOfBoundsError` | Index exceeds list length | `"Index 5 out of bounds for list of length 4"` |
| `ConfigTypeError` | Indexing non-list | `"Cannot index non-list value at 'config.theme'"` |
| `ConfigInvalidSyntaxError` | Malformed list literal | `"Invalid list syntax"` |

---

## 7. Examples Summary

```python
from configx import ConfigX

c = ConfigX()

# --- Creation ---
c.resolve('scores=[95, 87, 92, 88]')
c.resolve('names=["Alice", "Bob"]')
c.resolve('matrix=[[1,2],[3,4]]')

# --- Retrieval ---
c.resolve('scores')          # [95, 87, 92, 88]
c.resolve('scores[0]')       # 95
c.resolve('scores[-1]')      # 88
c.resolve('matrix[1][0]')    # 3

# --- Modification ---
c.resolve('scores[0]=100')   # Update first
c.resolve('scores+=75')      # Append
c.resolve('names+0="Zara"')  # Insert at start

# --- Deletion ---
c.resolve('scores[1]-')      # Remove index 1
c.resolve('names-')          # Delete entire list

# --- Queries (Phase 5) ---
c.resolve('scores!len')      # 3
c.resolve('scores!sum')      # 267
```

---

## 8. Design Decisions for Contributors

> [!NOTE]
> **Finalized design decisions:**

| Decision | Answer | Rationale |
|:---|:---|:---|
| Slicing returns copy or view? | **Copy** | Explicit writes are safer; no accidental mutations |
| `items+=[1,2]` behavior? | **Extend** | More common use case; use `+=[[1,2]]` to append list as element |
| Strict index validation? | **Yes** | Raises `ConfigIndexOutOfBoundsError` for invalid indices |
| Nested list modification? | **Yes** | Supported via chained indexing: `matrix[0][1]=99` |

---

## 9. Test Cases for Acceptance

```python
# MVP Tests
assert c.resolve('items=[1, 2, 3]') == [1, 2, 3]
assert c.resolve('items') == [1, 2, 3]
assert c.resolve('items!') == [1, 2, 3]
assert c.resolve('missing!') is None

# Index Tests  
assert c.resolve('items[0]') == 1
assert c.resolve('items[-1]') == 3
assert c.resolve('items[99]!') is None

# Modification Tests
c.resolve('items[0]=99')
assert c.resolve('items') == [99, 2, 3]

# Persistence Test
c.close()  # Saves to snapshot
c2 = ConfigX(path="same/path")
assert c2.resolve('items') == [99, 2, 3]
```
