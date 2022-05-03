# Design Goals

1) Simply adding the framework in an environment that does not know about the framework should have no effect at all. This means we do not apply anything by default, we simply create variables, classes and mixins that can be used. We strive to avoid naming conflicts with often used naming conventions like a ".button" class.
2) We assume that actually writing raw HTML is a rare occurence, hence being verbose in for example naming is not an issue.
3) We want you to be explicit in your choices. If you define something as a "button" and don't explicitly choose "primary" or "secondary", we are not going to "assume" it is primary. You need to explicitly choose it because not setting a primary or secondary class is _also_ a choice.
4) We do not use class equivalents of css directives. No ".bold" class to indicate "font-weight: bold". This defeats the purpose as you might as well write actual css directives. From a content perspective you want a particular item to for example be highlighted. What this actually means from a theme perspective (bold, flashy color, border,...) is not related to the content and should not be "enforced" through essentially hard css rules captured in classes.
5) The end goal is that users can switch between themes and apply them to their application without having to know css details, guess class names or otherwise assume knowledge that they do not reasonably have. This means we can not expect the user to combine 5 separate utility classes nested in specific parent classes to achieve a certain goal.
6) The end goal is that users can create and tweak themes in a visual theme builder without necessarily knowing all the ins and outs of the framework or the finer details of the theme. This means strict metadata is required for enumeration and suggestion purposes. We considered adding JSON descriptor files next to the theme but instead opted for scss-inline metadata. This may run into restrictions that are severe enough that we might still add JSON at some point. Because we use scss iself, we combine comment-based structured metadata with strict naming conventions.
7) The framework should try and limit the specific HTML requirements (for instance sibling relationships, parent/child, first-child etc) to allow for flexibility in designing components. However, sometimes restrictions have to be enforced to know the "as is". A good example of this is a button with an icon and some text. If you can randomly choose to put the text on the left of the icon or on the right at the HTML level, we don't know whether or not we have to reverse the order in case a theme choice is made to always put the icon on one particular side. So instead we require you to put the icon on the left and the text on the right in the html.

The framework is called "aris" based on the name aristotle, the father of logic, most famous student of Plato.

# Composition

The framework focuses on composition where you have primary identification of an item, for example ".is-button" which enables us to scrape css to find composition options. For example if there is a rule ".is-button.is-primary", we know that is-primary is a composition option.
The question is what we split off for composition and what we keep in the core of a component. Is the ".is-button" only a marker class allowing you to composite everything about the button (e.g. .is-medium.is-primary....) or do we accept a baseline as being part of a button?
The design guideline is to opt for "explicit choice", however it can be frustrating to _have_ to make the same choices over and over again.

However, apart from know what you can composite, we need to know which general choices a user can make and which can not be combined. For example choosing both "is-large" and "is-small" at the same time probably does not work as intended.
We want to help the user make relevant choices. This means we add **dimensions** to the button in this example.

The basic is-button class creates the most neutral, but fully styled button relevant to the theme. (This already goes against the explicit design rule.)
You can then use dimensions to "overwrite" certain parts and templated mixins to create your own relevant variants.
Even though this is based on overwriting, the selectors should be limited (and thus easy to further overwrite) and the dimensions strictly regulated in the exact css rules they apply. This, combined with the templating, allows users to create own alternatives with little friction as to css overrides and yet makes it easy to use, hopefully keeping the balance between these two usecases.

A dimension starts with "is", followed by the dimension it stands for, followed by the name of this particular instance of the dimension: is-dimension-name.

For example we could do a spacing dimension for buttons where medium is assumed to be default:

```
@mixin button-spacing-medium {
	padding: $spacing-scale-s $spacing-scale-m;
}
@mixin button-spacing-large {
	padding: $spacing-scale-m $spacing-scale-l;
}
@mixin template-button-spacing {
	padding: $spacing-scale-s $spacing-scale-m;
}

.is-button {
	@include button-spacing-medium
}
.is-button.is-spacing-large {
	@include button-spacing-large
}
```

The dimension here is called "spacing". Page builder knows that only a single option for each dimension is relevant, so it can prompt the user to choose a "spacing" and know that all other spacing composites are irrelevant once a choice has been made.
Because we make sure spacing dimensions only ever work on the same css rule(s), we know that they will be overridden correctly.

In the theme builder we can create a cross section of all the possible compositions or allow the user to simply choose a particular set to see what it looks like.
We can also use the template mixin to determine which css rules are relevant to create our own spacing variant and the default value(s) for those css rules if the user does not fill them in.

## Default dimensions

The variants pull together some default dimension, as a general rule:

> if a variant applies a dimension, there _should_ be a template for that dimension

Some dimensions are entirely optional additions, e.g. the order reverse on a button to reverse icon and text, it is not meant to spawn countless implementations.
It is also not applied by default which is why there is no need for a template.

Once a dimension is applied by default (e.g. size medium), we need to be able to override that dimensional choice easily, which requires a template.

## Extension

We use the terminology "is-" to indicate that you are a particular component.
Some components have common logic. For instance form-text and form-combo might both "extend" from form-component.

You can tag it as both "is-form-text" and "is-form-component", it will inherit both dimensions.

## Variant

Because very few things have only one main variant, but we do want to limit the amount of choices an end user _has_ to make, we have one core dimension "variant" which is (in most cases) the only choice the user has to make. This will also have a more prominent place in page builder.

A variant mixin can, like the base, include multiple dimensions.

To make sure the overrides are done correctly however, the variants _have_ to be defined (in css) before the other dimensions because they have the same specificity at the css level.
The only other option is to write out all combinations of "variant" and their dimensions which would lead to css bloat.

This also means if you create custom variants, they have to be injected after any mixins that might be referenced but before the dimensions that might override specific parts.
As long as we define mixins at the very start (after variables), we should be fine?

### Custom type dimensions

It is OK to customize dimensions to particular types, for instance coloring.

```
.is-button.is-type-primary {
	background-color: $color-primary-darkest;
	color: $color-primary-lightest;
}
.is-button.is-type-primary.is-tint-light {
	background-color: $color-primary-base;
}
```

In this particular case the is-tint-light values are explicitly bound to the primary type. Even though each type variant might have its own version of is-tint-light, the actual css will be different.
Page builder can also conclude, from the css, that the is-tint-light option should only become available once you have made the choice is-type-primary.

By restricting when dimensions might be applied, page builder can calculate at which point in time dimensions become relevant/available.

# Sizes

A lot of things come in different sizes, for example buttons, fonts,...

We should always have at least these levels:

- xs: extra small
- s: small
- m: medium
- l: large
- xl: extra large

We always assume "medium" is the default option at which point it can be left out. For instance ".is-button" is equivalent to ".is-button-m".
If you want a smaller button you can specify ".is-button-s"

# Variables

We use tag notation in comments to add structural metadata to lines of scss.

- description: a human-readable description of what the variable stands for
- priority: how important a variable is, 1000+ means very important (should definitely check for each application), 500+ means it's likely relevant per application but not necessarily changed, anything below is detail work. No assigned priority is 0

# Components

A component is either a native html tag or a vue component. Complex components can consist of multiple other components.

## Has-is

We use "is" and "has" to indicate whether you are a certain component and/or have child components.

Let's take a button as example, if we write:

```
<button>Do something!</button>
```

The framework does nothing as it is a default tag which we do not interfere with.

```
<button class='is-button'>Do something!</button>
<a class='is-button'>Do something!</a>
```

At this point we will enable default styling for the button. We only apply the basics that apply for all buttons so we do not assume "primary" or even "neutral" at this point.

```
<button class='is-button is-primary'>Do something!</button>
```

At this point we will also add the styling for primary buttons. Note that we might have a rule for ".is-primary" in general which can be applied to other components than a button and a ".is-button.is-primary" to specifically enable rules for buttons that are of the primary variant.

```
<button class='is-button is-primary has-icon has-content'><span class='is-icon fa fa-play'></span><span class='is-content'>Do something!</span></button>
```

We now know that our primary button has an icon, the icon itself gets the class "is-icon" which allows us to style both the container for positioning and the element for other reasons.

At the scss level this could look like this:

```
.is-button.is-primary.has-icon {
	.is-icon {

	}
}
```

Note that at this point we do not yet know whether the icon is on the left or the right in the html, nor do we know what you actually want shown. At the html level it might be on the left while we want to show it on the right.
The problem is, when laying it out we might need to add additional whitespace between the icon and content, so we can either work defensively (this also assumes a sibling nature of the html which might not always be possible):

```
.is-button.is-primary.has-icon.has-content {
	.is-icon + .is-content {
		margin-left: 0.3rem;
	}
	.is-content + .is-icon {
		margin-left: 0.3rem;
	}
}
```

We could also target :last-child and :first-child and while this does get rid of the sibling requirement, it forces another layout requirement onto the resulting html:

```
.is-button.is-primary.has-icon.has-content {
	.is-content:last-child {
		margin-left: 0.3rem;
	}
	.is-content:first-child {
		margin-right: 0.3rem;
	}
}
```

Alternatively we could mandate that at the html level it is _always_ on the left and if you want to switch it to the right, it is a css directive to do so.
You could make a mixin that says that all primary buttons need an icon on the right:

```
@mixin icon-right {
	...
}
.is-button.is-primary {
	@include icon-right;
}
```

# Mixins

We use mixins to add characteristics to components dynamically.
There are two kinds of mixins that the user can create via the theme builder:

1) Template driven: for example we create a template for "border" mixins which basically combines all the interesting css directives that may apply and includes the default value if you don't set it.

```
@mixin template-border {
	border-width: 1px;
	border-style: solid;
	border-radius: 0px;
	border-color: $color-neutral-dark;
}
```

At this point the user can create a new directive border.

.is-button.is-primary {
	@include border-heavy;
}