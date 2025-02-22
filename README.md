# Factual Language VS Code Extension

This Visual Studio Code extension provides syntax highlighting and language support for the Factual language used in Jinaga. The Factual language allows you to declare fact models, scenarios, and specifications.

## Features

- Syntax highlighting for Factual language files (`.model.fact`, `.scenario.fact`, and `.specification.fact`).
- Support for comments, brackets, auto-closing pairs, and surrounding pairs.
- Example Factual language snippets for quick reference.

## Installation

To install the extension, copy it into the `<user home>/.vscode/extensions` folder and restart Visual Studio Code.

## Usage

1. Open a new window with your extension loaded by pressing `F5`.
2. Create a new file with the `.model.fact` extension.
3. Write your Factual language code and enjoy the syntax highlighting and language support.

## Examples

### Model

Here is an example set of Factual model:

```factual
fact Blog.Site {
    domain: string
}

fact Blog.Post {
    createdAt: datetime
    site: Blog.Site
}

fact Blog.Post.Title {
    post: Blog.Post
    value: string
    prior: Blog.Post.Title*
}
```

### Scenario

Here is an example set of Factual scenario:

```factual
let site: Blog.Site = {
    domain: "qedcode.com"
}

let post: Blog.Post = {
    createdAt: "2022-08-16T15:23:13.231Z",
    site
}

let title: Blog.Post.Title = {
    post,
    value: "Introducing Jinaga Replicator",
    prior: []
}
let title2: Blog.Post.Title = {
    post,
    value: "Introduction to the Jinaga Replicator",
    prior: [ title ]
}
```

#### Facts

All of the facts written in the request body will be inserted into the Replicator's database. Give each fact a variable name, a type, and a set of fields.

```factual
let site: Blog.Site = {
    domain: "qedcode.com"
}
```

#### Predecessors

You can use the variable to declare a predecessor of another fact.

```factual
let post: Blog.Post = {
    createdAt: "2022-08-16T15:23:13.231Z",
    site: site
}
```

If the predecessor has the same name as the variable, you can simplify it.

```factual
let post: Blog.Post = {
    createdAt: "2022-08-16T15:23:13.231Z",
    site
}
```

#### Arrays

To define an array of predecessors, use square brackets. You can supply empty square brackets for an empty array.

```factual
let title: Blog.Post.Title = {
    post,
    value: "Introducing Jinaga Replicator",
    prior: []
}
```

Or you can list any number of previously defined facts.

```factual
let title2: Blog.Post.Title = {
    post,
    value: "Introduction to the Jinaga Replicator",
    prior: [ title ]
}
```

### Specification

The Factual specification language lets you declare a set of given facts, a specification body, and projections. Here's an example specification:

```factual
(site: Blog.Site) {
    post: Blog.Post [
        post->site: Blog.Site = site
        !E {
            deleted: Blog.Post.Deleted [
                deleted->post: Blog.Post = post
            ]
        }
    ]
} => {
    id = #post
    createdAt = post.createdAt
    titles = {
        title: Blog.Post.Title [
            title->post: Blog.Post = post
            !E {
                next: Blog.Post.Title [
                    next->prior: Blog.Post.Title = title
                ]
            }
        ]
    } => title.value
}
```

#### Givens

The givens are the starting point for the specification.

```specification
(site: Blog.Site) {
    <specification body>
}
```

#### Specification Body

Once you have a starting point, you will want to look for related facts.
The specification body matches facts based on their relationship to the givens and to each other.

The body of a specification defines a list of unknowns.
Declare an unknown by name and type.
Then in square brackets, list conditions that relate the unknown to the givens.

```specification
post: Blog.Post [
    <conditions>
]
```

One kind of condition is a path.
Paths join the declared unknown to givens or previously declared unknowns.

```specification
post->site: Blog.Site = site
```

The arrow indicates that we are following an edge up to a predecessor.
The name and type of that predecessor appear after the arrow.
You can use as many arrows as you need on both the left and right side.

Another type of condition is an existential.
It says whether it's looking for something to exist (`E`) or not exist (`!E`).
And then it declares another tuple.

```specification
!E {
    deleted: Blog.Post.Deleted [
        deleted->post: Blog.Post = post
    ]
}
```

A positive existential condition will be true if any tuple of facts match the unknowns.
A negative existential condition will match if none do.

#### Projection

So far you have some facts that are used as givens, and a list of unknowns with conditions.
If you run this specification, you will see all of the tuples of facts that match these unknowns.

```json
[
  {
    "post": {
      "createdAt": "2023-04-28T23:53:00Z"
    }
  }
]
```

But you might want to reshape your results.

Projections turn tuples into meaningful results.
They can select fields or hashes of facts in the tuple.

```specification
=> {
    id = #post
    createdAt = post.createdAt
}
```

This should now return the following:

```json
[
  {
    "id": "SV3Yseb0FJ/uQoL4IelHHbo1g4PZjnP4bx+aivMiiCkSTn2vSaNR6314KXL+PgO9lX9jmJzWoZABTRHjiFHWdQ==",
    "createdAt": "2023-04-28T23:53:00Z"
  }
]
```

#### Child Specifications

A projection can also run child specifications.
A child specification has its own list of unknowns.
It can even have its own projection.

```specification
titles = {
    title: Blog.Post.Title [
        title->post: Blog.Post = post
        !E {
            next: Blog.Post.Title [
                next->prior: Blog.Post.Title = title
            ]
        }
    ]
} => title.value
```

Run it now and you will see these results:

```json
[
  {
    "id": "SV3Yseb0FJ/uQoL4IelHHbo1g4PZjnP4bx+aivMiiCkSTn2vSaNR6314KXL+PgO9lX9jmJzWoZABTRHjiFHWdQ==",
    "createdAt": "2023-04-28T23:53:00Z",
    "titles": [
      "Introduction to the Jinaga Replicator"
    ]
  }
]
```

This lets you build up a JSON structure containing just what you want.

## Requirements

- Visual Studio Code version 1.96.0 or higher.

## Known Issues

- No known issues at this time.

## Release Notes

### 0.1.0

- Initial release of the Factual language Visual Studio Code extension.

## For more information

- [Jinaga Factual Language Documentation](https://jinaga.com/documents/replicator/write/)
- [Visual Studio Code's Markdown Support](http://code.visualstudio.com/docs/languages/markdown)
- [Markdown Syntax Reference](https://help.github.com/articles/markdown-basics/)

**Enjoy!**
