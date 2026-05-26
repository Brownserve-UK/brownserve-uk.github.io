# Style Guidelines

## Writing cmdlets

### Don't construct paths manually

Don't form paths by passing in separators (e.g. `C:\`, `/usr/`), use the tools PowerShell gives you:

- [`Resolve-Path`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/resolve-path?view=powershell-7.1) can be used to validate user submitted paths or resolve the path to commands/aliases.
- [`Join-Path`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/join-path?view=powershell-7.1) can be used to construct paths (top tip: you can specify multiple values in `-AdditionalChildPaths`)
- [`Convert-Path`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/convert-path?view=powershell-7.1) converts a `PSPath` to the standard path format for your OS, which avoids issues with environment variables.

All these cmdlets handle path separators automatically and will ensure that the path is valid for the current operating system, giving your code a much higher chance of working cross-platform.

### OS specific cmdlets

By default we treat all cmdlets as cross-platform. If your code will only work on certain operating systems you should call [`Test-OperatingSystem`](https://docs.brownserve.co.uk/PSCommon/Cmdlet%20reference/Test-OperatingSystem/) at the beginning of your cmdlet with the supported OSes as the parameter. An exception will be raised if the current OS is not in the supported list.

**Example:**  
In this example we're running on a Linux based operating system so the cmdlet will run successfully:

```powershell
> $isLinux
> $True
> Test-OperatingSystem 'Linux','macOS' -Verbose
> VERBOSE: This cmdlet is supported on Linux
```

Moving over to a Windows based operating system we can see that the cmdlet will now throw an exception:

```powershell
> $isWindows
> $True
> Test-OperatingSystem 'Linux','macOS' -Verbose
> Exception: This cmdlet is not compatible with Windows
```

### Use `Begin`, `Process` and `End` blocks

Even if your cmdlet doesn't support pipeline input you should still use the `Begin`, `Process` and `End` blocks. This allows us to easily add pipeline support in the future and keeps the code consistent.

### Define parameter properties

When defining parameters you should always specify the parameter properties, including `Mandatory`, `Position` and `ValueFromPipelineByPropertyName` where applicable.

### Define parameter types

Always define a type for each parameter. This allows PowerShell to validate input and provide tab completion.

### Ensure parameter names are consistent

Parameter names should be consistent across all cmdlets. For example if one cmdlet uses `Token` then all cmdlets should use `Token`, not `PAT` or `AccessToken`.

## Writing documentation

### All `Public` cmdlets must have documentation

All `Public` cmdlets **must** have documentation, and all documentation **must** be complete (CI/CD will fail if it isn't).
Running any of the build tasks will automatically generate the documentation for you, however some sections will be missing and will need to be completed manually. These will be surrounded by `{{ }}` to make them easy to find (e.g. `{{ Fill in Synopsis }}`).
The `BuildAndTest` task includes tests that check for incomplete sections and will highlight them in the build output.

### Keep the synopsis short

The synopsis should be a concise description of what the cmdlet does, ideally a single line.

### Documentation must use LF line endings

Unfortunately for files created by PowerShell (such as our documentation) the line endings will always be set to the line endings of the operating system that was in use when they were generated (see <https://github.com/PowerShell/PowerShell/issues/2872>).
Therefore to ensure the line endings stay consistent (so we can avoid a messy git history) we require that all documentation is generated with LF line endings.
To make this easier we include an [editorconfig](https://editorconfig.org/) file in the repository which will automatically set the line endings to LF for those files.
For those not using an editor that supports editorconfig then the build tasks should automatically convert the line endings for you.
