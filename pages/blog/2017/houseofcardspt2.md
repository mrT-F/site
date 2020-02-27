title: Should there be a House of Cards Season 6? (pt 2)
date: 2017-06-09
published: true
summary: Hacking the NSA is .... easy?  
tags: tv, tech review

#Q - Should there be a House of Cards Season 6? (pt 2)

Ok, yes, the last blog post was a bit of a joke. I just think House of Cards is no longer a __great__ (and maybe not even good) show. Knowing when to give up on your favorite shows is an important skill - I hung on to _Lost_ a couple seasons too long (season 5). I gave up on _Heroes_ at just the right time (season 2 - lol). I think I'm there with _House of Cards_. Apart from the contrived plot lines, stalled character development, and general feeling of _déjà vu_ surrounding every scene, the technical portions of the show were poorly done. Let's break down a few of the scenes and see where they go wrong.

**\*Spoiler Alert - I wouldn't read farther if you haven't watched season 5 yet.\***

Aidan Macallan (aka the hacker guy) plays a major role in season 5. He is in deep with the Underwoods, as his brilliant, unexplained data science (buzzword alert!) algorithms deployed under the cover of a domestic terrorism surveillance program help Frank's campaign in season 4. Apparently, Macallan is not just a brilliant data scientist, but also 1337 as hell. That's my first issue with this character. Not all computer skills are equivalent! The world's best offensive operations/red team specialists are almost certainly not also data scientists in their spare time. I understand some character consolidation may be necessary to avoid audience confusion, but haven't we gotten past the point where its sufficient to just have one polymathic "computer guy"?

In season 5, the Underwoods send him to the NSA. There's not a lot of explanation on how Macallan gets hired, but that's ok. I can buy it - the president would surely have enough contacts and power to place someone in an analyst-level position at any arbitrary agency. We see Macallan "tailgate" his way into a restricted area and sit down in an abandoned computer lab.

![computer lab](/static/blog/2017/abandonedCompLab.png)

From this machine, he types a couple commands, apparently hooks into an NSA social media monitoring program, and sends the same program back to a desktop machine in his office.

![office callback](/static/blog/2017/officeCallback.png)

Hmmm. Ok. Where did he get credentials to log into this machine from? Why is this computer lab, which apparently is the only way to access sensitive NSA programs, totally empty during the middle of the day? Secondly, it's unclear what he did to maintain remote access to the program. Did he just install TeamViewer/VNC? Is he tunneling out an RDP session? Those are believable I suppose. However, why would he send anything directly back to his own NSA workstation? That seems like a pretty conspicuous operations security (OPSEC) mistake from this allegedly brilliant hacker. If he makes any mistake, there is direct evidence pointing right at him. Also, WHY IS HIS NSA DESKTOP UNLOCKED!?!? I don't know any security-conscious individual that would leave their office without locking their system. Unfortunately, the plot only gets worse from here.

The next episode, Macallan finds out that an NSA forensic audit team is in the office. He calls a meeting with Underwood to explain that he needs to create a distraction to cover his tracks. He says "I need to let hackers temporarily invade a capital east telecom switching center". False flag operations are real - creating noise and distraction while you accomplish your real objective is a valid operational strategy in some cases. In fact, there has been speculation that some recent ransomware outbreaks have been part of false flag operations. In this case though... wut? Why would Macallan need to do anything at a telecom switching center? Shouldn't all his activity be inside the NSA internal network? What "hackers" is Macallan calling up to invade? The dialogue here really makes no sense. I'm cool with glossing over some technical details, but at least make sure the plot makes sense from a technical perspective! Maybe this was supposed to be related to Macallan's work the previous season? I was thoroughly confused at this point.

We're then treated to a "hacking" scene - again, apparently all from Macallan's NSA office. We see Macallan text someone, informing them of some vulnerability at the telecom center. 

![hacker text](/static/blog/2017/hackerText.png)

A character like Macallan could be expected to have connections with criminal hacking groups. But it's ridiculous to think that you can "give" someone access to a large internal corporate network (such as a telecom center) and then just close the vulnerability off. There is no such thing as a "temporary" hacker invasion. It takes time and effort to eradicate attackers from a corporate environment. From an attacker's perspective, the first thing I do when I get access to a new network is ensure I don't lose that access. I deploy a backdoor. I deploy a persistence method. I ensure that even if my initial vector is removed, I still have a way back in. Why would these criminal hackers play by Maccallan's rules? What do they even have to gain by momentarily disabling a telecom center?

We then see Macallan making extensive use of the hacker tool "ping" ;). 

![ping](/static/blog/2017/leetPing.png)

After a while, Macallan pulls a terminal up, and uses the "shred" command to delete some files. Kudos to the _HoC_ team for at least using a real linux command here. 

![shred](/static/blog/2017/shred.png)

At the same time, this still makes no sense. _Macallan is doing all of this from his NSA-issued computer!!_ Trust me, the NSA has an endpoint agent on their systems. The NSA logs outgoing network connections. Doing anything illegal on a company-owned system that can be directly traced to the offending individual is not something an advanced insider threat would ever consider. Why even bother with this false flag operation? It seems as if this would only bring _more_ attention to Macallan's activities. Does Macallan really want a full-blown forensic investigation into this telecom center incident? Because that's what would happen next. If all Macallan had to do was delete some files, it makes no sense to plan an elaborate, infrastructure-affecting attack on a telecom center. As an attacker, cleaning up the forensic evidence you leave behind is extremely difficult and goes way beyond deleting some files. The much cleaner, better solution is simply not drawing attention to yourself in the first place.

Macallan then sends a "fix" (from his NSA computer ¯\\\_(ツ)\_/¯), and magically everything is alright again. The hackers are gone, the telecom center is back up, and Macallan is supposedly untraceable. 

TV shows, movies - let's move on from the idea of a lone, brilliant hacker. Sophisticated attacks these days are targeted, well-planned, and generally carried out by teams. There is no magic to security and defense bypass. I will absolutely believe that anything is vulnerable and can be exploited, even inside the NSA or other ostensibly secure environments. However - give me an explanation! Have Macallan spout something that may not make sense to the average viewer, but helps clarify technical details. Trust me, the techically savvy viewers will appreciate it. 

_"But Tim, that sounds hard."_ Is it though? Here's an example exchange, inspired by the recent nPetya ransomware and the speculation surrounding it:

>_**Underwood**: What do you mean we're fucked?  
**Macallan**: I've been aggregating most of our surveillance data in a major east coast telecom center. If the NSA audit team decides to review their surveillance infrastructure, they'll likely notice the code alterations and duplicate deployments.  
**Underwood**: Shut up. Fix this.  
**Macallan**: I have a contact in a South American hacking group. They're looking for a new target for a ransomware campaign. I'll anonymously point them towards a Java deserialization vulnerability on an Internet-facing server at the telecom center. They'll be able to pivot through most of the environment using credentials on that first system. I'll pay my contact to modify their ransomware to just wipe the hard drive and destroy the MBR. Most of the evidence will be destroyed, and the NSA will discourage extensive investigation so they can keep their social media surveillance program secret. The telecom center will be offline for several hours.
**Underwood**: I don't know what you're talking about, nerd. Just do it._

And scene. If _House of Cards_ paid a little more attention to coherent plotlines and details like this, maybe I'd still be watching.