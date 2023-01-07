+++
title = "Timestamp Troubles"
author = ["Walker Griggs"]
date = 2023-01-05
categories = ["writing"]
draft = true
creator = "Emacs 27.2 (Org mode 9.4.4 + ox-hugo)"
weight = 2013
+++

## Abstract {#abstract}

Video is hard, and reliable timestamps in increasingly virtual environments are even harder.

We at Mux recently broke ground on a new live video experience, one that takes a webite URL as input and outputs a livestream. We call it Web Inputs. As with any abstraction, Web Inputs hides quite a bit of complexity, so it wasn’t long before we ran up against our first “unexpected behavior”: our audio and video streams were out of sync.

This talk walks you through our experience triaging our timestamps troubles. It’s a narrative account that puts equal weight on the debugging process as the final implementation, and aims to leave the audience with new perspective on the triage process.

I hope you’ll learn from our mistakes, a bit about Libav audio device decoders, and hopefully a new pattern for web-to-video streaming.


## Transcript {#transcript}

Hey everyone, my name is Walker Griggs, and I’m software engineer at Mux.

I’m actually going to do something a little out of order here and introduce the “punchline” for my talk before I even introduce the topic.

The punchline is: “reliable timestamps in virtual environments are really hard.” I’m giving the punchline away because this talk isn’t about the conclusion, it’s about the story I’m going to tell you. It’s a story about our mistakes, a little bit about Libav audio device decoders, and a lot a bit about some old fashion detective work.

So where is this talk going

We’ll start by introducing the problem, of course. Every good story needs an antagonist. We’ll take a quick detour to talk about timestamps, and use that info to color how we triaged the problem. Finally, we’ll arrive back at our problem statement and how we fixed it.

One last piece of framing. Up until I joined Mux 9 months ago, I was at Heroku working databases. The jump from databases to video might seem huge, but they have a lot more in common than you think. They’re both sufficiently complex pillars of the modern internet, they both require a degree of subject matter expertise, and, at first glance, neither are exceptionally transparent.

That’s why this talk will be geared to those of us who are looking to level up deductive reasoning and add new triage skills to our toolbelt. Videos and databases are a deep, dense topics and we won’t always know the answer.

So let’s jump into it. On and off, the last 9 months, I’ve been working on a system called Web Inputs which takes a website URL as input, and outputs a livestream. Website in, video out. On the surface that seems pretty simple, but, as most abstractions do, that simplicity hides a great deal of complexity.

Web Inputs has to wear quite a few thats.

1.  First and foremost, it handles all of the website’s client-side interaction, For example, broadcasting WebRTC is a common use case, so the headless browser, chrome in our case, has to decode all participant streams.
2.  Chrome pushes all the video and audio onto intermediate frame buffers
3.  which we can then feed into FFmpeg to transcode and broadcast through Mux’s standard Livestream API.

Even still, this doesn’t show all of the interactions going on here.

An adjustment we made early on, and one that is the catalyst for this **entire** talk, is to hide the page load from the livestream.

We have to start the frame buffer and audio server before headless Chrome; Chrome won’t start without having **somewhere** to dump AV. If we start Chrome, request the URL, and immediately start transcoding video from the buffers, we’ll also catch the webpage loading in the video which isn’t a great customer experience

Instead, we can listen to Chrome’s events. One of which is called “First Meaninful Paint”, and that’s effectively Chrome saying “something cool is on the screen now, you’ll want to pay attention. Only then, should we start ffmpeg. .A colleague of mine, Garrett Graves actually came up with this. It worked really well from a timing perspective, but here’s also where we started to see some fun behaviors in our livestream.

Behavior number 1: the first 4-7 seconds of video looked like it was shot from a cannon. The audio was scattered all over the place, and frames were jumping left and right.

Behavior number 2: the audio + video would meander in and out of sync over the course of the broadcast. It was maybe plus or minus 300ms. And was if it even sunk up at all. In the extreme cases, the AV sync was 1000ms off.

That’s no good. So what did we do? We did, what I’m sure many of you all are guilty of, and stayed up late into the morning fiddling with ffmpeg flags. We read all the blog posts on AV sync. We tried various combinations filters. In fact, a colleague of mine put together a spread sheet of the flags we were using, links to the videos, and various, subjective scores.

The problem with this, as many of you are probably itching to call out, is it lacks evidence. We spent a day on what effectively amounted to trial and error. The most frustrating part: sometime’s we’d get close, and I mean really really close. And then one test run would fail, which would put us back on square one.

Another point to callout here: we were testing in different environments, and differences were staggering. In production, the Web Input runs on as many cores as are dedicated to our entire development stacks. So it didn’t take long before we noticed how inconsistent dev really was, and that our qualitative assessments weren’t going to get us there.

We had to face the reality, and get our hands dirty. Empirical evidence is and will always be the fastest way to understanding your problem.

Before we look at any logs or metrics, let’s run through a quick primer on timestamps so we’re all on the same page. You’ll often hear PTS and DTS talked about. For starters, they’re both types of timestamps and each frame has them. The PTS or “presentation time stamp” is when a player should present that specific frame to the viewer. The DTS is the “decoded time stamp” or when the player should decode the frame.

These stamps are different because frames aren’t always stored or transmitted in the order you view them. See, some frames actually refer back to one another. These are call predictive frames.

but if you take anything away from this slide, let it be that presentation time stamps and decode time stamps are different, and change how we order our frames.

With that out of the way, let’s jump back to our evidence. This is roughly where the real triage work begins.

I’m sure everyone has their own way of dumping timestamps from the VOD, but I wanted to go right to the source — I wanted to crack open the system and inspect the timestamps as they’re assigned. I jumped into the PulseAudio decoder, logged timestamps, a few other metrics, and dumped that file to disk.

I’ve done of bit of pruning so the folks in the back aren’t squinting too much.

Non-monotonic DTS in output stream. These can be the bane of your existence if you’re not careful. It means that your decoded time stamps are out of order. See, it easiest for your decoder if each frame arrives in the order it cares about. It buffers some of the that GOP (group of pictures) and displays the completely decoded frames in presentation order. If your DTS are out of order in the output stream, it’s up to your player to make sense of them. Your end result will vary depending on how severely misaligned your timestamps are.

Another bit to callout, the sample sizes. We’re seeing a huge push of these 65khz packets at the start of the stream, which settles down to a steady 4 thousand after the first few seconds. I’ll let you start to theorize about what that means; we’ll come back to it in a second.

The next bit to question: DTS… on audio samples. Audio ‘frames’ don’t form group of pictures like video frames do. Audio doesn’t have predictive frames, so why are they used, and why are they different?

Ultimately it comes down to Libav’s data models. Frames and packet are general structs and used for both video and audio, so we can think of “PTS” and “DTS” in this context as ‘appropriately typed fields that can store timestamps’. So that explains why we’re using this terminology, but it doesn’t explain why they’re different.

For that we have to look at the pulse decoder which does 3 things when it assigns timestamps to frames.

The first is to fetch the time according wall clock; that’s the DTS.

It then adjusts the DTS by the sample latency. That latency is just the time difference between when sample was buffered by pulse and requested by ffmpeg.

It then runs it through a filter to de-noise the DTS and smooth out the timestamps frame-frame. The wall clock isn’t always perfect, as we’ll see more of in a second, and it can be exceptionally sporadic in these virtual environments.

Keep in mind, this system is running a docker container, running on a VM, which is probably itself part of a hypervisor. We’re likely not using a hardware timing crystal here, so we de-noise that PTS to offset and inconsistencies.

We’re heading in the right direction, but at this point I’d say we have “data” — not “evidence”. Long log files aren’t exactly human readable, and certainly harder to reason about. I may not be a Python dev, but the one think I’ll swear by is it’s ability to visualize and reason about data sets.

So with some fun awk we were able to sanitize that data and reshape it a bit so Numpy and Matplotlib can make sense of it.

The first thing we wanted to visualize were these timestamps, of course. We expected to see a linear increase in timestamps maybe an artifact of those non-monotonic logs in the first few seconds.

Good news: we do! But, maybe not as clearly as we should.

Unfortunately this doesn’t tells us that much. We can see it follows a rough trend with “some” inconsistencies, but we can’t draw any conclusions from this data. What would be more helpful would be to graph the **rate** at which these timestamps fluctuate because what we really care about is “how reliable or consistent these timestamps are”. The derivative, or the rate of change, of this data might show us how unstable these timestamps actually are.

Lo and behold; the derivative is pretty telling. So what are we looking at? Well a derivative of a linearly increasing function is flat, so that tells us that after some number of seconds, our timestamps are dead close to linearly increasing. That’s what we want!

But the first few seconds — they tell another story. Every time the slope increases, timestamps are increasing in a super-linear way. When they slope decreases, our timestamps are slowing down or even “jumping back in time” in a sub-linear way. So that’s interesting, but maybe more interesting is that this is only occurring for the first few seconds.

Also worth calling out that our de-noising filter is doing it’s job, but it can’t spin gold from straw. It’s really only as good as the data it’s fed.’

There was another piece to the logs: that back pressure of buffered samples at the beginning of the stream. Sure, we factor that in, and we’ll see some rough correlation. Again, high pangs of latency early in the stream which settles down to something more consistent.

If we think back to those initial behaviors, I think this visualizes them pretty well. We see an initial scramble of timestamps which likely is causing the player to throw frames at us in a seemingly random or unpredictable order. We can also see that the timestamps aren’t perfectly linear, which would explain why AV sync meanders a little bit over the course of a stream.

Something to call out here though: this is just a correlational and not directly causational relationship. These are only part of the picture. It might be hasty to drop the gavel and blame Pulse. There’s a number of paths unexplored here. For example, these are only the audio samples. There’s a whole other side to the video samples to explore.

One of the good and bad thing about the libav it’s a common abstraction for all protocols, codecs, etc. Not every concept translates well. Some things can be represented differently in code than in the actual stream. We stream out with RTMP which uses an FLV bitstream. FLV only encodes 1 timestamp in audio packets, so time spent reasoning about the PTS and DTS tells us more about how libav audio device decoders work than the output stream.

But this is also the point where we needed to step back and consider our goals. It’s important to callout that these visualizations are just interpretations; they’re portholes into the side of the system so we can get a better sense of what’s going on. But they’re not hard evidence, We, like many of you are under deadlines.

So, we had to make the difficult decision here. Keep digging, or action what we already know. We went with the latter, and wanted to strip it back to first principals.

-   We already know that latency is at play here, and our Pulse is buffering more than we need.
-   We also can see that our timestamps, at their core, our based off the wall clock which isn’t always perfect even after de-noising.
-   We know some simple metrics like the starting timestamp, exactly how many samples we’ve decoded, and the target frequency.

We’re not super concerned with what came **before** us starting the tr In fact, if you remember where this entire saga began, we were trying to cleanly start headless chrome **without** broadcasting the loading screen. Any data buffered before the start of the transcode, can be tossed.

One thing we tried was flushing the buffer when we initialize the Pulse Audio device in FFmpeg. We found, limited success interacting with the audio server directly.

Another option, and one we used to validate our hypothesis, was to ignore all samples until we were pulling off nice, round, 4kb packets. That solution, for a number of reasons, was a naive one, but gave us fine results in a controlled environment.

The last option was the one we ultimately went with, which is counting the number of samples and computing the DTS on the fly.

So what does that look like for us? First , we record the wall time when we initialize the device decoder — that’s our ‘starting time’. We then ignore all samples with a DTS before that starting time. We count each sample we care about, and use that to determine sample perfect timestamps using our target frquency and timebase.

For example,  if our target frequency is 48khz, or 48000hz, and we’ve already decoded 96000, that means we’re exactly 2 seconds into the livestream.

The results were so much closer. Not perfect, but closer. In fact, the over the next few days, we ran a 8hour test stream and noticed that, over the course of the day, millisecond by millisecond, the video pulled ahead of the audio.

See: what we learned first hand was, when it comes to livestreaming timestamps, you can’t trust any one single method. What we found is that counting samples is great in theory, but not responsive. There are a number of reasons why we might drop samples, and this solution doesn’t have any way to recover if we do. Sharks do bite undersea cables.

So instead, we can double check and re-sync where appropriate. We can actually use the wall clock here for a system of checks and balances. If the two methods of determining the timestamp disagree by more than some threshold, re-sync. You could, for example, reset that initial timestamp and restart the frame counter.

This solution gives you the accuracy of a wallclock but the precision of sample counting.

So what are some take-aways here. Well the first might be:

1.  Don’t just collect evidence, visualize it! More often than not, staring at raw numbers wont get you there. Patterns, anomalies, outliers — they’re all far easier to identify when you can see what you’re working with. In our case, overlaying the sample size and timestamp data really shone a light on the problem.
2.  I think it’s accurate to say, “you can’t trust any single solution in live streaming, and you need a system of checks in balances”. Again, in our case, maybe counting samples isn’t enough. We need something more robust that combines the accuracy of a wall clock and precision of a sample counter.
3.  I could even stretch as far to say: get you hands dirty. We resisted diving into the decoders because libav was daunting from the surface, and that’s not the case.
4.  But I think the big take away here, and I told you it was coming, “reliable timestamps in virtual environments are really, really hard.”
