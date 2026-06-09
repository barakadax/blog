# C.R.U.D
- [What it is](#what-it-is)
- [Why do we need it](#why-do-we-need-it)
- [Where to use](#where-to-use)
- [Sources](#sources)

## What it is

CRUD stands for **Create, Read, Update, and Delete**. These are the four basic operations of persistent storage.
They represent the lifecycle of data in almost every software application:

- **Create**: Add new records or data entries (e.g., publishing a blog post).
- **Read**: Retrieve and view existing data (e.g., reading a blog post).
- **Update**: Modify or edit existing data (e.g., editing a blog post's content).
- **Delete**: Remove data from the system (e.g., deleting a blog post).

## Why do we need it

CRUD operations are the building blocks of data management.
Without them, applications would be static and unable to manage user-driven state.
They:
- Provide a standardized framework for interacting with databases.
- Map directly to standard user interface actions (forms, buttons, and views).
- Align with modern API designs, such as REST, which uses HTTP methods (POST, GET, PUT/PATCH, DELETE) to manage resource states.

## Where to use

CRUD is implemented across various layers of software development:
- **Relational Databases (SQL)**: Executed via queries (`INSERT`, `SELECT`, `UPDATE`, `DELETE`).
- **RESTful APIs**: Exposing endpoints that handle HTTP requests mapping to CRUD actions.
- **User Interfaces (UI)**: Dashboards, admin panels, and data-entry forms where users manage data records.
- **NoSQL Databases**: Interacted with via document or key-value APIs to store and retrieve data.

## Sources

- [C.R.U.D](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)
- [MDN Web Docs: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
