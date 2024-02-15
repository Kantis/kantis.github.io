---
title: A lesson in idempotency
author: Kantis
date: 2023-05-24
tags:
  - Postmortem
---

Today a client reached out. They use software which I've built to manage invoicing between companies in their group. Problem was that some intra-group invoices had been handled but no invoices had been sent to their customer. 

The software integrates with the economy system used by _Consulting Group_. Every invoice received from _Consulting Company_ needs to generate an invoice to _Customer_. In order to keep the solution simple, I decided to keep the economy system itself as the single source of information. The list of received invoices acts as the "Todo list". So for each received invoice, we look up the invoice sent from _Consulting company_ to create a mirrored version of it which will be sent to _Customer_. After creating the mirrored client invoice, we mark the received invoice as processed and our todo-list is ticked off.

![A sketch showing that Consulting company sends invoices to consulting group, which sends invoices to Customer](/assets/Untitled_Artwork.png)

## Timeouts
So after running this for quite a while, we ran into an issue were requests to create the mirrored client invoice occasionally timed out. No problem, the software was already set up to do retries so we self-healed from this situation. Except, sometimes the timed out request had actually succeeded which left a "dangling" invoice in the system.

## Solving the dangling invoices
After discussions with the vendor of the economy system, they added support for including an idempotency key in the "create invoice"-requests. I started using this, and any retried create would simply return the already created invoice. All was well.. Until about a year later.

## Adding new features
In April 2023 we added a feature to include expenses, such as receipts, in the invoices managed by the software. After release, things went south. The economy system didn't  support fetching expense data for mileage reimbursements. This resulted in a few botched client invoices in _Consulting Group_ which were manually deleted and the software was rerun after some hotfixing, managing to simply skip such expenses. 

Three weeks later, _Consulting group_ reaches out regarding a few invoices which ended up not being sent. Namely, the ones that had been manually deleted. By generating an idempotency key that was stable between runs of the software, we had accidentally instructed the economy system that we wanted the result of the old create invoice request which had been deleted. 

## What happened
When the idempotency key was added, we reasoned that any given invoice in _Consulting group_ should be managed just once. By using the id of the invoice being processed as key, we were sure that any invoice would just be mirrored once. 

Morale of the story, make sure that idempotency keys are re-used only when you definitely want the already existing result, e.g. when performing a retry.




