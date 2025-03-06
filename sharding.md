**The Journey of Scaling a Database in Banking-as-a-Service (BaaS)**

### **Once Upon a Time – The Small Fintech Startup**

In the competitive world of digital banking, a fintech startup launched its Banking-as-a-Service (BaaS) platform. They had a simple Postgres database running on a single EC2 instance, handling all customer transactions, account details, and KYC verifications. Life was good, and their platform performed smoothly.

However, as businesses started integrating their banking APIs, traffic surged. The single database, once sufficient, began struggling under the load, causing slow transactions and delayed account updates.

### **Phase 1: Scaling Vertically – More Power, More Problems**

To handle the rising number of transactions, the engineers upgraded the server with more CPU, RAM, and disk space. Performance improved temporarily. But then came the next wave – thousands of API requests per second for payments, user verifications, and account statements. The database was now gasping for breath.

“Why don’t we just keep adding more power?” someone asked.

The lead engineer sighed. “We can… but only until we hit the hardware limit. Plus, high-end servers are expensive.”

### **Phase 2: Read Replicas – A Helping Hand**

To ease the load, they introduced **read replicas**. Now, the main database (master) handled only writes, while read replicas answered read requests for customer balances, statements, and KYC status.

For a while, things were smooth. The application directed read queries to replicas, and the database was no longer overwhelmed. But soon, customers reported an alarming issue.

> “I just made a transfer, but my balance still shows the old amount!”

The issue? **Replication lag.**

### **The Mystery of Missing Transactions**

One day, a customer initiated a fund transfer. The write operation went to the master database. Immediately after, they checked their account balance, but the read request was served by a replica that had not yet received the updated data. The result? The old balance was displayed, confusing the customer.

The engineers had a dilemma. Should they:

1. Force all balance-related reads to go to the master (slowing performance)?
2. Accept eventual consistency and tell customers to “wait a bit”?
3. Implement a smarter solution?

### **Phase 3: The Workaround – Read After Write Consistency**

They decided on a hybrid approach:

- **Recent transactions? Read from the master.**
- **Older data? Read from replicas.**

This reduced inconsistencies while still offloading most read queries.

### **Phase 4: The Explosion – Beyond One Database**

Then came the fintech boom. Businesses integrated the BaaS platform at an unprecedented rate. Traffic surged past anything they had seen before. Even with read replicas, the master database became the bottleneck.

The engineers had no choice: **It was time to scale horizontally.**

### **Phase 5: Sharding – Breaking the Monolith**

Instead of one giant database, they divided customer data into smaller chunks, distributing them across multiple database instances.

- Customers **A-M** were stored in one database.
- Customers **N-Z** were stored in another.
- More shards could be added as needed.

Each shard operated independently, reducing the load on any single machine.

But sharding brought new challenges:

- **Routing:** How to know which database a customer's account resided in?
- **Joins:** Queries spanning multiple shards became complex.
- **Rebalancing:** If one shard grew too large, data had to be redistributed.

Despite the complexity, sharding saved the day. The fintech’s database could now scale infinitely by simply adding more shards. Transactions were faster, and customers no longer faced delays in balance updates.

### **Final Thoughts – The Moral of the Story**

- **Scaling vertically is easy but has limits.**
- **Read replicas help, but watch out for replication lag.**
- **Sharding is powerful, but it requires careful planning.**
- **For BaaS, ensuring transactional consistency is key to customer trust.**
- **There’s no one-size-fits-all solution – the right choice depends on system needs.**

And so, the fintech thrived, serving millions of users worldwide – all thanks to the power of database scaling.

