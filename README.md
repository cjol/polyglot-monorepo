# Pymonorepo

NX is a good tool for managing monorepos. IT's complex to configure, which the community has solved by using generators. This hides the complexity, but swtill makes it difficult to understand what is going on.

In this repo, I'm going to use the generators but try to keep everything documented and as minimal as possible to improve my intuition.

# Steps

```
npx create-nx-workspace pymonorepo --preset=npm
```

This creates an initial nx workspace. It prompts me about distributed caching, which I enabled. It has created:

- a `.nx` hidden directory, which seems to contain gitignored cache. Hoping I can ignore it completely
- a `.vscode` directory with extension recommendations. I've accepted them, partly because I'm hoping that the extension will give me a more discoverable way to use generator parameters
- A classic `.gitignore`
- a nx-flavoured `README` which I've removed in favour of this one
- a minimal package.json containing just `nx` and `@nx/js` as dependencies. It's also set up the npm `workspaces` key to point to the `packages` directory (and a corresponding `package-lock.json`)
- an `nx.json` file which builds ontop of the `npm` preset (which I haven't investigated). It defines two targets which it assumes will be relevant for all projects in this repository: `build` and `lint`. It hasn't specified an executor for either of these, so I guess it doesn't know how to _do_ those things yet. Instead, the `targetDefaults` field specifies that these tasks can have their results cached, and also that the `build` task depends on the `build` task of its dependencies. That key feature is the main reason I'm here!

# Adding the python project

(see commit history)

# Generating a Python Liberary

At this point the tutorials and docs get you `nx generate ...` to create new projects. There are ~20 CLI parameters for the python poetry generator, and it doesn't seem useful to learn about them or juggle between the CLI and the docs. Instead, I've used the VS Code extension by just clicking "generate" in the sidebar, which has then given me a nice form listing all the options (which I guess is generated from a schema provided by the author).

Here's the command it ended up generating for me:

```
@nxlv/python:poetry-project --name=pythonlib --projectType=library --directory=python/libs/pythonlib --codeCoverage=false --codeCoverageHtmlReport=false --codeCoverageXmlReport=false --linter=none --projectNameAndRootFormat=as-provided --pyenvPythonVersion=3.11.2 --pyprojectPythonDependency=">=3.9,<3.12" --unitTestHtmlReport=false --unitTestJUnitReport=false --unitTestRunner=none --no-interactive --dry-run
```

Notable things:

- I've disabled lints, tests and code coverage to keep this minimal. I'll want to learn how to add those myself later.
- I bumped the Python version because I don't have the default version installed.
- I've set the project directory to include a `python/libs` prefix because I want to keep different languages separate in future.

# Generating a Python app

Same as above but with `--projectType=application` and `--directory=python/apps/pythonapp`

# Building the python projects

```
nx build pythonlib
nx build pythonapp
```

This can also be done from the VS Code extension (after refreshing the project tree).

Note that re-running these tasks once does not return a cached result. That's because the first invocation creates a new poetry.lock file, which then busts the cache for the second invocation. The third+ invocations returns a cached result as expected.

# Running python app

I couldn't see a builtin way of actually running our application, so I've added a main file and an executor called `dev` inside the project.json for the pythonapp. It's just running a shell command so nothing fancy.

I can now run the app with `nx run pythonapp:dev`
(or again through the UI). I expect I could run dependent projects by specifying dependencies in the `nx.json` file, but I haven't tried that yet(partly because we have no dependencies yet). This could be useful for e.g. watch-mode compilation of libraries.

# Declaring internal dependencies

The main event: can we set up internal dependencies in such a way that nx automatically runs dependent tasks when we build a project? Nx wraps the dependency management commands to make things easier for us, so I run

```
nx run pythonapp:add --name pythonlib --local
```

This has added the dependency to pythonapp's `pyproject.toml` (and updated the poetry.lock file). I now make some code changes to pythonapp to actually import and use pythonlib, and then run `nx run pythonapp:dev` again -- it works first time, so Poetry has correctly installed the dependency. Similarly, I can now run `nx run pythonapp:build` and it first builds the library and then the app (and then caches the result for the second run).

> > NX Successfully ran target build for project pythonapp and 1 task it depends on (1s)

There is a bit of a problem because VS Code can't cope with the virtualenvs involved here. NX has created a new venv for both the app and the lib, but VS Code doesn't know about them. This means that I get import errors highlighted in my code in the app's mainfile (because it doesn't know where to find pythonlib). I could tell VS Code where to find the pythonapp venv, but then I'd have to change context each time I wanted to work on a different project.

Instead, one option is to tell poetry to use a shared venv for all projects in development. nx will still isolate and bundle deps correctl when bundling for production. Theres a list of pros and cons here: https://github.com/lucasvieirasilva/nx-plugins/blob/main/packages/nx-python/README.md#shared-virtual-environment

For now, I'm swayed by the argument that it's easier to debug and develop when all projects are in the same venv, so I'll try that setup next.

# Shared Venvs

```
npx nx generate @nxlv/python:migrate-to-shared-venv  --pyenvPythonVersion=3.11.2 --pyprojectPythonDependency=">=3.9,<3.12"
```

This seems to have resolved the VS Code import errors (although there's now one about type stubs so I guess the lib build script needs to be updated to include those). I assume that future dependencies will be added as appropriate.

Again I have only overriden the python version parameters.

# TypeScript projects

Speedrun this section by just generating a lib and an app (see commit history). I had to restart VS Code to get the new projects to show up in the sidebar, but they were immediately visible in `nx graph`. Don't be tempted to call a project `tslib` because that will conflict with the basic typescript types library!

I've changed the workspaces entry in the top-level package.json. It seems weird to me that there's no package.json inside the new app. I'm not sure if that's a bug or a feature. My current guess is that it doesn't need one because (a) scripts are replaced by nx (b) as an app, it doesn't need to be published (c) as an app, all deps are effectively dev deps (which are bundled into a release artifact for deployment), and so can be installed at the workspace level (d) local dependencies are resolved via `tsconfig.base.json` (which is generated by nx).

I've added an import from `typescriptlib` in `typescriptapp` and it shows no import errors, so the TS server seems happy. The arrow also shows up in `nx graph`. The first time I tried to run `nx run typescriptapp:serve:dev`, I got an error which I think was because `typescriptlib` hadn't been built yet. I ran `nx build typescriptlib` and then `nx run typescriptapp:serve:dev` again and it worked. I would have expected nx to handle this for me (and it does for `nx run typescriptapp:build`). I've noticed that there's no `dev` command for the lib, so maybe that's the problem.
