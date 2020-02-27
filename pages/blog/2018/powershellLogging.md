title: A Note About PowerShell Logging
date: 2018-02-17
published: true 
summary: Some notes about PowerShell logging 
tags: security, windows, powershell, defense 

#Some PowerShell Logging Observations

I hear a lot about PowerShell logging and the capability that it gives to defenders. I currently spend more time on offensive security but I always attempt to understand defensive capabilities.

PowerShell briefly became every penetration testers post-exploitation tool of choice, and defensive products adapted to this quickly. Now, I rarely use PowerShell to execute any payloads. I do however still like to use various scripts to conduct recon and otherwise make my life easier. There are two ways of doing this that bypass nearly all modern EDR products:

1. Load PowerShell into some remote process through process injection. Although there are various ways of doing this, for some quick testing here I used the PSInject PowerShell script, which  can be downloaded here: [PsInject](https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerPick/PSInjector/PSInject.ps1)

2. Copy and paste scripts into the PowerShell or PowerShell ISE console. Most EDR products currently focus mainly on command line arguments, so other than a few scripts which trigger Windows AMSI or something like that, this technique nearly always works.

In this testing, I focused more on process injection since I think that more closely emulates the techniques of attackers.

Anyway, I enabled PowerShell script block logging in the local group policy editor as shown below.

![gpo](/static/blog/2018/gpoedit.png)

The biggest issue with this logging level is it didn't exist until PowerShell v5.0. And although its relatively straightforward to install v5.0, it's not so easy to complete remove previous versions. And tools like PSInject that are linked against the PowerShell v2 reference assemblies will first attempt to use the older PowerShell engine, even if a newer one is installed.

So preventing use of PowerShell v2 (which doesn't have the logging) is a separate and non-trivial issue. It's covered really well here: [Preventing PowerShell v2](http://www.leeholmes.com/blog/2017/03/17/detecting-and-preventing-powershell-downgrade-attacks/)

For example, you can execute PowerShell 2.0 like this:

![downgrade](/static/blog/2018/psversiontable.png)

Nothing run in PowerShell v2 would be logged, as shown by the last thing in the Microsoft/Windows/PowerShell/Operational event log:

![avoid logging](/static/blog/2018/avoidlogging.png)

![eventlog](/static/blog/2018/eventlog1.png)

At first, everything run with PSInject wasn't logged either.

![avoid logging](/static/blog/2018/injected1.png)

![event log](/static/blog/2018/eventlog2.png)

Then, I turned off PowerShell v2:

![powershell off](/static/blog/2018/powershell2off.png)

![powershell off](/static/blog/2018/powershell2off2.png)

This actually stopped PSInject from working, since that project loads DLLs compiled to reference PowerShell v2 assemblies. As a quick workaround, I rebuilt the [Sharp Pick](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick/SharpPick) project to target the 4.5 .NET CLR, and executed unmanaged PowerShell code that way:

![sharppick](/static/blog/2018/sharppick.png)

This time, the unmanaged PowerShell execution was logged:

![logged](/static/blog/2018/logging.png)

The obvious limitation here is if I didn't bother disabling PowerShell v2, most tools will just load the older version. However - disabling it broke at least one commonly used tool. That seems like an added benefit to me. It would also log any actions by an attacker copying and pasting commands into a console.

I don't think a lot of penetration testers (or attackers) realize this. Unmanaged PowerShell is seen as a silver bullet to most modern detection strategies. If you took away my ability to run any of the various recon and post-exploitation scripts I use, accomplishing objectives would become much more difficult. Because these techniques are seen as "safe", this could give security teams a chance to see actors they otherwise wouldn't...

If I was in a security position at any large enterprise, I would make every effort to collect these logs at scale. Enabling script block logging and disabling PowerShell v2 across an enterprise is well worth it in my opinion. 

