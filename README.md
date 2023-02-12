# Cli Demo

## Goals

Provides a pattern for creating shareable command namespaces and orchestrating them into a single CLI ||DONE||

This toolchain abstracts bundler and rubygems and can be configured with gems outside of rubygems that will be sideloaded after being included in the lockfile. This ensures that we will not have dependency conflicts but keeps the end user from having to manage the gemfile which is intentional. While this is written in Ruby with Thor and its extension commands are the same, a consumer must be able to operate this without knowledge of ruby. ||DONE||

Allow for dependencies to be transient of their use. For example say we would like to allow extensions to access a persistence like SQLITE we can allow them to request a persistence interface provider from the root without including SQLITE as a direct dependency. This allows commands to act more consistently and ensures that there is a single source of truth for dependenceis. Nothing requires that this be how extensions consume providers as each gem is part of the LOAD_PATH and can use what it likes. ||WIP||

Allow for the root command and extension commands to be delivered over a wire protocol. Right now command registration is handled by requiring commands exist locally within the LOAD_PATH. Its important that some commands might be network services and will register with a command directory allowing commands to appear to be installed locally but instead perform their actions with RPC. This can elimiate the complexity of sideloading and bundler hacks currently required as well as allow for some interesting use-cases for integration with remote development environments encapsulating secrets at the locality of the command, expanding the agility to give developers access to tools ||WIP||

Produce an example where an existing non ruby command binary is wrapped with a Thor DSL so we can integrate it along with any secrets into our root command. ||WIP||

## The future

## Install

Clone the repo and run ./cli

### What happens next

1. The root cli toolchain will evaluate current dependencies using bundler. This will happen evertime the cli boots.
2. Once base deps are resolved the cli will autogenerate a config file in ~/.cli/config
3. On the next boot the config will cause the gemspec to drift from the Gemfile.lock causing bundler to sync again
4. The root cli will evaluate its deps and sideload commands defined in the config
5. The combined comands will be displayed in a combined menu

## The Demo

Commands here are incomplete outside the bounds of the `config` namespace. Which allow for the config file located with ~/.cli/config to be reset or initialized.

For example the `survey` namespace is more a demo of the command registration and sideload process.

## How does this work

Well, a lot of this is just a hack on bundler to externalize how deps from the gemfile. The config options are:

1. `- name:` which specifies a gem, without additional modification this assumes the gem is published in rubygems and will fetch the latest
2. include `path:` which specifies that the gem is located within the `~/.cli/command/{gem_name}` this option eventually will let you specify the relative path to the config directory but not in scope for now
3. include `path_relative:` which specifies that the gem is located with `{cli_demo}/{gem_name}` this option eventually will let you specify the path relative to the executable but not in scope for now

Sideloading requires that command gems sometimes refered to extensions implement a refinedment for Thor that changes the signature of the `sub_command` directive. Instead of extending the procedurally context collecting command to build a namespace it identifies the namespace and the class that implements the commands for that namespace. This does break Thor for now but will eventually allow for extensions to be run standalone in the event the framework is no longer needed without modifying the commands.

The refinement allows each gem to create a command registry that we can register in our root gem. This when included as an extension command gems create a registry and Thor command registration is handled in the root or parent command only loading behavior from the extension.

The sideload process is included in the cli_toolkit gem so its possible for command gems to be organized into trees each level able to sideload and publish an aggregate to its parent.

Of course this is a little out of scope for now and we currently don't have a mechanism for configuring nested command extensions in a dynamic way so this is feature is a solution looking for a problem really. But expresses the flexability of the pattern at least.
