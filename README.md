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
