# Composite Design Pattern - Mission Management System

A Java implementation demonstrating the **Composite Design Pattern** for managing hierarchical mission structures with nested journeys and sub-missions.

## Design Pattern Overview

### What is the Composite Pattern?

The **Composite Pattern** is a structural design pattern that allows you to compose objects into tree structures to represent part-whole hierarchies. It lets clients treat individual objects and compositions of objects uniformly.

**Core Principle**: *"Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly."* - Gang of Four

### Problem Solved

**Scenario**: A logistics company needs to manage complex missions that can contain:
- Simple journeys (single trips from point A to point B)
- Sub-missions (groups of journeys)
- Nested sub-missions (missions within missions)

**Challenges Without Composite Pattern**:
- Different handling logic for single journeys vs. groups of journeys
- Complex code with type checking and casting
- Difficult to calculate total costs across nested structures
- Hard to add new hierarchy levels

**Solution**: Treat both individual journeys and groups of journeys uniformly through a common interface.

## Architecture

### Class Diagram

```
                    ┌──────────────────────┐
                    │  MissionAbstrait     │ (Component)
                    │  <<abstract>>        │
                    ├──────────────────────┤
                    │ + add(component)     │
                    │ + remove(component)  │
                    │ + getCout(): double  │ (abstract)
                    │ + view(indent): void │ (abstract)
                    └──────────┬───────────┘
                              △
                              │
                ┌─────────────┴─────────────┐
                │                           │
    ┌───────────▼──────────┐    ┌──────────▼─────────┐
    │   TrajetLeaf         │    │     Mission        │ (Composite)
    │   (Leaf)             │    │                    │
    ├──────────────────────┤    ├────────────────────┤
    │ - startLocation      │    │ - List<Mission     │
    │ - endLocation        │    │   Abstrait>        │
    │ - cout               │    │                    │
    │ - startTime          │    │ + add()            │
    │ - endTime            │    │ + remove()         │
    ├──────────────────────┤    │ + getCout()        │
    │ + getCout(): double  │    │ + view()           │
    │ + view(): void       │    └────────────────────┘
    │ + Builder            │            │
    └──────────────────────┘            │
                                       ◇ contains
                                        │
                                        └─ 0..* MissionAbstrait
```

### Tree Structure Example

```
Main Mission (Cost: 600)
├── Sub-Mission (Cost: 300)
│   ├── TrajetLeaf: Paris → Lyon (Cost: 100)
│   │   ├── Start: 2023-05-20
│   │   └── End: 2023-05-21
│   │
│   └── TrajetLeaf: Lyon → Marseille (Cost: 200)
│       ├── Start: 2023-05-22
│       └── End: 2023-05-23
│
└── TrajetLeaf: Marseille → Nice (Cost: 300)
    ├── Start: 2023-05-24
    └── End: 2023-05-25
```

## Pattern Components

### 1. Component (MissionAbstrait)

**Role**: Declares the interface for objects in the composition

```java
abstract class MissionAbstrait {
    // Composition operations (for Composite only)
    public void add(MissionAbstrait component) {
        throw new UnsupportedOperationException();
    }
    
    public void remove(MissionAbstrait component) {
        throw new UnsupportedOperationException();
    }
    
    // Operations for both Leaf and Composite
    public abstract double getCout();
    public abstract void view(String indent);
}
```

**Key Design Decisions**:

**Default Implementation in Abstract Class**:
```java
public void add(MissionAbstrait component) {
    throw new UnsupportedOperationException();
}
```
- Leaf nodes don't need to override (they can't have children)
- Composite nodes override to provide actual implementation
- Throws exception if client tries to add to a leaf (fail-fast principle)

**Alternative Approaches**:
1. **Interface-based**: Methods would be in interface, each class implements
2. **Separate interfaces**: One for Leaf, one for Composite (more complex for clients)
3. **No default implementation**: Force both classes to implement (more boilerplate)

**Why This Approach?**
- **Transparency**: Clients can treat all components uniformly
- **Type Safety**: Compile-time guarantee of method availability
- **Simplicity**: Leaf nodes inherit default behavior without extra code

---

### 2. Leaf (TrajetLeaf)

**Role**: Represents leaf objects in the composition (has no children)

```java
public class TrajetLeaf extends MissionAbstrait {
    private String startLocation;
    private String endLocation;
    private double cout;
    private LocalDate startTime;
    private LocalDate endTime;
    
    @Override
    public double getCout() {
        return cout;  // Simply returns own cost
    }
    
    @Override
    public void view(String indent) {
        // Displays details of this single journey
    }
    
    // Inherits add()/remove() that throw UnsupportedOperationException
}
```

**Characteristics**:
- **Terminal Node**: Cannot contain other components
- **Concrete Implementation**: Implements all abstract operations
- **Data Holder**: Contains actual journey information
- **No Composition Logic**: No child management needed

**Builder Pattern Integration**:
```java
TrajetLeaf trajet = new TrajetLeaf.Builder()
    .withStartLocation("Paris")
    .withEndLocation("Lyon")
    .withCout(100)
    .withStartTime(LocalDate.of(2023, 5, 20))
    .withEndTime(LocalDate.of(2023, 5, 21))
    .build();
```

**Why Builder Pattern Here?**
- **Readability**: Clear which parameter is which
- **Optional Parameters**: Can omit some fields if needed
- **Immutability**: Object created in final state
- **Validation**: Can add validation in build() method

---

### 3. Composite (Mission)

**Role**: Defines behavior for components having children and stores child components

```ava
public class Mission extends MissionAbstrait {
    private final List<MissionAbstrait> missionAbstraitList = new ArrayList<>();
    
    @Override
    public void add(MissionAbstrait mission) {
        missionAbstraitList.add(mission);  // Adds child component
    }
    
    @Override
    public void remove(MissionAbstrait mission) {
        missionAbstraitList.remove(mission);  // Removes child component
    }
    
    @Override
    public double getCout() {
        // Recursive calculation: sum of all children's costs
        return missionAbstraitList.stream()
            .mapToDouble(MissionAbstrait::getCout)
            .sum();
    }
    
    @Override
    public void view(String indent) {
        System.out.println(indent + "Mission (Total Cost: " + getCout() + ")");
        // Recursively display all children with increased indentation
        for (MissionAbstrait mission : missionAbstraitList) {
            mission.view(indent + "    ");
        }
    }
}
```

**Key Features**:

**1. Polymorphic Storage**:
```java
private final List<MissionAbstrait> missionAbstraitList = new ArrayList<>();
```
- Can store both `TrajetLeaf` and `Mission` objects
- Clients don't need to know the difference
- Enables uniform treatment

**2. Recursive Operations**:
```java
public double getCout() {
    return missionAbstraitList.stream()
        .mapToDouble(MissionAbstrait::getCout)  // Calls getCout() on each child
        .sum();
}
```
- Each child could be a Leaf (returns own cost) or Composite (recursively calculates)
- Automatically handles arbitrary depth
- Clean, functional programming style

**3. Tree Traversal**:
```java
public void view(String indent) {
    System.out.println(indent + "Mission (Total Cost: " + getCout() + ")");
    for (MissionAbstrait mission : missionAbstraitList) {
        mission.view(indent + "    ");  // Recursive call with increased indent
    }
}
```
- Depth-first traversal of the tree
- Visual hierarchy through indentation
- Each level increases indent by 4 spaces

---

## How It Works: Step-by-Step

### Cost Calculation Example

```
mainMission.getCout() is called
│
├─ Iterates over children:
│
├─ Child 1: subMission (Composite)
│   │
│   ├─ subMission.getCout() called (recursion)
│   │
│   ├─ Iterates over subMission's children:
│   │   │
│   │   ├─ trajet1 (Leaf): returns 100
│   │   │
│   │   └─ trajet2 (Leaf): returns 200
│   │
│   └─ subMission returns: 100 + 200 = 300
│
├─ Child 2: trajet3 (Leaf)
│   │
│   └─ trajet3.getCout() returns: 300
│
└─ mainMission returns: 300 + 300 = 600
```

### Visual Output Flow

```
mainMission.view("$ ") is called
│
├─ Prints: "$ Mission (Total Cost: 600)"
│
├─ Iterates children:
│
├─ Child 1: subMission.view("$     ")
│   │
│   ├─ Prints: "$     Mission (Total Cost: 300)"
│   │
│   ├─ Child 1.1: trajet1.view("$         ")
│   │   │
│   │   └─ Prints:
│   │       $         TrajetLeaf {
│   │       $           Start Location: Paris
│   │       $           End Location: Lyon
│   │       $           Cost: 100
│   │       $           Start Time: 2023-05-20
│   │       $           End Time: 2023-05-21
│   │       $         }
│   │
│   └─ Child 1.2: trajet2.view("$         ")
│       └─ Prints similar structure
│
└─ Child 2: trajet3.view("$     ")
    └─ Prints similar structure
```

---

## Key Concepts Demonstrated

### 1. **Uniform Treatment**

```java
// Client code treats Leaf and Composite the same way
MissionAbstrait component;  // Could be TrajetLeaf OR Mission

component = new TrajetLeaf.Builder()...build();  // Leaf
System.out.println(component.getCout());  // Works

component = new Mission();  // Composite
component.add(someTrajet);
System.out.println(component.getCout());  // Also works
```

### 2. **Recursive Composition**

```java
Mission level1 = new Mission();
Mission level2 = new Mission();
Mission level3 = new Mission();

level3.add(trajetA);
level2.add(level3);
level2.add(trajetB);
level1.add(level2);
level1.add(trajetC);

// Arbitrary depth - still works!
System.out.println(level1.getCout());  // Calculates entire tree
```

### 3. **Type Safety with Polymorphism**

```java
List<MissionAbstrait> components = new ArrayList<>();
components.add(new TrajetLeaf.Builder()...build());  // Leaf
components.add(new Mission());  // Composite

for (MissionAbstrait comp : components) {
    comp.view("");  // Polymorphic call - correct method invoked
}
```

### 4. **Transparency vs Safety Trade-off**

**Transparency Approach** (Used in this project):
```java
// All methods in base class - uniform interface
abstract class MissionAbstrait {
    public void add(MissionAbstrait component) {
        throw new UnsupportedOperationException();
    }
}
```
**Pros**: Clients treat all objects uniformly
**Cons**: Runtime exception if misused

**Safety Approach** (Alternative):
```java
// Separate interfaces
interface Component { double getCout(); }
interface CompositeComponent extends Component {
    void add(Component c);
}
```
**Pros**: Compile-time safety
**Cons**: Clients must handle types differently

---

## Usage Example

### Building a Complex Mission

```java
// Create individual journeys
TrajetLeaf paris_lyon = new TrajetLeaf.Builder()
    .withStartLocation("Paris")
    .withEndLocation("Lyon")
    .withCout(100)
    .withStartTime(LocalDate.of(2023, 5, 20))
    .withEndTime(LocalDate.of(2023, 5, 21))
    .build();

TrajetLeaf lyon_marseille = new TrajetLeaf.Builder()
    .withStartLocation("Lyon")
    .withEndLocation("Marseille")
    .withCout(200)
    .withStartTime(LocalDate.of(2023, 5, 22))
    .withEndTime(LocalDate.of(2023, 5, 23))
    .build();

// Create sub-mission
Mission franceTour = new Mission();
franceTour.add(paris_lyon);
franceTour.add(lyon_marseille);

// Create main mission
Mission europeTour = new Mission();
europeTour.add(franceTour);
europeTour.add(new TrajetLeaf.Builder()
    .withStartLocation("Marseille")
    .withEndLocation("Nice")
    .withCout(300)
    .withStartTime(LocalDate.of(2023, 5, 24))
    .withEndTime(LocalDate.of(2023, 5, 25))
    .build());

// Get total cost (automatically calculates entire tree)
System.out.println("Total: " + europeTour.getCout());  // 600

// Display hierarchical structure
europeTour.view("→ ");
```

### Expected Output

```
Total Cost of Mission: 600.0

Mission Details:
$  Mission (Total Cost: 600.0)
$      Mission (Total Cost: 300.0)
$          TrajetLeaf {
$            Start Location: Paris
$            End Location: Lyon
$            Cost: 100.0
$            Start Time: 2023-05-20
$            End Time: 2023-05-21
$          }
$          TrajetLeaf {
$            Start Location: Lyon
$            End Location: Marseille
$            Cost: 200.0
$            Start Time: 2023-05-22
$            End Time: 2023-05-23
$          }
$      TrajetLeaf {
$        Start Location: Marseille
$        End Location: Nice
$        Cost: 300.0
$        Start Time: 2023-05-24
$        End Time: 2023-05-25
$      }
```

---

## Design Pattern Benefits

### Advantages

✅ **Simplicity**: Clients use single interface for all components
✅ **Flexibility**: Easy to add new component types
✅ **Scalability**: Handles arbitrary tree depths automatically
✅ **Open/Closed Principle**: Add new components without modifying existing code
✅ **Recursive Operations**: Complex operations become simple
✅ **Polymorphism**: Leverages OOP power effectively

### When to Use

- You need to represent part-whole hierarchies
- You want clients to ignore differences between compositions and individual objects
- You have tree-structured data
- Operations should work uniformly across the structure

### When NOT to Use

- Simple linear structures (overkill)
- Performance-critical applications (recursion overhead)
- Need compile-time type safety for operations
- Components have very different behaviors

---

## Real-World Applications

### 1. **File System**
```
Folder (Composite)
├── File (Leaf) - 10MB
├── Folder (Composite)
│   ├── File (Leaf) - 5MB
│   └── File (Leaf) - 3MB
└── File (Leaf) - 2MB

folder.getSize() → 20MB (recursive calculation)
```

### 2. **GUI Components**
```
Panel (Composite)
├── Button (Leaf)
├── Panel (Composite)
│   ├── TextField (Leaf)
│   └── Label (Leaf)
└── Checkbox (Leaf)

panel.render() → renders entire tree
```

### 3. **Organization Chart**
```
Company (Composite)
├── Department (Composite)
│   ├── Employee (Leaf)
│   └── Team (Composite)
│       ├── Employee (Leaf)
│       └── Employee (Leaf)
└── Department (Composite)

company.getSalaryTotal() → sum of all employees
```

### 4. **Menu System**
```
Menu (Composite)
├── MenuItem (Leaf)
├── SubMenu (Composite)
│   ├── MenuItem (Leaf)
│   └── MenuItem (Leaf)
└── MenuItem (Leaf)

menu.display() → displays entire menu tree
```

---

## Extension Possibilities

### 1. Add Iterator Pattern
```java
public class Mission extends MissionAbstrait implements Iterable<MissionAbstrait> {
    @Override
    public Iterator<MissionAbstrait> iterator() {
        return missionAbstraitList.iterator();
    }
}

// Usage:
for (MissionAbstrait component : mission) {
    System.out.println(component.getCout());
}
```

### 2. Add Visitor Pattern
```java
interface MissionVisitor {
    void visit(TrajetLeaf trajet);
    void visit(Mission mission);
}

abstract class MissionAbstrait {
    public abstract void accept(MissionVisitor visitor);
}

// Use case: Different reporting formats without modifying classes
```

### 3. Add Caching
```java
public class Mission extends MissionAbstrait {
    private Double cachedCost = null;
    
    @Override
    public double getCout() {
        if (cachedCost == null) {
            cachedCost = missionAbstraitList.stream()
                .mapToDouble(MissionAbstrait::getCout)
                .sum();
        }
        return cachedCost;
    }
    
    @Override
    public void add(MissionAbstrait mission) {
        cachedCost = null;  // Invalidate cache
        missionAbstraitList.add(mission);
    }
}
```

### 4. Add Search Functionality
```java
public MissionAbstrait findByLocation(String location) {
    // Search in tree structure
}

public List<TrajetLeaf> getAllTrajets() {
    // Collect all leaf nodes
}
```

---

## Performance Considerations

### Time Complexity
- **getCout()**: O(n) where n = total nodes in tree
- **add()**: O(1) for adding to immediate children
- **view()**: O(n) for traversing entire tree
- **remove()**: O(n) in worst case (ArrayList remove)

---

## Key Takeaways

### Core Principles
1. **Treat objects uniformly**: Composite and Leaf share same interface
2. **Recursive composition**: Composites contain other composites
4. **Tree structure**: Natural representation of hierarchical data

### Implementation Checklist
- ✅ Abstract Component class/interface
- ✅ Leaf class (terminal nodes)
- ✅ Composite class (contains children)
- ✅ Uniform operations (getCout, view)
- ✅ Recursive implementations in Composite
- ✅ Child management (add, remove) in Composite

### Best Practices
- Use `final` for component list to prevent reassignment
- Provide meaningful default implementations
- Document which methods throw `UnsupportedOperationException`
- Consider caching for expensive operations
- Use Builder pattern for complex Leaf objects

---

## Project Structure

```
src/
└── ma/
    ├── Main.java                      # Client code demonstrating usage
    └── trajet/
        ├── MissionAbstrait.java       # Component (abstract base)
        ├── TrajetLeaf.java            # Leaf (terminal node)
        └── Mission.java               # Composite (container)
```

---

## Getting Started

### Prerequisites
- Java 8+ (LocalDate requires Java 8)
- No external dependencies

### Running the Project
```bash
# Compile
javac ma/Main.java ma/trajet/*.java

# Run
java ma.Main
```

---


---

**Project by**: *(Your educational institution or personal project)*
