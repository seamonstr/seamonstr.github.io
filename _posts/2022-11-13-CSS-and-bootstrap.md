---
title: CSS and Bootstrap
---

* Including a stylesheet in an HTML file directly:

```html
<head>
	<link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
</head>
```

* Bootstrap is installable with npm using `npm install bootstrap`; puts it into `node_modules/bootstrap/dist/css/bootstrap.min.css`

## CSS @-rules

@-rules are a set of CSS directives that begin with @ and end with a semi-colon. Most of them have their own, unique syntax, although there are a set of them that are called "group rules" - they allow creation of a group of CSS rules that only apply in the case of the @-rule's condition evaluates to `true`.

These @-rules can be nested inside other group rules.

*@media* is used to apply CSS rules only in the case of specific media attributes being present ("media" meaning characteristics of the output device).

```css
/* Apply the group only if the screen is at least 900px wide */
@media screen and (min-width: 900px) {

}
```

*@supports* is used to apply a group of CSS rules only in the case that the browser supports a given feature. Eg. the following checks if a particular format of font is supported:

```css
@supports font-format(opentype) {
	/* Stuff */
}
```
