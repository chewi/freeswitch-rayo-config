Minimal FreeSWITCH + Rayo configuration
=======================================

Rationale
---------

The default FreeSWITCH configuration is huge because it is designed to show off what FreeSWITCH can do. You should not use it in production, especially if you are using Adhearsion, because most of it will be redundant. Trimming it down can be daunting and creating a new configuration from scratch can be equally daunting so I have done the hard work for you.

Notes
-----

### Sofia profiles

The conventional approach is to have an internal and external profile. I have split the internal profile into lan and localhost because I do isolated load testing while off the network. This is necessary because a single profile cannot listen on more than one IP address and port number pair.

The rtp-timer-name parameter is not documented anywhere so I will take the opportunity to do so here. The developer of mod\_rayo says that FreeSWITCH will block if it does not receive RTP traffic unless soft timers are enabled. This will cause problems for mod\_rayo as well as SIPp. For some silly reason, the internal code default is "none" even though the default configuration sets it to "soft".

### Log levels

Log levels in FreeSWITCH can be a little confusing. The loglevel setting in switch.conf.xml dictates the overall log level used by the daemon and the log file. The loglevel setting in console.conf.xml only affects what is seen in the console. It is important to note that the console will only ever see log levels that the daemon itself uses. For example, if the switch.conf.xml loglevel is crit but the console.conf.xml loglevel is info, the console will only see crit and alert messages.

### Host-based configuration

My configuration lives under version control and this gets checked out onto different machines. I don't want to have to tweak each machine manually so I use the `$${hostname}` variable to load host-specific settings from the hosts directory. Rename the my-server.xml file to `hostname`.xml.

### ssml.conf.xml

If you don't use TTS then you don't need anything in this file. Otherwise check out conf/rayo/autoload\_configs/ssml.conf.xml from the FreeSWITCH sources. Don't forget to load mod\_flite.

### rayo.conf.xml

max-idle-sec is used to kill off calls that haven't terminated properly. It kicks in when there has been no Rayo activity for a given length of time. The default is 30 seconds, which is probably too short if callers need to wait for others to answer. Being on hold counts as idle activity, even if there is music playing in the background. I have therefore set this to 5 minutes instead.

Launching
---------

I run FreeSWITCH as the same user that runs Adhearsion and Rails. All three are started using [god](https://github.com/mojombo/god). The FreeSWITCH configuration, database, log, and PID file all live under the Adhearsion + Rails project directory tree. The command used to start it looks something like this.

    freeswitch -ncwait -nonat -conf config/freeswitch -db db/databases -log log -run tmp/pids

Console
-------

If FreeSWITCH is running in the background, you can connect to it using fs\_cli and the event socket interface. Use the password given in event\_socket.conf.xml.

    fs_cli -p foobar

Adhearsion
----------

Configure Adhearsion in the same way that you would configure Voxeo PRISM. Do not use old FreeSWITCH-related instructions! Use the name and password given in rayo.conf.xml. If your hostname doesn't resolve using DNS or you're having other DNS-related issues then Adhearsion may fail to start. This can be worked around by presetting the XMPP resource in the username as shown below.

    config.punchblock.platform = :xmpp
    config.punchblock.username = "adhearsion@localhost/#{::Process.pid}"
    config.punchblock.password = "barfoo"

Author
------

James ["Chewi"](https://github.com/chewi) Le Cuirot

License
-------

These files are in the public domain. Make ransom notes with them if you like.
