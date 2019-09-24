otopi -- API
============

The installer is fully pluggable framework, plugins are assigned
to groups, each groups can be loaded at startup.

For example two plugins within group1:
/usr/share/otopi/plugins/group1/plugin1
/usr/share/otopi/plugins/group1/plugin2

Groups are loaded by looking at the BASE/pluginGroups environment
variable.

At the core of the implementation there is the 'environment'.

The environment is the state of the installation, all plugins
have access to the environment. Environment can be loaded at
initialization from configuration file, and can be examine using
command-line triggered by the DIALOG/customization environment
variable.

Plugins entries are loaded and sorted by Stages, and within each
stage by priority, order by before and after hints. Then entries
are called one by one by their order.

Plugin class inherit from PluginBase and uses @plugin.event
decoration in order to declare entry points (see example bellow).

Plugins are loaded per python module, using the createPlugins()
method.

Please notice that installer change working directory to '/',
every file that is being access that has the potential to be
relative should be resolved using plugin.resolveFile() function.

NOTICE: Boot exceptions (ImportError and such) are not printed, in
order  to see then set OTOPI_DEBUG=1 environment before
running the script.

PRIORITIES
----------

PRIORITY_FIRST
PRIORITY_HIGH
PRIORITY_MEDIUM
PRIORITY_DEFAULT
PRIORITY_POST
PRIORITY_LOW
PRIORITY_LAST

STAGES
------

By order.

STAGE_BOOT
    Use to setup boot environment.
    Usually avoid.

STAGE_INIT
    Use this stage to initialize components.
    Also initialize key environment.
    Use only setdefault() to set environment.

STAGE_SETUP
    Use this stage to setup environment.
    Use only setdefault() to set environment.

STAGE_INTERNAL_PACKAGES
    Install local packages required for setup.
    No rollback for these packages.

STAGE_PROGRAMS
    Detect local programs.

STAGE_LATE_SETUP
    Late setup actions.

STAGE_CUSTOMIZATION
    Customization phase for dialog, avoid.

STAGE_VALIDATION
    Perform any process validations here.

STAGE_TRANSACTION_BEGIN
    Transaction begins here, you can add elements
    before, at this point these will be prepared.

STAGE_EARLY_MISC
    Early misc actions.

STAGE_PACKAGES
    Package installation.

STAGE_MISC
    Misc actions go to here.

STAGE_TRANSACTION_END
    Transaction commit.

STAGE_CLOSEUP
    Non destructive actions.
    Executed if no error.

STAGE_CLEANUP
    Clean up.
    Executed always.

STAGE_PRE_TERMINATE
    Termination dialog, avoid.

STAGE_TERMINATE
    Termination, avoid.

STAGE_REBOOT
    Reboot, avoid.

BUNDLE
------

An installer bundle allows transferring of an installer to
remote machine via ssh and executing it directly with no
dependencies other than python.

To create a bundle use the otopi-bundle script
located at the datadir of the package.

Usage:
 otopi-bundle gettext_domains target [root]

After doing so symlink any of the required plugins, and add
installation script such as the following:

 #!/bin/sh
 exec "$(dirname "$0")/otopi" "APPEND:BASE/pluginGroups=str:my-group $*"

Bundle can be executed using the following command, provided initial script is called
setup:

    bundledir=LOCATION
    ( tar -hc -C "${bundledir}" . && cat) | \
        ssh "${HOST}" '( \
            dest="$(mktemp -t install-XXXXXXXXXX)"; \
            trap "chmod -R u+rwX \"${dest}\" > /dev/null 2>&1; \
                rm -fr \"${dest}\" > /dev/null 2>&1" 0;
            rm -fr "${dest}" && mkdir -p "${dest}" && \
            tar -C "${dest}" -x && "${dest}"/setup \
        )'

EXAMPLE
-------

We write example plugin within group1.

/usr/share/otopi/plugins/group1/example1

---
__init__.py
---
from otopi import util


from . import example1


@util.export
def createPlugins(context):
    example1.Plugin(context=context)
---

---
example1.py
---
import platform
import gettext
_ = lambda m: gettext.dgettext(message=m, domain='otopi')


from otopi import constants
from otopi import util
from otopi import plugin
from otopi import filetransaction


@util.export
class Plugin(plugin.PluginBase):

    def __init__(self, context):
        super(Plugin, self).__init__(context=context)

    #
    # Register init stage at default priority.
    #
    @plugin.event(
        stage=plugin.Stages.STAGE_INIT,
    )
    def _init(self):

        #
        # Use only setdefault to keep existing environment
        #
        self.environment.setdefault('var1', False)

    #
    # perform validation, last chance before changes.
    #
    @plugin.event(
        stage=plugin.Stages.STAGE_VALIDATION,
        priority=plugin.Stages.PRIORITY_LOW,
    )
    def _validate(self):
        if not self._distribution in ('redhat', 'fedora'):
            raise RuntimeError(
                _('Unsupported distribution for iptables plugin')
            )

    #
    # perform some action.
    #
    @plugin.event(
        stage=plugin.Stages.STAGE_MISC,
        condition=lambda self: self.environment['var1'],
    )
    def _store_iptables(self):
            self.environment[constants.CoreEnv.TRANSACTION].append(
                filetransaction.FileTransaction(
                    name='/etc/example1.conf',
                    content=(
                        'hello',
                        'world',
                    )
                )
            )


@util.export
def createPlugins(context):
    Plugin(context=context)
---
