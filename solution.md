## Solution for job challenge 2023(a)

Source code for the solution is available at [gushakov/library-clean](https://github.com/gushakov/library-clean) GitHub
repository. It is just one of posible solutions for [job-challenge-2023a](https://github.com/unildev/job-challenge-2023a)
challenge. We are making it publicly available as of April 2024. Special thanks to all the participants who participated 
in the challenge and the discussions which followed.

_Bugfixes_: There were a couple of small bugs (not intended) in the source code of the challenge. We have corrected them
since the initial publication of the challenge. Please, consult the commit annotated with
`bugfix, 29.06.2023, job-challenge-2023(a)` for more details.

Here are the main ideas behind the proposed solution.

### Setting up security

For this challenge we were not trying to integrate any RBAC or ABAC security. For reference, here is an
[article](https://medium.com/unil-ci-software-engineering/securing-use-cases-in-clean-architecture-7f39d07b8ed2) which
describes specifically the way security may be handled within an implementation following Clean Architecture principles.

For this challenge if suffices to simply configure web security via Spring Security Boot starter. The
[configuration](https://github.com/unildev/library-clean/blob/main/src/main/java/com/github/libraryclean/infrastructure/adapter/security/SecurityConfig.java)
is straight forward and simply requires all access to be authenticated.

We set up a simple in-memory user details source with a single patron user using username and password from the
description of the challenge.

Once the security is set up this way, `org.springframework.security.core.Authentication` instance containing the
principal of the logged-in user can be injected into any request handling method of any controller. This way, for
example,
we can get the username of the logged-in user in the "hold book" request handling to be sent to the `HoldBookUseCase`
which requires `Patron`'s ID (username) as a parameter.

### Preparing the sample data

Our implementation follows closely the persistence schema described in this
[article](https://medium.com/unil-ci-software-engineering/hand-crafted-persistence-for-clean-architecture-3fe46cbef531).
In particular, it is very easy to add any sample data which we need for the scenario we are trying to implement. There
is [Flyway script](https://github.com/unildev/library-clean/blob/main/src/main/resources/db/migration/V1.2__Add_sample_data.sql)
which does just that. It will prepare the DB to contain a row for `PatronDbEntity` with the `patron_id`
matching `patron1`
user, and it will add two rows to `catalog` table to represent our sample catalog entries.

### Setting up presentation infrastructure

This is probably the "trickiest" part of the assignment. We need to separate Presenters and Controllers, following
the insights
of [this article](https://medium.com/unil-ci-software-engineering/flow-of-control-in-clean-architecture-23662dfd24f7).

On our blog we mention two working implementations which provide all the technical details of how we can go about such
separation using (there are references to GitHub repositories with full source code in each article):

- [ZK framework](https://medium.com/unil-ci-software-engineering/clean-testable-applications-with-zk-framework-mvvm-and-spring-boot-2d306608d6fd)
- [Spring MVC + Thymeleaf](https://medium.com/unil-ci-software-engineering/revisiting-cargo-tracking-application-using-clean-ddd-4ed16c0e6ae1)

For the solution presented here we focus on Spring MVC integration.
[Here](https://github.com/unildev/library-clean/blob/main/src/main/java/com/github/libraryclean/infrastructure/adapter/web/LocalDispatcherServlet.java)
is the `LocalDispatcherServlet` which allows us to interact with Spring's Thymeleaf presentation logic from our
presenters.

All presenters inherit
from [AbstractWebPresenter](https://github.com/unildev/library-clean/blob/main/src/main/java/com/github/libraryclean/infrastructure/adapter/web/AbstractWebPresenter.java)
which provides methods for constructing `org.springframework.web.servlet.ModelAndView` used by Spring MVC for
presentation
of such and such view.

The actual Thymeleaf templates are very simple since we do not at all care about the visual aspects of the UI.

### Controller

For the challenge we were interested principally in the implementation of a
[controller](https://github.com/unildev/library-clean/blob/main/src/main/java/com/github/libraryclean/infrastructure/adapter/web/hold/HoldBookController.java)
executing "hold book" use case through `com.github.libraryclean.core.usecase.hold.HoldBookInputPort`. The main idea,
of course, is to have a clean separation between `HoldBookController` and `HoldBookPresenter`. In the controller's
method we simply retrieve a prototype bean for `HoldBookUseCase` using its port `HoldBookInputPort` from the application
context. We then execute the use case providing the required parameters: ID of the patron (username of the currently
logged-in user), ISBN of the selected catalog entry, start date of the hold, flag specifying that this is a closed-ended
hold. After this point on it's up to the use case (and `Patron` aggregate) to perform all needed business logic and
execute appropriate presentation logic depending on the resulting outcome (success or error).

### Use case bean

We are setting up
a [configuration](https://github.com/unildev/library-clean/blob/main/src/main/java/com/github/libraryclean/infrastructure/config/UseCaseConfig.java)
for a bean instantiating `HoldBookUseCase` so that it is available in the application context and can be retrieved by
the controller on demand. This allows us to have a singe centralized place to provide all the collaborators needed for
all use cases, as well.

### Other use cases

For a fully working solution, there are other controllers, presenters, and use cases needed. We have implemented these
components in this illustration for completeness and as a reference. For the challenge, this may have been a bit too
much effort, and a simple pseudocode would have sufficed.

### Discussion

As we have certainly mentioned during the interview, the challenge was not meant to test any specific knowledge of any
particular aspect of Java and/or Spring Boot framework. The idea was to provide a hands-on experience with a working
application implemented following the principles of Clean Architecture and DDD, as we have been practicing them at the
university. We were interested in the reaction of a senior Java developer when it comes to development under the
guidelines of Use Case driven approach.

In order to understand how these principles in action, quite a bit of context and preliminary work is needed. That's why
we opted for this format for the challenge: some reading, some research on the existing solutions mentioned in our blog
and the work on the application itself.

We hope that you will find worth investigating further this approach which focuses on the use cases and the model. In
our opinion, it allows for the development which closely follows the real world (thorough use of Ubiquitous Language)
and which is very extensible (addition of new use cases). At the same time, it is very easily tested and comprehensible.

You can easily find more resources on the topic of Clean Architecture and DDD (our
[blog](https://medium.com/unil-ci-software-engineering) is good starting point, there more articles in the archives
section of the blog).

### Notes on running the application

1. start Docker container running `librarydb` Postgres DB
2. make sure the database is empty (no tables from integration tests, previous runs, etc.)
3. run application from the IDE with default profile, main class: `LibraryCleanApplication.java`
4. go to http://localhost:8080
5. login with `patron1` and `secret4p1`
6. try to place a hold on an entry from the catalog
7. try to place two consecutive holds on the same entry (should result in an error)
