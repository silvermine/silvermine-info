# Silvermine Organization

This organization is comprised of a small group of dedicated developers who
work together on a variety of projects. Our main development objective is to
produce high-quality products for our users. Most of the projects released by
this organization are released as a by-product of that work. That is, these
projects are not the primary output of our organization, but are used within
the products that are our primary objective.

Still, we use a great deal of open-source software in our work and feel that
it's important to contribute back to the community as much as we can. Thus, we
open-source many of our general-use libraries and codebases in an attempt to
help others. Doing so also helps us to maintain our focus on high-quality,
granular codebases.

We hope you find these codebases useful, either to use directly in your own
software, or to serve as a template that you use for writing your own code to
solve a similar problem.


## Contributing

We appreciate and welcome external contributions. Pull requests are welcome,
especially for bug fixes. If you are suggesting a new feature, it's often best
to open the feature idea as an issue first so that we can have a discussion of
the idea so that when the code is implemented everyone is already on the same
page as to what, exactly, is needed. The discussion process also generally
helps the review process go quicker.

Please note that we hold all external contributions to the same high standard
that we hold ourselves to. Thus, all pull requests submitted to our
organization must go through a thorough code review process by one of our core
developers, and all review comments must be resolved before the code can be
merged.

**Important:** We have gone to great lengths to codify as many of our coding
standards into linting tool configurations as possible and will continue to do
so (for example, see [eslint-config-silvermine][eslint-config] and
[eslint-plugin-silvermine][eslint-plugin]). For the coding standards that can
not be checked programmatically, please make sure you are familiar with our
[coding standards](coding-standards.md). We are particularly nitpicky about
commit messages and commit history, so be sure you read the dedicated page on
[commit history](commit-history.md).


### Automated Testing

Additionally, please note the following about automated testing:

Our goal is 100% unit test coverage, with **good and effective** tests (it's
easy to hit 100% coverage with junk tests, so we differentiate). We **will not
accept pull requests for new features that do not include unit tests**. If you
are submitting a pull request to fix a bug, we may accept it without unit tests
(we will certainly write our own for that bug), but we *strongly encourage* you
to write a unit test exhibiting the bug, commit that, and then commit a fix in
a separate commit. This *greatly increases* the likelihood that we will accept
your pull request and the speed with which we can process it. Simply put, unit
tests are the best way to communicate to a group of developers exactly what a
bug is, and to guarantee that the bug does not happen again in the future.
Also, when we open up a pull request and see that someone put effort into
writing unit tests for it, we automatically feel better about that pull request
and just naturally try harder to get the PR reviewed and merged sooner.

[eslint-config]: https://github.com/silvermine/eslint-config-silvermine
[eslint-plugin]: https://github.com/silvermine/eslint-plugin-silvermine
