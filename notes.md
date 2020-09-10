# Using Utility-First CSS to Manage CSS at Scale

This presentation won't be in-depth enough to convince you of anything, but I want to introduce a concept that's been gaining traction in the CSS world.

I'm going to start off with a statement that you may or may not agree with: Styling with CSS is very easy, but managing and scaling CSS stylesheets is very hard.

In my experience, there are a few key challenges:

## Challenges

### CSS is global

Because CSS is global, the selectors and properties defined in the stylesheet will be applied to any HTML document that loads it. This global nature and the cascade that defines the "C" in "CSS" are incredibly powerful, but because most apps or websites consist of multiple pages or screens that share the same stylesheet, these language features also make it very easy to end up with **leaky styles**.

### Specificity is hard

Because CSS selectors are global, one approach for keeping styles scoped to a specific interface element is to chain or nest selectors. Chaining and nesting increase a selector's _specificity_, which means that in order to override or modify a given property on that element you need to write a selector that's _more_ specific.

```css
.quote {
  border-left: solid 1px;
  display: block;
  font-size: 1em;
  font-style: italic;
  padding-left: 1em;
}

.quote .citation {
  color: gray;
  font-style: normal;
}


/* A handy utility class, right? */
.emphasized {
  color: blue;
  font-weight: bold;
}
```

```html
<blockquote class="quote">
  <p>I love deadlines. I like the whooshing sound they make as they fly by.</p>
  <footer className="citation emphasized">
      Douglas Adams
  </footer>
</blockquote>
```

The emphasized color won't be applied because of specificity. So you might be tempted to do one of two things to increase the specificity of the `.emphasized` selector:

```css
.citation.emphasized {
  color: blue;
  font-weight: bold;
}

/* or */

.emphasized {
  color: blue !important;
  font-weight: bold;
}
```

It's easy to end up in a specificity arms race.

### Duplication and bloat

To avoid leaky styles or to avoid the hassle of refactoring specificity issues, the path of least resistance is typically to start fresh with a whole new selector and duplicate styles.

### The result

In a large codebase being maintained by many developers with mixed abilities, it becomes very easy to mess things up. Even for a codebase with just one maintainer, it's easy to mess things up without strong organizing principles and naming conventions. As a result, devs often become afraid of touching CSS, and stylesheets end up becoming an "append-only" codebase.

This was the case at dscout pre-Particle, and we're not immune from reaching this point again, especially if we write large amounts of custom CSS on top of Particle to build our apps.

## Solutions

Over the years, people have proposed various CSS-writing architectures that aim to solve these challenges. Here are some of the names and acronyms you may have come across:

- OOCSS (Object Oriented CSS)
- SMACSS (Scalable and Modular Architecture for CSS)
- BEM (Block Element Modifier)
- Atomic CSS
- Utility-First CSS

### Object Oriented Paradigm

OOCSS, SMACSS, and BEM are all based on an object-oriented paradigm. In fact, OOCSS has object-oriented explicitly in it's name. Unfortunately I don't have the time, or enough knowledge to be honest, to provide in-depth explanations of these three, but here's a quick overview:

#### OOCSS

OOCSS is a mental framework that has you break an interface into reusable "objects" and how to write reusable and sharable CSS selectors. From what I can tell, the architecture surfaced around 2009 and has since lost traction. It's two principles are:

- Separate structure from skin
- Separate containers from content

"Structure" refers to things like dimensions, padding, margins, layout, etc. "Skin" refers to visual features, like colors, border radii, drop shadows, etc.

Separating containers from content means that content should be styled the same regardless of what container it is in.

OOCSS does not enforce any specific naming convention.

#### SMACSS (Scalable and Modular Architecture for CSS)

Like OOCSS, SMACSS is more of a style guide. It does not enforce any specific naming convention, but encourages you to think about and separate styles into 5 categories:

- Base
- Layout
- Module
- State
- Theme

The argument is that writing CSS selectors that intertwine these categories increases complexity and makes stylesheets difficult to manage.

SMACSS also calls out the importance of limiting specificity in CSS selectors.

#### BEM

BEM is the most specific of the object-oriented solutions. BEM seems to be the most popular, which may be in part because it has explicit naming conventions and a rigid structure that make it easier to understand and apply.

BEM has you define and style your interface with the terms "block," "element," and "modifier." A block is defined as a "standalone entity that is meaningful on its own." An example of a block might be a dropdown menu. An element is "part of a block that has no standalone meaning and is semantically tied to the block." An example of an element might be a dropdown menu's toggle. A modifier represents a change in state, appearance, or behavior. An example might be the open vs closed state of a dropdown menu.

BEM would look like this:

```html
<div class="dropdown dropdown--open">
  <button class="dropdown__toggle">Open</button>
  <ul class="dropdown__content">
    <li class="dropdown__item">Option 1</li>
    <li class="dropdown__item dropdown__item--selected">Option 1</li>
  </ul>
</div>
```

```css
.dropdown {
  position: relative;
}

.dropdown__toggle {
  border: solid 1px #ddd;
  display: block;
}

.dropdown__content {
  display: none;
  position: absolute;
  top: 100%;
  width: 100%;
}

.dropdown--open .dropdown__toggle {
  border-color: #333;
}

.dropdown--open .dropdown__content {
  display: block;
}

...
```

#### Pros

These object-oriented solutions have a lot of merit and, when applied rigourously and consistently, can address many of the challenges I outlined earlier.

Particularly when following BEM naming conventions, you end up with:

- minimal nesting, which alleviates specificity challenges (only modifier classes rely on nesting) but doesn't fully eliminate it
- clear relationships between elements
- separation of HTML structure from CSS selectors

#### Cons

However, these approaches have some downsides:

- Naming is hard, and pretty much everything in an object-oriented architecture needs to be named carefully
- CSS is still likely to bloat as you add more features and interfaces, as many selectors can end up applying an overlapping set of styles. Two different objects might look awfully similar.
- Changing the appearance of an interface means modifying CSS, which is inherently risky.
- This is the big one: An object-oriented naming convention (like BEM) forces you to start with abstractions, which are likely to be premature abstractions


### Functional Paradigm

CSS architectures that follow a functional paradigm have been gaining popularity. You'll see terms like Atomic CSS and Utility-First CSS. Two examples of frameworks that follow this paradigm are Tachyons and TailwindCSS.

Functional CSS an architecture that favors small, single-purpose classes that describe visual function. It favors flexibility and speed, acknowledges that designs and patterns will constantly change.

> You can't predict the future. This is why you should always favor composition over inheritance. - Sarah Dayan, [In Defence of Utility-First CSS](https://frontstuff.io/in-defense-of-utility-first-css)

Here's what a responsive "card" component might look like in BEM:

```css
.card {
  border-radius: .25em;
  box-shadow: 0 1em 1em rgba(0,0,0,0.1);
  display: flex;
}

.card__body {
  flex-grow: 1;
  padding: 0.5em;
}

.card__avatar {
  border-radius: .25em;
  display: block;
  margin-bottom: .5em;
}

.card__title {
  font-weight: bold;
  text-align: center;
}

.card__footer {
  border-top: solid 1px gray;
  padding: 0.5em;
}

.card.card--clickable {
  cursor: pointer;
}

.card.card--clickable:hover {
  box-shadow: 0 2em 1em rgba(0,0,0,0.2);
}

@media (min-width: 20em) {
  .card {
    width: 12em;
    height: 20em;
  }
  
  .card__body,
  .card__footer {
    padding: 1em
  }
}

```

```html
<div class="card">
  <div class="card__body">
    <img class="card__avatar" src="face.jpg" />
    <div class="card__title">
      Bad User
    </div>
  </div>
  <div class="card__footer">
    Demerits: 5
  </div>
</div>

<a class="card card--clickable" href="somewhere.html">
  <div class="card__body">
    <img class="card__avatar" src="face.jpg" />
    <div class="card__title">
      Bad User
    </div>
  </div>
  <div class="card__footer">
    Demerits: 5
  </div>
</a>
```

And here's what the HTML would look like with functional CSS:

```html
<div class="radius--small shadow--medium breakpoint-medium:width--12 breakpoint-medium:height--20 flex">
  <div class="padding--small breakpoint-medium:padding--medium flex-grow--1">
    <img class="radius--small margin-b--medium" src="face.jpg" />
    <div class="font-weight--bold text-align--center">
      Bad User
    </div>
  </div>
  <div class="padding--small breakpoint-medium:padding--medium border-t--thin border-color--gray">
    Demerits: 5
  </div>
</div>

<a class="radius--small shadow--medium breakpoint-medium:width--12 breakpoint-medium:height--20 flex hover:shadow--large cursor--pointer" href="somewhere.html">
  <div class="padding--small breakpoint-medium:padding--medium flex-grow--1">
    <img class="radius--small margin-b--medium" src="face.jpg" />
    <div class="font-weight--bold text-align--center">
      Bad User
    </div>
  </div>
  <div class="padding--small breakpoint-medium:padding--medium border-t--thin border-color--gray">
    Demerits: 5
  </div>
</a>
```

At first glance that might look really offensive. And to be honest, it kind of is.

#### Pros:

- No time spent coming up with semantic names (remember, naming is hard)
- Less duplication in stylesheet
- Creating new interfaces require no new CSS
- Safer – Changes happen in HTML, which is local, instead of CSS, which is global

#### Cons:

Here are some arguments against utility-first CSS.

- The resulting HTML is ugly
- No context-aware styles (no inheritance, so styles don't adapt to their context, and styles also can't adapt to sibling relationships)
    - Some would argue that this is a pro, as the result of adding a utility class should always be predicatble. But this comes at the cost of ease-of-use.
- CSS-powered themeing is more limited
    - Individual utility classes can have their properties customized for a certain theme, but broader or more coordinated style variations can't be accomplished with only CSS
        - e.g. consider a card component. In one theme, it has square edges, a small amount of padding, and a small drop shadow. In the other theme, it has round edges, larger padding, and a large, soft drop shadow. If all of the card styles were encapsulated within `.card`, a theme could control all of these with only CSS. But with utility-first CSS, JS would need to know to conditionally apply a different set of utility classes.
- Stylesheet size: utility classes for every property and variant need to be defined upfront, but many may never be used
    - Can be addressed with build tools, but that can be complicated
- Global design changes can be tedious
    - depending on the change, may need to find and replace every instance of a UI pattern and update the CSS
    - with a component-based UI framework like React, this is largely a moot point, as a resuable component should have already encapsulated the UI pattern

#### Notes

- All your utility classes should have the same specificity score (10), so priority is determined by their order in the stylesheet. That means you have to make a decision on which variants within a specific utility category take precedence. For example, does `.margin-b--none` take precedence over `.margin--m`?

## Takeaways:

- Utility first allows for rapid prototyping and building customized interfaces with no custom CSS
- Favors composition instead of inheritance (just like React best practices)
- A combination of BEM-like component classes and utility classes strikes a nice balance (and is what we do in Particle). E.g. use utility classes for one-off cases, prototyping, exploring new patterns, etc. Extract utility classes into "component" classes when patterns emerge or when benefits of context-aware styles outway the costs
- Reusable React components can abstract away the complexity and "mess" of applying many utility classes, so developers only need to think about the API of the component (via props) and not about the styling concerns
    - so we can probably lean more heavily on utility classes in our Particle components than we currently do


## Resources

- https://css-tricks.com/growing-popularity-atomic-css/
- https://frontstuff.io/in-defense-of-utility-first-css
- https://www.browserlondon.com/blog/2019/06/10/functional-css-perils/
- https://tailwindcss.com/docs/utility-first
- https://johnpolacek.github.io/the-case-for-atomic-css/
