# Design Goals

1) Simply adding the framework in an environment that does not know about the framework should have no effect at all. This means we do not apply anything by default, we simply create variables, classes and mixins that can be used. We strive to avoid naming conflicts with often used naming conventions like a ".button" class.
2) We assume that actually writing raw HTML is a rare occurence, hence being verbose in for example naming is not an issue. We will write "small" rather than abbreviating to "s"
3) We want to generate predictable css that can be easily overridden as needed. This means we want to avoid high specificity selectors.
4) Layout is done by parents, not by components. For example we never put margin on a button as is. A button container may choose to add margin to all its buttons however.
5) We do not use class equivalents of css directives. No ".bold" class to indicate "font-weight: bold". This defeats the purpose as you might as well write actual css directives. From a content perspective you want a particular item to for example be highlighted. What this actually means from a theme perspective (bold, flashy color, border,...) is not related to the content and should not be "enforced" through essentially hard css rules captured in classes. Dimensions are sometimes tightly coupled to a single rule of css (especially when it comes to layout) but sometimes they encapsulate multiple css rules that belong together.
6) We assume users are not aware of the css details, the most important goal of the framework is to be able to guide the user, we want to show him checkboxes where he can simply check the particular combinations he wants, he should not have to "guess" which classes exist and how they work together.
7) The framework should try and limit the specific HTML requirements (for instance sibling relationships, parent/child, first-child etc) to allow for flexibility in designing components. However, sometimes restrictions have to be enforced to know the "as is". A good example of this is a button with an icon and some text. If you can randomly choose to put the text on the left of the icon or on the right at the HTML level, we don't know whether or not we have to reverse the order in case a theme choice is made to always put the icon on one particular side. So instead we require you to put the icon on the left and the text on the right in the html.

The framework is called "aris" based on the name aristotle, the father of logic, most famous student of Plato.

# Mixins

Aris relies heavily on mixins to define all the necessary rules.
From there, it will generate SCSS which is then further compiled into css.

There are actually multiple SCSS generation strategies possible, depending on what you need. The most lightweight strategy for example is not IE-compatible because it relies heavily on :where and :is selectors.
The SCSS generation is done in a specific order to maintain css override guarantees.

The naming strategy for the mixins consists of:

```
[component]-[dimension]-[option]
```

Each dimension encapsulates one or more CSS rules that are grouped together. Each option within a dimension has specific values for these rules.
An option must _not_ have css that does not belong to its dimension as this would break predictable overriding.

For example

```css
@mixin button-color-primary {
	color: $color-primary-dark;
}
```

This is a mixin for the component button. The dimension it is referring to is "color" and the option in this case is "primary".

## Composition

Each component can have multiple dimensions and each dimension can have multiple options.

Through class composition, the user can (in HTML) choose which dimension options they want to apply. While multiple dimensions can be applied on an element, only one option per dimension is allowed.

So you shouldn't do:

```html
<button class="is-button is-color-primary is-color-secondary">A button</button>
```

As different options for the same dimension intentionally affect the same css rules, only one can successfully be applied.

What you can do however is:

```html
<button class="is-button is-color-primary is-size-small">A small button</button>
```

A single option from multiple dimensions.

## Class names

As you saw in the above example, aris heavily relies on the "is-" naming convention. An element +is+ something.
A secondary naming that is sporadically used in aris is "has-".

In some cases (hopefully one day fully replaced with the :has selector), the parent of a particular component must have some css applied for the child to render correctly. In this case we put a "has-" on the parent. For instance:

```html
<div class="has-tooltip">
	<div class="is-tooltip">My tooltip!</div>
</div>
```

## Variants

Having to repeatedly state the same combination of dimensions again and again across your application becomes tiresome very quickly. Not to mention that it is a very fragile approach.

Component variants are a reserved dimension which acts differently from other dimensions: the main purpose of variants is to combine multiple dimensions into a preset class that can be applied. For instance suppose have an "OK" button on forms that should be styled the same way everywhere.
If we change our mind, we want to change the styling of said OK button for the whole application. A variant is a good solution for this:

```css
@mixin button-variant-form-ok {
	@include button-color-primary;
	@include button-size-small;
}
```

In html we can then do:

```html
<button class="is-button is-variant-form-ok">OK!</button>
```

As you can see, we apply both the default button styling and the specific variant styling we have. Since both should simply mix and match dimensions, the variant should override anything it might want to in the default button class.
Additionally you can still apply dimensions to this variant if needed, the overriding is structured correctly to allow for this. For instance suppose in some exceptional case you want a large OK button:

```html
<button class="is-button is-variant-form-ok is-size-large">OK!</button>
```

## Inheritance

Because there is a lot of overlap between components, they can extend one another. When one component extends another component, it automatically inherits the available variants and the available dimensions.
It does +not+ however inherit the default variant, this needs to be redefined for each component.

## Patterns

While the distinction is irrelevant on most levels, we do distinguish (in name) between components and patterns.
A component is mostly styling itself and possibly positioning its children.

This means a component typically has a lot of dimensions and options to play with. An example is the button.

A pattern is a much more complex component which forces much more complex styling and layouting on its children. This means it has fewer degrees of freedom and will generally only come with a select amount of variations. An example of this is a top menu bar which may need to make sure children appear in dropdowns etc.
For components, dimensions and predictable overriding is very important, for patterns this is not always feasible.

# Templating

The end user of the styling (so the one making the application, +not+ the one making the theme) should rarely (and preferably never) have to write custom css.
Because of how aris is structured, we can provide helpful suggestions (e.g. available dimensions and options) to the user, allowing him to simply check the combination he wants. 
We can also provide a simplistic variant builder where you simply capture those dimensional choices in a prebuilt variant.

But sometimes the user will have to make their own custom dimension option.

For instance, suppose in all those color selections for button he can not find the one he wants, he wants to create a new option for his custom color choice.
At this point it is important to reiterate how crucial it is that dimensions only implement a select set of css rules. Once you start creating dimension options that perform arbitrary CSS, they can no longer be mixed and matched with other dimensions. While this is of course still possible if you have no alternative, the user should be dissuaded from doing so.

This is the point of templates. Each dimension can (but is not required to) have a template.

```css
@mixin template-button-color {
	color: $color-primary-dark;
	background-color: $color-primary-lightest;
}
```

Based on this we can give the user a basic css editor where he can only choose custom values for the available rules in the template. He gives his dimension option a name and he at that point he has added his own custom option to the application which is still compatible with all the other options.
He can then proceed to make a variant that embeds the custom dimension option if he so chooses.

# Overriding mixins

As someone who creates themes or has more complex usecases, it is interesting to note that mixins can also be overriden (much like variables).
Because of how aris works, the last mixin (even if it is loaded "later" in the bundle) will be used when generating the resulting css.

This means if you for some reason disagree with standard mixins provided in the base, you can provide your own alternative implementations. Take care not to break the aris goals of predictable overriding.

# Smart styling

When you are writing HTML, you don't want to know how the css is structured. You don't want to know how the extensions are set up and you don't want to add multiple classes which embed some sort of css knowledge into the html.
This may sound vague, but let's take the example of "form components".

We have a base component "form-component" and two extensions "form-text" and "form-combo".

There are at least two usecases for this:

- we want to write css that targets +all+ form-components (so both form-text and form-combo in this case)
- we want to write css that targets a specific component (e.g. form-text)

We could embed this knowledge in html by adding enough classes:

```html
<div class="is-form-component is-form-text"></div>
<div class="is-form-component is-form-combo"></div>
```

However, this arbitrarily limits the amount of combinations I can do from css. Suppose we have more form components and all of them are obviously an extension of form components, but perhaps we want to make an additional distinction between form components that have a label that is separate from the input field and those where they are together. For instance we add "form-checkbox" and "form-switch".

```html
<div class="is-form-component is-form-checkbox"></div>
<div class="is-form-component is-form-switch"></div>
```

We may want to have the label for the checkbox floating next to the actual checkbox but for the other components we want the label to float above the input.

However, because we have made a particular choice in our html, it is hard to group checkbox and switch on the one hand and text and combo on the other. We can only target form-component but that would affect them all, or start a fragile listing:

```css
.is-form-checkbox, .is-form-switch {
	...
}
```

We want to allow the aris implementors as many degrees of freedom as they want. Are there 5 finegrained extensions between form-component and form-text for whatever reason? No problem.
From the HTML perspective it becomes easier:

```html
<div class="is-form-text"></div>
<div class="is-form-combo"></div>
```

From the scss perspective we could do this:

```css
// @component form-component
@mixin form-component-default {
	...
}
// @component form-labelled-component form-component
@mixin form-labelled-component-default {

}
// @component form-text-component form-labelled-component
@mixin form-text-default {

}
// @component form-combo-component form-labelled-component
@mixin form-text-default {

}
// @component form-checkbox-component form-component
@mixin form-text-default {

}
// @component form-switch-component form-component
@mixin form-text-default {

}
```

At this point we can target all form-components together (and of course separately) but also target all "labelled" components. We can arbitrarily make different choices in aris without having to change the HTML.
But of course, when writing your scss, you may have set up the base concept of a labelled component, but you can't really know how many extensions there are at any given time.

You don't want to revert to the fragile listing.

Instead of

```css
.is-form-text, .is-form-combo {
	...
}
```

We can do this:

```css
:is-form-labelled-component {

}
```

Part of the aris generation will make sure that that particular selector is expanded to include all available extensions that are available at that point.

We can do the same with dimensions. Because dimensions are generally only relevant in combination with a component, we assume when you are writing custom css you will combine them:

```css
.is-form-labelled-component:is-color-primary {
	...
}
```

You can also combine these two:

```css
:is-form-labelled-component:is-color-primary {
	...
}
```

## Example

To show you what this does, let's take this test component:

```css
// @component test
@mixin test-variant-default {
	color: yellow;
}
@mixin test-color-red {
	color: red;
}
:is-test:is-color-red {
	text-transform: uppercase;
}
```

If we assume nothing else exists, the resulting css is pretty plain:

```css
.is-test.is-color-red {
	text-transform: uppercase;
}
.is-test {
	color: yellow; 
}
.is-color-red {
	color: red; 
}
```

This is pretty boring and predictable, the fun starts when we start adding extensions. We simply add this line:

```css
// @component test2 test
```

This tells aris that there is an extension test2 for test. There is no default variant so aris won't actually make a ".is-test2" class.
However, it will assume test2 can appear and the generated css changes:

```css
:is(.is-test, .is-test2).is-color-red {
	text-transform: uppercase;
}
.is-test {
	color: yellow; 
}
.is-color-red {
	color: red; 
}
```

Aris has multiple generation strategies, this one prefers using :is and :where, another one duplicates selectors:

```css
.is-test.is-color-red, .is-test2.is-color-red {
	text-transform: uppercase;
}
.is-test {
	color: yellow; 
}
.is-color-red {
	color: red; 
}
```

## Hidden dimension

This is all fun and games...until someone embeds the dimension you are targetting into a variant.
Once a dimension is embedded in a variant, it will +not+ appear in the html class list. For instance suppose we have these aris rules:

```css
// @component test
@mixin test-variant-default {
	color: yellow;
}
@mixin test-color-red {
	color: red;
}
@mixin test-variant-special {
	@include test-color-red;
}
:is-test:is-color-red {
	text-transform: uppercase;
}
// @component test2 test
```

We can now have html that says:

```html
<div class="is-test is-variant-special"></div>
```

A plain selector for:

```css
.is-test.is-color-red {
	...
}
```

Will not target our element anymore. Damn those variants!

When you write your selector with the above ":" syntax however, it will still work:

```css
:is-test:is-color-red {
	...
}
```

Aris knows that we also need to take into account the variants of the component (and its extensions), so it will generate this:

```css
:is(.is-test, .is-test2):is(.is-color-red, .is-variant-special) {
	text-transform: uppercase;
}
```

The exact same css is generated if that variant was for component test2 instead of test.
This allows you to write css that targets components and dimensions without having to worry about all the extension hierarchies and variants.

# Generation

There are currently two generation strategies, one duplicates rules and one uses a smart combination of :where() (for zero-specificity rules) and :is().

## Duplication

The duplication strategy has the following specificity:

```css
.is-button {
	// the core component
}
.is-button.is-variant-primary {
	// primary variant stuff with double specificity
}
.is-button.is-size-large {
	// dimension with same specificity as variant but always positioned _after_ the variant in the css
}
```

In this generation strategy we have at most 2 class selectors specificity but this approach tends to blow up the css as extension hierarchies get more complex. The problem is that any dimension is automatically inherited by the extensions and must be made available to them by combining the dimension definition with that particular extension component.

In a particular test this generation strategy created 147kb of css in 1.5 seconds.
When more complexity is added, this can start to increase rapidly. The resulting css also contains massive selectors.

Upside: it is backwards compatible with for example IE11.

## Non-duplication

The default generation of aris uses this strategy. When "explosion" into multiple components is necessary, it will use :where() or :is() selectors to minimize the additional overhead, you can have massively complex css rules that only get a tiny bit larger when new extensions are added rather than having to be duplicated in full.

This strategy always uses 1 class specificity generation. For instance a variant of a component that has a lot of extensions might be generated like this:

```css
:where(.is-content, .is-padded-content, .is-tooltip, .is-h1, .is-h2, .is-h3, .is-h4, .is-h5, .is-h6, .is-p, .is-icon, .is-button, .is-badge).is-variant-neutral {
	...
}
```

The :where() makes sure that it adds zero specificity, resulting in a one class selector.
The advantage of guaranteeing one class selectors is that aris can optimize further.

It will check for every variant and dimension/option combination whether it is unique. If it is uniquely named (there is no other component with the same variant or dimension/option), it will drop the :where() alltogether. 

For example if we add a variant to the same component as our above example but make sure our variant is uniquely named across our application, we get this:

```css
.is-variant-unique-name {
	background-color: transparent;
	color: #198754;
	border-color: #198754;
}
```

So by choosing more unique naming and/or optimizing our extension hierarchy so dimensions with the same name actually mean the same thing, we can actually reduce the generated css considerably.

In the same test as our above example, this strategy (which was not optimized for unique naming yet) generated 85kb in 450ms.
There was no visual difference with the first strategy in the showcase application the test was run upon.

## Future

Other generation strategies are possible. If dimensions are guaranteed to be unique (so no longer need to be tied to their component), it takes away the source of the css bloat in the first strategy.
At this point we could generate IE11 compatible code in a rather clean way.

If variants are also unique in this particular setup, we can generate rules that only have 1 class specificity:

```css
.is-button {
	...
}
.is-button-variant-primary {
	...
}
.is-size-large {
	...
}
```

If variants are not guaranteed to be unique but dimensions are, we can still do this:

```css
.is-button {
	...
}
.is-button.is-variant-primary {
	...
}
.is-size-large.is-size-large {
	...
}
```

This strategy is currently not implemented.
