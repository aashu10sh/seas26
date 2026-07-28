---
---
# Day 2 - Compute

---
layout: statement
---
Compute is the act of processing information according to a set of instructions, or the computational resources used to perform that processing.

---
---

Types of compute
<v-clicks>

1. CPU bound tasks
2. IO bound tasks
</v-clicks>

---
---
# CPU bound tasks
<v-clicks>

A CPU-bound task is a task whose performance is limited primarily by how fast the CPU can execute instructions.

Think about:

- performing arithmetic
- comparing numbers
- branching

Activities:
- Video encoding (FFmpeg)
- Image processing (Photoshop filters)
- Data compression (gzip, zstd)
- Encryption/decryption (AES, RSA)
- Scientific simulations
- AI inference and model training (often GPU-bound instead if run on GPUs)

</v-clicks>

---
---

# IO bound tasks
A program is IO bound when its speed is limited by how long it takes to interact with external systems. These systems might be disks, databases, networks, or even a user’s input.

<v-clicks>

Think about:

- Network (HTTP requests, APIs)

- Disk (reading/writing files)

- Databases

- Another process or service

- User input

This type of work is actively waiting for components out of their control. The CPU itself has no trouble keeping up—it is the external resource that introduces delays.

</v-clicks>

---
---
## Let's look at an example!
Take a look at this code snippet.

```js
app.get("/users/:id", async (req, res) => {
    const user = await db.query(
        "SELECT * FROM users WHERE id = ?",
        [req.params.id]
    );

    res.json(user);
});

```
```mermaid
graph LR
  RequestArrives --> ParseHTTPRequest
  ParseHTTPRequest --> SendSQLQuery
  SendSQLQuery --> WaitingForDB
  WaitingForDB --> SerializeJSON
  SerializeJSON --> ReturnResponse
```
```
CPU
│
├── Parse HTTP request       (50 μs)
├── Send SQL query           (20 μs)
│
│   WAIT 15 ms
│
├── Serialize JSON           (100 μs)
└── Send response
```

---
layout: statement
---
Let's do a little excercise?

---
---
Is it CPU bound or is it IO bound?

<v-clicks>

- Downloading a 2 GB file from the internet
- Resizing 10,000 images
- Decrypting a large encrypted file
- Reading a 50 GB log file from an SSD
- A Node.js server receives a request, queries PostgreSQL, and returns JSON.
</v-clicks>

---
layout: statement
---
Backend services don't spend most of their time thinking - they spend most of their time waiting.

---
---
IO-bound work wastes CPU if handled naively - the thread just sits there blocked, doing nothing, while the OS could've run something else. That single insight motivates everything that follows.

<v-click>

This profund realisation has led to many asynchronous runtime optimised for IO bound work rather than CPU bound work.
</v-click>

<v-click>

Node JS Runtime ( async event loop architecture )
</v-click>

---
---
nodeloops.com

---
---

![IO vs CPU bound work](https://substackcdn.com/image/fetch/$s_!4XiT!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8e0d481e-4c53-492b-8722-32ddfb58dadc_1400x704.png)

---
---
# Processes and threads
The building blocks of an operating system.

<v-clicks>

Process

Process is a program that is currently in execution within an operating system. It operates in an independent environment and is managed by the OS for proper scheduling and execution. Processes form the basis of program execution in a multitasking system.

Threads

Thread is a smallest unit of execution within a process. It enables a program to perform multiple tasks concurrently while sharing the same memory and resources. Threads improve application performance and responsiveness in multitasking environments.


### Threads run on CPU Cores - The more CPU cores you have the more things you can run in parallel.
</v-clicks>

---
---
An example - MS Docs!


---
---
<v-clicks>

Threads are excellent for CPU bound work!

What happens with threads on a single-core system?

But how does a thread perform for IO bound work?

**Threads are very expensive, there is a genuine cost when creating threads in an operating system.**

</v-clicks>

---
layout: statement
---
![Threaded System](https://substackcdn.com/image/fetch/$s_!fRmu!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F20224380-b92e-4697-86b6-3809ba0b0e7c_1024x604.jpeg)


---
layout: statement
---
Parallelism vs Concurrency

---
---

Concurrency is about **dealing with multiple tasks**.

Parallelism is about **doing multiple tasks at the same time**.

---
---

# Example
## Concurrency
Concurrency means your program can make progress on multiple tasks by switching between them.

> Imagine a chef trying to cook pasta, He first boils the water, as the water is boiling he starts chopping the onions, once the water is boiled - he adds the pasta to the water and gets back to chopping the onions, once the onions are done, he starts cooking the sauce and everything happens. The chef does not sit idle waiting for one task for finish.

The chef isn't doing two things at once.

They're simply not wasting time while waiting.

This is concurrency.

---
---
# Example
## Parallelism
Parallelism means two or more CPUs (or cores) are literally executing instructions simultaneously.

Imagine four chefs.
> Chef 1 - Cutting Onions, Chef 2 - Cook Pasta, Chef 3 - Grill Chicken, Chef 4 - Make dessert

Everything happens at the same time.

That's parallelism.

---
---
https://algomaster.io/animations/concurrency/concurrency-vs-parallelism

---
layout: statement
---
# Vertical vs Horizontal Scaling

---
---
# Server vs Serverless Compute ( lambdas )

<v-clicks>

With traditional servers, you own (or rent) machines that are always running.

With serverless, there is no always-running server from your perspective.

</v-clicks>

---
---
Something like this:

```js
export async function handler(event) {
    return {
        statusCode: 200,
        body: "Hello!"
    };
}
```

Can be priced:
- per running time
- per invocation

---
---

# Huge difference in what you pay.

An stateful server running 24/7 vs an function that only runs when invoked.

## Tradeoffs:

1. cold starts, 
2. execution time limits, 
3. statelessness forced on you


# Lambda is a valid compute option
cloudflare containers, cloudflare workers, aws lambda, google functions, ECS with EC2 and Fargate

---
---
# Case studies

<v-clicks>

- video transcoding pipeline
- a chat backend or API gateway fanning out to multiple downstream services
</v-clicks>

---
layout: quote
---

"Compute is the currency of the future"

Sam Altman
