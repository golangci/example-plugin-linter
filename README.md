This is an example linter that can be compiled into a plugin for `golangci-lint`.

### Create the Plugin From This Linter

1. Download the source code
2. From the root project directory, run `go build -buildmode=plugin plugin/example.go`.
3. Copy the generated `example.so` file into your project or to some other known location of your choosing. [^1]

### Create a Copy of `golangci-lint` that Can Run with Plugins

In order to use plugins, you'll need a golangci-lint executable that can run them.

Plugin dependencies defined in the `go.mod` file MUST have a matching version (or hash) as the same dependency in th `golangci-lint` binary if the dependency is used in both.

Because of the high probability of this both using the same dependency, it is recommended to use a locally built binary.

To do so:

1. Download [golangci-lint](https://github.com/golangci/golangci-lint) source code
2. From the projects root directory, run `make build`
3. Copy the `golangci-lint` executable that was created to your path, project, or other location

### Configure Your Project for Linting

If you already have a linter plugin available, you can follow these steps to define its usage in a projects `.golangci.yml` file.

If you're looking for instructions on how to configure your own custom linter, they can be found further down.

1. If the project you want to lint does not have one already, copy the [.golangci.yml](https://github.com/golangci/golangci-lint/blob/master/.golangci.yml) to the root directory.
2. Adjust the YAML to appropriate `linters-settings.custom` entries as so:
    ```yaml
    linters:
      settings:
        custom:
          example:
            path: /example.so
            description: The description of the linter
            original-url: github.com/golangci/example-linter
            settings: # Settings are optional.
              one: Foo
              two:
                - name: Bar
              three:
                name: Bar
    ```

That is all the configuration that is required to run a custom linter in your project.

Custom linters are enabled by default, but abide by the same rules as other linters.

If the disable all option is specified either on command line or in `.golang.yml` files `linters.disable-all: true`, custom linters will be disabled;
they can be re-enabled by adding them to the `linters.enable` list,
or providing the enabled option on the command line, `golangci-lint run -Eexample`.

The configuration inside the `settings` field of linter have some limitations (there are NOT related to the plugin system itself):
we use Viper to handle the configuration but Viper put all the keys in lowercase, and `.` cannot be used inside a key.

### To Create Your Own Plugin

Your linter must provide one or more `golang.org/x/tools/go/analysis.Analyzer` structs.

Your project should also use `go.mod`.

All versions of libraries that overlap `golangci-lint` (including replaced libraries) MUST be set to the same version as `golangci-lint`.
You can see the versions by running `go version -m golangci-lint`.

You'll also need to create a Go file like `plugin/example.go`.

This file MUST be in the package `main`, and MUST define an exposed function called `New` with the following signature:
```go
func New(conf any) ([]*analysis.Analyzer, error) {
	// ...
}
```

See [plugin/example.go](https://github.com/golangci/example-plugin-linter/blob/master/plugin/example.go) for more info.

To build the plugin, from the root project directory, run:
```bash
go build -buildmode=plugin plugin/example.go
```

This will create a plugin `*.so` file that can be copied into your project or another well known location for usage in `golangci-lint`.

[^1]: Alternately, you can use the `-o /path/to/location/example.so` output flag to have it put it there for you.
