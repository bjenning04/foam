# Mermaid Test
```mermaid
%%{
  init: {
    "class": {
      "hideEmptyMembersBox": true
    }
  }
}%%
classDiagram
    class Context {
        - strategy
        + setStrategy(strategy)
        + doSomething()
    }
    class Strategy {
        <<interface>>
        + execute(data)
    }
    class ConcreteStrategy {
        + execute(data)
    }
```
