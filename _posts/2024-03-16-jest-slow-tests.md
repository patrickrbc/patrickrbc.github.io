---
layout: post
title: Migrating away from Jest and achieving over 90% reduction in running time
date: 2024-03-16 13:42:00 -0300
comments: true
---

Unfortunately, I've been using Jest since 2019. No offense to the Jest team – I'm pretty sure they're awesome and way smarter than I am, but I wish I had made this move earlier in this project.

I'm not talking about UI testing here, so assume everything I'm saying is about server-side JS. If you're happy with Jest, don't switch! I'm not trying to convince anyone. I'm just sharing my experience because I bet there are people out there in the same situation as I was.

The decision to go with Jest wasn't well thought out. At the time, I wasn't heavily involved in software development, so I opted for the well-known, fully-fledged test framework. It seemed promising initially, with its great API.

However, after hundreds of tests, things started to break a lot. Memory leaks began to surface, the number of hacky flags multiplied, and visiting Jest's issues tab became routine.

## The problem

Jests loads [modules over and over again,](https://github.com/jestjs/jest/issues/10550) leading to a memory leak. It is [designed in a way](https://chanind.github.io/javascript/2019/10/12/jest-tests-memory-leak.html) that makes those problems arise [very likely](https://github.com/jestjs/jest/issues/6814), specially in backend projects that are getting big. I'm not going to try to describe the kind of issues here, but you can find them pretty easy [there](https://github.com/search?q=repo%3Ajestjs%2Fjest+memory&type=issues). I'm just going to leave some leads for the poor users that are still fighting this battle.

1. **jest —logHeapUsage**
\
As I've mentioned before, [memory](https://github.com/jaredjj3/jest-memory-leak-demo) [leaks](https://github.com/smolijar/jest-is-a-rude-needy-clown-and-eats-lot-of-memory?tab=readme-ov-file) [will](https://github.com/jestjs/jest/issues/12142) [be](https://github.com/jestjs/jest/issues/7311) [your](https://github.com/jestjs/jest/issues/7874) [best](https://github.com/jestjs/jest/pull/8331) [friend](https://github.com/jestjs/jest/issues/11956), so you'd better watch your heap usage to spot sudden growth.

2. **jest —maxWorkers=50%**
\
Some [benchmarks](https://ivantanev.com/make-jest-faster/) show that running with this configuration might make your tests run up to 20% faster. However, you have to test it in your project; some folks said it made things worse.

3. **jest —runInBand**
\
This command runs all tests serially in the current process instead of creating a worker pool of child processes. They say it's useful for debugging, but oddly enough, some people reported that it can actually improve performance.

4. **jest —changedSince**
\
If you run a workflow for each PR update, your tests will run a LOT. So, if you have limited compute minutes or a constrained CI pipeline, this flag can significantly reduce the time your PR workflows take.

5. **jest-slow-test-reporter**
\
You can use this reporter to view which tests are the slowest in your project and start tackling them first.

6. **Exposing Node.js gargabe collector**
\
In some cases, running Node with the flag **`--expose-gc`** seems to handle memory leaks better.

### Not good enough

Some of these strategies significantly reduced the running time during this period. However, the process of learning and implementing them came at the expense of delivery time, which is ultimately more critical.

The tests were so sluggish that I resorted to running them only on the module we were currently developing, then solely on the changed modules in the PR, and finally, all tests were run only when merging to the main branch. Unfortunately, this approach resulted in delays in identifying bugs.

The tests were so time-consuming that I found myself hesitating to write tests for certain features, worrying about the additional build process time they would cause. At this point, I realized it was time to make the switch.

## Switching to Mocha

I used Mocha a decade ago and it was awesome. So, I thought transitioning back to it would be smooth sailing. Over the past years, I've been seeing [people](https://github.com/jestjs/jest/issues/7832#issuecomment-1053545855) [throwing](https://github.com/jestjs/jest/issues/7963#issuecomment-802129978) [out](https://github.com/jestjs/jest/issues/7631#issuecomment-496335097) the idea of switching from Jest to Mocha, and always I found it [funny](https://github.com/jestjs/jest/issues/9980#issuecomment-1808310819). I remember there were a lot of guides and people talking about migrating from Mocha to Jest. Like me, most of people would assume the newer tool would have a better or at least similar performance.

The migration was much easier, than expected. A couple of replace cases and and less than an hour refactoring some code. The tricker part was the mock engine, which isn't included in Mocha.

I could have used [Sinon.js](https://sinonjs.org/) for that, but I really like the idea of one day not relying on any testing libraries. I even considered using only the new Node.js built-in test runner, but it's not quite there yet for me. So, I decided to stick with just the built-in [MockTracker](https://nodejs.org/api/test.html#class-mocktracker).

Trying things out was mind-blowing. Single tests that were taking 3 seconds with Jest would run in less than 200ms with Mocha. It shouldn't be a surprise – what I was running shouldn't take so long, but I had gotten used to that slowness. In the end, our test running time reduced from more than 12 minutes to less than 40 seconds.

The speed of Mocha helped us uncover hidden bugs that would occasionally fail tests because they only occurred under very specific conditions – conditions that were unlikely to be met in Jest due to its slower performance.

## Conclusion

I still use Jest on some smaller repositories that I maintain, and I'm not mad enough to migrate them until they become a problem. However, for future projects, I'll definitely opt for Mocha or the Node.js test runner.

The thing is, even if there's a way to optimize Jest and run thousands of tests in a reasonable time, there's something wrong when simply switching the testing framework results in significantly faster performance. Do you agree? Do you have any similar experiences? I'd love to hear about them.
