+++
title = "Rx"
date = 2022-10-16 22:11:37
tags = ["roblox", "programming", "fusion"]
+++

Ripped a standalone version of @Quenty's Rx module out of Nevermore. Looks very
very promising.

![](00.mp4)

Interfacing with Fusion's Value objects is really easy:

```lua
function Rxf.value(value: Fusion.Value): Rx.Observer
	return Rx.observer(function(sub: Rx.Subscriber): Maid.Task
		local conn = Fusion.Observer(value):onChange(function()
			sub:Fire(value:get())
		end)
		sub:Fire(value:get())
		return conn
	end)
end
```

> How's performance? I've been looking to write a version of what you're writing
> here for a while, but query performance seems scary.
>
> Using RxInstanceUtils for now, but this is looking a lot cleaner.
>
> --- [@Quenty, 7:13 PM · Oct 17, 2022](https://twitter.com/Quenty/status/1582087487610302464)

I haven't deliberately optimized anything, but it's not the worst. I might be
comfortable with one query that updates every frame, for example. It's a mess,
but you can [play with it here][rx].

[rx]: https://gist.github.com/Anaminus/1f31af4e5280b9333f3f58e13840c670

> hey wait a minute, arent you not a fan of observing any kind of behavior on
> instances that enter the data model?
>
> --- [@Kampfkarren, 7:14 PM · Oct 17, 2022](https://twitter.com/Kampfkarren/status/1582087755311349760)

That's right. Which is why Rx is great, because it simplifies a bunch of
boilerplate that would otherwise be need to ensure that an observation is
correct.
