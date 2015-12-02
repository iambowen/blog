---
layout: post
title: Invalid gemspec because of the date format in specification
tag: ruby
categories: ["ruby"]
---

In the morning, i was writing a automation sql script for QA. And after move the source file from mac mini to my machine. It met such warning as i tried to run the file.

```bash
Invalid gemspec in [/Users/twer/.rvm/gems/ree-1.8.7-2011.03@development/specifications/childprocess-0.2.1.gemspec]: invalid date format in specification: "2011-08-11 00:00:00.000000000Z"
```

It's ok because it's just warning. But i am just a little bit agnoied by the message. So i decided to kill this warning message. After search in google, i found the [answer](http://stackoverflow.com/questions/5771758/invalid-gemspec-because-of-the-date-format-in-specification).


And the steps should be:

```bash
/Users/twer/.rvm/gems/ree-1.8.7-2011.03@develoment/specifications/childprocess-0.2.1.gemspec
```

- Find the spec that is causing the problem, search it in vi with string "2011" and i found that line.
- Change `s.date = %q{2011-08-11 00:00:00.000000000Z}` to `s.date = %q{2011-08-11}`



Cool, done, no more bothering information.
