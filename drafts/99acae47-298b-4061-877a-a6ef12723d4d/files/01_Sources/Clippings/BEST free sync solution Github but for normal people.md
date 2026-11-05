---
title: "BEST free sync solution: Github but for normal people"
source: "https://www.reddit.com/r/ObsidianMD/comments/1o7tztj/best_free_sync_solution_github_but_for_normal/"
author:
  - "[[Valiantay]]"
published: 2025-10-16
created: 2026-11-05
description: "Hey everyone, As a newcomer to Obsidian, I spent a lot of time searching for the right sync solution. The paid option wasn't for"
tags:
  - "clippings"
---
Hey everyone,

As a newcomer to Obsidian, I spent a lot of time searching for the right sync solution. The paid option wasn't for me, and other methods seemed overly complicated, often requiring third-party apps to be running constantly just to sync a folder.

I knew I wanted to use a private GitHub repository for my vault - it’s free, reliable, and gives me automatic version history for all my notes, which is a huge plus. The problem was that most Git-based solutions for mobile are clunky and a pain to manage.

I decided to forego the "usual" sync options and just started browsing the community plugin list and found [silvanocerza/github-gitless-sync](https://obsidian.md/plugins?id=github-gitless-sync) - it's legit as seamless as its going to get without paying for the official Obsidian Sync.

The plugin syncs directly with the GitHub API, meaning ***you don't need to install or manage a Git client on your phone at all***. The setup was incredibly simple: just enter your Github username, repository name, and a Personal Access Token. Just be careful to make sure there are no spaces before or after the text you enter in those fields. Also disable "config" sync as it's going to royally fuck up the plugin.

Besides those two things, it has been absolutely flawless. It works silently and only when actually using Obsidian. It keeps my desktop, phone and tablet perfectly in sync and honestly feels as smooth as the native, paid solution.

The absolute BEST free, reliable way to use GitHub for syncing without the usual headaches, I can't recommend this plugin enough. It just works.

**TL;DR:** The `github-gitless-sync` plugin lets you use a private GitHub repo for syncing without needing Git installed. The setup is super easy, and it works flawlessly across devices. Highly recommended!

EDIT: If you're running into the issue of `Cannot set properties of undefined 'sha'` here's the solution [https://github.com/silvanocerza/github-gitless-sync/issues/21](https://github.com/silvanocerza/github-gitless-sync/issues/21)

---

## Comments

> **dmantisk** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqa9tt/) · 72 points
> 
> There's actually another GitHub based sync solution that doesn't use git called "FIT" as well.
> 
> Both are great as long as you don't have any large attachments over 25 or so MB. This is a limitation imposed by GitHub when not using git, so it's not the plugins' fault.
> 
> Because of this I ended up using remotely save instead.
> 
> I would love to make use of GitHub due to the version control like you, but until I can sync large attachments (around 70MB PDFs) I'm stuck.
> 
> Mega sync offers limited version control even on free, but there's no plugin that takes advantage of it sadly. Probably due to MEGA's zero knowledge structure though.
> 
> > **Aregalle7** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqijxf/) · 7 points
> > 
> > If one were to have a file over this limit, would it just skip it or would it stop syncing? Thanks for the info either way
> > 
> > > **dmantisk** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqoxru/) · 8 points
> > > 
> > > It got stuck and caused some errors.
> > > 
> > > To be safe is recommend only using it if you don't have large files.
> 
> > **Valiantay** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqrce7/) · 2 points
> > 
> > Yeah in the dev's blog they said there may be a way to overcome the various GitHub limits at a later date.
> > 
> > He's going to focus on other projects before coming back to this one.
> > 
> > > **dmantisk** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njr5wqg/) · 1 points
> > > 
> > > One of the plugins (don't remember which) had blacklisting files in the road block, which should hopefully be an easier task.
> 
> > **Valiantay** · [2025-10-18](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nk2jqni/) · 1 points
> > 
> > Yeah I'm testing FIT now too because it's still in active development, works fantastic, same as Gitless.
> > 
> > **KuSi833** · [2026-04-27](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/oimjsf6/) · 1 points
> > 
> > Late to this thread, but I built a free iOS app that fixes exactly this (and other things), [ObSync](https://apps.apple.com/gb/app/obsync/id6762672240). It uses real git (libgit2) instead of the GitHub API, so it's fast and there's no 25/100 MB cap (my own vault has 3+ GB of images and PDFs). Free, no tracking, token sits in the iOS Keychain. Caveat: iOS only.
> > 
> > Source: [https://github.com/KuSi833/ObSync](https://github.com/KuSi833/ObSync)
> > 
> > Hope this helps!

> **Thalimet** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqm1ku/) · 67 points
> 
> “For normal people”
> 
> Developers and people who can learn new things -are- normal people too.
> 
> > **Fluffy-Snow-3938** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njr2wq9/) · 21 points
> > 
> > And.
> > 
> > Annnd.
> > 
> > I setup my sync using syncthing recently. And op is calling using github as FOR NORMAL PEOPLE. Shit doesnt add up in any way.
> > 
> > > **henry\_tennenbaum** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrt0vx/) · 9 points
> > > 
> > > Just remember that sync isn't backup. Love Syncthing but boy is it inadequate if you fuck things up.
> > > 
> > > > **Fluffy-Snow-3938** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrv4gi/) · 4 points
> > > > 
> > > > Can you give me an example of a fuck up you had to go through?
> > > > 
> > > > I set it up a week ago. I haven't invested much time either on obsidian or syncthing even though I've known them for long. I might be missing key fuckups. Please let me know
> > > > 
> > > > > **henry\_tennenbaum** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njs1qs3/) · 10 points
> > > > > 
> > > > > The fuckups are too numerous to list. The basic difference between syncing and backup is that syncing gives you no history, no snapshots to go back to.
> > > > > 
> > > > > Syncthing has some basic optional versioning, but it is complicated to grasp and not something I'd rely on.
> > > > > 
> > > > > I personally am grandfathered into a cheaper Obsidian Sync tier, but even though that has a quite robust file history, I wouldn't consider it actual backups.
> > > > > 
> > > > > Large scale changes can only be reverted with some kind of external backup tool. I personally use restic, but any good modern snapshot-based backup tool will work. Arq backup is a proprietary option for Windows and Macs.
> > > > > 
> > > > > I *also* use obsidian-git, but not to sync changes. I have separate branch for each machine I have obsidian installed on and merely push changes to each branch from each machine. Never pull.
> > > > > 
> > > > > That gives me a great view into what actual changes happened and when. Most files are of course text, something git is made for. Having actual diffs has helped me figure out some weird plugin related changes that were unintended. Easy to revert that way.
> > > > > 
> > > > > As privacy is the core reason I prefer using Obsidian, I wouldn't use github. I personally have my own private forgejo instance, accessible via Tailscale.
> > > > > 
> > > > > If you can't or don't want to host your own git forge, I recommend using codeberg, who use forgejo under the hood.
> > > > > 
> > > > > There is of course no need to have any "forge" at all. Git has been built and is intended to be used just by itself. One can use tools like vscode or any other graphical local git interface to access it if one doesn't like the cli.
> > > > > 
> > > > > I get that that's a big task for some people. I personally think learning git is worth it even for non-programmers (I'm not a professional programmer), but I get not wanting to.
> > > > > 
> > > > > > **TimeYaddah** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njtzs5t/) · 1 points
> > > > > > 
> > > > > > That sounds like you have multiple independent vaults.  
> > > > > > Backup yes, sync no.
> > > > > > 
> > > > > > I am using syncthing too. And I use the obsidian trash and syncthing as "backup". My vault is on 5 machines, some are always online and some are not. So I feel quite safe 😅 There's like 1 thing to know, never open the vault before syncrthing did it's work. I've had to many conflicted files.
> > > > > > 
> > > > > > EDIT: now i am thinking about using a local repo aswell as synthing (just commits no push or pull)
> > > > > > 
> > > > > > > **henry\_tennenbaum** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njuaji2/) · 1 points
> > > > > > > 
> > > > > > > No, no independent vaults. Just one.
> > > > > > > 
> > > > > > > > **RhysieB27** · [2025-10-19](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nk8zk5q/) · 1 points
> > > > > > > > 
> > > > > > > > Using git branches seems like a pretty cool way to do granular vault sync (no private or overly personal notes on my work machine, for instance) but...
> > > > > > > > 
> > > > > > > > > I also use obsidian-git, but **not to sync changes**. I have separate branch for each machine I have obsidian installed on and merely push changes to each branch from each machine. **Never pull**. 
> > > > > > > > 
> > > > > > > > Your branches are functionally independent, therefore so are your vaults. They may share a common ancestor but unless you're pulling and merging then they're no longer working together.
> > > > > > > > 
> > > > > > > > > **henry\_tennenbaum** · [2025-10-20](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nkgsokt/) · 1 points
> > > > > > > > > 
> > > > > > > > > Partly correct. It is just one vault, but one independent branch per machine in the same repo. So, yes, from git's view they're independent "vaults", but not for Obsidian (sync).
> > > > > > > > > 
> > > > > > > > > I'm using git in a very unconventional way and unlike I do for programming. The reason is that I don't want two different sync solutions (obsidian sync and obsidian-git) to interfere with each other.
> > > > > > > > > 
> > > > > > > > > Last thing I need is having to deal with merge conflicts while away on the phone.
> > > > > > > > > 
> > > > > > > > > I could just as well have a different repo for each machine, but I've noticed that for this specific use case there isn't any benefit to that, and git still does deduplicate most files this way.
> > > > > > > > > 
> > > > > > > > > I'd do things very differently had I not been grandfathered into a cheap obsidian sync tier. Either going for a proper git-based workflow or using some unofficial obsidian sync solution along with this unorthodox git backup.
> 
> > **Physical-Rough-709** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njtezme/) · 7 points
> > 
> > Nah, all devs are lizards 🦎, this is well known

> **Gelu6713** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqjh4d/) · 57 points
> 
> But then GitHub gets your private information. Defeats the purpose of a local storage system imo
> 
> > **Faaier** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nju8xy9/) · 14 points
> > 
> > You also have the risk of accidentally setting your Github to public, exposing your vault to the whole wide world.
> > 
> > **FearlessFaa** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njtbegs/) · 1 points
> > 
> > They have incentive to protect your data because they get paid by their services.
> > 
> > > **Gelu6713** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njtbncr/) · 5 points
> > > 
> > > They’re likely one of the better online data storage custodians but when you could do it locally, it feels silly to trust anyone when you just don’t have to. Either use something like Remotely Save, a local hosted git, or pay obsidian for sync
> > > 
> > > > **FearlessFaa** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njthj12/) · 4 points
> > > > 
> > > > I agree. If you want to use multiple devices then I recommend paying for Obsidian Sync which is extremely secure and doesn't require special expertise to use.
> > > > 
> > > > > **tribak** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njwi09q/) · 6 points
> > > > > 
> > > > > But then Obsidian gets your private information. Defeats the purpose of a local storage system imo
> > > > > 
> > > > > > **FearlessFaa** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxk4oh/) · 3 points
> > > > > > 
> > > > > > What do you mean exactly? Obsidian Sync has End-to-end encryption so they only have your scrambled data. Possibly some telemetry data gets collected (I'm not sure). [https://help.obsidian.md/sync/security](https://help.obsidian.md/sync/security)
> > > > > > 
> > > > > > > **tribak** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxkr9o/) · 2 points
> > > > > > > 
> > > > > > > Maybe git-crypt could help. Also, you don’t need to use GitHub, git is still local.
> > > > > > > 
> > > > > > > **Gelu6713** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njzgffd/) · 1 points
> > > > > > > 
> > > > > > > Exactly, this is a much stronger security model and they dont have any incentive to sell data to advertisers
> 
> > **digitalmaster147** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nk0qos6/) · 1 points
> > 
> > I wonder if there's a way to use github while still mitigating this risk? Like maybe can encrypt before we push somehow.
> > 
> > > **RhysieB27** · [2025-10-19](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nk92y04/) · 2 points
> > > 
> > > At that point, all of GitHub's value (as a hub for managing git) is essentially lost. By the time you're thinking about encryption-at-rest you may as well start considering self-hosting a basic git remote, since that's all GitHub would be if all it sees (and thus, diffs) is ciphertext.
> > > 
> > > > **digitalmaster147** · [2025-10-19](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nkb9bnt/) · 1 points
> > > > 
> > > > Not sure I follow this rationale. GitHub doesn’t care about what is being stored. Why take on the burden of self hosting?
> > > > 
> > > > > **OnlyWearsAscots** · [2025-10-21](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nkjf912/) · 2 points
> > > > > 
> > > > > Git tracks incremental changes. So if you had "My name is John" and then updated to "My name is Joe", Git stores the diff (John --> Joe) instead of the whole string.
> > > > > 
> > > > > If you encrypt so it's gibberish, Git can no longer track those incremental changes. Your repo grows large quickly.
> > > > > 
> > > > > > **digitalmaster147** · [2025-10-23](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nl1sfeu/) · 1 points
> > > > > > 
> > > > > > Ahh i see what you mean. Makes sense. It'd have to be a kind of encryption that's consistent enough to recognize diffs (if that's even a thing).
> 
> > **the\_ballmer\_peak** · [2025-11-04](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nn5c58n/) · 1 points
> > 
> > That's not much of a real-world concern. Their entire business model is protecting private data.

> **creativ3ace** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrhc1o/) · 17 points
> 
> Github?
> 
> No. So many things wrong with that. For starters; data security and risk of it being used for training even if a private option. Its owned my Microsoft for one.

> **philosophical\_lens** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqew52/) · 6 points
> 
> Is there any way I can use this just for iOS while using normal git on macOS?
> 
> > **tribak** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njwhq58/) · 2 points
> > 
> > Obsidian config is split in two parts, one for mobile and the other for desktop, just because of these concerns.
> > 
> > > **philosophical\_lens** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njwmobd/) · 1 points
> > > 
> > > So you configure obsidian-git plugin in the desktop config and gitless-git plugin in the mobile config and have everything in the same vault / git repo? Do you need to gitignore anything?
> > > 
> > > > **tribak** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxgktm/) · 1 points
> > > > 
> > > > Sounds about right, I use the Git plugin and make it sync fast so that it feels transparent for me. If you edit the config on mobile a new file will be there to commit that’s mobile specific. No need to gitignore nothing. You can even select an specific file to use as config for each client if you feel like it (iPhone, Android, iPad, Mac, Windows)
> > > > 
> > > > > **philosophical\_lens** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxifss/) · 2 points
> > > > > 
> > > > > Thanks I’ll give this a try!
> > > > > 
> > > > > I really dislike obsidian’s approach to config management where the config is tied to the vault. Ideally vaults and configs should be independent of each other and let users mix and match. 
> > > > > 
> > > > > Your approach might be a good solution. I’m thinking I might place my configs in ~/.config and symlink them to my vaults as needed. But for mobile configs I’ll probably leave them in the vault, because I don’t think iOS can even handle symlinks. 
> > > > > 
> > > > > > **tribak** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxk3yh/) · 1 points
> > > > > > 
> > > > > > Yeah, symlinks sound like the best approach, not sure if the override would help you use a single .obsidian file for all the mobile clients. Let me know how this ends, I haven’t poked that much with the config files and agree that would prefer to have global config and allow specific overrides that are added to that particular file, maybe in the future.

> **Thin\_Rip8995** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njr5slv/) · 9 points
> 
> solid find. using GitHub without touching Git directly is a game-changer for non-devs who still want versioning and cross-device sync
> 
> only other step i’d add for folks new to GitHub: lock down that Personal Access Token with the least permissions possible and NEVER paste it raw into shared configs
> 
> once that’s set, you’ve basically got a free, resilient, and private Obsidian sync stack without running extra apps or services

> **FYNE** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njr026l/) · 4 points
> 
> I was using Syncthing, which works but just a little bit annoying. Ive seen couple days ago a video which mentioned the "GIT" plugin and I've thought to myself maybe this is a good alternative... but in the options I couldnt point to my git, like in the video guide. Some options were just missing. Checked multiple devices and networks but nope.
> 
> This is a very nice tip gonna look into that ty
> 
> > **zabkasa** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njs8e2h/) · 4 points
> > 
> > might i inquire about the reason you consider SyncThing annoying? I've been using it for almost a year now, have no problems and feel like it is close to flawless.
> > 
> > > **FYNE** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njwxgsv/) · 2 points
> > > 
> > > It really is just a minor inconvience the program itself works great
> > > 
> > > its just that I need to launch 2 programs and on work syncthing is blocked so I also always have to connect to my smartphone hotspot to sync
> > > 
> > > > **zabkasa** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njydbap/) · 1 points
> > > > 
> > > > ahh! I get it. I didn't think of that.

> **GrainTamale** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqmi03/) · 7 points
> 
> I use OneSync to sync my phone with OneDrive.  
> To prevent conflicts, I hit sync on my way to/from my computer. Works alright for free.
> 
> > **dmantisk** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqzyij/) · 6 points
> > 
> > I actually use onedrive now too, except through the remotely save plugin. Do check it out.
> > 
> > **Scrung3** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njsf0np/) · 2 points
> > 
> > Same. Sometimes Onedrive randomly deletes my pictures tho. But then I do get an email warning me that a lot of pictures were deleted and it's easy to restore them from the bin.

> **liamdun** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njsbdb2/) · 3 points
> 
> Dude thank you so much for sharing this, I've been looking for something like this for a while that doesn't work in the background while I'm not using obsidian

> **naveenda** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njtakeh/) · 3 points
> 
> I’ve tried a lot of solutions after a while, but I thought, “F\*ck it, we don’t need mobile Obsidian. We can read the content directly on the GitHub mobile app.” 
> 
> This looks promising, thank you so much.

> **starkruzr** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxgzzd/) · 3 points
> 
> I just use Self-Hosted LiveSync [https://github.com/vrtmrz/obsidian-livesync](https://github.com/vrtmrz/obsidian-livesync) Sure I had to set up a CouchDB server but that really wasn't terribly difficult.
> 
> > **\[gelöscht\]** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxocsi/) · 1 points
> > 
> > I tried that and for reasons I don't remember it turned out to be a massive pain in the ass. ETA: oh right it turned out to be just another subscription service I didn't want
> > 
> > > **starkruzr** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxoi2x/) · 1 points
> > > 
> > > I tried that and for reasons I don't remember it turned out to be a massive pain in the ass. ETA: oh right it turned out to be just another subscription service I didn't want

> **literroy** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njuuw34/) · 2 points
> 
> Personally I just keep the files in iCloud Drive where I keep most of my other files anyway and it ends up syncing flawlessly. I know people have various issues with doing that for one reason or another, but if you really want a simple and free option, that is one of them. 

> **EnkiiMuto** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqqyzm/) · 1 points
> 
> I'm the opposite, I want git repos to share and SSGs but without github, they're hard to find.
> 
> I've been meaning to try Radicle for a while.

> **ItsAstray** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrc0kq/) · 1 points
> 
> I would love to use it but i got pdfs of file sizes greater than 100mb lol

> **farcical-flamingo** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrfw9s/) · 1 points
> 
> I'd also highly recommend syncthing and Synctrain for iOS. Keeps it all local and works really well. I set up a shortcut to syncronize every time I open or close obsidian and it works perfectly, is free, and has no cloud services involved.

> **TallestGargoyle** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrr96z/) · 1 points
> 
> I just use Dropbox, though I'm only really syncing between a desktop and a laptop. My tablet gets synced by a 3rd party dropbox app I set up once and never opened again, but I rarely use Obsidian on that.

> **Elvaric** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrrpq5/) · 1 points
> 
> You can use the plugin Descentralized Obsidian.

> **gmek210** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njs7dh1/) · 1 points
> 
> I have used a self hosted ownCloud to sync my notes across my devices for the past 3-4 years. I recommend it.

> **ScallionWilling4779** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njsjscv/) · 1 points
> 
> I mean I use the Git plugin on multiple devices and it doesn't need a Git client, you enter your login and token and it works the same as your plugin mentioned? Or I missed something

> **\[gelöscht\]** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njsrq94/) · 1 points
> 
> Syncthing? Anyone tried that?

> **PlantOfJazz** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njsww1m/) · 1 points
> 
> I found syncthing to actually be quite easy to set up as an amateur.

> **Casually-Passing-By** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njubfvp/) · 1 points
> 
> Honestly, i like your set up, but let me tell ypu something set up that could interest you anyway. I have my github repo, but in addition, i have a cron job where it just updates it. So daily, i have my notes synced without doing anything.

> **smoothieMcSmoothFace** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njuh5ua/) · 1 points
> 
> I use Syncthing and on the iPhone its syncthing client called Möbius. Syncthing is great with Obsidian, with Möbius you need to make sure that it does not get killed by iOS by manually restarting it every now and then.

> **NoozPrime** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njvm9jz/) · 1 points
> 
> I get a error remote manifest do you have any idea why ?
> 
> > **Valiantay** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njvnsuz/) · 1 points
> > 
> > Sounds like you're trying to sync the config file?
> > 
> > In the plugin there's an option to enable logs. I do that, then run the sync and copy the logs into an AI to tell me what's up lol
> > 
> > > **NoozPrime** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njvrk01/) · 1 points
> > > 
> > > What file that you added in your repos ? Because nothing work for me
> > > 
> > > > **Valiantay** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njvs0cz/) · 2 points
> > > > 
> > > > What did the AI say?
> > > > 
> > > > > **NoozPrime** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njvv9il/) · 1 points
> > > > > 
> > > > > to create multiple files it's not working i'm new with github

> **Blueman0110** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njvx508/) · 1 points
> 
> Do I need to install GitHub and let it run in the background on my PC? My PC with 4GB RAM is too weak. I have to run Dropbox and I'm using DropSync for my phone. Sometimes it gets conflict errors.

> **crypt0n0m1c0n** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njwa4zb/) · 1 points
> 
> how do I get to see my notes on my iphone then? do I need a git client on my iphone?
> 
> > **\[gelöscht\]** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxo6kh/) · 1 points
> > 
> > thanks so much for the fast reply. I tried it and it works well on both devices.
> > 
> > I have some questions:
> > 
> > 1. how is the diff managed on the phone?
> > 2. the vault name in my macbook is different than in my iphone... is that a problem?
> > 3. what best practice do you recommend to manage different vaults?
> > 
> > I think these would be nice additions for the README later on the project too ;)
> > 
> > thanks again for this nice piece of software.
> > 
> > > **crypt0n0m1c0n** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njygdz4/) · 1 points
> > > 
> > > thanks so much for the fast reply. I tried it and it works well on both devices.
> > > 
> > > I have some questions:
> > > 
> > > 1. how is the diff managed on the phone?
> > > 2. the vault name in my macbook is different than in my iphone... is that a problem?
> > > 3. what best practice do you recommend to manage different vaults?
> > > 
> > > I think these would be nice additions for the README later on the project too ;)
> > > 
> > > thanks again for this nice piece of software.
> > > 
> > > > **\[gelöscht\]** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njzeg5e/) · 2 points
> > > > 
> > > > yep just tried all that and i’m aligned. thanks!!!
> > > > 
> > > > > **crypt0n0m1c0n** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nk15174/) · 1 points
> > > > > 
> > > > > yep just tried all that and i’m aligned. thanks!!!

> **tribak** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njwipn4/) · 1 points
> 
> I use Git plugin and has been great, any other solution I tried failed to me, like syncthing that was dropping files

> **treetexan** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njwtm5r/) · 1 points
> 
> Is it a bad idea to sync via iCloud on a PC? Asking for a friend.
> 
> > **Valiantay** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxhq3z/) · 1 points
> > 
> > iCloud is a walled garden, if you ever try to switch to another platform like Android it doesn't work at all.
> > 
> > Even on PC it's pretty awful imo

> **Swimming\_Ninja\_7981** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njxtssw/) · 1 points
> 
> can i edit the files directly into github (web-editing on another pc) and than sync the files to obsidian (pc at home)?

> **Ivan\_croissant** · [2025-10-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nk0ij08/) · 1 points
> 
> Did you try Syncthing?

> **ggStrift** · [2025-10-20](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nkgaqrz/) · 1 points
> 
> Obsidian sync

> **obesefamily** · [2025-10-25](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nlbmdu9/) · 1 points
> 
> Thanks for sharing. Im just stating to dive into obsidian beyond basic note taking. Have you tried Syncthing? How does a gitless solution (either the one you posted about or FIT) compare to Syncthing?
> 
> > **Valiantay** · [2025-10-25](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nlcna7f/) · 1 points
> > 
> > One is just a folder sync, one is a document sync.
> > 
> > FIT is pretty good because it's actively maintained, I switched to that one instead of the one in my OP

> **the\_ballmer\_peak** · [2025-11-04](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nn5byw9/) · 1 points
> 
> I AM a developer, and I just spent an hour trying to get this working, including rotating tokens, creating and cleaning new repos, deleting content on one side or the other.
> 
> I have two vaults. I got one of them to perform an initial sync, but it would not update after that. The other only ever synced the .obsidian folder.
> 
> I have no idea how you can consider this an "easier" solution.
> 
> > **Valiantay** · [2025-11-05](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nn5oque/) · 1 points
> > 
> > Well I warned against syncing the .obsidian folder.
> > 
> > I switched to FIT as this one looks like it's in maintenance mode.

> **CucumberAfter6608** · [2025-11-20](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/npx9szn/) · 1 points
> 
> how do you trust this kind of plugins not steal your doc?

> **ContentInflation5784** · [2026-01-02](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nxb1fn6/) · 1 points
> 
> Does gitlab not support this kind of use, or do people just not think of it when making these kinds of tools. I don't use github so sadly doesn't work for me.
> 
> > **Valiantay** · [2026-01-02](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/nxb231g/) · 2 points
> > 
> > Gitlab does not have this mechanism I believe.
> > 
> > But worth raising it to the dev's attention, maybe there is something similar they can use to achieve the same effect.
> > 
> > Also though, why not use GitHub just for this? Not like you have to actually use the GitHub interface or anything

> **Synthetic\_Liquicity** · [2026-01-07](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/ny72a5y/) · 1 points
> 
> But you have a maximum memory threshold of 1 gb in your github repository

> **Gicig** · [2026-01-28](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/o27e12o/) · 1 points
> 
> can it do two-way sync?

> **garvit\_\_dua** · [2026-04-12](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/ofshbdq/) · 1 points
> 
> Sync is such a pain but honestly even when everything is synced, I still struggle with actually finding the note I need when it matters. I'll remember writing something down during a call or meeting, know it's somewhere in my vault, but searching for it is like playing keyword roulette. Sometimes I give up and just ask the person again which defeats the whole purpose of taking notes. At least you've got your sync sorted, that's a win.

> **Frankie-Li** · [2026-04-20](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/ohbxq5r/) · 1 points
> 
> 👀

> **98\_110** · [2026-05-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/om9cyr8/) · 1 points
> 
> hey OP, [u/Valiantay](https://www.reddit.com/user/Valiantay/), how do you set this plugin up for mobile use? I created a PAT and its working well for my desktop, but IDK how to bring the vault being synced to git into my mobile Obsidian.
> 
> > **Valiantay** · [2026-05-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/ombjj7z/) · 1 points
> > 
> > Try the FIT plugin instead, much easier imo
> > 
> > > **98\_110** · [2026-05-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/ombouhz/) · 1 points
> > > 
> > > do you no longer recommend the gitless sync plugin your post is about?
> > > 
> > > > **Valiantay** · [2026-05-17](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/ombqgqr/) · 1 points
> > > > 
> > > > Both are the same, the fit plugin is just easier in some ways if you're having trouble with the one I posted about

> **Nijat01\_** · [2026-08-14](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/p3let7v/) · 1 points
> 
> i had some issues with [https://obsidian.md/plugins?id=github-gitless-sync](https://obsidian.md/plugins?id=github-gitless-sync) but Fit plugin is much easir to use

> **Quetzal\_2000** · [2026-09-10](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/p8xf6pi/) · 1 points
> 
> Looks great. Works great for me on installation. Can you syncrhonize several repository in the same vault?  
> Edit: Error on my second try. Obsidian didn't synchronize all the files in my repo (only 3 out of 29, in a subfolder). Obsidian says "Sync successful", but 26 files where not downloaded to my vault.

> **Linux\_19** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqblzh/) · 1 points
> 
> Use "drive sync" android app with Google drive.
> 
> > **redfox\_seattle** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njqjicd/) · 3 points
> > 
> > I definitely had issues with Google Drive not working well with a vault syncing between multiple platforms (Mac, PC, Android). Obviously that's a special case, but it would sync too slowly or overwrite with the wrong version, I found it really hard to work like Obsidian Sync for me. Also did worry a bit about privacy of my vault.

> **\[gelöscht\]** · 2025-10-16 · 0 points
> 
> Yes. No company would store their source code on GitHub otherwise…
> 
> > **NizmoxAU** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrsdxo/) · 4 points
> > 
> > Yes. No company would store their source code on GitHub otherwise…

> **\[gelöscht\]** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njrys4x/) · 3 points

> **\[gelöscht\]** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njs1l6p/) · 1 points

> **\[gelöscht\]** · [2025-10-16](https://reddit.com/r/ObsidianMD/comments/1o7tztj/comment/njs1r06/) · 1 points