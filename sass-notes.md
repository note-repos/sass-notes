# Notes from [SASS](https://sass-lang.com/)

The **@extend** rule may only be used within style rules.

// This comment won't be included in the CSS.

/_ But this comment will, except in compressed mode. _/

/_! This comment will be included even in compressed mode. _/

The deeper you nest, the more bandwidth it takes to serve your CSS and the more work it takes the browser to render it. Keep those selectors shallow!

Some of these CSS properties have shorthand versions that use the namespace as the property name. For these, you can write both the shorthand value and the more explicit nested versions.

Exp:

scss:

```scss
.info-page {
  margin: auto {
    bottom: 10px;
    top: 2px;
  }
}
```

css:

```css
.info-page {
  margin: auto;
  margin-bottom: 10px;
  margin-top: 2px;
}
```

---

scss:

```scss
$primary: #81899b;
$accent: #302e24;
$warn: #dfa612;

:root {
  --primary: #{$primary}; // to give sass variable to css variable use #{}
  --accent: #{$accent};
  --warn: #{$warn};

  // Even though this looks like a Sass variable, it's valid CSS so it's not
  // evaluated.
  --consumed-by-js: $primary;
}
```

css:

```css
:root {
  --primary: #81899b;
  --accent: #302e24;
  --warn: #dfa612;
  --consumed-by-js: $primary;
}
```

---

Unfortunately, interpolation removes quotes from strings, which makes it difficult to use quoted strings as values for custom properties when they come from Sass variables. As a workaround, you can use the meta.inspect() function to preserve the quotes.

scss:

```scss
@use "sass:meta";

$font-family-sans-serif: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto;
$font-family-monospace: SFMono-Regular, Menlo, Monaco, Consolas;

:root {
  --font-family-sans-serif: #{meta.inspect($font-family-sans-serif)};
  --font-family-monospace: #{meta.inspect($font-family-monospace)};
}
```

css:

```css
:root {
  --font-family-sans-serif: -apple-system, BlinkMacSystemFont, "Segoe UI",
    Roboto;
  --font-family-monospace: SFMono-Regular, Menlo, Monaco, Consolas;
}
```

The parent selector, &, is a special selector invented by Sass that’s used in nested selectors to refer to the outer selector.

Sass has a special kind of selector known as a “placeholder”. It looks and acts a lot like a class selector, but it starts with a % and it's not included in the CSS output. In fact, any complex selector (the ones between the commas) that even contains a placeholder selector isn't included in the CSS, nor is any style rule whose selectors all contain placeholders.

What’s the use of a selector that isn’t emitted? It can still be extended! Unlike class selectors, placeholders don’t clutter up the CSS if they aren’t extended and they don’t mandate that users of a library use specific class names for their HTML.

Ex:
scss:

```scss
%toolbelt {
  box-sizing: border-box;
  border-top: 1px rgba(#000, 0.12) solid;
  padding: 16px 0;
  width: 100%;

  &:hover {
    border: 2px rgba(#000, 0.5) solid;
  }
}

.action-buttons {
  @extend %toolbelt;
  color: #4285f4;
}

.reset-buttons {
  @extend %toolbelt;
  color: #cddc39;
}
```

css:

```css
.action-buttons,
.reset-buttons {
  box-sizing: border-box;
  border-top: 1px rgba(0, 0, 0, 0.12) solid;
  padding: 16px 0;
  width: 100%;
}
.action-buttons:hover,
.reset-buttons:hover {
  border: 2px rgba(0, 0, 0, 0.5) solid;
}

.action-buttons {
  color: #4285f4;
}

.reset-buttons {
  color: #cddc39;
}
```

---

### Sass Variables

Sass variables are imperative, which means if you use a variable and then change its value, the earlier use will stay the same. CSS variables are declarative, which means if you change the value, it’ll affect both earlier uses and later uses.

Sass variables, like all Sass identifiers, treat hyphens and underscores as identical. This means that `$font-size` and `$font_size` both refer to the same variable

Variables defined with `!default` can be configured when loading a module with the @use rule. Sass libraries often use !default variables to allow their users to configure the library’s CSS.

If you need to set a global variable’s value from within a local scope (such as in a mixin), you can use the !global flag. A variable declaration flagged as `!global` will always assign to the global scope.

scss:

```scss
$variable: first global value;

.content {
  $variable: second global value !global;
  value: $variable;
}

.sidebar {
  value: $variable;
}
```

css:

```css
.content {
  value: second global value;
}

.sidebar {
  value: second global value;
}
```

_The !global flag may only be used to set a variable that has already been declared at the top level of a file. It may not be used to declare a new variable._

Define a map from names to values that you can then access using variables.  
scss:

```scss
@use "sass:map";

$theme-colors: (
  "success": #28a745,
  "info": #17a2b8,
  "warning": #ffc107,
);

.alert {
  // Instead of $theme-color-#{warning}
  background-color: map.get($theme-colors, "warning");
}
```

css:

```css
.alert {
  background-color: #ffc107;
}
```

---

### Interpolation

Interpolation can be used almost anywhere in a Sass stylesheet to embed the **result** of a SassScript expression into a chunk of CSS. Just wrap an expression in `#{}`

Interpolation in SassScript always returns an __unquoted__ string.

While it’s tempting to use this feature to convert quoted strings to unquoted strings, it’s a lot clearer to use the `string.unquote()` function. Instead of `#{$string}`.

---

### @use

A stylesheet’s @use rules must come before any rules other than @forward, including style rules. However, you can declare variables before @use rules to use when configuring modules.

You can do this by writing `@use "<url>" as <namespace>`.

You can even load a module without a namespace by writing `@use "<url>" as *`

Sass makes it easy to define a private member by starting its name with either `-` or `_`

If you want to make a member private to an entire package rather than just a single module, just don’t forward its module from any of your package’s entrypoints (the stylesheets you tell your users to load to use your package). You can even hide that member while forwarding the rest of its module!

scss:

```scss
// _library.scss
$black: #000 !default;
$border-radius: 0.25rem !default;
$box-shadow: 0 0.5rem 1rem rgba($black, 0.15) !default;

code {
  border-radius: $border-radius;
  box-shadow: $box-shadow;
}
```
```scss
@use 'library' with (
  $black: #222,
  $border-radius: 0.1rem
);
```
A stylesheet can define variables with the !default flag to make them configurable.
Exm:
scss:
```scss
// _library.scss
$black: #000 !default;
$border-radius: 0.25rem !default;
$box-shadow: 0 0.5rem 1rem rgba($black, 0.15) !default;

code {
  border-radius: $border-radius;
  box-shadow: $box-shadow;
}
```
```scss
// style.scss
@use 'library' with (
  $black: #222,
  $border-radius: 0.1rem
);
```

Built-in module variables (such as math.$pi) cannot be reassigned.

---

### @forward

If you do write both a @forward and a @use for the same module in the same file, it’s always a good idea to write the @forward first. That way, if your users want to configure the forwarded module, that configuration will be applied to the @forward before your @use loads it without any configuration.

This is written @forward "<url>" as <prefix>-*, and it adds the given prefix to the beginning of every mixin, function, and variable name forwarded by the module. For example, if the module defines a member named reset and it’s forwarded as list-*, downstream stylesheets will refer to it as list-reset.

scss:
```scss
// src/_list.scss
@mixin reset {
  margin: 0;
  padding: 0;
  list-style: none;
}
```
```scss
// bootstrap.scss
@forward "src/list" as list-*;
```
```scss
// styles.scss
@use "bootstrap";

li {
  @include bootstrap.list-reset;
}
```
css:
```css
li {
  margin: 0;
  padding: 0;
  list-style: none;
}
```

The `hide` form means that the listed members shouldn’t be forwarded, but everything else should. The `show` form means that only the named members should be forwarded. In both forms, you list the names of mixins, functions, or variables (including the $).

---

### @import
The Sass team discourages the continued use of the @import rule. Sass will gradually phase it out over the next few years, and eventually remove it from the language entirely. Prefer the @use rule instead. (Note that only Dart Sass currently supports @use. Users of other implementations must use the @import rule instead.)

---

### @mixin and @include

Mixin names, like all Sass identifiers, treat hyphens and underscores as identical. This means that reset-list and reset_list both refer to the same mixin

Argument lists can also have trailing commas! This makes it easier to avoid syntax errors when refactoring your stylesheets.

scss:
```scss
@mixin rtl($property, $ltr-value, $rtl-value) {
  #{$property}: $ltr-value;

  [dir=rtl] & {
    #{$property}: $rtl-value;
  }
}

.sidebar {
  @include rtl(float, left, right);
}
```

css:
```css
.sidebar {
  float: left;
}
[dir=rtl] .sidebar {
  float: right;
}
```

Normally, every argument a mixin declares must be passed when that mixin is included. However, you can make an argument optional by defining a default value which will be used if that argument isn’t passed. Default values use the same syntax as variable declarations: the variable name, followed by a colon and a SassScript expression.

scss:
```scss
@mixin replace-text($image, $x: 50%, $y: 50%) {
  text-indent: -99999em;
  overflow: hidden;
  text-align: left;

  background: {
    image: $image;
    repeat: no-repeat;
    position: $x $y;
  }
}

.mail-icon {
  @include replace-text(url("/images/mail.svg"), 0);
}
```

css:
```css
.mail-icon {
  text-indent: -99999em;
  overflow: hidden;
  text-align: left;
  background-image: url("/images/mail.svg");
  background-repeat: no-repeat;
  background-position: 0 50%;
}
``` 
Default values can be any SassScript expression, and they can even refer to earlier arguments!

Keyword Arguments: When a mixin is included, arguments can be passed by name in addition to passing them by their position in the argument list.

scss:
```scss
@mixin square($size, $radius: 0) {
  width: $size;
  height: $size;

  @if $radius != 0 {
    border-radius: $radius;
  }
}

.avatar {
  @include square(100px, $radius: 4px);
}
```
css:
```css
.avatar {
  width: 100px;
  height: 100px;
  border-radius: 4px;
}
``` 

Sometimes it’s useful for a mixin to be able to take any number of arguments. If the last argument in a @mixin declaration ends in `...`, then all extra arguments to that mixin are passed to that argument as a list. This argument is known as an argument list.

scss:
```scss
@mixin order($height, $selectors...) {
  @for $i from 0 to length($selectors) {
    #{nth($selectors, $i + 1)} {
      position: absolute;
      height: $height;
      margin-top: $i * $height;
    }
  }
}

@include order(150px, "input.name", "input.address", "input.zip");
```

css:
```css
input.name {
  position: absolute;
  height: 150px;
  margin-top: 0px;
}

input.address {
  position: absolute;
  height: 150px;
  margin-top: 150px;
}

input.zip {
  position: absolute;
  height: 150px;
  margin-top: 300px;
}
``` 

In addition to taking arguments, a mixin can take an entire block of styles, known as a content block. A mixin can declare that it takes a content block by including the @content at-rule in its body. The content block is passed in using curly braces like any other block in Sass, and it’s injected in place of the @content rule.

scss:
```scss
@mixin hover {
  &:not([disabled]):hover {
    @content;
  }
}

.button {
  border: 1px solid black;
  @include hover {
    border-width: 2px;
  }
}
````

css:
```css
.button {
  border: 1px solid black;
}
.button:not([disabled]):hover {
  border-width: 2px;
}
```
A content block is lexically scoped, which means it can only see local variables in the scope where the mixin is included. It can’t see any variables that are defined in the mixin it’s passed to, even if they’re defined before the content block is invoked.

---

### functions

While it’s technically possible for functions to have side-effects like setting global variables, this is strongly discouraged. Use mixins for side-effects, and use functions just to compute values.

Sometimes it’s useful for a function to be able to take any number of arguments. If the last argument in a @function declaration ends in ..., then all extra arguments to that function are passed to that argument as a list. This argument is known as an argument list.

scss:
```scss
SCSS SYNTAX
@function sum($numbers...) {
  $sum: 0;
  @each $number in $numbers {
    $sum: $sum + $number;
  }
  @return $sum;
}

.micro {
  width: sum(50px, 30px, 100px);
}
```
css:
```css
.micro {
  width: 180px;
}
```

It’s only allowed within a @function body, and each @function must end with a @return.

Because any unknown function will be compiled to CSS, it’s easy to miss when you typo a function name. Consider running a CSS linter [stylelint](https://stylelint.io/) on your stylesheet’s output to be notified when this happens!

---

### @extend

@extend <selector>, and it tells Sass that one selector should inherit the styles of another.

scss:
```scss
.error {
  border: 1px #f00;
  background-color: #fdd;

  &--serious {
    @extend .error;
    border-width: 3px;
  }
}
```
css:
```css
.error, .error--serious {
  border: 1px #f00;
  background-color: #fdd;
}
.error--serious {
  border-width: 3px;
}
````

Of course, selectors aren’t just used on their own in style rules. Sass knows to extend everywhere the selector is used. This ensures that your elements are styled exactly as if they matched the extended selector.

scss:
```scss
.error:hover {
  background-color: #fee;
}

.error--serious {
  @extend .error;
  border-width: 3px;
}
````
css:
```css
.error:hover, .error--serious:hover {
  background-color: #fee;
}

.error--serious {
  border-width: 3px;
}
```

Extends are resolved after the rest of your stylesheet is compiled. In particular, it happens after parent selectors are resolved. This means that if you @extend .error, it won’t affect the inner selector in .error { &__icon { ... } }. It also means that parent selectors in SassScript can’t see the results of extend.

Like module members, a placeholder selector can be marked private by starting its name with either - or _. A private placeholder selector can only be extended within the stylesheet that defines it. To any other stylesheets, it will look as though that selector doesn’t exist.

Like module members, a placeholder selector can be marked private by starting its name with either - or _. A private placeholder selector can only be extended within the stylesheet that defines it. To any other stylesheets, it will look as though that selector doesn’t exist.

As a rule of thumb, extends are the best option when you’re expressing a relationship between semantic classes (or other semantic selectors). Because an element with class .error--serious is an error, it makes sense for it to extend .error. But for non-semantic collections of styles, writing a mixin can avoid cascade headaches and make it easier to configure down the line.

While @extend is allowed within @media and other CSS at-rules, it’s not allowed to extend selectors that appear outside its at-rule. This is because the extending selector only applies within the given media context, and there’s no way to make sure that restriction is preserved in the generated selector without duplicating the entire style rule.

scss:
```scss
@media screen and (max-width: 600px) {
  .error--serious {
    @extend .error;
    //      ^^^^^^
    // Error: ".error" was extended in @media, but used outside it.
  }
}

.error {
  border: 1px #f00;
  background-color: #fdd;
}
```

---

### @error

When writing mixins and functions that take arguments, you usually want to ensure that those arguments have the types and formats your API expects. If they aren't, the user needs to be notified and your mixin/function needs to stop running.

Sass makes this easy with the @error rule, which is written @error <expression>. It prints the value of the expression (usually a string) along with a stack trace indicating how the current mixin or function was called. Once the error is printed, Sass stops compiling the stylesheet and tells whatever system is running it that an error occurred.

SCSS:
```scss
@mixin reflexive-position($property, $value) {
  @if $property != left and $property != right {
    @error "Property #{$property} must be either left or right.";
  }

  $left-value: if($property == right, initial, $value);
  $right-value: if($property == right, $value, initial);

  left: $left-value;
  right: $right-value;
  [dir=rtl] & {
    left: $right-value;
    right: $left-value;
  }
}

.sidebar {
  @include reflexive-position(top, 12px);
  //       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  // Error: Property top must be either left or right.
}
```

---

### @warn

When writing mixins and functions, you may want to discourage users from passing certain arguments or certain values. They may be passing legacy arguments that are now deprecated, or they may be calling your API in a way that’s not quite optimal.

The @warn rule is designed just for that. It’s written @warn <expression> and it prints the value of the expression (usually a string) for the user, along with a stack trace indicating how the current mixin or function was called. Unlike the @error rule, though, it doesn’t stop Sass entirely.
```scss
$known-prefixes: webkit, moz, ms, o;

@mixin prefix($property, $value, $prefixes) {
  @each $prefix in $prefixes {
    @if not index($known-prefixes, $prefix) {
      @warn "Unknown prefix #{$prefix}.";
    }

    -#{$prefix}-#{$property}: $value;
  }
  #{$property}: $value;
}

.tilt {
  // Oops, we typo'd "webkit" as "wekbit"!
  @include prefix(transform, rotate(15deg), wekbit ms);
}
```

---





