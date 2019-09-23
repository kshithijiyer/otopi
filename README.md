otopi -- oVirt Task Oriented Pluggable Installer/Implementation
===============================================================

Standalone plugin based installation framework to be used to setup
system components. The plugin nature provides simplicity to
add new installation functionality without the complexity of the state
and transaction management.

At the core of the implementation there is environment dictionary and
a flow of stages within plugins. The environment can be modified using
command-line parameters, configuration file, or dialog customization.

Features:

 * otopi is a library for component installation.

 * Modular, task oriented implementation.

 * Supports pluggable manager dialog protocol, provides
   human and machine dialogs.

 * Localization support, gettext enabled.

 * Local and remote execution modes are supported.

 * Distribution independent implementation (core).

 * Compatible with python-2.6, python-2.7, python-3.2

USAGE
-----

otopi [variables]

variables ::= name=type:value variables | APPEND:name=type:value | ''
type ::= none | bool | int | str | multi-str

APPEND: prefix appends as colon list string.

CUSTOMIZATION
-------------

Set the following environment:

    DIALOG/customization=bool:True

This will trigger command-line prompt before validation and
before termination.

Refer to README.dialog for more information.

FILES
-----

CONFIGURATION

Configuration files used to override the environment.

System environment:
    OTOPI_CONFIG
Environment:
    CORE/configFileName
Default:
    /etc/otopi.conf

Config files to be read:
    @configFileName@
    @configFileName@.d/*.conf (sorted)

Structure:

    [environment:default]
    key=type:value

    [environment:init]
    key=type:value

    [environment:override]
    key=type:value

    [environment:enforce]
    key=type:value

default is applied during setup without override.
init is applied during setup with override.
override is applied before customization with override.
enforce is applied after customization with override.

type ::= none | bool | int | str | multi-str

ENVIRONMENT
-----------

Refer to README.environment

UNPRIVILEDGE EXECUTION
----------------------

Using sudo it is possible to escalate privilege. Use the following
configuration:

/etc/sudoers.d/50-otopi.conf

    Defaults:user1 !requiretty
    user1 ALL=(ALL) NOPASSWD: /bin/sh

COMPATIBILITY
-------------

- Python-2.6
- Python-2.7
- Python-3.2
