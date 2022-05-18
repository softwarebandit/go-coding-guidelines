# go-coding-guidelines

The following is a list of coding guidelines and recommandations for go (golang) development. They have been compiled from multiple sources. See credits at the end of the document.

# 1. Code Organization

1. Packages should contain code that has a single purpose: _cmd, archive, crypto_. Avoid catchall packages like _util, helpers, common_. Catchall packages often cause problems with testing and are a common cause of circular dependencies

2. Different implementations for a common set of functionality should be grouped under a single parent. See the contents of the package _encoding_

3. Package names should be short but descriptive. Take advantage of the parent package name and don't repeat in children. _encoding/base64_. Prefer "transport" over "transportmechanism"

4. When building an application, the executable should be separate from the other packages

5. Application packages should center around _Domain Types_ and _Services_

6. The package containing domain types should also defined the interfaces that define what you can do with the domain types:
_ProductService, AuthenticationService, UserStorage_

7. The domain types package should be the root of the application. This makes it clear to everyone what the application does. It should not contain any external dependecies.

8. The implementation of the interfaces should be in separate packages organized by dependency

9. There should be a single package per dependency: data source, transport protocol, etc. This makes it easier to test/mock, substitute and prevents circular dependencies

10. Inside a package separate code into logical concerns. If the package deals with multiple types, keep the lgoic for each type in its own source file:

```
package: postgres

orders.go
products.go
customers.go
```

11. In the package that defines your domain objects, define the types and interfaces for each object in the same source file:

```
package: inventory

orders.go
-- contains Orders type
   and OrdersStorage interface
```

# 2. Naming Conventions

1. When naming variables use camelCase and not snake_case

2. Use single letter variables for indices

```
for i:=0; i < 100; i++ {}
```

3. Use short but descriptive variable names for everythign else

```
var count int
var cust Customer
```

4. Take scope of the variable into consideration. The farther away from declaration it is used, the longer the name should be

5. Use repeated letters to represent a collection/slice/array outside of a loop/range and a single letter inside:

```
var tt []*Thing
for i, t:= range tt {}
```

These conventions are common inside of Go's own source code

6. Avoid package-level functions that repeat the package name:

```
GOOD: log.Info()
BAD:  log.LogInfo()
```

7. Go doesn't have setters and getters:

```
GOOD: custSvc.Customer()
BAD:  custSvc.GetCustomer()
```

8. If the interface has only a single function, append "-er" to the function name:

```
type Stringer interface{
  String() string
}
```

9. If the interface has more than one function, use a name to represent its functionality:

```
type CustomerStorage interface {
  Customer(id int) (*Customer, error)
  Save(c *Customer) error
  Delete(id int) error
}
```

# 3. Other Conventions

1. Make comments in full sentences always:

```
// An order represents an order from a customer
type Order struct {}
```

2. Use `goimports` to manage import and they will always appear in canonical order. Standard libs first, external next.

3. Avoid `else` clause. Especially in error handling

```
if err != nil {
  //error handling
  return // or continue, etc.
}
// normal code
```

4. Avoid broad interfaces. Functions should only accept interfaces that require the methods they need. Functions should not accept broad interfaces when a narror one would work. 

6. Compose broad interfaces made from narrow ones:

```
type File interface {
  io.Closer
  io.Reader
  io.ReaderAt
  io.Seeker
  io.Writer
  io.WriterAt
}
```

7. Do not overuse methods. Methods should only be use if they modify the object/struct state that they belong to. They define the behavior of a type. They use state and are logically connected. They are bound to a specific type.

8. Functions should be used when they don't depend on state: Same inputs always returning same outputs. They can be used with interfaces.

9. When deciding between values and pointers consider if shared access is needed (not performance). Pointers should be used when the value is to be shared with a method or function. Value (copy) should be used when we don't want to share it.

10. When sharing a value with it's method, use a pointer receiver. This is a common usage when managing state. But it is not safe for concurrent access.

11. Use values when not sharing or using with an empty struct that is stateless with only behavior. Values are safe for concurrent access.

12. Don't forget to treat errors as interfaces and not just strings. You can create custom errors to provide additional context.

13. Data structures are not safe for concurrent access. Values aren't safe. You need to create safe behavior around them.
  - _Sync_ package provides behavior to make a value safe (Atomic/Mutex)
  - _Channels_ coordinate values across go routines by permitting one go routine to access at a time
  - Safety comes at a cost. It imposes behavior on the consumer. Safety is often unnecessary.
  - Proper API allows consumers to add safety as needed (using channels and mutexes)
  - Maps are unsafe by design. They enable consumers to implement safety as needed.

14. Review Go StdLib for examples of good Go conventions.

# Credits
- Brian Ketelsen @bketelsen
- Ben Johnson (https://medium.com/@benbjohnson/standard-package-layout-7cdbc8391fc1)
- Steve Francia @spf13
