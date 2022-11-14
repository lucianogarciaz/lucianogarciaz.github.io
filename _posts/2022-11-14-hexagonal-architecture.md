---
layout: post
title:  "Hexagonal architecture"
author: luciano
featured: true
categories: [ Software Development, XP, Software Economics, Design, Software Architecture ]
image: assets/images/hexagonal-architecture.png
---
Hexagonal architecture (also called **ports & adapters**) is a type of **clean architecture** that has been around for a while but is gaining popularity in the age of microservices. 
The idea is to design your software so that the different components are **loosely coupled** and **can be easily swapped out or replaced**. This makes your software more **maintainable** and **scalable in the long run**.


To understand what is clean architecture, we need to talk about software architecture.
## Software Architecture: 
Is a set of patterns and decisions that provide a necessary **frame of reference to guide the construction of software**, allowing developers to work in the same line and be focused on **business decisions**. 
It will define the structure, operation and interaction between the parts of the software.
 
## Clean Architectures
The clean architecture was introduced by Uncle bob, they define, guide and structure our code by a series of layers that has a **golden rule**.

## Golden Rule
<img class="shadow-lg" src="{{site.baseurl}}/assets/images/golden-rule.png" alt="Golden Rule"/>
It's a dependency rule which says that source code **dependencies can only point inwards**. 
Nothing in an inner circle can know anything at all about something in an outer circle. In particular, the name of something declared in an outer circle must not be mentioned by the code in the inner circle. That includes functions, and classes. variables, or any other named software entity.

So let's review each layer:

_Note: all examples are oversimplified, and it will have some code smells for sure (I will write another post about that 😉)_  
### Infrastructure
Is the code that changes based on external decisions (like external libraries, database connections, etc). 
Is the layer where the implementation of the interfaces is defined in the domain layer.
We are going to rely on the SOLID DIP to be able to decouple from external dependencies.
```go
package infra

import (
	//...
	"github.com/go-sql-driver/mysql"
	"domain"
)

var _ domain.AccountRepository = &MysqlAccountRepository{}

type MysqlAccountRepository struct {db *sqlx.DB}

func (m MysqlAccountRepository) Save(ctx context.Context, account domain.Account) error {
	// ...
	_, err := m.db.ExecContext(ctx, query)
	if err!= nil {
		return err
    }
	// ...
}

```
This is a very simplified implementation of an account repository interface with mysql. 
The key here is that we are using an **external library (mysql)** to persist our accounts.


### Application
This is where the use case lives. Sign-In, Sign-Up, Create User, etc.
It orchestrates what are the business steps to execute successfully (or not) the use case.
this simple sign-up example is written in go:
```go
package app

import (
	//...
	"domain"
)

type SignUpCommandHandler struct{
	ar domain.AccountRepository
}

func (s SignUpCommandHandler) Exectute(ctx context.Context, cmd SignUpCommand) (domain.Event[], error) {
	// validates the parameters are correct and instantiate an account
	account, err := domain.NewAccount(cmd.id, cmd.email)
	if err!= nil {
		return nil, err
        }
	
	// stores the account in a repository . 
	err = s.ar.Save(ctx,account)
	if err!= nil {
		return nil, err
        }
	
	return account.Pull(), nil
}
```
Here we can see how the `SignUp` use case orchestrates between validating with business rules in the domain the inputs, calling the account repository and last but not least returning the domain events recorded by the account.

### Domain
It's where we design our business logic and define contracts with the outside world (infrastructure)
```go
package domain

type Account struct {
	id string
	email string
} 

// NewAccount is a named constructor
func (a Account) NewAccount(id, email string) (Account,error) {
	// ... validates email, id etc...
	return Account{id, email}, nil
}
```

## Why should you give it a try?
#### Maintainable code:
**Reduce the time needed** to maintain and modify your code over time by writing maintainable code. A general rule of thumb is that maintainability increases with decreasing technical debt. 

**Reduce/Avoid Accidental Complexity:** This means that we can produce value as quickly as the necessary complexity permits.
<img class="shadow-lg" src="{{site.baseurl}}/assets/images/accidental-complexity.png" alt="Accidental complexity"/>

#### Simple and easy to code: 
The **code flow is quite similar among the features** that we have to develop. We have **less cognitive load** and we can focus more on business logic.

#### Separation of concerns: 
Since our **business logic is isolated** the domain depends on nothing but itself. Also, the code is more **cohesive**.

#### Ready for changes (Adaptability & Flexibility): 
The core business logic talks with the other parts of the application through contracts a.k.a interfaces. Therefore, we can have several implementations for a given contract and swap between them when needed.

#### Testability: 
We can write tests for each layer. Is always easier to test in isolation, we can mock any dependency if needed. Translated into faster tests and confidence to deploy a new feature.

#### Easy to onboard (software economics)
Since the code is easier to follow and most use cases will follow the same pattern, new joiners will be able to pick tasks faster.