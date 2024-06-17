<div style="text-align: justify">

# What you'll find here

I believe that "Culture eats strategy for breakfast". A strong culture can offset the deficiencies in strategy or lack of company processes.

This page lists some of the concepts that I think are very important to be established in a successful engineering culture. Implementation details are included to better describe what I have in mind by a particular _fancy word_. I am cognisant that other implementations of said concepts could work equally well.

# Ownership

A product shipped is not a product that can be left to itself. Well, it likely can for some time, but not forever.

As an Engineering team is scaled, it is necessary to establish ownership. I like [Alex Ewerlöf's take on ownership](https://blog.alexewerlof.com/p/broken-ownership), where the ideal is defined as:
- Knowledge
- Responsibility
- Mandate

It is useful to establish ownership around:
- All system features. One of the first improvements I did at my current and previous job was to tag application entrypoints (http handlers, background jobs) with code owners, so that runtime errors are reported to respective owners. It's also helpful to tag APM traces or application/database logs with code owners.
- Agreed-upon SLAs. Only the people who have knowledge, responsibility and mandate should be alerted about SLA breaches. There is no point in receiving an alert if you cannot do anything about it.
- Engineering tools, practices and processes. Having a Directly Responsible Individual helps to avoid decision paralysis.

# Treat uptime and speed like product features

At a product's early stage, those may be taken for granted. There isn't a lot of data, a high throughput, a lot of code written. Things just work.

But eventually, they stop *just working*. The database is on fire, endpoints are slow to respond, users are frustrated, and you hardly know what to do. You don't have observability to triage, the personnel is not trained to handle incidents, there are no playbooks, failures occur in 5-year-old solutions written by some folks who are no longer with the company.

There is cost associated with uptime and performance.
- Observability setup, personnel training, on-call duty compensation
- Definition of Service Level Indicators and Objectives/Agreements, to trigger automated alerts
- Capacity allocated for preparing playbooks, improving the performance of existing solutions in light of higher throughput and larger data volumes

No software scales till the end of the universe. There are limits, which can be changed with applying architectural changes. But it costs - engineering time and compute resources.

It is beneficial to consciously state what you aim for in terms of reliability. This statement can (and should!) change as the product evolves and is adopted by users. The Google SRE concept of [error budgets](https://sre.google/sre-book/embracing-risk/#id-na2u1S2SKi1) explains that concept well.

# Testing

The world of software is all too volatile. Code executes on hardware, most calculations happen in volatile memory, machines communicate over the network... all of those components (and more) may be a root cause of issues. It is virtually impossible to be perfectly certain of how a block of code will behave under all potential circumstances - there's simply too many of them to consider. There are types of issues which are not easily reproducible and start to become more prevalent at a certain scale (hello, concurrency problems).

1. Write automated tests on a module's public API level. Too long have I lived under a rock and written tests that break when a small detail in a class definition changes. Uncle Bob helped me to understand my folly with his article on [test contravariance](https://blog.cleancoder.com/uncle-bob/2017/10/03/TestContravariance.html). I live and breathe that now, and higher level, behaviour tests are my starting point for automated tests. I still see merit in lower-level tests that may break easier on code change, but those are reserved for critical elements of the software puzzle.
2. Get a good number of life-saving end-to-end tests. Tag tests according to area, so that they can be executed selectively, depending on the modules which changed in the shipped diff. A CI/CD process which can finish in a few minutes is a great productivity boost.
3. Production is your real test environment. Ship often, in small increments, with an option for a quick roll-back when necessary. To quote a few reasons for this:
    - You won't get the same data quality or quantity in your test environment. You will also not get the same data quality in an anonymized production dump.
    - You will not replicate production traffic on a feature before it hits production.
    - Your skilled 10-person engineering team will not find as many ways to fiddle around with your production application's state as an army of end-users.

# Minimizing interruptions

There's a popular wording of _being in the zone_. One take to describe that could be _a state of relaxed focus_. To say simply - a very productive time. The biggest enemy of _the zone_ is interruption. Interruption comes in many forms.

- Internal requests: some teams may be required to answer questions from other company members. This is often the case when technical support is engaged and a deeper debugging session is needed. Or if we talk about a _platform team_ (or any product enablement team, really), that may just be their daily bread. It's helpful to establish a rotational duty in such teams, so that there is one person focused on going through support requests, and may be doing other low-intensity work in the meantime, which won't be signifiacntly hurt by frequent interruptions. This also pushes folks who wouldn't otherwise do it, to step outside their comfort zone and better understand the system they're responsible for.
- Long wait times: this can be long build times (there's a famous [xkcd comic](https://xkcd.com/303/) about this), long testing pipelines, or long wait time for feedback from peers. A human can only efficiently think of a limited number of topics. If those topics are _easy_ for the particular human - great (hence the proposition to handle low-intensity work during a rotational duty in the previous point). If the topics are complex, [the tax for context switching is enormous](https://www.atlassian.com/blog/productivity/context-switching). Fast feedback cycles allow for quick completion of units of work.
- Remove meetings which do not have clear goals and minimize gaps between valuable meetings (perhaps you'll be left with no gaps after removing the unnecessary meetings). Also, if a meeting can be an async discussion - make it so. By the power of word written, it becomes easier to refer to it later.

# Automating the world

There's a great opening article in the Google SRE book on [toil](https://sre.google/workbook/eliminating-toil/). Automating manual tasks is a guaranteed productivity booster. Just make sure you automate the right things; if you spend 5 minutes per quarter on a manual action, spending 2 weeks to automate doesn't sound like a good investment.

Even though many may understand toil as being primarily related to operational activities, there's more to it. I'll give a few examples:
- You just had an incident because of a misuse of a volatile, internal API. You can fix this with an automation to request code review from a group of subject matter experts when it is used. Take a look at [Danger](https://danger.systems/).
- Engineers sometimes forget to update the Changelog, which is displayed on your product page, and you don't want your customers to miss the cool updates. [Danger](https://danger.systems/) is here to help again.
- You migrate a frequently used component to a newer implementation. While the old implementation is still there to support old use cases, you want to make engineers use the new one instead. Look for the pattern described [here](https://sirupsen.com/shitlists).

If the language you write in supports reflection/metaprogramming, you might be able to automate some checks by validating the state of a fully-loaded application at runtime. For example, you could easily check that all HTTP Handlers descend from a specific parent.

# On the practice of blitzscaling

When the time comes to scale engineering headcount, some companies choose to do it very fast. They may triple headcount within half a year. Then triple that again in the same time.

I do not endorse this. I believe that it is possible to perform successfully, but very hard. Specifically, trying to maintain a similar culture to what you had before scaling might prove impossible. Onboarding engineers into a living culture presupposes there are others who practice that culture. If you onboard a new team comprised only of new hires, chances are they will find it harder to _speak the same language_ as the previous hires. On the other hand, if you take it slower and onboard a new hire to an existing team every a few months, and then set up a new team with a portion of the new and previous hires, you are more likely to maintain the culture.

For a successful implementation of this, I imagine that previous engineers would mostly need to be busy with onboarding new hires, regular 1:1s - essentially, "sharing the culture". There would be little space for the experienced folks to directly work on the codebase.

</div>
