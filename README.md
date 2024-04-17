This repository contains sample bitbake-setup configurations for the purpose of testing and refining the tool.

The current revision of bitbake-setup
(https://github.com/kanavin/bitbake/commits/akanavin/bitbake-setup/)
is now as good as I can make it (given the currently implemented
features), so here's the plan to take it into use for setting up Yocto
builds. Step by step:

0. Terminology used:

https://lists.openembedded.org/g/openembedded-architecture/topic/terminology_for_bitbake_setup/110775384

1. What are the configurations that Yocto project should offer?

The official configuration registry (in a repository called
yocto-configurations, hosted on git.yoctoproject.org) would contain
configurations for tip of master, and tips of all stable branches from
some future release onward. E.g.

yocto-master.conf
yocto-5.2-walnascar.conf
yocto-5.3-xomething.conf
and so on.

Stable release configurations are never removed from the registry even
when they become archaic. Rather, they contain a field with their
expiry date, and when that date passes bitbake-setup will hide them
from 'list' output, and politely note that the respective release is
no longer supported or maintained upstream when its status is queried.

There will be no configurations with specific point releases; it's too
much ever-growing clutter (especially for LTS) and not what people
really need when they get started. At some point they'll create their
own private configurations, and then they can pin layer revisions as
they please.

Each configuration checks out bitbake/oe-core/meta-yocto/yocto-docs
repos separately. Poky repository will not be used. Note that there's
an issue here described separately in [1].

[1] https://lists.openembedded.org/g/openembedded-architecture/topic/layer_layout_poky_vs/111010762

2. What bitbake configurations should each configuration offer?

This is sort of subject to debate, but here's the starting point:

- qemux86-64 and qemuarm64 as machines. No retro computing, no
unsupportive vendors.
- poky, poky-altcfg, poky-tiny as distros

That makes for six configurations.

Targets for each of them (except tiny) would be the images: minimal,
full-cmdline, weston, sato-sdk.

Each bitbake configuration is defined as 'blank' template (e.g. empty
local.conf) provided in meta-poky (this needs to be added there) plus
fragments that set machine and distro, and sstate mirror or anything
else needed for a good first experience.

Each configuration is guaranteed to come with pre-built sstate. This
is the killer feature of the whole project.

3. Containerized builds?

Not at the moment. This is a future project. I tend to dislike
containers, and think we should rather detect if buildtools-tarball is
needed, and pull that in if so.

4. How is sstate availability guaranteed?

There would be a separate autobuilder job, triggered by new
commits in master (or stable branches) that would populate sstate (via
bitbake-setup plus autobuilder sstate tweak). Existing a-full selftests
already cover for that sstate propagating to CDN. I need to check if buildbot
and autobuilder code can support jobs triggered by new commits,
otherwise it has to be time-scheduled like nightly or auh, which I'd like
to avoid.

There could be a short window of not yet provided sstate until that job
runs, but I'm not sure how to address that, short of asking maintainers
to run the sstate population job from staging branch before they push
that staging branch to master/stable branch.

5. Is cloning poky and using oe-init-build-env deprecated?

Yes... but in the softest, gentlest manner possible. Documentation
should put using bitbake-setup first, and explain why it is better.

6. Should autobuilder use bitbake-setup?

Yes, for credibility and validation reasons, but it's a much higher
hanging fruit. There are a lot of tightly coupled moving pieces there,
and bitbake builds are also distributed between multiple workers,
something bitbake-setup doesn't currently account for. It's a
separate, future project.
