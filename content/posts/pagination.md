---
title: "Pagination"
date: 2023-05-11T09:16:48-07:00
---

So fun thing about pagination, its a pain.

When querying a large dataset, many apis will provide a "page" parameter
(and also limit the maximum results that can be returned).

A serious problem though with APIs that do this is that they can result
in an inconsistent view of the data. The worst thing that can happen is
if something that does exist at the initial query request does not get
returned after iterating across all objects. This can happen if rows
get added or removed between requests.

This issue is extremely common since the dead simplest solution to solve
the problem "How can I divvy up the results of this SQL query?" is to use
`LIMIT X OFFSET Y`.  Unfortunately, in addition to performance issues
(which I haven't investigated thoroughly, but seems plausible), this has
the issue mentioned above on this not necessarily returning entries when
rows get added or deleted. Also, internal consistency (but a number of
popular databases go with "meh; eventually consistent. Deal with it".).

The generally recommended solution if you don't care about sort order is
to use an additional column that is unique and immutable and do `where
COLUMN > PREVIOUS_KEY and LIMIT PAGE_SIZE`. This ensures that every row
that exists at the initial query request time, and continues to exist,
will eventually get enumerated.

If you do care about sort order (or about the internal consistency of
the data), the best solution I came across was to use temporal tables
(these have other advantages) to ensure that you're iterating over a
self consistent view of the database.

From an API perspective, you should probably return a continuation token
as part of the initial request.

However, as was pointed out yesterday over coffee, there's another use
case for pagination that makes this more complicated, namely, UI that
is explicitly using pages to represent part of the data. This has the
same issue that data can screw up your ordering and make you miss an entry.
Typically, these UI pretend that they are an up to date representation
of the data, and clicking between pages, or updating
the values in a row suggests that the data should be a current representation
of what's in the database.

I don't have a good solution to the UI problem. It's probably underspecified,
which means that if you *do* have paginated data in your UI, you need to think
*very* carefully about what the correct behavior should be.

