# versadep

`versadep` is a versatile application dependency manager. It is a tool like `bundler` rather than `gem` in Ruby, but for any sets of files.

## Rationale

Your shell scripts, [variant commands](https://github.com/mumoshu/variant), [helmfiles](https://github.com/roboll/helmfile) depends on various executables and other files to accomplish their jobs, from libraries, templates, external commands, and so on.

We tend to use `Dockerfile` to describe the dependencies and distribute container images containing the apps and deps via container registries so that the jobs are reproducible.

The downside of `Dockerfile` is that it is unable to run natively on non-linux machines, it isn't declarative and build instructions in Dockerfiles are not reusable.

`versadep` strievs to solve this problem by filling the gap between your runtime environments, including machines and containers.

That is, `versadep` fetches files and executables in a versioned manner, and generates any configuration files from templates, and it works on operating systems like macOS, Linux and probably Windows.

Now you can either run your application natively on any runtime environment, including host OSes, guest OSes, or on containers, in a consistent manner. It is just manner of running `versadep` to prepare the deps on the runtime environment of your choice and run your application as usual.

## Uage

Place a `versadep.yaml`:

```console
files:
- name: commons
  version: 0.5.0
  repository: s3://examplecom-versapack-ap-northeast-1/

executables:
- name: github.com/rogpeppe/gohack
  version: v1.0.0

builds:
- source: .genrun/.envrc.gotmpl
  target: .envrc
  datasources:
  - name: cities
    url: env:///CITIES?type=application/yaml
  - name: weather
    url: https://wttr.in/?0
    header: "weather=User-Agent: curl"
- source: .genrun/helmfile.yaml.lua
  target: helmfile.yaml
```

Resolve and fetch all dependencies by running:

```console
$ versadep ensure
```

Add paths to necessary executables fetched by `versadep`:

```console
$ eval $(versdep env -)
```

And you can run your application as usual.

You can even stream-line this process by using `exec` - it runs `ensure`, `env`, and then your application:

```console
$ versadep exec -- helmfile sync
```

## Features

### Shebang support

Tired of typing `versadep exec` many times? Make your `versadep` configuration an executable by adding a shebang to `versadep exec` and a default command:

```yaml
#!/usr/bin/env versadep exec

command:
- helmfile
```

so that your `versadep.yaml` looks like:

```yaml
#!/usr/bin/env versadep exec

command:
- helmfile

files:
- name: commons
  version: 0.5.0
  repository: s3://examplecom-versapack-ap-northeast-1/

executables:
- name: github.com/rogpeppe/gohack
  version: v1.0.0

templates:
- source: .genrun/.envrc.gotmpl
  target: .envrc
  datasources:
  - name: cities
    url: env:///CITIES?type=application/yaml
  - name: weather
    url: https://wttr.in/?0
    header: "weather=User-Agent: curl"
- source: .genrun/helmfile.yaml.lua
  target: helmfile.yaml
```

Now rename the config file as you like, and set the executable bit on the config file:

```console
$ mv versadep.yaml helmfile
$ chmod +x helmfile
```

And run it:

```console
$ ./helmfile sync
```

Voila! Your command `helmfile sync` is now executed in the runtime environment filled with all the necessary files and executables prepared by `versadep`.

## Developing

### Architecture

Under the hood, `versadep` delegates its jobs to the following tools:

- [sing](https://github.com/mumoshu/sing) for managing single-command executable dependencies
- [genrun](https://github.com/mumoshu/genrun) to dynamically generate configuration file(s) required to run your application
- [versapack](https://github.com/mumoshu/versapack) for fetching any files from versioned sources

## Acknowledgements

This project has been insipired by awesome projects below! Thanks for all the maintains of these projects.

- davidovich's [summon](https://github.com/davidovich/summon)
- geodesic's [deps](https://github.com/cloudposse/geodesic/pull/466)
