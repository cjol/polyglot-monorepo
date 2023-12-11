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
