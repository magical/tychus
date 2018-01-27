tychus
========

`tychus` is a command line utility to live-reload your application. `tychus`
will watch your filesystem for changes and automatically recompile and restart
code on change.

`tychus` is language agnostic - it can be configured to work with just about
anything: Go, Rust, Ruby, Python, etc. Should you desire you can use `tychus`
as a proxy to your application. The proxy is pretty smart - it will pause
requests while your app rebuilds and won't let you run into that super annoying
case where you refresh your webpage after your app boots, but before it can
serve a request.


## Installation

### Homebrew on macOS
Coming soon...

### With Go
Assuming you have a working Go environment and `GOPATH/bin` is in your `PATH`

```
go get github.com/devlocker/tychus
```

## Getting Started
You will need to create a `.tychus.yml` file configuration file. Easiest way is
to generate one with:

```
$ tychus init
```

## Usage

```
tychus run
```

Want to pass additional arguments?

```
tychus run bundle exec ruby myapp.rb
```

Need to pass flags? The following are equivalent:

```
# Yep, that will just run "ls -al" on any file change with the build step
# disabled. You can really run anything you want.
tychus run "ls -al"
tychus run ls --with=-al
tychus run ls -w -al
```

## Configuration

```yaml
# Settings for the file watcher
watch:
  # List of extentions to watch. A change to a file with one of these extensions
  # will trigger a fresh of your application.
  extensions:
  - .go
  # List of folders to not watch.
  ignore:
  - node_modules
  - tmp
  - log
  - vendor

# Build settings.
build:
  # Disable this if you don't have a compile step (Ruby, Python, etc.).
  enabled: false
  # Command to run to rebuild your binary. Tychus will automatically tack on a
  # -o bin_name so it can be omitted.
  build_command: go build -i
  # Name of binary that gets built.
  bin_name: tychus-bin
  # Where to put your built binary.
  target_path: tmp

# Proxy settings.
proxy:
  # If not enabled, proxy will not start.
  enabled: true
  # Port your application runs on. NOTE: a PORT environment will take overwrite
  # whatever you put here.
  app_port: 3000
  # Port to run the proxy on.
  proxy_port: 4000
  # In seconds, how long the proxy will attempt to proxy a request until it
  # gives up and returns a 502.
  timeout: 10
```

### Sample Configuration for a Sinatra Application

```yaml
watch:
  extensions:
  - .rb
  - .erb
  ignore:
  - node_modules
  - tmp
  - log
  - vendor
build:
  enabled: false
  build_command: ""
  bin_name: ""
  target_path: tmp/
proxy:
  enabled: true
  app_port: 4567
  proxy_port: 4000
  timeout: 10
```

```
tychus run bundle exec ruby myapp.rb

# Or in a Procfile (for use with foreman)
# Foreman will set a PORT env variable, which will automatically get picked up
# by tychus. No need to modify your config file.
web: tychus run bundle exec ruby myapp.rb
```

### Sample Configuration for a Rust Application

```yaml
watch:
  extensions:
  - .rs
  ignore:
  - node_modules
  - tmp
  - log
  - vendor
build:
  enabled: true
  build_command: rustc hello.rs
  bin_name: tychus-bin
  target_path: ./
proxy:
  enabled: true
  app_port: 3000
  proxy_port: 4000
```

Sample program

```rs
// hello.rs
fn main() {
    println!("Hello World!");
}
```

Each save rebuilds and runs the hello.rs program.
```
$ tychus run

[tychus] Starting: build [enabled], proxy [enabled]
[tychus] Build: Successful
Hello World!
[tychus] Build: Successful
Hello World!
[tychus] Build: Successful
Hello World!
```