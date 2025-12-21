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
    Context o--> Strategy
    Context <-- Client
    Client ..> ConcreteStrategies
    Strategy <|.. ConcreteStrategy1
    Strategy <|.. ConcreteStrategy2
    class Context {
        - strategy
        + setStrategy(strategy)
        + doSomething()
    }
    class Strategy {
        <<interface>>
        + execute(data)
    }
    package "ConcreteStrategies" {
        class ConcreteStrategy1 {
            + execute(data)
        }
        class ConcreteStrategy2 {
            + execute(data)
        }
    }
```
