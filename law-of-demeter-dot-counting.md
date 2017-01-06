http://haacked.com/archive/2009/07/14/law-of-demeter-dot-counting.aspx/

# _The Law of Demeter Is Not A Dot Counting Exercise_
# Закон Деметры не о том, как считать точки

 July 14, 2009
_Recently I read a discussion on an internal mailing list on whether or not it would be worthwhile to add a null dereferencing operator to C#._

Недавно во внутренней рассылке обсуждалось, нужен ли добавить в C# оператор разыменовывания. 

_For example, one proposed idea would allow the following expression._

Например, предлагалось разрешить такое выражение.
```
object a = foo.?bar.?baz.?qux;
```

_This would assign the variable a the value null if any one of foo, bar, or baz is null instead ofthrowing a NullReferenceException. It’s a small, but potentially helpful, mitigation for the billion dollar mistake._

Если хотя бы одно из свойств foo, bar, или baz окажется null, вместо выброса NullReferenceException переменной будет тоже присвоено значение null. Несложный, но перспективный способ смягчить последствия [ошибки на миллион](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare).

_Sure enough, it did not take long for someone to claim that this syntax would be unnecessary if the code here was not violating the sacred Law of Demeter (or LoD for short). I think this phenomena is an analog to Godwin’s Law and deserves its own name. 
Let’s call it the “LoD Dot Counting Law”:_

И конечно, тут же нашелся человек, заявивший, что оператор был бы не нужен вовсе, если бы код в примере не нарушал Закон Деметры (Law of Demeter, LoD)

As a discussion of a code expression with more than one dot grows longer, the probability that someone claims a Law of Demeter violation approaches 1.

What is wrong with the claim that the above expression violates LoD? To answer that let’s briefly cover the Law of Demeter. I’m not going to cover it in detail but rather point to posts that describe it in much better detail than I would.

The Many Forms of Demeter

The formal object form of the law can be summarized as:

A method of an object may only call methods of:

The object itself.

An argument of the method.

Any object created within the method.

Any direct properties/fields of the object.

A general formulation of the law is:

Each unit should have only limited knowledge about other units: only units “closely” related to the current unit. Or: Each unit should only talk to its friends; Don’t talk to strangers.

This of course leads to the succinct form of the law:

Don’t talk to strangers

In other words, try to avoid calling methods of an object that was returned by calling another method. Often, people shorten the law to simply state “use only one dot”.

One of the key benefits of applying LoD is low coupling via encapsulation. In his paper, The Paperboy, The Wallet, and The Law of Demeter (PDF) (it’s a relatively quick read so go ahead, I’ll be here), David Bock provides a great illustration of this law with an analogy of a paperboy and a customer. Rather than having a customer hand over his wallet to pay the paperboy, he instead has the paperboy request payment from the customer.

In answer to “Why is this better?” David Bock gives these three reasons.

The first reason that this is better is because it better models the real world scenario…  The Paperboy code is now ‘asking’ the customer for a payment.  The paperboy does not have direct access to the wallet.  

The second reason that this is better is because the Wallet class can now change, and the paperboy is completely isolated from that change…

The third, and probably most ‘object-oriented’ answer is that we are now free to change the implementation of ‘getPayment()’.

Note that the first benefit is an improvement not only in encapsulation but the abstraction is also improved.

Dot Counting Is Not The Point

You’ll note that David doesn’t list “50% less dots in your code!” as a benefit of applying LoD. The focus is on reduced coupling and improved encapsulation.

So going back to the initial expression, does foo.bar.baz.qux violate LoD? Like most things, it depends.

For example, suppose that foo is of type Something and it contains properties named Bar, Baz, and Qux which each simply return this.

In this semi-contrived example, the expression is not an LoD violation because each property returns the object itself and according to the first rule of LoD, “you do not talk about LoD” … wait … sorry… “a method is free to call any properties of the object itself” (in a future post, I will cover the class form of LoD which seems to indicate that if Bar, Baz, and Qux return the same type, whether it’s the same object or not, LoD is preserved).

This pattern is actually quite common with fluent interfaces where each method in a calling chain might return the object itself, but transformed in some way.

So we see that counting dots is not enough to indicate an LoD violation. But lets dig deeper. Are there other cases where counting dots leads do not indicate an LoD violation? More importantly, is it always a bad thing to violate LoD? Are there cases where an LoD violation might even be the right thing to do?

Go Directly to Jail, Do Not Pass Go

Despite its name, violating the Law of Demeter will not get you on an episode of Cops nor is it some inviolable law of nature.

As the original paper points out, it was developed during design and implementation of the Demeter system (hence the name) and was held to be a law for the developers of that system.

The designers of the system felt that this practice ensured good Object-Oriented design:

The motivation behind the Law of Demeter is to ensure that the software is as modular as possible. The law effectively reduces the occurrences of nested message sendings (function calls) and simplifies the methods.

However, while it was a law in the context of the Demeter system, whether it should hold the weight that calling it a Law implies is open to debate.

David Bock refers to it as an idiom:

This paper is going to talk about a particluar (sic) ‘trick’ I like, one that is probably better classified as an ‘idiom’ than a design pattern (although it is a component in many different design patterns).

Martin Fowler suggests (emphasis mine)

I’d prefer it to be called the Occasionally Useful Suggestion of Demeter.

Personally, I think most developers are guilty of bad encapsulation and tight coupling. I’m a bit more worried about that than applying this law inappropriately (though I worry about that too). Those who have deep understanding of this guideline are the ones who are likely to know when it shouldn’t be applied.

For the rest of us mere mortals, I think it’s important to at least think about this guideline and be intentional about applying or not applying it.

I Fought The Law and The Law Won

So what are the occasions when the Law of Demeter doesn’t necessarily apply? There’s some debate out there on the issue.

In his post, Misunderstanding the Law of Demeter, Daniel Manges argues that web page views aren’t domain objects and thus shouldn’t be subject to the Law of Demeter. His argument hinges on a Rails example where you send an Order object to the view, but the view needs to display the customer’s name.

<%= @order.customer.name %>

Counting two dots, he considers the change that would make it fit LoD:

<%= @order.customer_name %>

He then asks:

Why should an order have a customer_name? We’re working with objects, an order should have a customer who has a name.

…when rendering a view, it’s natural and expected that the view needs to branch out into the domain model.

Alex Blabs of Pivotal Labs takes issue with Daniel’s post and argues that views aredomain objects and an order ought to have a customer_name property.

It’s an interesting argument, but the following snippet of a comment by Zak Tamsen summarizes where I currently am on this subject (though my mind is open).

because they don’t. the primary job of the views (under discussion) is to expose the internal state of objects for display purposes. that is, they are expressly for data showing, not data hiding. and there’s the rub: these kind of views flagrantly violate encapsulation, LoD is all about encapsulation, and no amount of attribute delegation can reconcile this.

The problem as I see it with Alex’s approach is where do you stop? Does the Order object encapsulate every property of Customer? What about sub-properties of the Customer’s properties? It seems the decision to encapsulate the Customer’s name is driven by the view’s need to display it. I wouldn’t want my domain object’s interface to be driven by the needs of the view as that would violate separation of concerns.

There’s another option which might be more common in the ASP.NET MVC world than in the Rails world, I’m not sure. Why not have a view specific model object. This would effectively be the bridge between the domain objects and the view and could encapsulate many of the these properties that the view needs to display.

Another case where an LoD violation might not be such a bad idea is in cases where the object structure is public and unlikely to change. While in Norway, I had the opportunity to briefly chat with Michael Feathers about LoD and he pointed out the example of Excel’s object model for tables and cells. If LoD is about encapsulation (aka information hiding) then why would you hide the structure of an object where the structure is exactly what people are interested in and unlikely to change?

Use It or Lose It

When I learn a new guideline or principle, I really like to dig into where the guideline breaks down. Knowing where a guideline works, and what its advantages are is only half the story in really understanding it. When I can explain where it doesn’t work and what its disadvantages are, only then do I feel I’m starting to gain understanding.

However, in writing about my attempts at understanding, it may come across that I’m being critical of the guideline. I want to be clear that I think the Law of Demeter is a very useful guideline and it applies in more cases than not. It’s one of the few principles that can point to an empirical study that may point to its efficacy.

In a Validation of Object-Oriented Design Metrics as Quality Indicators, the authors of the study provide evidence that suggests The Law of Demeter reduces the probability of software faults.

Still, I would hope that those who apply it don’t do it blindly by counting dots. Dot counting can help you find where to look for violations, but always keep in mind that the end goal is reducing coupling, not dots.
