# Calculate Code Complexity and Churn

To figure out which files deserve refactoring the most we use the approach described
by Michael Feathers in [Getting Empirical about Refactoring](https://www.stickyminds.com/article/getting-empirical-about-refactoring). The idea is that the code that is _complex_ (structurally) and changes frequently is harder to maintain and thus should be refactored.

We use [`attractor`](https://github.com/julianrubisch/attractor) gem to calculate these metrics.

## Usage

* Generate a plain text report for the directory:

```sh
$ bundle exec attractor calc -p app/models

Calculated churn and complexity

file_path                                                     complexity   churn
--------------------------------------------------------------------------------
app/models/deal.rb                                                 925.4     946
app/models/user.rb                                                 858.5     484
...
```

* Generate an HTML report:

```sh
$ bundle exec  attractor report -p app/controllers
Generating an HTML report
Generated HTML report at attractor_output/index.html
```
