# From Database Transactions to Business Transactions

_A fundamental question in transaction-processing design is:_ **_Are we trying to make the user live within the limits of the system, or are we building the system to give the user and the business what they actually need?_**

The distinction between what "we could deliver" vs. "what do the users want" helps explain an important difference between the transaction-processing benchmarks that emerged around TPC-A and the philosophy embodied in IMS/Fast Path.

TPC-A modeled a hypothetical bank and measured a debit/credit-style transaction involving an account, teller, branch, and transaction history. Its database represented the cash position of those entities and a history of recent transactions. This was useful because it provided an objective way to compare the transaction-processing performance of systems. But the benchmark necessarily reduced the banking problem to a standardized database operation.

The result was a database-centered definition of the transaction.

The transaction began, in effect, when the application asked the database to perform work and ended when that database work completed. What happened before and after that database interaction was largely outside the benchmark's concern. In an ATM withdrawal, however, that leaves out much of what matters to the customer. The customer inserts a card, authenticates, requests \$100, receives an approval, waits for the ATM, and finally receives—or does not receive—the cash.

The actual business transaction therefore cannot be understood merely as an update to an account balance.

The ATM and bank participate in a message-driven workflow. The ATM sends an authentication request, receives authentication, sends a cash request, receives authorization, and dispenses cash. From the customer's perspective, the transaction is successful only if the requested business outcome occurs. Debiting an account without dispensing the money is clearly not success.

TPC-A was valuable for measuring a portion of this process, but it did not attempt to represent that entire interaction. The benchmark asks, essentially, **How efficiently can the system perform the recoverable database work?** IMS/Fast Path addressed the larger question: **How can the system reliably deliver the business service the customer requested?**

That difference substantially changes the architecture.

IMS/Fast Path treated messages as part of transaction processing rather than as peripheral communications. A message arriving from an ATM initiated work. The account might be checked, an amount placed in escrow, statistics updated, a ledger entry created, and a response returned to the ATM. Message numbering allowed missing or duplicated transmissions to be detected, supporting effectively exactly-once processing across an unreliable communications environment.

This makes an ATM withdrawal resemble what would later be called a **saga**, even though that terminology did not yet exist. Garcia-Molina and Salem formally introduced the saga concept in 1987, describing a long-lived activity as a sequence of transactions whose partial execution can be corrected through compensating transactions. The IMS/Fast Path ATM interaction was much shorter and simpler, but structurally similar: several recoverable steps were connected by messages, and intermediate state could exist while the overall business activity remained incomplete.

Escrow was particularly important. When the customer requested \$100, the bank did not have to pretend that the complete withdrawal had already occurred. It could reserve \$100 so that it was unavailable for another transaction while awaiting the next step. The system represented both the current state and an intended future state. If the ATM dispensed the cash, the withdrawal could be finalized. If the workflow failed, the escrow could be released.

This is a much richer concept of a transaction than simply changing a database record.

It also changes the importance of messages. Between autonomous parties, the message is often the only mutually visible evidence of the interaction. Each participant has its own private database and internal workflow; what both parties can agree upon is what was communicated. Consequently, messages should not be ephemeral notifications. They should become immutable records of what was requested, accepted, rejected, or completed. A transactional messaging system should keep message creation and consumption consistent with the associated database state.

Together with the recovery journal and business ledger, those retained messages form an authoritative history of what actually happened.

This distinction remains important today. Much of modern application architecture still begins with the needs of the database or infrastructure: what can conveniently be committed, what an API exposes, what a message broker guarantees, or what a particular cloud service supports. The application is then designed around those limitations.

IMS/Fast Path demonstrated the opposite direction of reasoning. Start with the outcome required by the customer and the business. Determine what must be recoverable, what must be communicated, what must temporarily be reserved, and what evidence must remain afterward. Then make the infrastructure support that complete interaction.

TPC-A helped the industry measure an important component of transaction processing. But the danger is mistaking the measurable component for the entire problem. The technology required to do more already existed. IMS/Fast Path showed that transaction processing could encompass the database, messages, recovery, escrow, and the user's experience as parts of one coherent business process.

The larger lesson is that the purpose of OLTP is not to make users conform to database transactions. It is to make database transactions serve the transactions that users and businesses actually conduct.