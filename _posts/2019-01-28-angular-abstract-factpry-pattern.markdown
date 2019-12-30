---
layout: post
title:  "Angular Tips | Combine Abstract Factory Pattern & Injector to inject a service depends on parameter 👷 📐"
date:   2019-01-28 19:39:05 +0100
categories: angular software injector pattern architecture
thumbnail: "https://miro.medium.com/max/2050/1*29aT2IueecGIskkY0UPIIw.png"
---

![Header image](https://miro.medium.com/max/2050/1*29aT2IueecGIskkY0UPIIw.png)

The good practices guide of Angular proposes that the transfer and storage of information is done through services.

In many occasions, we will want to create generic components (to manage different resources of a family) that will have to __handle different services depending on how they were created.__

The challenge to be solved in this context (and the topic of the article) is to manage this __dynamic injection__ of services in a __clean, scalable and efficient way__.

In this article we will use the __Abstract Factory pattern in combination with the Angular Injectors__ to solve the mentioned problem.

# A poor and ugly solution

One way to solve this problem is to inject each service and use whatever is necessary:

```typescript
@Component({
  . . .
})
export class GenericComponent implements OnInit {
  public resource: any;
  constructor(
    private service1: Service1,
    private service2: Service2,
    // Rest of services
    . . .
  ) {}
  ngOnInit() {
    // Get parameter to resolve Service to use
    const serviceType = this.route.snapshot.data['type'];
    
    // Resolve service to use
    if (serviceType === 'SERV1') {
      this.foods = this.service1.get();
    }
    if (serviceType === 'SERV2') {
      this.foods = this.service1.get();
    }
    // Everything else
    . . .
  }
}
```

However, __it is not a good solution__, since the logic and the constructor of our component will grow as we need to add more services. _What about if you have to solve the injection of 50 services?_

# Applying method
If you are not acquainted with the Abstract Factory pattern, I recommend this reading before applying the method: [Abstract Factory Pattern](https://www.baeldung.com/java-abstract-factory-pattern).

To implement this creational pattern, we should consider coding the following classes and interfaces:

* __AbstractFactoryInterface__: interface that will implement each class.
* __AbstractFactoryProvider__: will solve a ConcreteFactory upon request.
* __ConcreteFactory(s)__: create an instance of a class that implements the Abstract Factory Interface.

> 💡 _Keep in mind that implementation doesn’t strictly follow the principles of the Abstract Factory pattern, this is an adaptation combined with the Angular Injectors_.

## AbstractFactoryInterface & AbstractFactoryProvider

The first step is to create the __common interface__ (methods that will implement the concrete classes) and the __provider__ responsible for resolving which class to instantiate.

```typescript
// food.ts
import { PastaService } from './pasta.service';
import { PizzaService } from './pizza.service';

// AbstractFactoryInterface
export interface Food {
  get(): Observable<any>;
}

// AbstractFactoryProvider as a HashMap
export const foodMap = new Map([
  ['PASTA', PastaService],
  ['PIZZA', PizzaService]
]);
```

## ConcreteFactory
Once our interfaces and the provider are created, we will create our concrete classes.
```typescript
// pasta.service.ts
import { Injectable } from '@angular/core';
import { Food } from './food.interface';
. . .

// ConcreteFactory
@Injectable()
export class PastaService implements Food {
  constructor() {}
  public get(): Observable<any> {
    return Observable.of([
      {
        name: 'Carbonara'
      },
      {
        name: 'Pesto'
      }
    ])
  }
}
```

Note that the _PizzaService_ class implements the methods of the Food interface.

## Angular Injector

Once our ConcreteFactories are implemented, we will proceed to __solve the one that proceeds and inject it__ using the __Angular Injector__.

```typescript
// generic.component.ts
import { Component, OnInit, Injector, Input } from '@angular/core';
import { Food, foodMap } from './food.interface';

@Component({
  selector: 'generic-food',
  templateUrl: './app.component.html',
  styleUrls: [ './app.component.css' ]
})
export class GenericFoodComponent implements OnInit {
  @Input() type: string; // 'PASTA' or 'PIZZA'
  public foods: Array<any>;
  public service: Food;
  
  constructor(private injector: Injector) {}
  ngOnInit() {
    // Resolve AbstractFactory
    const injectable = foodMap.get(this.type);
    // Inject service
    this.service = this.injector.get(injectable);
    // Calling method implemented by Food interface
    this.service.get().subscribe((foods) => {
      this.foods = foods;
    })
  }
}
```

# Considerations

Please __consider not applying this method when it is not strictly necessary__ and only when you need to instantiate services in a generic and dynamic way.

# Conclusions

We have seen how to apply the _AbstractFactory pattern_ in combination with the Angular Injectors, which can resolve the coding of bad smells when you need to __inject a service depending on a parameter instance of inject all of them__.

# [Example](https://stackblitz.com/edit/rjlopezdev-injector?embed=1&file=src/app/app.component.ts){:target="_blank"}