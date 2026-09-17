# Case Study — תוכי פרינט · בונה הקטלוג (Catalog Builder)

## Summary

A Hebrew-RTL catalog builder for a print shop that carries hundreds of products.
Every season the business needs to send customers an updated catalog. Rebuilding it
by hand is repetitive and error-prone. The catalog builder turns it into a few
minutes of work: the owner ticks which products are in this month, sets a per-unit
and a bulk price, and the system assembles a designed catalog — a festive cover, the
products grouped by category, and a ready-to-fill client order form.

## The problem

- Hundreds of products, and a fresh catalog needed each season.
- Assembling it manually repeats the same work and never guarantees the right price
  landed on the right page.
- When a customer finally chooses, the order comes back as scattered messages that
  have to be collected and totaled by hand.

## Before

Everything lived in separate files — images in one folder, prices in another list,
and a small tool where every change meant reopening, copying and pasting. Each new
catalog started almost from scratch, and any small mistake carried through to the
customer's order.

## The solution

One working screen holding every product with its image, where you pick — with a
simple toggle — what goes into this month's catalog. You enter a per-unit price and a
bulk (25+) price, mark the products you want, and that's it. Everything saves as you
go, so nothing gets lost.

## The automation

The moment a product is ticked, it already sits in the catalog cover and in the order
form, sorted by category with the correct price beside it. The festive cover, contact
details and voucher/CTA badges update on their own, so the catalog always looks whole
and ready to send.

## What the system does on its own

- Gathers every selected product into a designed catalog, sorted by category.
- Builds a festive cover with the logo, tagline and business contact details.
- Picks the correct price by quantity — per unit, or the bulk price from 25 units.
- Opens the customer a comfortable order form where they only choose quantities.
- Totals the order with clear numbers, including VAT when needed.
- Saves continuously while working, with no "save" button to remember.

## The result

What used to be a whole season's repeated work becomes a few minutes. You pick what's
in this month, and customers get a complete catalog with a festive cover, correct
prices and an inviting order form — every time, from scratch, without the grind.

## Scope note

The portfolio deliverable is a faithful, single-file interactive **demo** plus a
designed project page. It reproduces the real product's screens and flows with
example data; production persistence and PDF/catalog export are beyond the demo.
