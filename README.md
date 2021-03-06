# Github as a database

This repository explores and documents the idea of using github as a database for an application


## Deviation from the classic architecture of a web application

The classic architecture of a website is composed of a front-end, a back-end and a database. Information flows from the front-end (HTTP client) to the backend (HTTP server) to the database and back to the backend and back to the front-end

![classic web architecture](images/Classic-architecture.svg)

The proposed *github-as-a-database* architecture lets the developers focus on the client-side code of the application allowing to prototype the UI quickly by reducing the feedback loop between the developer and the end-user

In this architecture, the developers write the client-side code, embed a [library](https://www.npmjs.com/package/github-api) interacting with the [Github API](https://developer.github.com/v3/) and that's how data is persisted to be later retrieved

![Github as a database architecture](images/Github-as-a-database-architecture.svg)



## Trade-off

The idea presented here is not meant to be perfect or solve all problems, but presents numerous advantages. The idea is to stay as long as reasonnably possible with these benefits and move from this solution as soon as the downsides outweigh the advantages

### Advantages

- Low-cost prototyping
    - it's possible to iterate over the data model freely
- low-cost hosting ([github pages](https://pages.github.com/))
- zero deployment headache
- lots of features out of the box (versioning, easy-revert, change author, change date, diffs, HTTPS)


### Inconvenients

- No/low write-concurrency
- Identity is a bit hard to manage
- The database is available in a public github repository



## How

1. Start the front-end with HTML and CSS and host on [Github Pages](https://pages.github.com). Hey! Github Pages is actually a [Jekyll](http://jekyllrb.com/) website by default, so you can write content pages in [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) out of the box!
2. If you need to read data, [store it in the repo manually](https://help.github.com/articles/adding-a-file-to-a-repository/) and [fetch](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch) it from JavaScript. It's that simple! This doesn't work well if the file is large (> 5MB ?)
3. Do the data manipulation from JavaScript (no need for SQL queries). JS has a [JSON built-in parser](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Objets_globaux/JSON/parse), but the [npm](http://npmjs.com/) community got you covered if you need a [csv](https://www.npmjs.com/package/d3-dsv#csvParse) or [yaml](https://www.npmjs.com/package/js-yaml) parser. And when you have JavaScript objects and arrays, `Array.prototype.map/filter` and `Object.keys` are your best friends.
4. Write the UI from HTML, CSS and whatever you favorite JS UI library/framework is. React? Vue? Angular? Bouture? Everything works.

Tips:
- During the prototyping phase, it's ok to use CDNs like https://unpkg.com/ or https://cdnjs.com/ or https://rawgit.com/


### Storing data

If you need to store data, the first idea would be to store locally using [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)/IndexedDB (data generated by the user) or [ServiceWorker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) (as a cache of server-side resources)

If you want remote storage, you will need write privileges to a github repo. To do that, you can setup a server, have people [login with github](https://developer.github.com/v3/guides/basics-of-authentication/). The server is only an intermediary between your code and the github identity. blaahh... server...

Another solution is to have every user create a github account and create a [personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/). Depending on what you're building you might need to add the user as a collaborator with write priviledges to the repo you want them to write to.
You also need to create a UI in your application to receive the token and keep it in [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage. One downside here is that each user needs a github account. It can be a annoyance for people who don't already have an account, but it's possible (easy?) to sit down with users in a prototyping phase to reduce the annoyance

Another solution is to create a [machine account](https://developer.github.com/v3/guides/managing-deploy-keys/#machine-users), add it as write-level collaborator to the repo, create a personal access token for this account and add the token to the source code. Don't forget to [protect](https://help.github.com/articles/about-protected-branches/) the main branch from force-push. This way, the token being effectively public cannot cause more harm than it can cause good


#### Where to store the data within Github

Various answers are possible here. One is to let the user store the data under their own account. Another is to have a "centralized database": either the code source repo or a dedicated corresponding "database repo"


#### Handling conflicts gracefully

Git isn't designed to handle concurrency at all, so as soon as you need concurrency because enough users use the application often enough, **move away from github as a database**

When two users try to write at the same time, one will fail to write to the database. This can be handled gracefully either by storing the changes locally (localStorage), retrying automatically a couple time with exponential delay and if failing too much, warning the user to maybe retry manually later. Another option is to create a user-specific branch and let the user work on it and merge manually via git/github and delete the branch (on the client side, if the branch is deleted, the UI goes back to the main branch). In any case, **NEVER LOSE THE USER'S WORK**

Maybe record every such failures as analytics to know when moving away would be a good idea


### Identity/user accounts

If needed to store per user infos, one solution is to leverage github user accounts either by providing a "login with github" (need a server), or have each user create a personal access token, or manage identity externally.

In any case, it's possible to track which user did which change to the database by filling the [`author` field of the commit](https://developer.github.com/v3/git/commits/#optional-parameters) created to update the database


### Private data in plain sight

If prototyping an application which needs privacy from day one, there are solutions to work around the public aspect of the github repo. One idea can be to ask a password on the client-side and use it to encrypt/decrypt the data. The password remains in the client-side ([hashed](https://www.owasp.org/index.php/Cryptographic_Storage_Cheat_Sheet#Architectural_Decision) in localStorage) at all time. The hashed password is used to encrypt/decrypt the content.

/!\ localStorage is per-[origin](https://html.spec.whatwg.org/multipage/origin.html#concept-origin), so it's better to store a hashed password so other scripts on the same origin do not have access to the password (important for `user.github.io` domains)






- identity
- privacy/secrets
- combo with Travis CI
    => Continuous integration/continous deployment
- admin interface
- inspirations : 
    - Prose.io
    - multibao
- dealing with large data


