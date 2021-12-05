---
title: Samples
main_menu: true
weight: 60
description: >
  Samples of cool stuff we can do with Hugo and Docsy.
---

## PlantUML support

We can include inline plant UML markdown

```plantuml
Alice -> Bob: Authentication Request
Bob -> Alice: Authentication Failure
group My own label [My own label 2]
    Alice -> Log : Log attack start
    loop 1000 times
        Alice -> Bob: DNS Attack
    end
    Alice -> Log : Log attack end
end
```

And even specify the theme we wish to render to:

```plantuml
!theme plain
Alice -> Bob: Authentication Request
Bob -> Alice: Authentication Failure
group My own label [My own label 2]
    Alice -> Log : Log attack start
    loop 1000 times
        Alice -> Bob: DNS Attack
    end
    Alice -> Log : Log attack end
end
```
