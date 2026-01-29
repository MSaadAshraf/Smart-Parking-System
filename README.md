# Smart Parking System (C++ Backend)

Welcome to the **Smart Parking System** backend project. This documentation is designed not only to explain how the code works but also to help you prepare for your viva/interview questions regarding C++ development.

## 1. Project Overview
This project is a backend server for a Smart Parking System implemented in C++. It manages parking zones, areas, slots, and requests. It simulates a real-world scenario where vehicles request parking, and the system allocates the best available slot based on logic.

**Key Features:**
*   **Zone-based Architecture**: The city is divided into Zones, Areas, and Slots.
*   **Automatic Allocation**: Finds the optimal slot for a vehicle.
*   **State Management**: Tracks which vehicle is in which slot.
*   **Rollback Mechanism**: Supports "undoing" operations (Command Pattern).
*   **HTTP API**: Exposes endpoints via `httplib` for clients (frontend/mobile apps).

---

## 2. Architecture & Flow

The system follows a layered architecture:

`HTTP Server (server.cpp)` -> `Controller (ParkingSystem)` -> `Logic (AllocationEngine)` -> `Data Models (Zone, Slot)`

### Typical Request Flow:
1.  **Request**: User hits `POST /api/request` with `vehicleId` and `zoneId`.
2.  **Server**: `server.cpp` parses params and calls `ParkingSystem::requestParking`.
3.  **Core Logic**: `ParkingSystem` asks `AllocationEngine` to find a slot.
4.  **Allocation**:
    *   Checks the requested Zone.
    *   If full, checks adjacent/other zones (Cross-zone allocation).
    *   Returns a `Slot` and `Zone`.
5.  **State Update**: `ParkingSystem` marks slot as occupied, creates a `ParkingRequest`, and logs the operation to `RollbackManager`.
6.  **Response**: Returns JSON with the assigned Request ID.

---

## 3. Key C++ Concepts Used

Be prepared to explain these concepts in your viva:

### A. STL Containers (Standard Template Library)
*   **`std::vector`**: Used extensively (e.g., `vector<Zone>`, `vector<ParkingSlot>`) to store dynamic lists of objects.
    *   *Viva Q:* Why vector? *Ans:* It provides dynamic resizing and O(1) random access.
*   **`std::stack`**: Used in `RollbackManager` to store history.
    *   *Viva Q:* Why stack? *Ans:* We need Last-In-First-Out (LIFO) behavior to "undo" the most recent action first.

### B. Object-Oriented Programming (OOP)
*   **Encapsulation**: Classes like `ParkingSlot` and `Zone` hide their internal data (`private:` members) and expose methods (`public:`) to manipulate them.
*   **Composition**: A `ParkingSystem` *has-a* `Zone`, a `Zone` *has-a* `ParkingArea`.

### C. Pointers & References
*   **Const References (`const Zone&`)**: Used in function parameters to avoid copying large objects while protecting them from modification.
    *   *Efficiency:* Passing by value copies data; passing by reference passes the address.
*   **Pointers (`ParkingSlot*`)**: Functions like `findSlotById` return a pointer. If the slot isn't found, it returns `nullptr`.

### D. Memory Management
*   The project largely uses stack allocation and STL containers which manage memory automatically (RAII - Resource Acquisition Is Initialization).

---

## 4. Viva Preparation: Questions & Answers

**Q1: How does your system handle concurrent requests?**
*   *Answer:* Currently, `httplib` handles threads for connections, but our `ParkingSystem` logic is single-threaded (or not explicitly protected by mutexes). In a production system, we would add `std::mutex` locks around shared resources like the `zones` vector to prevent race conditions.

**Q2: Explain the "Rollback" feature.**
*   *Answer:* It implements the **Command Pattern**. Every time we allocate or cancel a slot, we push an `Operation` object (containing details like request ID and slot ID) onto a `std::stack` in `RollbackManager`. To rollback, we pop from the stack and perform the inverse operation (e.g., if we allocated, we now free the slot).

**Q3: What is the time complexity of finding a parking slot?**
*   *Answer:* If there are $Z$ zones, $A$ areas per zone, and $S$ slots per area:
    *   Worst case is $O(Z \times A \times S)$ as we iterate through everything linearly.
    *   *Optimization:* We could use a Hash Map (`unordered_map`) or a Set of free slots to make this $O(1)$ or $O(\log N)$.

**Q4: Why `server.cpp` instead of a framework like Django/Node?**
*   *Answer:* C++ provides high performance and low latency, which is critical for real-time systems. Using `httplib` keeps it lightweight without the overhead of a heavy framework.

**Q5: What happens if `new` fails?**
*   *Answer:* (Trick question, we mostly use stack/vectors) If `push_back` fails due to memory, `std::bad_alloc` is thrown. We would need try-catch blocks to handle it gracefully, though modern OSs rarely run out of memory for small apps.

---

## 5. API Reference (Quick Cheatsheet)

*   `GET /api/data`: detailed JSON of all zones and slots.
*   `POST /api/request`: `vehicleId=ABC&zoneId=1` -> Returns `{requestId: ...}`
*   `POST /api/leave`: `requestId=...` -> Returns success status.
*   `POST /api/rollback`: `k=1` -> Undoes the last k actions.

---

*Good luck with your Viva! Focus on knowing the flow of data and why you chose specific data structures.*
