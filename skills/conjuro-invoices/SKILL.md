---
name: conjuro-invoices
description: List Conjuro invoices, draft new ones with their lines, and finalize them so they are sent to the customer. Use when the user asks what is owed or paid, wants to bill a customer, or wants to send an invoice.
---

# Conjuro invoices

Use the Conjuro MCP server (`plugin:conjuro:conjuro`) to work with invoices in the user's current Conjuro workspace.

## Permissions

| Action | Tools | Permission |
|---|---|---|
| Find invoices | `conjuro_list_invoices` | **Read invoices** |
| Draft an invoice | `conjuro_create_invoice` | **Create draft invoices** (off by default) |
| Finalize and send | `conjuro_finalize_invoice` | **Finalize invoices and email them to customers (uses AMPS)** (off by default) |

If a tool returns `[permission_missing:<scope>]`, tell the user to turn that permission on in Conjuro under **Account → Agent connections → Manage permissions**, then try again. Don't look for a workaround.

## Find invoices

Call `conjuro_list_invoices`. Filter with `status` (`draft`, `finalized`, `paid`, `overdue`, `archived`), `paid` (`true` / `false`), `customerId`, `projectId`, `issuedAfter` / `issuedBefore`, `dueBefore` (`YYYY-MM-DD`), or `query`. "What's unpaid for Acme?" is `paid: false` with Acme's `customerId` (from `conjuro_search_customers`). Summarise number, customer, total, due date, and status; give totals per currency, never mixed.

## Draft an invoice

1. Get the customer (`conjuro_search_customers`), optionally the project (`conjuro_list_projects`), issue and due dates, currency, and each line: description, quantity, unit price, and tax rate in percent.
2. Show the user the lines and the expected total and wait for a clear yes.
3. Call `conjuro_create_invoice` with all the lines and a fresh `idempotencyKey`. Conjuro numbers the draft and calculates the totals; nothing is sent.
4. Report the invoice number and the total Conjuro calculated. If the result says only some lines were saved, tell the user to finish the draft in Conjuro. Don't create another invoice.

## Finalize and send

Finalizing locks the invoice, finalizes it in Stripe, **emails it to the customer to pay**, and uses 25 AMP cents. It cannot be undone.

1. Only finalize when the user asks to send or finalize a specific invoice.
2. Show the user the invoice number, customer, and grand total, and get an explicit yes for that invoice.
3. Call `conjuro_finalize_invoice` with `confirmTotal` set to that grand total and a fresh `idempotencyKey`. The tool refuses if the total has changed, the invoice has no lines, or the customer has no email address.
4. Report the result: finalized (share the payment link), Conjuro's reason for refusing (for example too few AMPS), or still in progress. If it's still in progress, don't finalize again; check with `conjuro_list_invoices` later.
5. Never finalize several invoices from one confirmation. Confirm each one.
