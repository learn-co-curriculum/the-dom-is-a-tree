The DOM Is a Tree
---

## Objectives

1. Explain what a tree is in computer science
2. Describe how the DOM works as a tree
3. Practice exploring the DOM

## A tree? Like, with leaves?

What do we mean when we say that the DOM is a tree? Are we saying it has leaves?

Well, yes, metaphorically. To understand what a tree is, we first have to take a quick look at graphs.

### Graphs

In computer science, a graph is a set of vertices — or "nodes" — and their connections ("edges"). We'll often see them represented like so

``` shell
     A — C
    / \
   B   F — I
  / \ /|\
 D   E G H
```

The above graph represents the set of nodes, `A-I`, and their connections. Notice that `A` is connected to `B` and `C` as well as `F`. The edge between `F` and `A` creates what's called a _cycle_ — this means, essentially, that the graph has a loop in it.

This is perfectly okay! Graphs can have cycles, and we can get lost in them — it just means that the graphs can be a bit tedious to search, as we need to make sure that we don't find ourselves in an infinite loop.

Note too that graphs can be **directed** or **undirected**: in _directed_ graphs, nodes are not only connected but they are also connected in specific ways (the most common is a parent-child relationship, such that (in the graph above), `B` and `C` could be said to be the children of `A`). But the graph above might be undirected, too, where the connections just describe paths and no other relationship is implied. We could, if we treat the above graph as undirected, redraw it as:

``` shell
      E   I
     / \ /
    B   F — H
   / \ / \
  D   A   G
     /
    C
```

This is _the same graph_. One advantage of this representation is that it makes it trivial to notice the cycle introduced by the connection between `A` and `F`.

What if we break that connection?

``` shell
      E   I
     / \ /
    B   F — H
   / \   \
  D   A   G
     /
    C
```

We have now removed all cycles from our graph. And if we make the graph directed — where the connections between each node describe a parent-child relationship — we have a **tree**.

### Trees

As we demonstrated above, a tree is effectively a _directed graph ("digraph") with a root value and without cycles_. These properties make trees remarkably easy to traverse. Every tree has a hierarchy that's easy to reason about, understand, modify, and explore.

For example, let's take the cycle-less, digraph version of the graph above:

``` shell
     A
    / \
   B   C
  / \
 D   E
      \
       F
      /|\
     G H I
```

With `A` as the root, can find our way to the deepest leaves (see!? a metaphor!) of the tree: `A -> B -> E -> F -> [G, H I]`.

In pseudo-code, we explore the tree using the breadth-first search algorithm that we learned about previously.

```shell
root = A
current = root
next = []
visited = []

while (current) {
  visited.push(current)

  if (children = root.children) {
    for (i = 0, l = children.length; i < l; i++) {
      next.push(children[i])
    }
  }

  current = next.shift()
}

// visited is now [A, B, C, D, E, F, G, H, I]
```

Well, why deal with pseudo-code? We've seen a tree before — we just called it a "nested object."

We can represent the tree above as a series of objects:

``` javascript
var tree = {
  A: {
    B: {
      D: {},
      E: {
        F: {
          G: {},
          H: {},
          I: {}
        }
      }
    },
  C: {}
}
```

Represented this way, an empty object means that its parent (its key) has no children. Objects are a great candidate for representing trees in JavaScript because we can store virtually limitless amounts of metadata (data that describes our data) on them in an easily accessible way. We'd want to change our representation above slightly so that we didn't accidentally traverse the metadata when we meant to traverse the tree.

``` javascript
var tree = {
  A: {
    children: {
      B: {
        children: {
          D: { children: {}, metadata: { status: "lonely" } },
          E: {
            children: {
              F: {
                children: {
                  G: { children: {} },
                  H: { children: {} },
                  I: { children: {} }
                },
                metadata: {
                  nickname: "Mr. F"
                }
              },
              metadata: {
                callSign: "Red 5"
              }
            },
            metadata: "so meta"
          },
          metadata: {
            favoriteFood: "tomatoes"
          }
        },
        metadata: {
          favoriteNumber: 2
        }
      },
    C: {}
  }
}
```

It's easy to see how that's overkill for such a spares tree, though. We could, to take a super minimalist route, represent the tree as nested arrays:

``` javascript
var tree = ["A", ["B", ["D", "E", ["F", ["G", "H", "I"]]]], "C"]
```

This is a pretty barebones representation of the tree, but it sure is easy to traverse:

``` javascript
let current = tree
let next = []
let visited = []

while (current) {
  if (Array.isArray(current)) {
    for (let i = 0, l = current.length; i < l; i++) {
      next.push(current[i])
    }
  } else {
    visited.push(current)
  }

  current = next.shift()
}

// `visited` is ["A", "C", "B", "D", "E", "F", "G", "H", "I"]
// "C" comes before "B" because we visit "C" when we visit
// _the array containing "B"_, but before visiting "B" itself
```

The ease with which we can traverse this tree — **knowing nothing about it to start other than its root and that its nodes can have children!** — makes the tree an excellent data structure for the DOM.

## The DOM

You might have noticed an interesting property in the above trees: each tree can contain _subtrees_, which we can, for all intents and purposes, treat independently of their parent trees.

Practically speaking, the DOM begins at `<html>`, but we should think carefully about manipulating what's between the `<head></head>` tags. Instead, we can look at the DOM subtree with its root at `<body>` and only deal with things that will be visible on the page. Within that tree, we might also deal with subtrees. So, for example, if we have

``` html
<body>
  <div>
    <p>Hi!</p>
  </div>

  <div>
    <p>Bye!</p>
  </div>
</body>
```

We would have a tree that looks like

``` shell
        body
        /  \
      div   div
      /      \
     p        p
    /          \
 "Hi!"        "Bye!"
```

Pretty simple, right? Similarly, if we had a DOM subtree that looked like

``` html
<div>
  <div>
    <h1>Hello!</h1>
  </div>

  <div>
    <h5>Sup?</h5>
  </div>
</div>
```

We could simply treat it as the tree

``` shell
      div
     /   \
    h1   h5
   /       \
"Hello!"  "Sup?"
```

### Finding a Node

Remember when we said that we could organize our tree in such a way that a node's metadata didn't interfere with finding its children? It turns out that not only does providing additional information about a node make it more _useful_, it also makes it easier to find.

JavaScript exposes a few ways of finding DOM nodes more or less directly, courtesy of the `document` object.

#### `document.getElementById()`

This method provides the quickest access to a node, but it requires that we know something very specific about it — its `id`. Since IDs must be unique, this method only returns one element. (If you have two elements with the same ID, this method returns the first one — keep your IDs unique!) Given the following DOM tree

``` html
<div>
  <h5 id="greeting">Hello!</h5>
</div>
```

we could find the `h5` element with `document.getElementById('greeting')`. Notice how the `id` that we pass to `getElementById` is identical to the `id` in `<h5 id="greeting>`. We can assign properties to HTML nodes (or elements) simply by including them between the `<>` tags at the start of the element (so not in the `</h5>` tag, in this case).

**Try it out!**

Open up your web inspector (command+option+I on OS X) and find an element on the page — make note of its `id`. Then open up your console, type `document.getElementById({theIdYouTookNoteOf})`, and check out your handy dandy DOM node. Try changing a few of its properties!

#### `document.getElementsByClassName()`

This method, as its name implies, finds elements by their `className`. Unlike `id`, `className` does not need to be unique; as such, this method returns an HTMLCollection (basically an list of DOM nodes — note that it is _not_ an array, even though it has a `length` property) of all the elements with the given class. You can iterate over an HTMLCollection with a simple `for` loop.

Given the following DOM tree

``` html
<!-- the `className` attribute is called `class` in HTML -- it's a bummer -->
<div>
  <div class="banner">
    <h1>Hello!</h1>
  </div>

  <div class="banner">
    <h1>Sup?</h1>
  </div>

  <div class="banner">
    <h5>Tinier heading</h5>
  </div>
</div>
```

we could find all of the elements with `className === 'banner'` by calling `document.getElementsByClassName('banner')`.

**Try it out!**

Inspect the web page again, this time making note of a className. Get all elements with that className and give 'em a look. Remember, you can use a `for` loop to loop through them. (You can also assign the return value of `document.getElementsByClassName()` to a variable: `var elements = document.getElementsByClassName('banner')`.)

#### `document.getElementsByTagName()`

If you don't know an element's ID but you do know its tag name (the tag name is the main thing between the `<>`, e.g., `'div'`, `'span'`, `'h1'`, etc.). Since tag names aren't unique, this method returns an HTMLCollection of 0 to many nodes with the given tag.

**Try it out!**

Explore the DOM in console by typing `document.getElementsByTagName('div')`. Remember, you can iterate through these elements using a simple `for` loop.

### Finding a Node without knowing anything about it

What if we know next to nothing about an element? Or what if we're just interested in finding out more about the child nodes of a given element? This is where our knowledge of trees and nested data structures comes in handy!

Given the following DOM tree (which you can find in `index.html` — feel free to open the file up in your browser to play along!)

``` html
<main>
  <div>
    <div>
      <p>Hello!</p>
    </div>
  </div>
  <div>
    <div>
      <p>Hello!</p>
    </div>
  </div>
  <div>
    <div>
      <p>Hello!</p>
    </div>
  </div>
</main>
```

how would we go about changing only the second "Hello!" to "Goodbye!"? We couldn't just iterate over `document.getElementsByTagName('div')`, checking for `textContent === "Hello!"`, because we'd inadvertently change all three "Hello!"s. More importantly, the DOM might change (more on that later), and we want to make sure that we're updating the right element.

Let's start by getting the `<main>` element:

``` javascript
const main = document.getElementsByTagName('main')[0]
```

Then we can get the children of `main` using `main.children`, so we can get the second child with `main.children[1]`.

``` javascript
const div = main.children[1]
```

Finally, we can get and update our `<p>` element with

``` javascript
// we can call getElementsByTagName() on an _element_
// to constrain the search to its children!
const p = div.getElementsByTagName('p')[0]

p.textContent = "Goodbye!"
```

Obviously, this way of accessing that text isn't fully generic, but it does a good job of demonstrating the basic tools available to us for finding and manipulating HTML elements.

## Resources

- [MDN - Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
