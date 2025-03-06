**The Journey of Scaling a Database – A Banking-as-a-Service (BaaS) Story**

### **Once Upon a Time – A Fintech Startup is Born**
In a quiet corner of the fintech world, a small startup launched its Banking-as-a-Service (BaaS) platform. They provided digital wallets, real-time transactions, and instant settlements. At first, their MySQL database running on a single EC2 instance handled everything—account balances, transactions, and user profiles. Life was good.

But as word spread, businesses flocked to their platform, and transaction volumes soared. The single database, once enough, began struggling under the load.

### **Phase 1: Scaling Vertically – More Power, More Problems**
To handle the rising transactions, the engineers upgraded the server with more CPU, RAM, and disk space. Performance improved for a while. But then came the next surge – from 100 to 1000 transactions per second. The database was now gasping for breath.

“Why don’t we just keep adding more power?” someone asked.

The lead engineer sighed. “We can… but only until we hit the hardware limit. Plus, these powerful machines cost a fortune.”

### **Phase 2: Read Replicas – A Helping Hand**
To ease the load, they introduced **read replicas**. Now, the main database (master) handled only writes, while read replicas answered read requests.

For a while, things were smooth. The application directed read queries to replicas, and the database was no longer overwhelmed. But soon, users reported something strange.

> “I just transferred money, but my balance still shows the old amount!”

The issue? **Replication lag.**

### **The Mystery of Missing Transactions**
One day, a user transferred $500 from their wallet to a merchant. The write operation went to the master database. Immediately after, they refreshed their app to check their balance – but the data was fetched from a read replica, which had not yet received the update. The result? The user still saw the old balance, causing confusion.

This was unacceptable in a financial system. The engineers had a dilemma. Should they:
1. Force all reads to go to the master (slowing performance)?
2. Accept eventual consistency and tell users to “wait a bit”?
3. Implement a clever workaround?

### **Brainstorming a Solution**
The engineering team gathered in a war room, whiteboards filled with diagrams and question marks.

- **Option 1: Read from Master Only**
  - ✅ Solves the consistency issue immediately.
  - ❌ Increases load on the master, making scaling harder.

- **Option 2: Delayed Reads with a Retry Mechanism**
  - ✅ Gives replicas time to catch up.
  - ❌ Adds complexity, and users might still see outdated data.

- **Option 3: Read After Write Consistency (Hybrid Approach)**
  - ✅ Ensures recent writes are always read from the master.
  - ✅ Most other reads can still be served from replicas.
  - ✅ Balances consistency and performance.

After much debate, they chose **Read After Write Consistency**:
- **Recent updates? Read from the master.**
- **Everything else? Read from replicas.**

This reduced inconsistencies while still offloading most of the read queries.

### **Phase 4: The Explosion – Beyond One Database**
Then came the viral moment. A major neobank partnered with them, causing traffic to surge past anything they had seen before. Even with read replicas, the master database became the bottleneck.

The engineers had no choice: **It was time to scale horizontally.**

### **Phase 5: Sharding – Breaking the Monolith**
Instead of one giant database, they divided data into smaller chunks, distributing them across multiple database instances.
- Customers **A-M** went to one database.
- Customers **N-Z** went to another.
- More shards could be added as needed.

Each shard operated independently, reducing the load on any single machine.

But sharding brought new challenges:
- **Routing:** How to know which database a user belonged to?
- **Joins:** Queries spanning multiple shards became complex.
- **Rebalancing:** If one shard got too big, data had to be moved.

Despite the complexity, sharding saved the day. The BaaS platform could now scale infinitely by simply adding more shards.

### **Final Thoughts – The Moral of the Story**
- **Scaling vertically is easy but has limits.**
- **Read replicas help, but watch out for replication lag.**
- **Sharding is powerful, but it requires careful planning.**
- **There’s no one-size-fits-all solution – the right choice depends on your system’s needs.**

And so, the startup thrived, serving millions of transactions daily – all thanks to the power of database scaling.

