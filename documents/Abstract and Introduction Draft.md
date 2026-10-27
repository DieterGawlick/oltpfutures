# The Beginnings of OLTP Systems  

## Abstract

Most online transaction systems can be classified as 'inventory managers' with inventories that range from a stock of physical assets (books, clothing) through more abstract assets (seat or room bookings) to purely digital ones (balances in bank accounts). The ultimate measure of success for any of these systems is whether they solve the requirements of the complete problem: How well the system meets the users' needs.

The advent of business applications that handled this complete problem moved addressing those requirement from being the responsibility of individual applications to be the responsibility of the software infrastructure ("application servers," "transction managers," and so on). generated critical advances in the design and implementation of software.

This paper discusses, at a conceptual level, the solutions developed for creating effective OLTP applications (with references to &lt;…&gt; for the details) and draws some conclusions about which of those requirements are still relevant today.

## Introduction

At the start of the 1970s, the most challenging applications in IT were online systems that managed bank accounts and airline seats.

Prior to moving to OLTP, activities in these areas began with professionals (bank tellers, travel agents, airline employees) interacting with users. Those activities gathers the information required for those business transactions but the transactions were actually processed outside of business hours in batch mode.

Processing in batch mode provided multiple benefits:

- There was, potentially, only one actor (the batch transaction) processing changes against the database
- All the steps making up any transactions did not have to be complete until the batch application finished processing all transactions (the database would be unavailable during processing)
- If there was a problem during batch processing, recovery was relatively simple: The database was restored to the start of processing and the application was rerun (after addressing any errors)

The new OLTP applications imposed a new set of requirements:

- Typically, at any time, multiple actors would be simultaneously executing overlapping transactions
- Individual transactions had to complete independently of any other transactions executing at the same time
- With multiple, overlapping transactions constantly completing, there is no clear recovery point for restarting processing

In addition, in a "self-serve" environment, users might no longer be interacting with a professional who knew how to use the system but would have to figure out the user interface for the application on their own.

Specifically, the user's criteria were:

- Managing the transaction from the user to the database and back to the user in a way that made sense to the user
- Executing those transactions at the "speed of life"

The business' criteria were

- Accuracy
- Reliability

In this paper, we'll focus on the solutions for those problems, as developed for OLTP debit/credit systems (specifically, banking systems) because the criteria that had to be met for banking systems were more stringent than, for example, those for airline booking systems:

- Banking systems worked with a purely digital inventory while airline booking systems managed had, in airline seats, at least a partially physical resource
- Airline systems had a physical "record of transaction" in the issued airline ticket that was the ultimate arbitrator of the result of the transaction
- While airline systems could live with a low level of inaccuracy (double booking seats was considered a "cost of doing business"); Banking systems had to be completely accurate.
- At the time that OLTP systems were being developed, airline booking systems were still run by professionals that user interacted with; With banking systems, users at an ATM were often interacting directly with the application

## Defining the Problem

A key part of building the initial OLTP systems was to recognize the nature of the complete problem. It is possible to define the problem of "electronic banking" purely from the point of view of the database. In that case, the problem appears to be:

1. Begin a transaction
2. Within that transaction, make one or more updates
3. Commit (or rollback) the transaction

This is a form of tunnel vision that ignores real issues around meeting the users' criteria. To address the complete problem, system developers need to consider the user's point of view, which was:

1. Prepare a sign-in message from some device (ATM, teller's terminal)
2. Send the message
3. Receive a response message with account information
4. Allow the user to specify the request
5. Prepare a message with the request
6. Send the request
7. Receive an acknowledgement
8. Present the user with updated account information (and, for a withdrawal, dispense cash)

- Functional requirements of debit/credit
  - Communication between the end user and the database
    - Clerk
    - ATM
  - Transactionality (a set of things have to be correct or not correct: ACID)
    - Previously a responsibility of the application
    - Batch processing more forgiving than managing multiple, overlapping transactions
      - Only need to be correct at the end of the batch
      - Supports restart
  - Reliability
    - No data loss
    - Isolate any problems
      - Any part of the database
      - Dealing with disk errors: if the disk doesn't work – use a copy of the disk
    - Restart in the event of unrecoverable errors without
      - Programmer intervention
      - Interruption to the UX
  - Operational requirements
    - Dealing with hotspots
    - Performance
    - Throughput
- Key Concepts
  - Messaging system
  - Database system
- Reflection
  - Messaging
    - As part of the transaction
    - Stateful
    - Workflows
      - User and database as executing coordinated workflows
  - Database
    - Stateless
    - Hierarchies
    - Networks
  - The Great Detour: SQL
