.. _manual-ar-unix: 

UNIX: Active Response Configuration
===================================

The Active response configuration is divided into two parts. In the first one you
configure the commands you want to execute. In the second one, you bind the
commands to rules or events.

Commands Configuration
^^^^^^^^^^^^^^^^^^^^^^

In the commands configuration you create new “commands” to be used as responses.
You can have as many commands as you want. Each one should be inside their own
“command” element. You can see an example here (for the host-deny.sh) and one
here (for disable-account.sh).

.. code-block:: xml 

    <command>
        <name>The name (A-Za-Z0-9)</name>
        <executable>The command to execute (A-Za-z0-9.-)</executable>
        <expect>Comma separated list of arguments (A-Za-z0-9)</expect>
        <timeout_allowed>yes/no</timeout_allowed>
    </command>

- **name**: Used to link the command to the response.
- **executable**: It must be a file (with exec permissions) inside 
  “/var/ospatrol/active-response/bin”.
  
  You don’t need to provide the whole path.
- **expect**: The arguments this command is expecting (options are srcip and
  username).
- **timeout_allowed**: Specifies if this command supports timeout.


Responses Configuration
^^^^^^^^^^^^^^^^^^^^^^^ 

In the active-response configuration, you bind the commands (created) to events.
You can have as many responses as you want. Each one should be inside their own
“active-response” element. Examples are here (for blocking based on the
severity) and here (for blocking on specific rules).

In the active-response configuration, you bind the commands (created) to events.
You can have as many responses as you want. Each one should be inside their own
“active-response” element. Examples are here (for blocking based on the
severity) and here (for blocking on specific rules).

.. code-block:: xml 

    <active-response>
        <disabled>Completely disables active response if "yes"</disabled>
        <command>The name of any command already created</command>
        <location>Location to execute the command</location>
        <agent_id>ID of an agent (when using a defined agent)</agent_id>
        <level>The lower level to execute it (0-9)</level>
        <rules_id>Comma separated list of rules id (0-9)</rules_id>
        <rules_group>Comma separated list of groups (A-Za-z0-9)</rules_group>
        <timeout>Time to block</timeout>
    </active-response>


- **disabled**: Disables active response if set to yes.
- **command**: Used to link the response to the command
- **location**: Where the command should be executed. You have four options:

    - **local**: on the agent that generated the event
    - **server**: on the OSPatrol server
    - **defined-agent**: on a specific agent (when using this option, you need to set the agent_id to use)
    - **all**: or everywhere.

- **agent_id**: The ID of the agent to execute the response (when defined-agent is set).
- **level**: The response will be executed on any event with this level or higher.
- **timeout**: How long until the reverse command is executed (IP unblocked,
  for example).


Active Response Tools
^^^^^^^^^^^^^^^^^^^^^

By default, the ospatrol hids comes with the following pre-configured
active-response tools:

- **host-deny.sh**: Adds an IP to the /etc/hosts.deny file (most Unix systems).
- **firewall-drop.sh** (iptables): Adds an IP to the iptables deny list (Linux 2.4 and 2.6).
- **firewall-drop.sh** (ipfilter): Adds an IP to the ipfilter deny list (FreeBSD, NetBSD and Solaris).
- **firewall-drop.sh** (ipfw): Adds an IP to the ipfw deny table (FreeBSD).

    .. note:: 

        On IPFW we use the table 1 to add the IPs to be blocked. We also
        set this table as deny in the beginning of the firewall list. If you use the
        table 1 for anything else, please change the script to use a different
        table id.
    
- **firewall-drop.sh** (ipsec): Adds an IP to the ipsec drop table (AIX).
- **firewall-drop.sh** (pf): Adds an IP to a pre-configured pf deny table (OpenBSD and FreeBSD).

    .. note:: 

        On PF, you need to create a table in your config and deny all the
        traffic to it. Add the following lines at the beginning of your
        rules and reload pf (pfctl -F all && pfctl -f /etc/pf.conf):
        table <ospatrol_fwtable> persist #ospatrol_fwtable

        block in quick from <ospatrol_fwtable> to any
        block out quick from any to <ospatrol_fwtable>
