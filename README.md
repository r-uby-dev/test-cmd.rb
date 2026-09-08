<p align="center">
  <a href="https://r.uby.dev">
    <img
      src="rubydev.svg"
      width="400"
      height="200"
      border="0"
      alt="a r.uby.dev project"
     >
  </a>
</p>

> [r.uby.dev](https://r.uby.dev) project.

Welcome to the canonical test-cmd.rb repository.

test-cmd.rb is a Go-inspired, object-oriented interface for running
commands on UNIX-like systems, in the spirit of Go's `os/exec`. A
command can be built by chaining multiple method calls that impact
different command attributes. The environment the command executes
with can be set, its standard input stream can be written to, and
the standard output and standard error streams are captured so the
parent process can read them.

It has zero runtime dependencies (stdlib-only).

## Install

test-cmd.rb can be installed via rubygems.org:

    gem install test-cmd.rb

## Quick start

### Commands

```ruby
require "test-cmd"

##
# Call the 'ls' command
p Test::Command
  .new("ls")
  .arguments("-l")
  .stdout

##
# Call the env command with the given environment
# variables set.
p Test::Command
  .new("env")
  .env("FOO" => "bar")
  .stdout

##
# Write to the stdin stream of the command that
# is executed.
p Test::Command
  .new("tr", "a-z", "A-Z")
  .stdin("hello")
  .stdout  # => "HELLO"

##
# Limit how much data is read from stdout and
# stderr. At most 20k bytes is read from both:
p Test::Command
  .new("git")
  .arguments("diff", "--cached")
  .limit(stdout: 20_000, stderr: 20_000)
  .stdout
```

<details>
<summary>Environment</summary>
<br>

A command that is not given an environment inherits the parent's
environment. When `env` is called, only the variables that are
given to `env` are set on the spawned process, alongside `$PATH`
which is always inherited. `$PATH` is inherited so that the
command can still be found via path lookup.

```ruby
##
# Inherit the parent's environment. All variables
# from the parent process are available.
p Test::Command
  .new("env")
  .stdout

##
# Only $FOO and $BAR are set, plus the inherited
# $PATH. Everything else is left behind.
p Test::Command
  .new("env")
  .env("FOO" => "42", "BAR" => "7")
  .stdout

##
# An empty hash yields only $PATH and the rest of
# environment is left empty.
p Test::Command
    .new("env")
    .env({})
    .stdout
```
</details>

<details>
<summary>Standard input</summary>
<br>

```ruby
require "test-cmd"

p Test::Command
  .new("cat")
  .stdin("hello world")
  .stdout  # => "hello world"

p Test::Command
  .new("tr", "a-z", "A-Z")
  .stdin(Test::Command.new("echo", "hello"))
  .stdout  # => "HELLO\n"
```
</details>

<details>
<summary>Callbacks</summary>
<br>

```ruby
require "test-cmd"
Test::Command.new("ruby", "-e", "exit 0")
  .success { print "The command [#{_1.pid}] was successful", "\n" }
  .failure { print "The command [#{_1.pid}] was unsuccessful", "\n" }
```
</details>

<details>
<summary>Assertions</summary>
<br>

The following example demonstrates how tests might be written with
test-unit from the standard library. A
[Test::Command](https://r.uby.dev/api-docs/test-cmd.rb/Test/Command.html)
takes the name or path of a command, alongside any arguments. The tests
assert against the exit status, standard output stream, and standard error
stream of the spawned ruby process:

```ruby
require "test/unit"
require "test-cmd"

class CmdTest < Test::Unit::TestCase
  def test_ruby_stdout
    assert_equal "42\n", ruby("puts 42").stdout
  end

  def test_ruby_stderr
    assert_equal "42\n", ruby("warn 42").stderr
  end

  def test_ruby_success_exit_status
    assert_equal 0, ruby("exit 0").exit_status
  end

  def test_ruby_failure_exit_status
    assert_equal 1, ruby("exit 1").exit_status
  end

  private

  def ruby(code)
    Test::Command.new("ruby", "-e", code)
  end
end
```
</details>

## License

This software is released under the MIT license. <br>
See [LICENSE](LICENSE) for details.
