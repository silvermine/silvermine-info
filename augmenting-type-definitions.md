# How to Augment Existing Type Definitions

Use this pattern if you need to add methods or properties to existing TypeScript
classes, interfaces, or objects. This is often required when writing plugins.

## Quick Reference

If the type that you'd like to augment was declared within the `global`
namespace, then use this format:

```typescript
declare global {
   export namespace ExampleLibrary {
      export interface InterfaceYouWantToAddTo {
         methodYouWantToAdd(value: any): void;
      }
   }
}
```

Otherwise, use this format to augment what the main module exports:

```typescript
declare module 'npm-module-name' {
   export interface InterfaceYouWantToAddTo {
      methodYouWantToAdd(value: any): void;
   }
}
```

If you want to augment something that is a property of a property of the
library's main export, e.g. `myLib.interfaces.InterfaceYouWantToAddTo`, then
use a `namespace` nested within the `declare module` statement:

```typescript
declare module 'npm-module-name' {
   export namespace interfaces {
      export interface InterfaceYouWantToAddTo {
         methodYouWantToAdd(value: any): void;
      }
   }
}
```

Continue reading for an in-depth explanation of why these formats are necessary
and when to use them.

## The Problem

In JavaScript, it's very easy to dynamically add properties and functions to
existing objects or classes:

```javascript
let foo = { bar: true };

a.baz = true;
```

However, in TypeScript, the code above causes a type error because the `type` of
the `foo` object is defined as `{ bar: boolean }` and so `baz` is not
recognized.

You can see how this limitation could be a problem when implementing a plugin
for an existing library that has TypeScript types. How do you dynamically assign
properties or methods to an object or class when the interface is already
defined and used by the library internally? Also, how do you produce types so
that users of your plugin can have type definitions for the properties or
methods that your plugin adds to the existing object or class?

Here is a concrete example of this problem:

Imagine that you want to add a plugin for the [Chai test assertion library][1]
that adds a `strictlyEqual` assertion to Chai's `Assertion` interface:

```typescript
const one = 1;

expect(one).to.strictlyEqual(1); // It passes. Go figure.
```

To do so, you follow [Chai's plugin documentation][2] and refer to other plugin
implementations and come up with something like this:

```typescript
export default (chai: any, utils: any) => {
   utils.addMethod(chai.Assertion.prototype, 'strictlyEqual', () => {
      // ... implementation
   });
};
```

Chai's `Assertion` class receives the `strictlyEqual` method. For JavaScript
users, this is all that's needed. However, TypeScript knows knothing of the
`strictlyEqual` method, and trying to use it in your tests will result in a type
error:

```typescript
expect(one).to.strictlyEqual(1); // Property 'strictlyEqual' does not exist on type 'Assertion'
```

How do you add the `strictlyEqual` method to the existing `Assertion`
`interface`?

## The Solution

The solution is [declaration merging][3]. The documentation says:

> ...“declaration merging” means that the compiler merges two separate
> declarations declared with the same name into a single definition. This merged
> definition has the features of both of the original declarations.

In other words, you can augment many existing types by re-declaring a type of
the same name and scope. The TypeScript compiler will automatically merge them
into a single type.

Returning to our example of a Chai plugin, Chai's type definition file declares
a [`namespace` called `Chai`][4]. That `namespace` exports the `Assertion`
interface, which we'd like to augment by adding a `strictlyEqual` method. Chai's
type definition file does not import or directly export anything, so all of its
`namespace` declarations end up in the global scope* (see the footnote at the
end of this section for an explanation of why this is so). Therefore, the type
declaration that we write must also be in the `global` scope:

```typescript
declare global {
   // TODO
}
```

Then, inside of the `declare global` statement, we export a `namespace` called
`Chai`, which [corresponds to the `namespace` in Chai's type definition file][4]:

```typescript
declare global {
   export namespace Chai {
      // TODO
   }
}
```

Lastly, inside of the `Chai` `namespace` we re-define and export the `Assertion`
interface:

```typescript
declare global {
   export namespace Chai {
      export interface Assertion {
         strictlyEqual(value: any, message?: string): void;
      }
   }
}
```

Now, whenever a user of our plugin `import`s the plugin's code, the TypeScript
compiler will detect this `Chai` `namespace` and `Assertion` `interface`
declaration and merge it with any existing `Assertion` `interface` declarations
in the `global` `Chai` `namespace`, and the TypeScript compiler will be aware of
our `strictlyEqual` method.

```typescript
expect(one).to.strictlyEqual(1); // No type error!
```

### Augmenting One of a Module's Exported Types

If the type that you want to augment is *not* in the `global` namespace, then it
must be exported by the library's main module in order for you to augment it.
For example:

```typescript
import { InterfaceYouWantToAddTo } from 'npm-module-name';
```

In this case, instead of using the `global` namespace you would `declare` the
module in order to augment it:

```typescript
declare module 'npm-module-name' {
   export interface InterfaceYouWantToAddTo {
      methodYouWantToAdd(value: any): void;
   }
}
```

Then, the TypeScript compiler will merge the definition of the
`npm-module-name`'s module with `npm-module-name`'s original types.

Note: this means that when you are writing your library in TypeScript, you
*must* `export` any interface that you want to allow users of your library to
augment.

*footnote: According to [the TypeScript Handbook's "Modules" page][5]:

> In TypeScript, just as in ECMAScript 2015, any file containing a top-level
> import or export is considered a module. Conversely, a file without any
> top-level import or export declarations is treated as a script whose contents
> are available in the global scope (and therefore to modules as well).

### "Gotchas"

The ["Declaration Merging"][3] section of the TypeScript documentation has a
more in-depth explanation of the declaration merging behavior. Some of the
behavior may not be obvious. Here, we document some of the "gotchas" you could
run into when augmenting types via declaration merging.

#### Type Declarations Are Merged When a File Is Imported, Not When the File's Code Runs

If you are importing a module that has type declarations that augment another
module's type declarations or that augment a global namespace, the type
declarations are merged because they are *imported*, not because the plugin code
is run.

For example:

```typescript
import enableStrictlyEqual from '@silvermine/chai-strictly-equal';

expect(1).to.strictlyEqual(1);
```

The TypeScript compiler recognizes the `strictlyEqual` method and no type error
is thrown because we imported from '@silvermine/chai-strictly-equal', which
contains the type declaration for `strictlyEqual`. However, running this code
causes a runtime error because we forgot to actually register the plugin!
Instead, we should:

```typescript
import * as chai from 'chai';
import enableStrictlyEqual from '@silvermine/chai-strictly-equal';

// Register the chai-strictly-equal plugin
chai.use(enableStrictlyEqual);

// No type errors AND no runtime errors
expect(1).to.strictlyEqual(1);
```

#### Adding Properties That Already Exist Causes an Error, but Methods Turn into Overloads

If you attempt to augment an `interface` with a property that already exists but
that has a different type, the TypeScript compiler will throw an error (a
property that already exists and is of the *same type* won't throw an error, it
just does nothing).

However, augmenting an `interface` with a method that already exists but with a
different signature makes that method an overload of the original method. See
the ["Merging Interfaces"][7] heading in the "Declaration Merging" documentation
for a detailed explanation of how `interface`s are merged.


[1]: https://www.chaijs.com/
[2]: https://www.chaijs.com/guide/plugins/
[3]: https://www.typescriptlang.org/docs/handbook/declaration-merging.html
[4]: https://github.com/DefinitelyTyped/DefinitelyTyped/blob/d00e7def9a2d6b0a8b6a75aebd260ad78be6e383/types/chai/index.d.ts#L15
[5]: https://www.typescriptlang.org/docs/handbook/modules.html#introduction
[6]: https://github.com/DefinitelyTyped/DefinitelyTyped/blob/d00e7def9a2d6b0a8b6a75aebd260ad78be6e383/types/chai/index.d.ts#L61
[7]: https://www.typescriptlang.org/docs/handbook/declaration-merging.html#merging-interfaces
