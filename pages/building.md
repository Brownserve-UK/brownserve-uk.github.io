# Building

We use [Invoke-Build](https://github.com/nightroman/Invoke-Build) to run our builds and [Pester](https://github.com/pester/Pester) to perform unit testing.
The build tasks are defined in the build tasks script under `.build/tasks/` and the tests are located in the tests directory under `.build/`.
There's a wrapper build script at `.build/build.ps1` which is the entry point for all builds; this takes care of initializing the build environment and calling the build tasks.

## Builds

The following build tasks are available:

* `Build` - Builds the basic module and copies it to the build output directory. (This is mostly a meta task that other builds can use as a base)
* `BuildAndImport` - Builds the module and imports it into the current session.
* `BuildWithDocs` - Builds the module, imports it into the current session and automatically creates documentation.
* `BuildAndTest` - Performs the same tasks as `BuildWithDocs` but also runs the unit tests
* `BuildTestAndCheck` - Performs the same tasks as `BuildAndTest` but also ensures there are no uncommitted changes resulting from the build (e.g. documentation). This task is used for all pull requests.
* `StageRelease` - Stages a release (see [RELEASING](./releasing.md) for more information)
* `DryRun` - Performs the same actions as `Release` but stops short of actually pushing the module out, mostly useful for debugging (see [RELEASING](./releasing.md) for more information)
* `Release` - Releases the module to the configured repositories (see [RELEASING](./releasing.md) for more information)

The build tasks script has detailed documentation for each task should you wish to know more.

## How to build

To run a build it's as simple as calling `./.build/build.ps1 <task>` from the root of the repository.  
This will first initialize the build environment using the `_init.ps1` script and then call the specified task using `Invoke-Build`.  

When developing we recommended running the `BuildAndImport` task often.
This will build the module and import it into your current session allowing you to easily test your changes manually.

When you're happy with your changes you should run the `BuildWithDocs` task to have the build create the relevant documentation for you. (Don't forget to fill out the missing information!)

When you're ready to test your changes properly you can run the `BuildAndTest` task which will run the unit tests as well as the above tasks.

> To ensure stability of builds the `_init.ps1` will always download and use the latest stable versions of the Brownserve build modules (Brownserve.PSCommon, Brownserve.PSSourceControl, and Brownserve.PSBuildTools). If you want to test how your changes are likely to affect the build process of the module then you can pass the `-UseWorkingCopy` parameter.

## Self-consuming builds

This repository uses itself to build itself: the build scripts call cmdlets from the Brownserve build modules (Brownserve.PSCommon, Brownserve.PSSourceControl, and Brownserve.PSBuildTools), and `_init.ps1` satisfies that dependency by downloading the latest stable published versions from NuGet. Only the PowerShell modules are packaged; the build scripts themselves always run from the local working copy.

### Shipping changes the build scripts depend on

Because the build scripts pull the modules from NuGet, you can't update them to use new cmdlet behaviour until that behaviour is published. The recommended sequence is:

1. Make your module changes and test locally. If the cmdlet you changed is one the build scripts call, pass `-UseWorkingCopy` so the build loads your local copy of the module rather than the stable NuGet version, confirming your changes don't break the build.
2. Cut a new release following [RELEASING](./releasing.md).
3. Open a new branch and update the build scripts to use the new functionality from the published version.

**Deprecate before removing.** If you need to remove or rename a cmdlet or parameter that the build scripts use, deprecate it in one release and remove it in the next. This keeps the build working across the version boundary and gives other consumers time to migrate.

**Pinning as an escape hatch.** If a bad release creates a bootstrapping problem, you can pin the affected module(s) to a known-good version in `paket.dependencies` while you work out a fix.

## Building locally

There are two ways to get all the dependencies you need to build locally, either using the included `.devcontainer` or manually installing them.

### .devcontainer

By far the easiest way to test things locally is to use the included `.devcontainer` within VS Code or other compatible IDE/code editor.

This will spin up an environment with all the prerequisite dependencies installed and relaunch VS Code connected to that container.
This allows you to test things quickly and easily and avoid the potential to pollute your local system.  

This also has the benefit of including the VS Code extensions we use for linting, formatting and spelling so you can ensure your code is compliant with our guidelines.

### Manual installation

If you don't use VSCode or want to build manually for whatever reason you will need to ensure you have:

* [dotnet](https://dotnet.microsoft.com/download) installed and available on your path
* [PowerShell Core](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.1) installed and available on your path
* If on a *nix system you'll also need [mono](https://www.mono-project.com/download/stable/) to be able to run NuGet.exe

## CI/CD

### GitHub Actions

Currently we use GitHub Actions to build and test the module on every PR to the `main` branch.  
The GitHub Actions workflow will run the `BuildTestAndCheck` task which will build the module, run the unit tests and then check for any uncommitted changes.  
This ensures that any changes to the module are thoroughly tested and that the documentation is fully up to date before merging.
There are also two release related workflows which can be manually triggered to stage and release a new version of the module (for more information see [RELEASING](./releasing.md)).
