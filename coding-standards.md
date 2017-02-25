# Silvermine Coding Standards

This document outlines the coding standards for the Silvermine organization.
Many of these principles and rules are based on accepted industry standards and
*Code Complete 2* (CC2). Others are based on our own decades of experience and
deep thought. Still others are based on an arbitrary and capricious decision
but have behind them the force of history and thousands of lines of code. In
any event, if you are contributing to our codebase, it is your responsibility
to follow these standards. They will be enforced in code reviews.


## Abbreviations

Abbreviations can be used where it is reasonable and length of a variable name
or similar entity is an issue. However, keep in mind the following principles:

   * Readability and clarity are both more important than horizontal space.
     Never use an abbreviation that sacrifices readability or clarity in your code.
   * If you choose to use an abbreviation, use it consistently. Do not use two
     abbreviations for the same word, and do not abbreviate a term in one place and
     not in another.
   * Similarly, if there is already a standard way of abbreviating a word in
     our codebase, use that way consistently throughout all codebases we maintain.
   * If you do need to create an abbreviation, these guidelines may prove helpful:
      * Create names that can be pronounced (if you cannot read your code out
        loud, use a different abbreviation).
      * Do not abbreviate by removing one character from a word. The
        one-character savings hardly justifies the loss in readability.
      * Remove all non-leading vowels.
      * Use the first few letters of each word.
      * Avoid combinations that could be misread or misunderstood.


## Acronyms

   * Occasionally an acronym may be used in a function or variable name.
     Examples include LDAP,  IMAP, or API. Whenever this is the case, the acronym
     should be written in all capital letters.

## Identification

Identify yourself by a public email address (such as gmail, etc) in all commit
messages.


## Classes

### Naming

Class names should be nouns, using UpperCamelCase (see [the
glossary](#glossary)). Try to keep your class names simple and descriptive.
Many classes represent the data stored in a single record of a table. In that
case the class should be named the same as the table, although depending on the
system you are working on, the table may be the plural form of the noun whereas
class names are always the singular form of the noun.


## Formatting

When formatting code, your key objectives are:

   * Accurately represent the logical structure of the code
   * Consistently represent the logical structure of the code
   * Improve readability
   * Withstand modifications

The rules below provide more details. However, if you are in doubt, copy the
formatting of the code you are working within.

If you encounter a file that has formatting errors in it, be cautious about
fixing these errors in the same commit as functional changes. If there will be
a large number of formatting changes made that do not impact functionality,
these should be made as a separate merge request from functional changes.
Otherwise the noise-to-signal ratio for the functional merge request is
distorted.


### Blank Lines

According to CC2, "in writing, thoughts are grouped into paragraphs...
Similarly, a paragraph of code should contain statements that accomplish a
single task and that are related to each other."

In accord with this:

   * Related statements that accomplish a single task should be grouped
     together without blank lines.
   * Use one blank line to separate unrelated statements.
   * Use one or two blank lines to separate functions and sections of your
     source file.

Again, the key is readability and clarity.


### Braces

   * Curly braces appear at the end of the line instead of on a new line. It
     saves considerable vertical space.
   * Never leave curly braces off of conditionals or loop control statements,
     even if the condition or statement is a single line. Reasons:
      * We are already saving enough whitespace with our EOL curly brace style,
        so you are really only saving the one line from the closing curly.
      * If you were to leave out curly braces and someone makes the condition a
        two line condition, or add a little debugging line, they now have to also add
        the braces or suffer a silent and subtle failure.

You can note other advantages to both of these rules [in K&R Variant
1TBS](https://en.wikipedia.org/wiki/Indent_style#Variant:_1TBS_.28OTBS.29)


### Indentation

*Yes, we know our indentation rules are different than any codebase you have
ever worked on. We have worked this way for many years and like it. Sorry if
you do not like it.*

   * Indent distance is three spaces.
   * Never use actual tab characters. Please configure your editor such that
     when you press the *tab* key, it inserts spaces instead of tab characters. Some
     prefer to turn on display of white space characters so you can easily identify
     when tab characters have been introduced.
   * If a statement wraps to more than one line, subsequent lines should be
     indented a single indentation from the first line. An exception is when the
     line wrap comes inside of a string. In those cases, such as SQL statements and
     HTML, you may indent each line as dictated by the code you are writing.
   * If you are declaring variables in an opening `var` block (as in
     JavaScript), do not allow any declaration to wrap more than one line. If you
     need to declare a variable that will take several lines to initialize, declare
     the variable as one statement and initialize it as a separate statement. This
     makes the code much easier to read and doesn't lead to weird indentation due to
     `var ` being four characters. See example with `someVariable` below.

Example:

```js
// in SomeFile.js
var Class = require('class.extend'),
    _ = require('underscore'),
    SomeClass;

SomeClass = Class.extend({

   myFunction: function doSomething(someVeryLongVariableNameThatShouldNeverActuallyBeUsedInOurCodebase) {
      var someVariable;

      someVariable = _.isEmpty(someVeryLongVariableNameThatShouldNeverActuallyBeUsedInOurCodebase) ?
         theTruePartOfMyTernaryStatementIsIndentedOneLevelBecauseTheLineWrapped() :
         theFalsePartIsIndentedTheSame;
   }

})
```

   * Where you have a chained set of function calls that will span two lines,
     the series of chained functions should be indented one level from whatever
     starts the chain.
      * As described elsewhere, avoid nesting too many levels of chained calls or functions.

Examples:

```js
// in SomeFile.js
function doSomethingEventually(bar) {
   return Q(bar)
      .then(function(result) {
         return database.findBySomething(_.pluck(result, 'column'));
      })
      .then(function(rows) {
         return convertToGroup(rows);
      });
}

// an unrelated example, this time with underscore - another common place to chain
function anotherExample(people) {
   return _.chain(people)
      .sortBy(function(person) {
         return person.ageInMilliseconds;
      })
      .filter(function(person) {
         return person.ageInYears < 65;
      })
      .groupBy('lastName')
      .value();
}

// this is fine to do as long as it does not wrap to two lines:
function anotherExample(people) {
   return _.chain(people).sortBy('ageInMilliseconds').groupBy('lastName').value();
}
// if it needed to wrap to two lines, you should break each piece of the chain
// to a new line
```


### Parentheses

   * Use parentheses to clarify the order of operations, even when they may not
     strictly be needed (see example).
   * A space is inserted between control structures (like for, foreach, if) and
     their parentheses. However, spaces are not put between function names and their
     opening parenthesis (see example).
   * Spaces should not be used between nested parentheses.

Example:

```php
// SomeFile.php
class SomeClass {

   function someFunction($articles) {
      // note the space after foreach and the curly brace on the previous line
      // but that there is not a space between "someFunction" and the parenthesis
      foreach ($articles as $article) {
         // Note the following where the parentheses are not strictly needed, but helpful for grouping
         $x = (empty($abc) ? 'truevalue' : 'falsevalue');
         $z = ((empty($a) || empty($b)) ? 'truevalue' : 'falsevalue');
      }
   }

}
```

### Miscellaneous

   * All files should end with a newline as this is the *nix standard (a line
     is defined as ending with a newline) and some tools will not operate properly
     otherwise.
      * **Please note**: this is a *newline character* (see
        [Wikipedia](https://en.wikipedia.org/wiki/Newline)), *not a blank line*. You
        should **not** see a blank line at the bottom of your files.
      * But, but, but, why do I need newlines at the end of my files? See [this
        answer](http://stackoverflow.com/a/729795/1011988) on StackExchange. Then try
        this:
         * `echo -n 'this is a test' > /tmp/test-nonewline.txt && echo 'this is a test' > /tmp/test-newline.txt && wc -l /tmp/test-*newline.txt`
         * Notice that the line that you echoed to the "nonewline" file doesn't
           get counted. All sorts of *nix tools start breaking if you don't terminate each
           line with a newline character.
   * Never commit Windows line endings into any of our codebases.
   * Separate operators from their operands with a single space. Exceptions are
     unary operators `!`, `++`, `--`
   * Use the ternary operator `x = condition ? true : false` only when your
     operands are simple and the statement can easily be read.
   * Separate array references and arguments by a space following the comma.
   * Empty array and object declarations should not have a space in them
     (e.g. `[]` or `{}`)
   * Multi-line array and object declarations should:
      * *always* have a dangling comma for node.js codebases
      * *never* have a dangling comma for browser-targetted JS codebases

Examples:

```php
// SomeFile.php
$x = array( 'a' => 1, 'b' => 2, 'c' => 3 );
```

```js
var foo = [ 1, 2, 3 ],
    bar = [],
    baz = {},
    bob = { yes: 'bob' };
```


## Functions

### Naming

   * **Static** functions are named all lower case with underscores separating
     words in the name.
   * **Instance** functions are named using camelCase (see [the
     glossary](#glossary)).
   * Describe everything the function does by its name. A very long or compound
     name likely indicates that you need to break the function up into smaller ones.
      * If the function returns a value, it is often good to name the function
        as a description of its return value.
      * If the function returns a single column value from a table, name it
        "get" followed by the column name.
      * If the function returns a single record primarily from a single table,
        name it "get" followed by the table name.
      * If the function returns multiple rows primarily from a single table,
        name it "list" followed by the table name.
      * If the function does not return a value, use a strong verb followed by
        a noun.
      * If a function inserts, updates, or deletes row(s) to a table, name it
        with the appropriate verb followed by the table name.
   * Give boolean functions names that naturally imply true or false, generally
     by starting them with the word "is" (e.g. `isFoo`).
   * Examples of good function names:
      * deletePage
      * getDocumentClassification
      * getPage
      * isPublished
      * isAllowedToBePublished
      * sendNotification


### Header

Each function must have a header. It should be formatted similar to the following code snippet:

```
  /**
   * Copy fields from this object to the object that you pass in, returning the names
   * of the fields that were edited on the object you passed in.
   *
   * @param Page $target - the page to copy fields to
   * @param boolean $persist (optional, default false) - if true, persists changes to the target object
   * @return array - names of the fields edited on the target, or an empty array if none
   */
```

   * You must begin the header with `/**` since this causes it to be picked up
     by the documentation system.
   * Enter a description of the purpose of the function on the line after the
     opening comment symbol.
   * Add a @param entry for each function parameter.
      * Include the type of the expected parameter.
      * Include a description of the purpose of the parameter.
      * If the parameter is optional, specify that and what the default is.
      * Document assumptions about parameters. Examples would be expected data
        type, numeric units, range of values, invalid values, and the meaning of status
        code and error values.
   * Add a @return entry for all functions even if the function does not return
     a value. If it returns a value, include the data type and a description of the
     return value. Describe the return value for success and failure conditions. If
     there is no return value specify "@return void".


### Control Structures

   * Avoid deep nesting of control structures.
      * This reduces readability and clarity, and is error-prone.
      * While we don't enforce specific [cyclomatic
        complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) metrics, you
        should understand what it is and try to keep yours low.
      * Checking error or exit conditions immediately at the beginning of a
        function or control structure and using return or break appropriately can help
        reduce nesting.
   * Put the most common case after the if rather than the else so that the
     focus of attention is on the most common code path.
   * Simplify complicated tests with boolean variables and function calls. If
     your condition is hard to read, break it out into separate variables or
     function calls and perhaps multiple conditional statements that are easier to
     read.
   * Use positive logic conditions rather than negative ones where possible.
     For example it is easier to understand `if ($isOK)` than it is to understand
     `if (!$isOK)`.
      * In general, try to use positive variable names as well (such as
        `$isOK`) rather than negative (e.g. `$isNotOK`)
      * Avoid ever using double negatives (e.g. `if (!$isBad)`) as it
        drastically reduces readability and clarity.


### Miscellaneous

   * In general code should not be commented out. We have version control, so
     there's hardly any need to commit commented-out code. If in some rare
     circumstance you commit commented-out code, you absolutely must supply a
     reason.
   * Be careful with functions that are extremely long, where your eyes have to
     do a great deal of vertical travel to see what's happening in the function.
     Keep your functions small and purpose-built, breaking large tasks into smaller
     functions for more readability and clarity. A function should optimally do one
     thing and do it well.
   * In general, try to avoid using input parameters as working variables. If
     you need to work with the value of an input parameter, assign the result to a
     local variable. This prevents confusion over the purpose of an input parameter
     and whether it has maintained the value supplied by the caller.
      * If you are going to modify a function input parameter, do so only in
        the first lines of the function so that it's easy to see that the input was
        immediately modified. Generally this is used only for sanitation of some sort
        (see example).

Example:

```js
// SomeFile.js
function doSomething(someString, someMixedValue) {
   // this is more preferable than modifying the input:
   var listOfThings = (_.isArray(someMixedValue) ? someMixedValue : [ someMixedValue ]);

   // if you really want to use the input parameter and just do sanitization on it, do it
   // as the first thing in the function (following the var declarations), such as this:
   someString = _.trim(someString);

   // do not continue modifying the input parameter throughout the function
}
```


## Variables

### Naming

   * **Static** variables are named all lowercase with underscores separating
     words in the name.
   * **Constants** (or publicly-accessible static variables) are named all
     uppercase with underscores separating words in the name.
   * **Instance** variables are named using camelCase (see [the
     glossary](#glossary)).
   * **Scoped** variables (those appearing within a function or other control
     structure) are also named with camelCase.

Other general principles to use in naming your variables:

   * Make the name as specific as possible. It should describe the entire
     purpose of the variable.
   * There is no optimum name length, but a good rule of thumb is 4 - 20
     characters.
   * Do not use Hungarian notation.
   * Array names should typically be in plural form. Exceptions are associative
     arrays (a.k.a. maps).
   * In PHP constants should be created using the `define` function. As
     mentioned above, they should be in `ALL_CAPS_SEPARATED_BY_UNDERSCORES`.
   * Give boolean variables names that naturally imply true or false, using
     true values as much as possible, as described in [the glossary](#glossary)
   * Never differentiate variables solely by capitalization.
   * CC2 - "Code is read far more times than it is written. Be sure that the
     names you choose favor read-time convenience over write-time convenience."
   * Make the distance between a variable's first reference and its last
     reference as short as possible. Define and initialize it as close as possible
     to where it is used.


### Scoping

Try to keep instance variables scoped to the private scope if at all possible.
It is generally better to provide mutators or other functions for editing and
accessing private variables than it is to simply make them non-private.

In languages where there is no scope, or everything is public scope (here's
looking at you, JavaScript), classes can use a leading underscore to indicate
to outside users that functions and variables are "private" and should not be
used or called from outside the class.

Example:

```js
var Class = require('class.extend');

module.exports = Class.extend({

   init: function(config) {
      // nobody outside of this code file should access the _config variable:
      this._config = config;
   },

   doSomething: function(a, b) {
      return this._somePrivateFunction(a, b);
   },

   // because it is named with a leading underscore, nobody outside of this
   // code file should access this function:
   _somePrivateFunction: function(a, b) {
      return a + b;
   },

});
```


### Global Variables

Avoid global variables at all costs. The only thing globally accessible should
be constants (PHP).


### Sanitization

If any variable comes from anything that can be supplied by the user (including
request headers supplied by the browser, etc), it absolutely must be sanitized
before being rendered into a template or used in a SQL query. Understand [data
sanitization](https://xkcd.com/327/) and be sure to protect yourself.


### Declaring

Declare variables within the lowest scope possible. For example:

```
function a() {
  var neededForFnA = 1;

  return function() {
     var neededForInnerFunctionOnly = 2;

     return neededForFnA + neededForInnerFunctionOnly;
  };
}
```

In the example above, `neededForInnerFunctionOnly` is not needed in the scope
above it (`function a`), so it is declared within the anonymous inner function.

(Note: javascript is function-scoped, not block-scoped as many other languages
are. That is, a variable declared within an if-statement block, for example, is
accessible outside of the if-statement block but within the current function.)
Declare the variable at the top of this scope, **before** any other statements.
(In javascript this should appear within a single `var` statement.) If you need
more explanation of this see http://eslint.org/docs/rules/vars-on-top and in
particular the resources listed on that page about variable hoisting.

Variables that are initialized with a value should be declared before those
that are not initialized with a value. For example:

```
var a = 1,
    b = 2,
    c, d;
```


## Source Files

### Naming

   * Class filenames should be named the same as the class with the addition of
     the file type suffix (e.g. `User.php` or `User.js`)
   * Tests in our JS codebases are named `ClassTheyAreTesting.test.js`


## Commit Messages and Commit History

See [our commit history](commit-history.md) page for rules about our commit
history guidelines.


## Glossary

   * **UpperCamelCase** (also known as **PascalCase**): mixed case with the
     first letter of the word and the first letter each internal joined word
     capitalized.
      * `ThisIsAnExample`
      * `thisIsNotAnExample`: see *camelCase*
   * **camelCase**: as we use it in this document, this means a text element
     that is made up of multiple words, with the first letter of each internal
     joined word being capitalized, but the first letter of the entire joined text
     element *not* capitalized.
      * `thisIsAnExample`
      * `ThisIsNotAnExample`: see *UpperCamelCase*
