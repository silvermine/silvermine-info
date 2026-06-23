# Styling Code Standards

Sass (file extension .scss) is a language which compiles to CSS. It is a powerful tool in the writing of CSS but does not absolve you as a
developer from the code which is output. You must understand what good CSS looks like before you can effectively write Sass.

Below are some guiding principles and rules when writing styling code.

## General Standards

Unless there is a compelling reason to use an ID we should always default to using classes [[1]](#references) for styling.

Code smells [[2]](#references) to watch out for might include:

   * >3 selectors (when compiled)
   * Using an ID or !important
   * <htmlElement>.class
   * z-index declarations

### Naming

Namimg of elements (e.g. classes, variables, mixins) should be camel case [[3]](#references) with hyphens being used to
distinguish logical structual elements as needed. Two hyphens should be used to seperate modifiers from the consistent body of a name.

Examples:
   * `$color-link--visited`
   * `.pm-mediaGroup-top--mobile`
   * `.pm-mediaGroup-top--desktop`
   * `$color-primButtonBg--hover`
   * `@mixin linkBackground-circle()`

## Sass-specific Standards

   * Sass Documentation [[4]](#references)

Like all other code we use three (3) space indents, no tabs.

We write all Sass code in the SCSS syntax [5](#references).

**Avoid using `@extend` in any form. [[6]](#references)**

If you are creating something new you will want to create a file in the correct place as a partial (`_name.scss`) which indicates to the
compiler that this file should not be compiled on its own. This partial would then be imported into `collector.scss` in the appropriate
place for the cascade.

**Almost never nest more than 3 levels deep.** If you're deeper than that, you're writing an ugly selector. Ugly in that it's too reliant on
HTML structure (fragile), overly specific (too powerful), and not very reusable (not useful). It's also on the edge of being difficult to
understand.

Avoid specificity wars at all cost.
> You fell victim to one of the classic blunders - The most famous of which is "never get involved in a land war in Asia",
> but only slightly less well-known is this: "Never go in against a z-index when the code is on the line"!

Keep DRY [[7]](#references) principles in mind and use Sass `@mixins` to your advantage.

Be generous with code comments. In general always use single-line comments `//` so they will never be included in compiled code. Try to
include an explanation that could be deciphered by someone new. Include an issue number and URL of the code being exercised when possible,
especially when the line does not have an obvious purpose.

```
// This is a comment which explains the code that immediately follows
// #97294 /example/url/of/where/code/is/exercised
```

**Use variables for all colors, shared common numbers, and z-index values**. Always go to the variables directory and check to see if a
standard color already exists which can be utilized.

Examples:
   * `$color-blue: #4a6da7;`
   * `$color-link: $color-blue;`

### Blank Lines

Follow our blank lines [[8]](#references) standards. There should always be a blank line seperating top level selectors.

In most cases multiple selectors in a single block should each be listed on a new line.

Examples:

```
.mainContainer {
   .nestedClassOne {
     color: tomato;
   }
   .nestedClassTwo {
      color: honeydew;
      &:hover {
         color: firebrick;
      }
   }
   // ----------------------
   // A major nested section
   // ----------------------
   .nestedClassThree {
      color: salmon;
   }
}

.secondContainer {
   .nestedClassFour {
      color: wheat;
   }
   .oneThing,
   .secondThing,
   .thirdThing {
      color: black;
   }
}
```

### Directional Content

Often styling needs to consider if the immediate context is being displayed in a left-to-right (LTR) or right-to-left (RTL) manner,
consistent with the language being displayed. Styling rules should be written to consider the closest language direction boundary instead of
relying solely on the page language and direction. An appropriate class will be available on any element which functions as a language
boundary.

Examples:

```
// Element which is considered a language direction boundary
.mainElement {
   &.dir-ltr {
      padding-left: 5px;
   }
   &.dir-rtl {
      padding-right: 5px;
   }
}
```
### References

[1] [http://oli.jp/2011/ids/](http://oli.jp/2011/ids/)
[2] [https://en.wikipedia.org/wiki/Code_smell](https://en.wikipedia.org/wiki/Code_smell)
[3] [Coding Standards Glossary](coding-standards.md#glossary)
[4] [http://sass-lang.com/documentation/file.SASS_REFERENCE.html](http://sass-lang.com/documentation/file.SASS_REFERENCE.html)
[5] [http://sass-lang.com/documentation/file.SASS_REFERENCE.html#syntax](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#syntax)
[6] [http://www.sitepoint.com/avoid-sass-extend/](http://www.sitepoint.com/avoid-sass-extend/)
[7] [https://en.wikipedia.org/wiki/Don%27t_repeat_yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
[8] [coding-standards.md#blank-lines](coding-standards.md#blank-lines)
