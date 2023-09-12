---
title: "Codeql"
date: 2023-05-03T10:10:39-07:00
---

# CodeQL
So there's an interesting bit of technology that Microsoft seems to have been developing to
perform static code analysis across all the code in GitHub called 'CodeQL', with a focus
on security vulnerability discovery. (Note, CodeQL was originally created by researchers
and developed by their company Semmle back in 2006; Microsoft acquired them 5 years ago in
2019 https://en.wikipedia.org/wiki/Semmle).
<!--more-->

Unfortunately, the system relies on creating a database, the format of which is not fully
documented and the CLI used to create it is not opensource. Its an odd choice. The
language specific libraries, written in the QL language, are available and browseable, but the thing
executing the langauage is not. This means the path to extend CodeQL for new languages (or raw
binaries) is not clear at all.

Since codeql binary is not open source, I'm not going to be able to extend and include it as an ashnazg
dependency; which is irritating, as it seems like an interesting piece of tech which I may
need to recreate; but eh, so be it if necessary.

At this point, I'm tempted to just read the papers that were published and call it a day rather
than waste more time trying to understand how this works at an operational level. I think understanding
how one would construct the sort of queries for dataflow analysis does seem worthwhile though.

## Graph Query Langauges
So it turns out that MSSQL has a way of declaring tables as NODE and EDGE tables which can
then be used with their MATCH query

https://learn.microsoft.com/en-us/sql/t-sql/queries/match-sql-graph?view=sql-server-ver16

The MATCH query seems to be an implementation of https://en.wikipedia.org/wiki/Graph_Query_Language
which I'm extremely annoyed that it has nothing to do with GraphQL's GQL.

So anyhow, it looks like the industry is standardizing on CQL (Cypher Query Language). This is
not what CodeQL is using, given that it's based off of datalog instead and has some crazy
existential predicates it can use instead. Want to know if nothing calls a function? Well you
can just do this:

```codeql
import java

from Callable callee
where not exists(Callable caller | caller.polyCalls(callee))
select callee
```

Which I'll grant is pretty slick. Unfortunately, I can't figure out if the QL language itself
is under an open licesnse. Blech.

## Monday's problem.
On Monday, I proposed tackling the following problem:

Write code that determines that the passed in values to `printf` are [7,8]
```c
int f(int b) {
    printf("%d\n", b);
}

int main() {
    f(7);
    f(8);
}
```

CodeQL can answer this with

```codeql
import semmle.code.cpp.dataflow.DataFlow

class ValuesOfFunction extends DataFlow::Configuration {
    ValuesOfFunction() { this = "EnvironmentToFileConfiguration" }

  override predicate isSource(DataFlow::Node source) {
    source.asExpr().isConstant()
  }

  override predicate isSink(DataFlow::Node sink) {
    exists (FunctionCall fc, Function printf |
      sink.asExpr() = fc.getArgument(1) and
      printf.hasGlobalName("printf") and
      fc.getTarget() = printf
    )
  }
}

from Expr src, Expr usage, ValuesOfFunction config
where config.hasFlow(DataFlow::exprNode(src), DataFlow::exprNode(usage))
select src
```

Which is pretty neat. That said, one of the key things this brings up is "Wait, what is a *source* exactly?".
Its not really clear when one should stop. Here, I stop when you hit a constant input. However, this doesn't
seem to work with the following

```c
#include <stdio.h>

int f(int b) {
    printf("%d\n", b);
}

int main() {
    int k = 9;
    int j = 11;
    f(7);
    f(8);
    f(k+1);
    f(j);
}
```

The query misses the `9`. This surprised me, I had expected it to hit the 9 and I was going to go on a
digression on how I'd need to basically track the 9 and how it got transformed into a 10 as an actual
output of this program. Looking at the docs for DataFlowConfiguration `hasFlow` reveals:

```codeql
 * Conceptually, this defines a graph where the nodes are `DataFlow::Node`s and
 * the edges are those data-flow steps that preserve the value of the node
 * along with any additional edges defined by `isAdditionalFlowStep`.
```

Ah yeah, that's an issue. `k+1` is most certainly not a value preserving node. I'm not sure how to
use `isAdditionalFlowStep` correctly to extend this. 

Anyhow, this is cool, but at the moment, a dead end. I'm going to have to implement something different
for ashnazg.
