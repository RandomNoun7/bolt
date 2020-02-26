# Speed Notes

## Thoughts

Bolt slowness seems caused in large part by consuming resources that are themselves slow.

Examples

- The [Bolt Logger] uses a gem called logger which users little-plugger. Even when loading as little as possible to bypass whatever I can to just return a value from `bolt --version`, a significant amount of time was consumed just on those gems.

- When running `bolt task show` fully 61% of Bolts time was spend processing `puppet_pal.rb` and `puppet.rb`. While it might be possible to defer loading library code like this as long as possible, eventually the libraries do have to get loaded, and in this case, the library that Bolt does not control is itself slow.

- Ethan was already attempting to optimize some of the performance operations that seemingly should not take very long like `bolt --help`. The Pull-Requests below are a part of that effort. As you will see if you open them however, even after significant refactoring, performance of simple tasks like `--help` still did not result in performance that most commandline users would consider normal.

- Further attempts at optimizing Bolts performance would likely require more drastic measures, and perhaps some architectural decisions, to avoid loading absolutely anything that isn't strictly required for the task at hand, especially around what does and does not need to be logged for analytics purposes.

- The biggest performance penalties, and thus the biggest possible wins, both for Bolt performance, and for Puppet as a whole, may in fact be to talk Puppet performance itself.

- Tackling Puppets performance as a whole, with a concerted engineering effort would yield gains that Windows users have been asking for going back quite a long time, and would give Bolt performance gains essentially for free, from the perspective of Bolt.

## Relevant Pull Requests Already Completed

- Ethan:  
  [Pull 900](https://github.com/puppetlabs/bolt/pull/900/files)  
  [Pull 906](https://github.com/puppetlabs/bolt/pull/906/files)

[Bolt Logger]: lib/bolt/logger.rb
