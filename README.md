# Stays — booking.com, built on forinda-cms

A working accommodation marketplace whose entire definition is the YAML in this
directory. It exists to answer one question honestly: **can this platform build
the thing it says it is for?**

Nothing here is a mock-up. Twelve properties across five cities, twenty-five
rooms, forty-two reviews, a search page with counted filters, availability
computed from real bookings, and a two-step booking journey that ends at a
payment page.

```sh
npm install
npm start                  # localhost:4711 — Postgres runs in the process
npx fcms apply --yes --content
```

Nothing else to install. No database, no Docker, no connection string.

## What it demonstrates

| | |
|---|---|
| **Counted filters** | `stars`, `kind`, `city`, cancellation, meals — each option carries how many properties it would leave, counted with its own filter lifted |
| **Numbers nobody types** | a property's guest score, review count and "from" price are aggregates over its reviews and rooms |
| **Availability** | `vacancy` is a *derived* type: the rooms free for every night between two dates, computed from bookings, never stored |
| **A price the guest cannot choose** | the deposit is a computed field — the room's own rate times the nights — and the schema refuses it as input |
| **A journey** | pick dates and a room, then your details; the room chosen is the room charged for |
| **Their own bookings** | a guest sees theirs and nobody else's |

## The files

```
site.yaml            name, theme, currency, header and footer, payment wiring
content/*.yaml       city, amenity, property, room, review, booking, vacancy
pages/*.yaml         home, search, property detail, book, trips
data/*.yaml          the rows — 96 of them, in version control
```

Every change went through `fcms plan` and `fcms apply`, and every one of them is
undoable from the admin's history screen.

## What building it found

Sixteen bugs and gaps in the platform, none of which would have surfaced from
inside its own repository. The ones worth knowing about:

- Validation errors never reached a self-hosted caller — the framework strips an
  exception's details in production, so a refused write said only "That entry is
  not valid" and named no field.
- `pull --content` followed by `apply --content` unpublished the whole site: one
  side wrote "draft" only for drafts, the other read absent as draft.
- Aggregates never matched a reference written the documented way, so every
  card's score, review count and price were empty.
- A `reference` field declared `many` could not hold many.
- Two processes on one embedded database corrupt it permanently — found by
  restarting this site too quickly, which destroyed its database.
- A booking journey could not filter and then choose: the filter's form ended up
  inside the choosing form.

Each one is fixed upstream, with a test that fails against the commit before it.

## Still missing

- **Photos.** Media works; these properties have none, so the cards are text.
- **A first step that only collects dates.** A flow step is answered by choosing
  a row, so "when are you coming?" cannot be a step of its own — the dates live
  in the same step as the rooms they filter.
- **Multi-property search by map**, currency per property, cancellation policies
  as data.
