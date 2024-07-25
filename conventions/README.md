# C# identifier naming rules and conventions
Article [Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names)
Article [Microsoft Learn](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md)
Article [Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/naming-rules)
Article [Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-namespaces)


## General syntax
To define any of the above entities—a naming rule, symbol group, or naming style—set one or more properties using the following syntax:

  ```ini
  <kind>.<entityName>.<propertyName> = <propertyValue>
  ```

## Default naming styles
### Pascal case
Use pascal casing ("PascalCasing") when naming a:
  - namespace
  - class
  - interface,
  - struct,
  - enumerations,
  - methods,
  - properties,
  - events with any accessibility, or
  - delegate type
  - Constant fields

### Camel case
Use camel casing ("camelCasing") when naming private or internal fields and prefix them with _. Use camel casing when naming local variables, including instances of a delegate type.

### Prefix
- I<name> for interface
  For interfaces with any accessibility, the default naming style is Pascal case with a required prefix of I.
- _<name> for private or internal fields
- <name>Async for Asynchronous methods
  Asynchronous methods end with "Async".

## Names of Namespaces
As with other naming guidelines, the goal when naming namespaces is creating sufficient clarity for the programmer using the framework to immediately know what the content of the namespace is likely to be. The following template specifies the general rule for naming namespaces:

`<Company>.(<Product>|<Technology>)[.<Feature>][.<Subnamespace>]`

The following are examples:

`Fabrikam.Math Litware.Security`


