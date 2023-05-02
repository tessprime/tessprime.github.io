---
title: "Getting Started"
date: 2023-05-01T10:29:49-07:00
summary: "Setting up hugo and Monday"
---

# Setting up Hugo
Alright, so I opted to go for using Hugo as the
static content generator.

Ultimately, I ended up just building it from source
with 

```
go install -tags extended github.com/gohugoio/hugo@latest
```

which, (after removing `asdf` shims from my toolpath) just
worked. Don't use the `apt-get` version, it'll be old and
incompatible with random themes you come across online.

Tricky bit is setting up the Github workflow pipeline.

Docs are here for that:

https://gohugo.io/hosting-and-deployment/hosting-on-github/

And wow, the workflow UI feels way better than gitlab.

Issues run into:
  - baseUrl needs to be https or its going to look gnarly
    when the page tries to fetch http resources. Oops!
    However, I discovered that the default gitlab workflow
    was overriding this with
    `--baseURL "${{ steps.pages.outputs.base_url }}/"`.
    In order to get the correct behavior I just removed this,
    
  - Github seems geared to use jekyll by default,
    but the above "Hosting on Github" page explains
    how to tweak that.
  - Old version of go didn't work with desired Hugo Theme.

# Plan for today.

For this week, I'm not going to have a week plan I think
(subject to change, otherwise this would be a plan ;) ),
just setup daily goals.

Goals Today:
  - [x] Get Hugo up and running on github.
  - [ ] Finish reading 'Chip Wars' and publish notes.
  - [ ] Get OpenFOAM up and running and figure out how
    to have moving boundary conditions.
  - [x] Groceries.
  - [ ] Go running.
