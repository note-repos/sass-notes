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























