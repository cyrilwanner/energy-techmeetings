# CSS Specificity / `!important`

> `!important` declarations **should not be used unless they are absolutely necessary** after all other avenues have been exhausted. If you use `!important` out of laziness, to avoid proper debugging, or to rush a project to completion, then you’re abusing it, and you (or those that inherit your projects) will suffer the consequences.
>
> If you never use `!important`, then that’s a sign that you understand CSS and give proper forethought to your code before writing it.

> Sources:
> * https://www.smashingmagazine.com/2010/11/the-important-css-declaration-how-and-when-to-use-it/
> * https://css-tricks.com/specifics-on-css-specificity/

## Example

* Open [resources/css-specificity.html](resources/css-specificity.html) in the editor
* What styles do you expect?
* Now open it in a browser, is it the same as you expected?
* Why do we see this behavior?

## How CSS Specificity gets calculated

General order when applying styles:

1. Declarations from the user agent *(default browser styles, e.g. blue box shadow on input focus on chrome)*
2. Declarations from the user *(e.g. bigger font size in chrome settings)*
3. Declarations from the author *(stylesheets from the website)*
4. Declarations from the author with `!important` added *(in stylesheets from the website)*
5. Declarations from the user with `!important` added *(e.g. bigger font size is enforced)*

Order within stylesheets:
https://css-tricks.com/specifics-on-css-specificity/#article-header-id-0

## How to avoid `!important`

<details>
  <summary>Declare multiple styles for an element in the same selector if possible using nesting with a preprocessor</summary>

```scss
#animals li {
    font-family: Arial, Helvetica, sans-serif;
    font-weight: normal;
    font-size: 20px;
    color: blue;

    &.favorite {
        color: red;
    }
}
```
</details>

<details>
  <summary>Or if you don't use sass (or another preprocessor), use the same selectors</summary>

```css
#animals li {
    font-family: Arial, Helvetica, sans-serif;
    font-weight: normal;
    font-size: 20px;
    color: blue;
}

#animals li.favorite {
    color: red;
}
```
</details>

<details>
  <summary>Avoid complex selectors</summary>

```css
#animals {
    font-family: Arial, Helvetica, sans-serif;
    font-weight: normal;
    font-size: 20px;
    color: blue;
}

.favorite {
    color: red;
}
```

You of course have to make sure they don't collide on your site when you use many selectors and pages.
Packages like [css-modules](https://github.com/css-modules/css-modules) solve this problem by automatically scoping your selectors.
</details>

<details>
  <summary>When using multiple files, make sure they are included in the correct order</summary>

```scss
// desktop.scss
.box {
    width: 500px;
    margin: 10px;
    padding: 10px;
    border: 1px solid black;
}
```

```scss
// mobile.scss
.box {
    width: 90%;
}
```

Make sure `mobile.scss` gets included *after* `desktop.scss`:
```scss
// main.scss
@include 'desktop';

@media only screen and (max-width: 600px) {
    @include 'mobile';
}
```
</details>

## Exceptions

Sometimes it is still valid to use `!important`.

<details>
  <summary>When is it okay to use !important?</summary>

* To overwrite inline styles (e.g. added by an included 3rd party script)
* For users with special needs, for example in user stylesheets to force a bigger font
* If you can't control the whole styling of the site and some later applied stylesheets overwrite your styles
</details>
