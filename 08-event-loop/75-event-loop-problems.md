# Lesson 75 — Event Loop Output Problems

This lesson is dedicated to practice.

The goal is not to memorize outputs.

The goal is to use a repeatable tracing method.

Use this every time:

```text
1. Run synchronous code

2. Record microtasks

3. Record tasks/macrotasks

4. Drain all microtasks

5. Run one task

6. Record new work

7. Drain microtasks again

8. Repeat
```

---

# Problem 1 — Basic Timer and Promise

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("C");
  });

console.log("D");
```

### Trace

Synchronous:

```text
A
D
```

Microtask:

```text
C
```

Task:

```text
B
```

### Output

```text
A
D
C
B
```

---

# Problem 2 — Chained Promises

```js
Promise.resolve()
  .then(() => {
    console.log("A");
  })
  .then(() => {
    console.log("B");
  });

Promise.resolve()
  .then(() => {
    console.log("C");
  });
```

Initial microtasks:

```text
A-handler
C-handler
```

After A executes, B becomes queued.

Queue:

```text
C-handler
B-handler
```

### Output

```text
A
C
B
```

---

# Problem 3 — Promise Inside Timer

```js
setTimeout(() => {
  console.log("A");

  Promise.resolve()
    .then(() => {
      console.log("B");
    });
}, 0);

setTimeout(() => {
  console.log("C");
}, 0);
```

### Trace

First timer task:

```text
A
```

It queues microtask:

```text
B
```

Before second timer task, microtasks drain.

Then:

```text
C
```

### Output

```text
A
B
C
```

---

# Problem 4 — Timer Inside Promise

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    setTimeout(() => {
      console.log("B");
    }, 0);
  });

setTimeout(() => {
  console.log("C");
}, 0);
```

### Trace

Initial microtask:

```text
A-handler
```

Initial task:

```text
C timer
```

A runs and schedules B timer later.

Tasks now:

```text
C
B
```

### Output

```text
A
C
B
```

---

# Problem 5 — Promise Constructor

```js
console.log("1");

new Promise((resolve) => {
  console.log("2");

  resolve();
}).then(() => {
  console.log("3");
});

console.log("4");
```

Promise executor runs synchronously.

### Output

```text
1
2
4
3
```

---

# Problem 6 — async/await

```js
async function test() {
  console.log("A");

  await Promise.resolve();

  console.log("B");
}

console.log("C");

test();

console.log("D");
```

### Trace

Synchronous:

```text
C
A
D
```

Then async continuation:

```text
B
```

### Output

```text
C
A
D
B
```

---

# Problem 7 — queueMicrotask

```js
console.log("1");

queueMicrotask(() => {
  console.log("2");
});

Promise.resolve()
  .then(() => {
    console.log("3");
  });

console.log("4");
```

Both callbacks are microtasks.

Queued order:

```text
2
3
```

### Output

```text
1
4
2
3
```

---

# Problem 8 — Nested Microtask

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    queueMicrotask(() => {
      console.log("B");
    });
  });

Promise.resolve()
  .then(() => {
    console.log("C");
  });
```

Initial queue:

```text
A
C
```

After A:

```text
C
B
```

### Output

```text
A
C
B
```

---

# Problem 9 — Mixed Complex Example

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("3");

    setTimeout(() => {
      console.log("4");
    }, 0);
  })
  .then(() => {
    console.log("5");
  });

queueMicrotask(() => {
  console.log("6");
});

console.log("7");
```

### Step 1 — synchronous

```text
1
7
```

### Initial microtasks

```text
3-handler
6
```

### Initial tasks

```text
2
```

Run 3-handler:

```text
prints 3
schedules timer 4
queues 5-handler
```

Microtask queue:

```text
6
5
```

Then tasks:

```text
2
4
```

### Output

```text
1
7
3
6
5
2
4
```

---

# Problem 10 — Multiple await

```js
async function run() {
  console.log("A");

  await 1;

  console.log("B");

  await 2;

  console.log("C");
}

run();

console.log("D");
```

### Trace

Synchronous:

```text
A
D
```

After first await:

```text
B
```

Then another await schedules the next continuation.

Then:

```text
C
```

### Output

```text
A
D
B
C
```

---

# Problem 11 — Promise Chain Interleaving

```js
Promise.resolve()
  .then(() => {
    console.log("A");
  })
  .then(() => {
    console.log("B");
  });

Promise.resolve()
  .then(() => {
    console.log("C");
  })
  .then(() => {
    console.log("D");
  });
```

Initial queue:

```text
A-handler
C-handler
```

Run A:

```text
queue B
```

Queue:

```text
C
B
```

Run C:

```text
queue D
```

Queue:

```text
B
D
```

### Output

```text
A
C
B
D
```

---

# Problem 12 — Timer Schedules Promise

```js
setTimeout(() => {
  console.log("A");

  Promise.resolve()
    .then(() => {
      console.log("B");
    });

  console.log("C");
}, 0);

setTimeout(() => {
  console.log("D");
}, 0);
```

First timer task:

```text
A
C
```

Then microtask:

```text
B
```

Then next timer:

```text
D
```

### Output

```text
A
C
B
D
```

---

# Problem 13 — Promise Schedules Promise and Timer

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    Promise.resolve()
      .then(() => {
        console.log("B");
      });

    setTimeout(() => {
      console.log("C");
    }, 0);
  });

setTimeout(() => {
  console.log("D");
}, 0);
```

Initial microtask:

```text
A
```

Initial task:

```text
D
```

A queues:

```text
microtask B
task C
```

Microtasks drain:

```text
B
```

Then existing task order:

```text
D
C
```

### Output

```text
A
B
D
C
```

---

# Problem 14 — Promise Error Handling

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    throw new Error("fail");
  })
  .catch(() => {
    console.log("B");
  })
  .then(() => {
    console.log("C");
  });

console.log("D");
```

Synchronous:

```text
D
```

Then Promise chain:

```text
A
↓
throw
↓
catch B
↓
catch returns undefined
↓
chain fulfilled
↓
then C
```

### Output

```text
D
A
B
C
```

---

# Problem 15 — Timer + async Function

```js
async function test() {
  console.log("A");

  await Promise.resolve();

  console.log("B");
}

setTimeout(() => {
  console.log("C");
}, 0);

test();

Promise.resolve()
  .then(() => {
    console.log("D");
  });

console.log("E");
```

### Synchronous

```text
A
E
```

### Initial microtasks

```text
resume test → B
Promise handler → D
```

### Task

```text
C
```

### Output

```text
A
E
B
D
C
```

---

# Problem 16 — Microtask Starvation Concept

```js
function repeat() {
  queueMicrotask(repeat);
}

repeat();

setTimeout(() => {
  console.log("timer");
}, 0);
```

The microtask queue keeps refilling.

Conceptually, the timer may never get a chance to run.

Lesson:

> Microtasks are high priority, but abusing them can starve tasks and rendering.

---

# Problem 17 — Blocking Synchronous Code

```js
setTimeout(() => {
  console.log("timer");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise");
  });

const start = Date.now();

while (
  Date.now() - start < 2000
) {}

console.log("done");
```

The loop blocks all async work.

Then:

```text
done
promise
timer
```

Full output:

```text
done
promise
timer
```

---

# Problem 18 — Final Challenge

Predict:

```js
console.log("1");

setTimeout(() => {
  console.log("2");

  Promise.resolve()
    .then(() => {
      console.log("3");
    });
}, 0);

Promise.resolve()
  .then(() => {
    console.log("4");

    queueMicrotask(() => {
      console.log("5");
    });
  })
  .then(() => {
    console.log("6");
  });

queueMicrotask(() => {
  console.log("7");

  setTimeout(() => {
    console.log("8");
  }, 0);
});

console.log("9");
```

### Step 1 — sync

```text
1
9
```

Initial microtasks:

```text
4-handler
7-handler
```

Initial tasks:

```text
2-timer
```

Run 4:

```text
print 4
queue 5
queue 6 after chain settles
```

Microtask queue becomes:

```text
7
5
6
```

Run 7:

```text
print 7
schedule timer 8
```

Then:

```text
5
6
```

Tasks:

```text
2-timer
8-timer
```

Run timer 2:

```text
print 2
queue microtask 3
```

Before timer 8:

```text
3
```

Then:

```text
8
```

### Final Output

```text
1
9
4
7
5
6
2
3
8
```

---

# The Interview Method

Never solve event-loop questions by intuition alone.

Write three buckets:

```text
SYNC

MICROTASKS

TASKS
```

Then simulate queue changes after every callback.

This approach scales much better than memorization.

---

# Section 8 Complete Mental Model

```text
current task starts
↓
sync code runs
↓
async APIs schedule work
↓
current stack finishes
↓
drain microtasks completely
↓
browser may render
↓
run next task
↓
drain microtasks again
↓
repeat
```

---

# Key Takeaways

- Always classify sync, microtask, and task work.
- Promise handlers are microtasks.
- Timers are tasks/macrotasks.
- Promise-chain handlers are queued progressively.
- A task can create microtasks that run before the next task.
- A microtask can schedule later tasks.
- async/await continuations follow Promise/microtask semantics.
- Long synchronous work blocks everything.
- Microtask starvation can delay tasks and rendering.
- Event-loop questions are best solved with explicit queue tracing.
