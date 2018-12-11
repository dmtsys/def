### Tryable:

Comment: somewhat obsolete because of new `?.` abilities in javascript.

Accessing elements can be done without danger of breakage because something in access chain returned `null`.

Root element after parsing any .def file is always wrapped in *Tryable* before returning.

Example:

Instead of writing:

```javascript
user.server.ip
```

which is usual if you are sure that each user has a server, you do:

```javascript
user.try('server.ip')
```

now the result will be `null` if `server` is `null` or `server` is not `null` but is missing the `ip` definition.

Root element is returned wrapped in `Tryable` already, but you can always make all nodes in def tree tryable, like so:

```javascript
def.tryable(user).try('server.ip')
```

There are however other benefits to `try` so it may stay as part of spec, will update on this soon.
