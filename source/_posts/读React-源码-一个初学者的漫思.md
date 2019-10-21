---
title: '读React 源码:一个初学者的漫思'
catalog: true
date: 2019-10-15 18:31:25
subtitle: 
header-img:
tags:
- 翻译
---

未完待续

# The React Source Code: a Beginner’s Walkthrough I 读React 源码:一个初学者的漫思

Ever been stuck in a video game? Do you remember reaching for that strategy guide while battling Shadow Link in the Water Temple? In the spirit of a video game walkthrough, this is a journey through the React Core source code so that beginners may achieve a deeper understanding of the tools they use. Many developers understand the what and how but missing the depth that conveys the why of React.

Furthermore, we write better code by reading good code. I know firsthand how intimidating the board-ranging scope of the React source can be. It is my hope that by building a line-by-line walkthrough of the React core source, its patterns and nuances will emerge to beginner developers. This knowledge should lead to a better understanding of the code developers write, encourage informated contributions to the community, and ultimately demystify open source projects.

I will try to make no assumptions about the reader, other than a familiarity with a JavaScript and experience with React. If there are any concepts that need further clarification, let me know and I will try to elaborate. Also, this is as much a journey for me as it will (hopefully) be for many of you, so I am certain to confound some important facts or concepts. Please feel free to kindly correct any of those misunderstandings and I will happy to edit this or any future pieces.

## what is React?

This section wasn't originally included, but is the result of a conversation with my dad. When he heard I was writing a article about React core, he said,'What's that? You are writing a reactor? Does the government know about that?'

We laugth but the truth is, we as a comunity take certain high-level assumptions as granted. So here is it- for all dads in the world who wandered what their sons and daughters are up to: React is a popular web framework(i.e. used to build web applications) that was open sourced by Facebook. Unfornately it was nothing to do with nuclear fusion or submation.

## Into the code

### package.json

When I first jump into a libary I begin at the package.json, which is often located at the root dictnary. The package.json's main filelds points to the project's entry point and is where your main module's exports object is returned. In other words, when a module is 'required in', this is the point where the API is imported from. If we look for the main field in the  package.json located at React's root, however, we find the main field is strangely absent. We can gain insight as to why this is case from the React Codebase overview, here. There are two important things to note from this source. The first is the React repository is a monprepo(i.e. React and all its related projects are kept in a signal repository), the second is Cool, the `packages` dictnary might be a good place to explore next. Before moving on, however, you maybe wondering why `package.json` exits at the root, if it doesn't actionly provide a entry point. One subtle clue to this question lies in the `package.json`'s dependnces. The only dependecies that the package relies on are DevDependcias(in other words, on direct dependencies). Furthermore, the npm scripts in the scripts fileds are all development scripts, giving access to the useful CLI commonds, like `npm run build`,`npm run flow`,etc. Therefore, it is resonable to assume this `package.json` exists to set up the development environment.

Let's now move on to the `packages` directory at the root. We are interested in the core React module for now so navigate to `react/packages/react/package.json`. You could notice that this `package.json` contains a main field that points to the `index.js` as well as a few dependencies. If we navigate to `react.js` we may be surprised to somethings looks like this:

```javascript
'use strict';

module.exports = require('./lib/React');
```

That's right-- only three lines of code, one of which is `'use strict';`,the other appears to be exporting a non-existent file in a non-existent directory. Once again, the resone for this mysterious file are answered by the Codebase Overview:
> When we compile React for npm, a script copies all the modules to a flatten directory called `lib` and prepands all `require()` paths with `./`

## A Brief Note On Haste

Navigating to [https://unpkg.com/react@15.4.1/lib](https://unpkg.com/react@15.4.1/lib), we can see that there is indeed a file named `React.js`,but from where did this file originate? The answer lies in Facebook's custom module system,*Haste*. *Haste* was developed to be convenients for projects with large codebases, so that files can be moved to different locations within a project without having to worry about specifying new relative file paths. The key distinction between *Haste* and *Common.js*, however, is that all filenames within a *Haste* project must be unique. When a new file is created in Facebook project, Facebook requires that file incloud  a linsinc header. Within that header appears a line like `* @provideModule React` where the name following `@provideModule` should match the name of newly created file. So, in this case `@provideModule React` imports the React Module. This is how modules are created in `Haste`.

## React.js

Having global filenames makes finding file incredebly easy on Github and most text editors. Press `t` anywhere in React's repository, we can type *react.js* to search for the core React Module. Indeed, we find a file named *react.js* located in `src/isomorphic` as the third hit from the top.

Navigating to `src/isomorphic/React.js` we finally see a javascript file that's more than three lines of code!

```javascript
/**
 * Copyright (c) 2013-present, Facebook, Inc.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @providesModule React
 */
```
`Lines 1-10`:As I mentioned before, each source file in React requires a linsenc header. This lines contain infomation about license and are used by *Haste* to uniquily identify the module.

---

```javascript
var ReactBaseClasses = require('ReactBaseClasses');
var ReactChildren = require('ReactChildren');
var ReactDOMFactories = require('ReactDOMFactories');
var ReactElement = require('ReactElement');
var ReactPropTypes = require('ReactPropTypes');
var ReactVersion = require('ReactVersion');
```

`Line 14-21`: These line leverage *Haste* to import `React.js`'s module dependencies. The module required in here will ultimately help comprise React's top-level public API. We will dive into their code in a bit.

```javascript
var createReactClass = require('createClass');
var onlyChild = require('onlyChild');
```

`Line 23-24`: These lines requires in two more 