---
layout: post
title: "Parallel Microservices with Dependencies"
description: "Using simple optimizations to make services run faster"
date: 2018-01-01
tags: [infrastructure, go, microservices, parallel, dependencies, graphs, bfs,
breadth, first, search, concurrency]


comments: true
---

_You can find the link to the actual code [here](https://github.com/blendlabs/go-taskrunner)._

## The problem

Right now, I'm a software engineering intern on the infrastructure team at
[Blend Labs](https://blend.com).
We run a standard microservice architecture over k8s running on AWS
instances. Internal apps make it convenient to run and deploy
microservices, abstracting away a lot of k8s stuff. It's simple. You just
register an image or a git repo and let our tools do the rest.

Internally, we have our own abstractions to help us deal with k8s stuff so
it's easier to manage common tasks that we'd like to do with k8s, Docker,
or etcd. Part of our workflow includes a component called `TaskRunner` that
does what you'd expect -- it runs tasks. There is an abstraction called a
`Module` that has a logically separated set of `tasks`. These tasks are assumed
to be linearly dependent. You have to run them in sequence. A `Module` has
a `Register()` function that adds a set of sequences to a queue for
`TaskRunner` to execute.

We realized that not every `Module` has to be run in a linear order. In fact,
things would probably be a lot faster if we could specify strict dependencies
and try to run things in parallel. One immediate issue was that we didn't
know how long it would take to run each set of steps, or `Module`, which rules
out a lot of optimizations.

## Optimization

You can very easily tell that this is a graph problem. We can abstract each
`Module` as a node. A node can keep track of their dependents and dependencies,
so we effectively have two directional graphs. Every time a `Module` is
registered, it also has to specify its dependencies.

### Acyclic graphs

It's pretty important to make sure that the graph doesn't contain a cycle
so we wrote checks to see if the graph contained a cycle. So how do you
check to see if a directional graph contains a cycle?

A graph is defined as cyclic if you can start a traversal down a node and
end up coming back to that node. So you can just arbitrarily pick nodes
to traverse down, and see if any of these paths lead you back to a node
that you already visited. Since we're talking about dependencies, you
might have thought of [topological graph sorting](https://en.wikipedia.org/wiki/Topological_group).
Topological sorting can actually detect cycles in a graph, but
topological sort itself won't solve our problems since it resolves
dependencies in a linear fashion.

It's actually a fairly easy problem to tackle. You need to have a set of
unvisited nodes, a set of visited nodes, and a set representing the
nodes in the current traversal stack.

Pick an arbitrary node in the set of unvisited nodes. Begin a traversal
down this path (depth-first). As you go down the path, add each node to
the stack, and then after you get through all of the nodes on the stack,
and you add all of those nodes to the set of visited nodes. When you are
traversing down the stack, don't visit already visited nodes, and if you
come across a node in the stack, you know you have a cycle.

If you get through the entire graph without coming across a node in the
stack, the graph is acyclic.

The pseudocode looks something like this.

```python
def check(graph):
    visited, unvisited, stack = set(), set(), set()

    while len(unvisited) > 0:
        root = unvisited.remove(random)

        if helper(graph, root, visited, unvisited, stack):
            return True
    return False


def helper(graph, root, unvisited, visited, stack):
    """ returns true if a particular path has a backedge
    """
    visited.add(root)

    if root in stack:
        return False

    stack.add(root)
    return helper(graph, root.child, unvisited, visited, stack)
```

### Traversing the graph

Once we've verified that there are no cycles in the dependency graph,
we can theoretically run asynchronous breadth-first searches from
every root in the graph.

Note that this dependency graph will need to have links for dependencies
and dependents. A task can be run if it has no dependents or if all of
the dependents have been run/fulfilled. While this can be better optimized,
at the moment, if a module is completed, the thread checks on all of its
dependents and if that dependent has all of its dependencies completed, it can
run. We could spawn off a new thread, but spawning a potentially infinite
number of threads isn't much good. Our solution involved using a fixed
number of worker threads that would take accept nodes off of a queue,
so we could limit the maximum number of threads at any given time.

## Implementation details

Of course, everything is always easier in theory. I had some trouble figuring
out how to set up the concurrency aspect of the code, but it turns out Go
actually has some pretty nice concurrency features.

I ended up following [this](https://gobyexample.com/worker-pools)
tutorial, which made things very easy.

When implementing these algorithms, we had a few things to think about. I was
also new to Go, so I had to get acclimated with Go's concurrency features.

- how do we protect the graph from data races?
- how do we manage threading?
- how do we properly shut down the threads?

Turns out, you can take care of all that with `chan`, go's "channel". It's
essentially a messaging system that lets you easily communicate between
threads. I won't go too in depth into it since the example covers it very
well.
