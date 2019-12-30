---
layout: post
title:  "Angular Tips | Use Projection technique to improve your components reusability 📈"
date:   2019-01-12 18:30:00 +0100
categories: angular software reusability content-projection architecture
thumbnail: "https://miro.medium.com/max/642/1*0KrUOMQuHd9jSJ5CcmuXaw.png"
---

__Reusability__ is one of the pillars of software engineering and an important indicator that informs us of, among others, the __quality of the architecture__ of a module, component, system …

__Angular__ proposes an architecture based on __Web Components__ with the aim of improving the quality of the project, the reuse of code blocks, modularization …

Some of the best known techniques for writing our components are __Composition and Parameterization__. With these techniques, we can __encapsulate blocks of code__ within each other (giving rise to new components) and __configure their appearance and behavior__ according to some parameters.

The objective of this article is to __introduce__ a lesser known __technique that improves the reusability of the components: the Projection__.

## Composition example

```html
// list.component.html

<h1> List of items </h1>
<item *ngFor="let item of items"> </item>
```
---

```html
// item.component.html

<div>
  <h2> {% raw %}{{ item.title }}{% endraw %} </h2>
  <p> {% raw %}{{ item.text }}{% endraw %} </p>
</div>
```

Applying this technique, we can __create components by composing others__.

## Example of Parameterization

```typescript
// parameterized.component.ts

@Component({
  ...
})
export class ParametrizedComponent {
  @Input() param1: string;
  @Input() param2: boolean;
  constructor() {}
}
```

---

```html
// parameterized.component.html

<h1> {% raw %}{{ param1 }}{% endraw %} </h1>
<span *ngIf="param2" class="badge badge-secondary"> New </span>
// foo.component.html
<h1> This is an example of ParameterizedComponent usage </h1>
<parameterized
  [param1]="This is a title"
  [param2]="true">
</parameterized>
```

Applying this technique, we can create components with __appearance and dynamic behavior dependent on parameters__.

# Improving reusability: Component Projection

By using the previous techniques, we have managed to encapsulate small blocks of code into components, composing them to give rise to others (composition) and giving them a different behavior and appearance depending on a series of predefined parameters.

However, __we can go a step further__.

Imagine that you wanted to create a __layout__ with a __hierarchy__ that could vary beyond configurations through parameters:

![Hierarchy layout](https://miro.medium.com/max/642/1*0KrUOMQuHd9jSJ5CcmuXaw.png)

We could use the technique of composition and parameterization, although we would limit the parameters and force us to write components and templates with the sole purpose of being used in … perhaps a single component?

Let’s see how our template could be using the projection to create a “card” component (with its header, body and footer):

```html
// tree.component.html

<card>
  <card-header> Some header </card-header>
  <card-body>
    <div class="content">
      <h2> Title </h2>
      <p> Lorem impsum... </p>
      <img src="awesome-foo.png">
    </div>
  </card-body>
  <card-footer> With love by @rjlopez </card-footer>
</card>
```

To make possible the construction of this hierarchy, we will use the angular tag `<ng-content>`. This tag, allows us to __load in the component the template that is found inside the tag of our component__.

```html
// card.component.html

<div class="card text-center">
  <ng-content select="app-card-header"></ng-content>
  <ng-content select="app-card-body"></ng-content>
  <ng-content select="app-card-footer"></ng-content>
</div>
```

---

```html
// card-header.component.html

<div class="card-header">
  <ng-content></ng-content>
</div>
```

With the previous sentence, using `<ng-content>`, we indicate that the __component accepts an embedded html tag__. And what tag do you accept? __Through the “select” attribute, we inform you that you must accept tags of type__ `<app-card-header>`. In case of not passing any value to “select”, it will accept all the content.


# Conclussions

By using this technique, we can create __hierarchical components and less dependent on others, which greatly improves the reusability of these__. As a result, we will achieve a more __flexible architecture__ and increase the degree of __code reusability__.

# [Example](https://stackblitz.com/edit/rjlopezdev-angular-projection?embed=1&file=src/app/app.component.ts){:target="_blank"}
