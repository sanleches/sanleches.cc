---
title: Parking Garage Scheduling, Because Apparently That Is Fun
date: 2026-08-06 13:00:00
category: projects
tags:
  - C++
  - algorithms
  - CMake
  - simulation
---

[PGOF](https://github.com/sanleches/PGOF) started as a Parking Garage of the Future design challenge. The scaffold and capacity-splitting problem came from the upstream project; I used the fork to build out my solution, tests, reports, logging, and a small deterministic demo.

The problem is simple enough to explain without a diagram. Cars have sizes from 1 to 3. Parking spaces also have sizes from 1 to 3. Cars arrive in a queue, ask to stay for a number of ticks, and should take the smallest compatible available space. Fees are `car size * requested time`.

Then the annoying cases show up.

## A queue that cannot always be FIFO

Suppose the first waiting car is size 3, but only a size 1 space is open. Strict FIFO would block every smaller car behind it even though one of them could park immediately.

My queue keeps arrival order as the default but can pull the first later car that fits. It temporarily moves skipped cars aside, selects one compatible car, and rebuilds the queue without changing the order of everybody else.

That is not a global fee optimizer. It is a practical first-eligible policy that keeps spaces moving without turning a small simulation into an operations-research thesis.

## The model

The C++17 project is split into small pieces:

- `PgofApp` coordinates arrivals, ticks, fees, reports, and shutdown.
- `ParkingLotHandler` owns spaces and handles parking, countdowns, and departures.
- `CarQueue` owns waiting order and compatible-car selection.
- `FeeHandler` tracks totals and writes operation logs.
- `garage_capacity` divides the requested number of spaces across the three size buckets.

Each tick advances time, removes cars whose stay has expired, and tries to park a waiting car. Spaces are stored in ascending size order, so the normal linear scan naturally finds the smallest compatible space first.

The demo is intentionally deterministic:

```bash
cmake -S . -B build
cmake --build build
ctest --test-dir build --output-on-failure
./build/apps/pgof_app/pgof 10 12
```

That makes it useful for watching state transitions and checking reports without needing an interactive UI.

## A pointer problem hiding in a value type

One implementation detail I enjoyed was `ParkingSpace`. It owns storage for a car but also keeps an internal pointer to the active car. Default copy and move behavior could leave that pointer aimed at the old object's storage. The class has explicit copy and move operations that rebind the pointer to the new object's owned value.

This is exactly the kind of C++ problem that looks boring until a vector moves an object and your apparently valid pointer starts referring to yesterday's memory.

## Prototype rules apply

The implementation uses linear scans and in-memory state. One tick parks one car, logs are plain files in the working directory, and input validation is not complete. That is fine for a design exercise, but I would not connect it to a million-space garage and hope for the best.

The useful part was turning a short list of rules into explicit objects, ownership decisions, queue behavior, tests, and a simulation I could reason about one tick at a time.

Code: [github.com/sanleches/PGOF](https://github.com/sanleches/PGOF)
