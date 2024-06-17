<div style="text-align: justify">

# What you'll find here

While I do not strive to be an expert in "how to build a product", I do have some thoughts on this matter, having seen some successes and failures firsthand, and having learned from resources throughout the Internet. I also firmly believe that the process of building a software product is an iterative cycle of ideation, implementation, improvements, and back to ideation. That cycle is most effective with different perspectives, and that includes engineers' perspective throughout the whole cycle, from ideation.

# The two dimensions of a product

We can think of a product offering in two dimensions:

- Depth: How well do we solve a specific problem space
- Width: How many adjacent problem spaces we solve

Investing in either of those can bring value depending on a specific business.

Solving a specific vertical extraordinarily well has been a prerequisite for the success of some companies. Google's search engine, as its primary customer-centered offering has shot the company to the moon. No one was doing internet-scale search nearly as well as Google did (sorry Bing). As far as search itself was not Google's profit center, it did enable their Marketing business to grow on the search engine's popularity.

A business needs to carefully adjust their appetite. It is tempting to become the one-stop-shop for end-users. The wider the offering you have, the more potential customers.

Under the hood, you may actually start acting like a few small sub-businesses under the hood. A wider offering may dilute the product's value proposition. It may make it harder to steer the Product/Engineering teams as a whole towards a single north-star goal. Multiple internal goals may compete, wasting time and resources. A small scale allows for better focus. At a certain point, it is virtually necessary to grow and expand one's offering. The hardest part is to choose that point appropriately.

For most products, I see that a reasonable path is that of _humble ambition_ - a ground somewhere between the two, adequate to your business's growth stage and risk appetite.

# Humble ambition

I'd expand a tiny bit what I mean with that phrase. In reality, this is about making rational decisions in the scope of your business.

Perhaps you can do those things:

- Reduce the First Contentful Paint time on your homepage by 30%
- Integrate with two more third-party providers, as requested by your largest current client
- Automate a step where your users had to input data manually

But can you do those, too?

- Build and maintain a robust in-app Chat experience
- Build and maintain a Sendgrid-grade in-app Marketing Campaigns experience
- Build a flawless full-text search experience

Perhaps you can. But building a successful product is not just a matter of _Can we?_. It more importantly is a matter of _Should we?_

# Profitability

Every line of code becomes a commitment in one way or another.

- When an engineer extends existing solutions, they need to understand what exists at the time of extension.
  - A lack of complete understanding is the cause of software bugs. Make no mistake: A human's capacity to comprehend a complex system is limited. If a system is easier to understand, it is easier to prevent bugs. When there is more code, the engineer needs to spend more time on validating how their proposed change would impact the system at large. It costs more for every new part to be added to a system.
- Code runs on machines. Machines cost money.

Is it not your goal to make a profitable business? To earn more than you spend (or, in case of non-profits: to provide enough value to offset costs)?

Thus, I believe that software should be periodically revised, and parts of the software which do not offset their costs shall be removed, especially:
- Dead code
- Features adopted by a low number of users
- Features which require beefy machines to execute

# _Good enough_ and adjacent problems

There are different ways to put it. The [Pareto principle](https://en.wikipedia.org/wiki/Pareto_principle) suggests that 20% of effort takes you to 80% of where you want to be. Similarly, 37signals points to the value in looking for a [judo solution](https://37signals.com/podcast/good-enough-is-fine/). Those are obviously generalisations, but here's the thing about good generalisations - they are pretty close to reality.

Building smaller initial offerings is a great way to start. It allows you to ship and learn from the experiment faster. It allows you to see what works and what does not work.

The same approach also works when improving on existing solutions. Especially, the ability to restate the problem that DHH talks about in the 37signals podcast is a superpower. When you think of a problem your users face, a UX flaw, there's a myriad of perspectives to look at that problem. You might actually be able to see a few similar, adjacent problems. You might find such a problem statement that will be a day's worth of work to fix, with 90% of what you aimed for. That is efficiency at its finest.
</div>
