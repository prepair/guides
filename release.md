Release
=======

Always use semantic release commit messages. See [here](http://karma-runner.github.io/1.0/dev/git-commit-msg.html).

Just a reminder: fix, feat, docs, chore, style, refactor, test. Use (scope), add "BREAKING CHANGE" etc.

----

Setup npm and login with your user.

```bash
npm set init.author.name "Your Name"
npm set init.author.email "you@example.com"
npm set init.author.url "http://yourblog.com"
npm adduser
```

Ask the project lead to do the initial publication (he will do an `npm publish --access=public`)
and then he shall add you to the list of maintainers (he will `npm owner add YOU @prepair/foo`).
This is needed since team management is not available in npm's free plan.

Avoid working in master. Use development, or feature/fix branches. When finished, merge
or rebase into master.

----

Use `npm run release` if it has been set up in _package.json_.

Otherwise do it manually (with the _standard-version_ npm package):

```bash
git checkout master
git pull origin master
standard-version
git push --follow-tags origin master
npm publish --access=public
```

Standard-version generates a CHANGELOG for you and does an `npm version major/minor/patch -m 'message'`.

----

* Connect CI (Travis, CircleCI etc.) - TODO
* WARN - the free CI plans may not have proper team management
