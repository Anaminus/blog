+++
title = "Syncing things"
date = 2022-11-06 20:00:00
+++

On my desktop, writings have to be technical, detailed, and correct. It's hard
to write blog posts from there. I find it a lot easier to write casually if it's
somewhere else, such as my laptop while sitting in a cushy chair.

But the blog's local repository is still on my desktop. How do I do things from
my laptop, then? It's just Git, so it should be possible.

Well, it is possible, but it's a pain to get going, and there are two parts. The
first part involves setting up the "remote" repository as a bare repo somewhere
on the desktop. Then separate repositories are cloned from it, one on the
laptop, and another also on the desktop. These are just regular local
repositories, but instead of the remote being GitHub or something, it's just
somewhere else on the desktop. There are some extra considerations, such as what
to do about syncing the "local remote" to the "remote remote" GitHub, but I
haven't gotten that far, and don't really care to.

Anyway, the second part is that I need the laptop to be able to access the file
system of the desktop, in order to push to and pull from the "remote" repo. The
laptop is Linux, while the desktop is Windows, so there's some finagling
required. I use a combination of [MSYS2 and SSHd][sshd] to get this done.

I've used this setup for exactly one repository so far, and that was mainly to
test whether it was even viable. I think I would only use it if I needed to do
the usual deep-focus work. But I've found that it's difficult to do that kind of
work on my laptop, so I'm not exactly enthusiastic about this setup.

The alternative setup, and what I'm currently doing, is to use
[Syncthing][syncthing]. I have Syncthing on both the laptop and desktop, and I
just configure the blog repository as a folder that syncs between them.

Now, I suspect that Syncthing and Git wont play nice together. Some rudimentary
internet research says as much. To get around this, I just don't sync the git
parts. There's some other stuff that gets ignored, too. The exact patterns look
like this:

```
.git*               # No git files.
.git/               # No git directory.
*.sublime-workspace # Stop Sublime Text from getting weird.
*.lock              # Hugo just drops this and leaves it lyin' around.
```

On the desktop, the blog is a full repository, but from the perspective of the
laptop, it's just a regular folder. I can still run Hugo on it, so that's
nothing to worry about.

I don't get to do Git things from the laptop, but that's fine. On the laptop,
the writing is the most important part. Proofreading and publishing can come
later, on the desktop. If anything, this setup is better, because it prevents me
from getting distracted.

It's interesting how I find it hard to do focused programming on my laptop, but
easy to do casual writing. While on the desktop, it's the complete opposite. If
there's anything to take away from this post, it's to use different devices for
different contexts.

[sshd]: https://www.msys2.org/wiki/Setting-up-SSHd/
[syncthing]: https://syncthing.net/
