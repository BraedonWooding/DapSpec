# Goals

> What does Dap try to accomplish

Dap focuses on the following principles:

- Clear design: that is code written in Dap should be clear, extensible, and reworkable.
  - One core tenant is having simpler ways to approach problems avoiding the "9 ways to do X" issue that is persistent in other languages.
    - i.e. having 1 syntax for 1 thing.
- Performant: code written in Dap should be performant and scalable.
  - A smaller point is that code written in Dap shouldn't result in large binaries.

## Clear Design

Dap is a task oriented language, that is you write programs by designing tasks.  This is closest to functional languages in concept, but allows a deeper level of mutability/clearness.

A very simple example, is a todo list application.  This application is shown in the todo list example section in this document.

In some ways Dap acts like a RESTful/GraphQL web application, however it's much more effective at quickly prototyping solutions, it also allows you to abstract complex low level operations such as protocol parsing in a simpler way.
