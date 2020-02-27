title: A Quick Question Answered with SysMon
date: 2018-06-07
published: true 
summary: 10 minutes of playing with sysmon
tags: security, windows, powershell, defense, sysmon 

#A Quick Question Answered with SysMon

A lot of attacker tradecraft today relies on process injection. Mainly in anticipation of increased awareness and monitoring of this in Windows environments, I think it's worth exploring how these detections can fail. One of the more well-known techniques attacker payloads might leverage is using the `CreateRemoteThread` Windows API call. Personally, I don't care if a technique is well-known or not, I care more about how difficult it is to generate security alerts using today's technology. So, a good question to ask is: "How often does this occur during normal software use?"

The Windows Sysinternals software Sysmon has the ability to help us start to answer that question in like 10 minutes. It's great. I created a dumb configuration file to log only CreateRemoteThread events, shown here:

    <Sysmon schemaversion="4.0">
        <!--No Hashes -->
        <HashAlgorithms></HashAlgorithms>
        <EventFiltering>
          <!-- no driver loggin -->
          <DriverLoad onmatch="include">
          </DriverLoad>
          <!-- Dont bother with process creation -->
          <ProcessCreate onmatch="include">
          </ProcessCreate>
          <!-- Log all CreateRemoteThread -->
          <CreateRemoteThread onmatch="exclude">
          </CreateRemoteThread>
          <!-- Do not log file creation time stamps -->
          <FileCreateTime onmatch="include" />
          <!-- Do not log raw disk access -->
          <RawAccessRead onmatch="include" />
          <!-- Do not log process termination -->
          <ProcessTerminate onmatch="include" />
          <!-- Do not log registry events -->
          <RegistryEvent onmatch="include">
          </RegistryEvent>
          <!-- Do not log file creation events -->
          <FileCreate onmatch="include" />
          <!-- Do not log if file stream is created -->
          <FileCreateStreamHash onmatch="include" />
          <!-- Do not log network connections -->
          <NetworkConnect onmatch="include">
          </NetworkConnect>
        </EventFiltering>
    </Sysmon>

Then I installed the program:

![install](/static/blog/2018/sysmoninstall.png)

I rebooted a couple of times, clicked around the system, downloaded and ran a few programs, and then checked out the event log.

![events](/static/blog/2018/sysmonevents.png)

There were quite a few instances of this API call. However, pretty much all of them came during Windows initialization, as you can see here:

![excludecsrss](/static/blog/2018/excludecsrss.png)

So, in about 10 minutes we saw that it definitely might be possible for defenders to reliably trigger on this type of action with a limited amount of whitelisting. However, I can think of at least one source process that might be whitelisted in an enterprise environment, on VMs at least. Maybe there are other programs that exhibit this type of behavior. A little more time might yield more, who knows ;).