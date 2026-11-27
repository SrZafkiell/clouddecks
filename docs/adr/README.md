# Architecture decision records

A decision that shapes more than one step of Clouddecks or of one of its tools is written
down here, in one short record, before the work that depends on it. The tool
repositories link to this folder instead of keeping records of their own.

## How a record is written

1. Copy [template.md](template.md) to `NNNN-short-title.md` with the next free number.
2. Fill in the context, the options and the decision, and name the plan step that asked
   for it. A new record starts as Proposed.
3. Open a pull request that adds the record and its row in the index below, and updates
   the decisions table of [PLAN.md](../PLAN.md). The record becomes Accepted when the
   maintainer accepts the decision; the pull request then says so.
4. An accepted record is never rewritten. A change of mind is a new record that marks
   the old one "Superseded by NNNN", and the old one links forward to it.

Until the CI gates exist (P0-15), the index is kept by hand in the same pull request as
the record; after that, CI checks that every record has a row and every row a record.

## Index

<!-- adr-index:start -->
| Number | Decision | Status | Step |
|---|---|---|---|
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | Accepted | P0-09 |
<!-- adr-index:end -->
