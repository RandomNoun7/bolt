# Speed Notes

## Thoughts

Bolt slowness seems caused in large part by consuming resources that are themselves slow.

Examples

- The [Bolt Logger][1] uses a gem called logger which users little-plugger. Even when loading as little as possible to bypass whatever I can to just return a value from `bolt --version`, a significant amount of time was consumed just on those gems.

- When running `bolt task show` fully 61% of Bolts time was spend processing `puppet_pal.rb` and `puppet.rb`. While it might be possible to defer loading library code like this as long as possible, eventually the libraries do have to get loaded, and in this case, the library that Bolt does not control is itself slow.

- Ethan was already attempting to optimize some of the performance operations that seemingly should not take very long like `bolt --help`. The Pull-Requests below are a part of that effort. As you will see if you open them however, even after significant refactoring, performance of simple tasks like `--help` still did not result in performance that most commandline users would consider normal.

- Further attempts at optimizing Bolts performance would likely require more drastic measures, and perhaps some architectural decisions, to avoid loading absolutely anything that isn't strictly required for the task at hand, especially around what does and does not need to be logged for analytics purposes.

- The biggest performance penalties, and thus the biggest possible wins, both for Bolt performance, and for Puppet as a whole, may in fact be to talk Puppet performance itself.

- Tackling Puppets performance as a whole, with a concerted engineering effort would yield gains that Windows users have been asking for going back quite a long time, and would give Bolt performance gains essentially for free, from the perspective of Bolt.

## Relevant Pull Requests Already Completed

- Ethan:  
  [Pull 900](https://github.com/puppetlabs/bolt/pull/900/files)  
  [Pull 906](https://github.com/puppetlabs/bolt/pull/906/files)

## Adventures in avoiding loading things to complete --version

This is where I will note all of the difficulties I run into while attempting to avoid loading things so that `--version` can run quickly without having to load `puppet.rb`.

- The [Bolt Options Parser][2] references a constant called `TRANSPORTS` twice, on lines [608][3] and [730-731][4]. This constant is used (in this context) to get the current list of all available transports. This constant is defined in [lib/bolt/config.rb][5]. The problem is that this is not a simple string list of available transports. It seems that in order to get the list of available transports, all of the transports are in fact loaded. So just to load enough of the commandline to be able to accept commandline commands, all of the available transports are loaded into the run, even though none of them are going to be used yet.

- Commenting out every requires statement I can find and only adding back the bare minimum requires I need to make `--version` return a value still results in loading `puppet/pal` and `puppet` gems.

## Packed vs Source Code Runs

When running the packed and installed version of Bolt to get `bolt --version`, the result is 91,500 file open events in about 5 seconds on my machine. If I switch into the source code directory however and invoke `bundle exec bolt --version` I get only 24,231 file open events for about 1.9 seconds. This is opposite to my expectation where I would have expected that running via bundler would result in more files opened and slower execution time. It seems there may be something about the way Bolt is packaged and distributed that is slowing down execution time.

<!-- Links -->

[1]: lib/bolt/logger.rb
[2]: https://github.com/puppetlabs/bolt/blob/master/lib/bolt/bolt_option_parser.rb
[3]: https://github.com/puppetlabs/bolt/blob/master/lib/bolt/bolt_option_parser.rb#L608
[4]: https://github.com/puppetlabs/bolt/blob/master/lib/bolt/bolt_option_parser.rb#L730-L731
[5]: lib\bolt\config.rb
