<div align="center">

```
          ___________________________________________
         /                                           \
        |   🌐  E Q U I R E C T A N G U L A R  🌐     |
        |   ┌───┬───┬───┬───┬───┬───┬───┬───┐         |
        |   │ ~ │ ~ │ ~ │ ~ │ ~ │ ~ │ ~ │ ~ │  world  |
        |   └───┴───┴───┴───┴───┴───┴───┴───┘   map   |
        |                  │                          |
        |                  ▼  🎞️ ffmpeg -vf transform  |
        |          ┌───┬───┬───┐                      |
        |          │ L │ F │ R │   🧊 C U B E M A P   |
        |          ├───┼───┼───┤                      |
        |          │ D │ B │ U │                      |
        |          └───┴───┴───┘                      |
         \___________________________________________/
```

# 🌐 VR Video Compressor

**pushed: may 2016 · revived: november 2026 · a sphere, a cube, and ten years**

*a [GitHub Run](https://github.com/saitec/github-run) revival by [sai](https://5aitec.com)*

</div>

---

> 🦕 **quick note before we start.**
> this repo is a fork of FFmpeg from 2016.
> it carries a video filter called Transform.
> Transform turns 360 video from a flat world-map layout into a cube.
> the filter came from Facebook's open-source 360 work. i forked it to learn and play.
> the code is untouched. the old README is in [`README-2016.md`](README-2016.md).
> this page is the story around it. pour a coffee. ☕

---

## 📍 what's in this article

```
 1. 🍿  the short version
 2. 🎥  2016: everyone wants to film in 360
 3. 🗺️  why a sphere is a pain to store
 4. 🧊  enter the cubemap
 5. 🔧  what this fork actually does
 6. 🦣  a love letter to FFmpeg
 7. 🌍  2016 → 2026: the world got rebuilt
 8. 📼  the codec timeline
 9. 🤖  video in the age of AI
10. 🛠️  what i'd build now
11. 📚  lessons
12. 💌  p.s.
```

---

## 🍿 1. the short version

360 videos are spheres.
Screens and codecs love rectangles.
So you flatten the sphere into a rectangle.
The usual way wastes a ton of pixels.
A cube wastes way less.
This fork converts sphere-flat video into a cube.
Same quality. Smaller files. Happier headsets. 🥽

That's it. That's the repo.
Now let me tell you why it mattered.

---

## 🕰️ this repo's life, in one strip

```
 may 2016   🍴 fork FFmpeg. add Transform. push. feel like a genius.
 may 2016   🖼️ add screenshots. fix the image links. feel like a bigger genius.
 2017       😴 nothing. the repo naps.
 2019       🧊 FFmpeg merges v360. my fork quietly becomes history.
 2020-25    😴 still napping. FFmpeg ships release after release.
 nov 2026   🏃 GitHub Run wakes it up. new README. same old code.
```

That's a very normal life for a side repo.
A burst of energy. Then a long nap.
Then, if you're lucky, a second chapter.
Most repos never get the second chapter.
This one got a long letter instead.
I think that counts. 💌

The honest scorecard looks like this.
Lines I wrote: very few.
Things I learned: a lot.
Projections. Codecs. How FFmpeg filters plug in.
How to read a giant C codebase without crying.
That was the real output.
The repo was just the receipt. 🧾

---

## 🎥 2. 2016: everyone wants to film in 360

Picture 2016 for a second.

YouTube had launched 360 video the year before.
Facebook was pushing 360 video in the News Feed.
You could drag your phone around and look behind the camera.
It felt like magic the first ten times. ✨
Then it felt like a gimmick.
But in 2016, we were still in the magic phase.

Cameras were wild.
Rigs of six GoPros taped into a cube.
Big studios used sixteen, even more.
Consumer 360 cameras started showing up.
Two fisheye lenses, back to back.
Point, shoot, and boom: a whole sphere.

```
   2016 CAMERA ZOO 🦓

   📷📷        two fisheyes       "cheap & cheerful"
   📷📷📷📷📷📷 six-GoPro cube      "stitch for a week"
   🎥 × 16     the pro rig         "we have a budget"
   📱 + 🔁     phone panorama     "it's fine, it's fine"
```

And everyone in VR had the same dream.
Headsets plus 360 video equals teleportation.
Watch a concert from the front row.
Stand on a mountain without the hike.
Visit a factory floor from your sofa.

The dream ran straight into a wall.
That wall was bandwidth. 🧱

---

## 🗺️ 3. why a sphere is a pain to store

Okay, tiny geography lesson. 🌍
Remember the world map in school?
Greenland looks the size of Africa.
In reality, Africa is way bigger.
That's because the map stretches the poles.

That stretched map has a name.
It's the equirectangular projection.
Longitude goes left to right.
Latitude goes top to bottom.
Simple. Every pixel has a clean address.

That's how most 360 video is stored.
And it has the same Greenland problem.

```
   EQUIRECTANGULAR: THE GREENLAND PROBLEM 🧊🌍

   ┌──────────────────────────────────────────┐
   │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ ← the sky / north pole
   │▓▓▓▓▓▓▓▓▓▓  stretched like crazy  ▓▓▓▓▓▓▓▓│   tons of pixels, tiny area
   │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
   │      👀  the good stuff, the horizon     │ ← where people actually look
   │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
   │▓▓▓▓▓▓▓▓▓▓  stretched like crazy  ▓▓▓▓▓▓▓▓│
   │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ ← the floor / south pole
   └──────────────────────────────────────────┘
       ▓ = wasted pixels on the sky and floor
```

The top and bottom rows are a single point in reality.
The very top is just "straight up".
But in the file, that point is smeared across a whole row.
Thousands of pixels. For one dot of sky. ☁️

Now you compress that with a normal codec.
The codec doesn't know it's a sphere.
It thinks it's a weird stretched movie.
It spends bits on the stretched sky.
Bits you'll barely look at.

People usually look at the horizon.
Faces, stages, roads, products.
The middle band of the sphere.
So the format spends bits backwards.
Most detail where you look least. 🙃

---

## 🧊 4. enter the cubemap

Okay so how do you fix it?
You stop pretending the world is a flat map.
Put the camera inside a cube instead.
Project the sphere onto six faces.
Front. Back. Left. Right. Up. Down.

```
   THE CUBEMAP TRICK 🧊

             ┌─────────┐
             │   UP ☁️  │
   ┌─────────┼─────────┼─────────┬─────────┐
   │ LEFT ⬅️  │ FRONT 👀│ RIGHT ➡️ │ BACK 🔙  │
   └─────────┼─────────┼─────────┴─────────┘
             │ DOWN 🌱 │
             └─────────┘

   unfold it, pack it into a rectangle, ship it 📦

        ┌─────┬─────┬─────┐
        │  L  │  F  │  R  │
        ├─────┼─────┼─────┤
        │  D  │  B  │  U  │
        └─────┴─────┴─────┘
```

Each face is a plain, flat photo.
No crazy stretching.
Pixels spread much more evenly over the sphere.
Fewer wasted pixels on the poles.

At the same file size, you get more detail.
Or at the same detail, a smaller file.
Facebook's engineers wrote that the cube saved a big chunk of file size.
That was a big deal for streaming.
Every percent saved was millions of mobile users buffering less. 📶

And cubes are GPU-friendly too.
Graphics cards have loved cubemaps forever.
Games used them for skies and reflections since the 2000s.
So a headset can draw a cubemap fast.
Win-win. 🏆

---

## 🔧 5. what this fork actually does

Let's open the repo. 📦

It's a full copy of FFmpeg from spring 2016.
Thousands of files. All of FFmpeg.
Plus the Transform filter sitting in `libavfilter`.

```
VR-Video-Compressor/   (a.k.a. FFmpeg, circa 2016)
├── 📜 COPYING.GPLv2 / GPLv3 / LGPLv2.1 / LGPLv3   ← the licence buffet
├── 🛠️ configure, Makefile                         ← the classic build
├── 🎛️ cmdutils.c                                  ← command-line glue
├── 📂 libavcodec/      ← encoders & decoders
├── 📂 libavformat/     ← containers: mp4, mkv, etc.
├── 📂 libavfilter/
│     ├── 🧊 transform.c     ← the sphere-to-cube maths
│     ├── 🧊 transform.h
│     └── ... 300+ other filters
└── 📖 README.md        ← "Transform" instructions
```

The README explains the setup.
Copy the filter file into FFmpeg's source.
Register it in the filter list.
Add it to the Makefile.
Build FFmpeg as usual.
Then run it like any other filter.

```bash
ffmpeg -i input.mp4 \
  -vf transform=input_stereo_format=MONO:w_subdivisons=4:h_subdivisons=4:max_cube_edge_length=512 \
  output.mp4
```

Let's read those options like humans.

- 🎭 **input_stereo_format=MONO.** One eye's view. Not 3D stereo.
- ✂️ **w_subdivisions / h_subdivisions = 4.** Chop each face into a grid. More pieces, smoother maths.
- 📏 **max_cube_edge_length = 512.** Each cube face, at most 512 pixels wide.

(And yes, the original README spells it "subdivisons". 2016 typo. Preserved forever. 🦕)

Under the hood, the maths is lovely.
For every pixel on a cube face:
1. Work out which direction it points from the centre. ➡️
2. Turn that direction into latitude and longitude. 🌐
3. Find that spot on the flat world-map image. 🗺️
4. Sample the colour. Blend nearby pixels for smoothness. 🎨
5. Paint it onto the cube face. 🖌️

Do that millions of times per frame.
Thirty or sixty frames a second.
That's why it lives inside FFmpeg, in C.
Speed is the whole game.

Why did I fork the whole of FFmpeg for this?
Honestly? Because that's how you did it then.
Filters lived inside the FFmpeg tree.
You couldn't just `pip install` a filter.
So you forked the beast and added your bit.
It made my repo look huge.
It was 99% other people's genius.
And 1% my curiosity. 🙈

---

## 🦣 6. a love letter to FFmpeg

I have to stop and say this.
FFmpeg is one of the most important pieces of software on earth.
And most people have never heard of it.

Watch a video on almost any site? FFmpeg is probably in the chain.
Edit a reel in some app? FFmpeg is often under the hood.
Run an AI video pipeline? FFmpeg cuts and stitches it.
My own reel workflows use it daily. 🎬

```
   WHERE FFMPEG HIDES 🕵️

   🎬 video editors      ─┐
   📺 streaming services ─┤
   📱 social apps        ─┤
   🤖 AI video pipelines ─┼──►  🦣 FFmpeg, quietly doing the work
   🎮 game capture       ─┘
```

It's mostly built by volunteers.
A small core team.
Fighting codec bugs nobody else wants to touch.
Two decades and counting.

In 2016 I forked it to add one filter.
In 2026 I'm way more grateful.
If you use video in any way,
go read about who maintains it.
Then go say thank you. 🙏

---

## 🌍 7. 2016 → 2026: the world got rebuilt

Alright. Big picture time.
I pushed this in May 2016.
It's November 2026.
What changed?

Short answer: almost everything about video.

```
   2016                                        2026
    │                                            │
 📺 TV is still a thing      ─────────►  📱 vertical video eats the world
 🎥 360 is the next big thing ────────►  🎥 360 found a niche, not the crown
 📶 4G in some cities        ─────────►  📶 5G and cheap data, way more places
 🧊 cubemap is clever        ─────────►  🧊 built into FFmpeg, no fork needed
 🎞️ H.264 everywhere         ─────────►  🎞️ AV1 and friends rising
 🧑‍🎨 humans make every frame ────────►  🤖 models generate video from text
 😷 "pandemic" = a movie     ─────────►  😷 we lived through one
 🐙 GitHub: indie            ─────────►  🐙 GitHub: Microsoft + Copilot
```

Let me go through the big ones.

### 📱 video went vertical

In 2016, video was mostly horizontal.
Widescreen. TV-shaped.
Then short vertical video exploded.
TikTok went global.
Instagram launched Reels in 2020.
YouTube launched Shorts.
Now most of the video people watch is tall and short.
Phone-shaped. Thumb-shaped. 👍

Funny thing.
360 promised "look anywhere".
What won was "look at this one thing, fast".
Attention went narrow, not wide.

### 📶 data got cheap, in India especially

In 2016, mobile data was expensive in a lot of places.
India flipped that in a big way.
Cheap 4G data plans arrived in late 2016.
Hundreds of millions came online fast.
Video use went through the roof.
Suddenly compression mattered for a new reason.
Not for headsets. For a billion phones. 📱📱📱

### 🧊 the cube went mainstream (and got smarter)

Remember how this repo needed a whole FFmpeg fork?
That's history now.
In 2019, a Google Summer of Code student built a full 360 conversion filter for FFmpeg called v360.
It handles a pile of formats.
Equirectangular. Cubemaps. Fisheyes. More.
It even supports the equi-angular cubemap YouTube uses.
And the barrel layout Facebook used.
It got merged into FFmpeg proper.

So in 2026 you don't fork anything.
You just run:

```bash
ffmpeg -i input.mkv -vf v360=e:c3x2:cubic output.mkv
```

One line. Built in. 🎉
My whole 2016 repo, replaced by eleven characters of filter.
That's how open source should work.
Experiments become features.
Forks become footnotes.

And it keeps growing.
Even in 2026, people are still adding new 360 camera support to FFmpeg.
The community never stopped.

### 🐙 FFmpeg changed how it works, too

In 2016, FFmpeg development ran on mailing lists.
Patches went out by email.
Reviews happened in long threads.
Very old-school. Very hardcore. 🧔

By 2026, FFmpeg also takes pull requests on its own code forge.
Patches and PRs, side by side.
Even legends modernise. 🦣➡️🚀

### 🤖 AI arrived and ate the timeline

This deserves its own section. Coming up.

---

## 📼 8. the codec timeline

Codecs are the secret heroes of video.
They decide how many bits a frame costs.
This is how they moved.

```
 2003 ┤ 🎞️ H.264 / AVC          the workhorse. still everywhere.
      │
 2013 ┤ 🎞️ H.265 / HEVC         better, but patent mess 😵
      │ 🎞️ VP9 (Google)         free, YouTube loves it
      │
 2016 ┤ 🦕 THIS REPO            cubemaps to save bits for 360
      │
 2018 ┤ 🆓 AV1 released         free, open, from a big alliance
      │                         (Google, Netflix, Amazon, more)
      │
 2019 ┤ 🧊 FFmpeg v360           cubemaps built in, no fork needed
      │
 2020 ┤ 🎞️ H.266 / VVC          even better, even more complex
      │
 2021+┤ 📱 phones decode AV1     hardware support spreads
      │
 2024+┤ 🤖 neural codecs         research: models that compress video
      │
 2026 ┤ 🎬 AI makes the video    sometimes you don't compress it,
      ▼                         you regenerate it 🤯
```

The big arc is simple.
Every few years, same quality, fewer bits.
Usually about half the bits per generation.
That's a huge deal over a decade.

And AV1 matters most for open folks like me.
No royalty mess.
Backed by the big streamers.
Hardware decoding on most new phones and laptops.
If I rebuilt this repo, AV1 would be my default.

---

## 🤖 9. video in the age of AI

Okay this is the wild part.

In 2016, video was something you captured.
Point a camera. Record photons. Compress. Ship.

In 2026, video is something you can *generate*.
Type a sentence.
A model paints every frame.
Sora. Veo. Kling. Seedance. MiniMax. And many more.
Some look real enough to fool you. 😳

At ShopOS we live inside this world now.
Our Sloosh platform makes images and videos for brands.
Ad creatives. Product shots. Reels.
Text in. Video out.
Ten years ago that was a sci-fi pitch.
Now it's a Tuesday.

```
   HOW VIDEO GETS MADE 🎬

   2016:  📷 camera ─► 💾 raw ─► ✂️ edit ─► 🗜️ compress ─► 📡 stream

   2026:  ✍️ prompt ─► 🤖 model ─► 🎞️ frames ─► 🗜️ compress ─► 📡 stream
                         ▲
                         └── or mix both: real footage + AI fill-ins
```

And this changes compression itself.
Think about it.
If a model can regenerate a frame from a description,
do you need to send the frame at all?
Maybe you send the description.
Plus a few hints.
And the viewer's device rebuilds it. 🤯

That's the frontier of neural compression.
It's not mainstream yet.
But the idea is spicy.
Send meaning, not pixels.

For 360 and VR, it's even spicier.
Generate the parts of the sphere you look at.
On the fly.
Skip the rest entirely.
That's foveation, cubemaps, and AI in one move.
2016 me would have lost his mind. 🤯🤯

There's a darker side, of course.
Fake videos got easy.
Deepfakes became a real problem.
Platforms now add labels and provenance tags.
Content credentials. Watermarks.
Trust became a feature you have to build.
That's a 2026 problem 2016 never had to think about.

---

## 🥽 bonus: how a headset actually plays 360

People ask me this a lot.
"So the cube goes into the headset. Then what?"
Fair question. Let's walk it.

**Step 1. Decode.** 🎞️
The headset gets a normal-looking video file.
To the decoder, it's just a rectangle.
It has no idea there's a cube folded inside.
It decodes frames like any movie.

**Step 2. Build the room.** 🧱
The player builds a cube around your head.
Six walls. You stand in the middle.
Each wall gets its slice of the decoded frame.
Front face on the front wall. And so on.

**Step 3. Track your head.** 🧭
Sensors in the headset know where you face.
Turn left, the view turns left.
Tilt up, you see the ceiling face.
This happens many times every second.

**Step 4. Draw only what you see.** 👀
The GPU only draws the walls in your view.
Maybe two or three faces at once.
The rest sits in memory, waiting.

**Step 5. Hide the seams.** 🪡
Where two faces meet, you can get a line.
Good players blend the edges.
Some layouts, like YouTube's equi-angular cube,
spread pixels more evenly so seams hurt less.

```
   INSIDE THE CUBE 🧊👤

        ┌───────────────┐
       ╱│     UP ☁️      │╲
      ╱ │               │ ╲
     │L │    👤 you      │ R│
     │⬅️│   ↻ turning    │➡️│
      ╲ │               │ ╱
       ╲│   DOWN 🌱     │╱
        └───────────────┘
          FRONT 👀 ahead
```

The magic part?
Your brain forgets the cube in seconds.
You just feel "there". 🏖️
That feeling is called presence.
Everything in VR chases it.
Compression, cubes, foveation, frame rates.
All of it serves one feeling.
Being somewhere you aren't.

---

## 🇮🇳 bonus: the india video story

I have to tell this bit from home.
Because the biggest video change of the decade
didn't happen in a headset.
It happened in a billion pockets. 📱

In 2016, a lot of India was still on 2G and 3G.
Data was precious.
People downloaded videos on wifi, then watched later.
Apps had "lite" versions.
Every megabyte counted.

Then cheap 4G arrived in late 2016.
Data prices crashed.
Overnight, video went from luxury to default.
People started watching hours a day.
On small phones. On busy trains. Everywhere.

```
   INDIA, VIDEO, AND A DECADE 📈

   2016  📶 2G/3G, data is gold      🎥 "download on wifi"
     │
   2017  📶 cheap 4G spreads         🎥 video use explodes
     │
   2020  😷 lockdowns                 🎥 everyone online, all day
     │
   2020  📱 short-video apps boom     🎥 vertical, local, fast
     │
   2022+ 💸 UPI everywhere            🛍️ shopping inside video
     │
   2026  🤖 AI makes the ads          🎬 brands need 100x more creatives
     ▼
```

That last line is my whole job now.
Brands used to make one TV ad a quarter.
Now they need hundreds of short videos a month.
Different languages. Different cities. Different moods.
No human team can make that by hand.
So AI tools fill the gap.
That's what Sloosh does at ShopOS.

And compression still matters.
Phones in small towns still have patchy networks.
A video that loads in one second sells.
A video that buffers gets scrolled past. 👋
Every bit you save is a customer you keep.

So the spirit of this 2016 repo lives on.
Not for VR this time.
For a kid on a train in a tier-3 town,
watching a ten-second ad,
on a phone that cost less than a nice dinner.
Same instinct. Bigger crowd. 🫶

---

## 🧪 bonus: three experiments you can run today

Want to feel the difference yourself?
Try these. Each takes ten minutes.

**🧊 Experiment 1: map vs cube.**
Grab any free 360 clip online.
Convert it to a cube with FFmpeg's v360 filter.
Encode both at the same bitrate.
Open them side by side in a 360 player.
Look at the horizon. Which looks sharper?

**🎞️ Experiment 2: H.264 vs AV1.**
Take the same clip.
Encode once with H.264, once with AV1.
Aim for the same visual quality.
Compare file sizes.
That gap is a decade of codec research. 📉

**👀 Experiment 3: where do people look?**
Show a 360 clip to three friends in a headset.
Ask them what they remember.
Bet you they all looked forward, mostly.
That's why "spend bits where eyes go" works.
Humans are predictable. Lovably so. 🙂

---

## 🛠️ 10. what i'd build now

Okay. Fresh start. 2026 brain.
What does this repo become?

Not a fork. Never again. 🙅
FFmpeg already has v360.
So the job changes.
Build on top. Don't copy the beast.

### 🎯 the idea: "spatial video kit"

A tiny toolkit for spatial and 360 video.
Glasses, headsets, phones.
Built on stock FFmpeg plus small AI helpers.

```
   SPATIAL VIDEO KIT 🧰

   📥 input (any 360 / spatial / fisheye clip)
     │
     ▼
   🧊 ffmpeg v360       ← convert to the right layout
     │
     ▼
   👀 saliency model    ← where will people look? faces, motion, text
     │
     ▼
   🎯 bit planner       ← more bits where eyes go
     │
     ▼
   🗜️ AV1 encode        ← region quality per tile
     │
     ▼
   🏷️ provenance tag    ← "real" or "AI-made", baked in
     │
     ▼
   📤 output for headsets, glasses, and the web
```

### 🧰 what's inside

- **A Python CLI.** `svk convert`, `svk encode`, `svk preview`.
- **Presets.** "For Quest." "For Vision Pro." "For glasses." "For the web."
- **A saliency model.** Small, open, runs on a laptop.
- **A web previewer.** Drag a file in. Look around. See where bits go.
- **Content credentials.** Every output says how it was made.
- **A Claude skill file.** So an agent can run the kit for you.

### 🔗 why it fits my life now

SeeIt builds open AI glasses.
Glasses will record spatial video.
Glasses will play spatial video.
Battery and bandwidth are tiny on glasses.
So this kit isn't a museum piece.
It's a missing part. 🧩

And at ShopOS, brands are starting to ask about spatial ads.
Products you can walk around.
Showrooms you can peek into.
Same pipes. Different customers.

---

## 📚 11. lessons

Ten-year-old code teaches good stuff.
These are the ones that stuck.

**1. Forks are for learning. Upstream is for living.** 🍴
Forking FFmpeg taught me a ton.
But a fork goes stale the day you stop touching it.
The real win was the v360 filter landing upstream.
If your idea is good, aim to get it merged.

**2. Maps lie. Choose your lie on purpose.** 🗺️
Every projection distorts something.
Equirectangular lies about the poles.
Cubes lie a little at the seams.
Pick the lie that hurts least for your use.
Same in business. Every metric distorts. Choose wisely.

**3. Spend bits where attention goes.** 👀
This is the whole repo in five words.
It applies to video. And to life.
And to your calendar, honestly.

**4. The boring layer wins.** 🦣
Everyone talks about apps.
FFmpeg is the boring layer under everything.
Boring layers last decades.
Apps come and go.

**5. Trends shrink into niches. That's fine.** 🎥
360 video didn't take over.
It found homes: tours, sports, training, art.
A niche isn't failure.
It's the idea finding its real size.

**6. Credit the giants.** 🙏
This repo is 99% FFmpeg and Facebook's filter.
Say that out loud.
Open source runs on gratitude.

---

## 🗺️ the cheat sheet: 2016 vs 2026

| | 2016 🦕 | 2026 🚀 |
|---|---|---|
| 🎥 360 video | the next big thing | a useful niche |
| 🧊 cubemaps in FFmpeg | fork it yourself | `-vf v360`, built in |
| 🎞️ codec of choice | H.264 | AV1 rising fast |
| 📱 video shape | horizontal | vertical, short |
| 📶 mobile data | pricey in many places | cheap, everywhere-ish |
| 🤖 who makes video | cameras + editors | cameras + editors + models |
| 🏷️ trust | "it's a video, so it's real" | check the provenance tag |
| 🐙 FFmpeg reviews | email patches only | patches + pull requests |

---

## 🏃 running the 2016 version

Real talk: don't, unless you love archaeology. 🏺
This is an old FFmpeg.
It'll fight modern compilers.

If you insist:

```bash
git clone https://github.com/saitec/VR-Video-Compressor
cd VR-Video-Compressor
./configure
make -j8     # go make tea. maybe dinner. 🍲
./ffmpeg -h filter=transform
```

Or, the 2026 way, with stock FFmpeg:

```bash
ffmpeg -i my360.mp4 -vf v360=e:c3x2 cube.mp4   # done. ✨
```

---

## 🤓 tiny fun facts, for the nerds

- 🗺️ **The world map you grew up with is a projection choice.** Every map lies somewhere. 360 video inherited that fight.
- 🧊 **Game skies have been cubes for decades.** Look closely at an old game sky. You can sometimes spot the corners.
- 🎞️ **A codec is mostly a prediction machine.** It guesses the next frame from the last one. It only stores the surprise.
- 📉 **Codec generations roughly halve bits.** Same quality, about half the size, every big jump.
- 🐢 **Encoding is way slower than decoding.** You encode once, people decode millions of times. So encoders can afford to think hard.
- 🎥 **Two-lens 360 cameras overlap their views.** The overlap is where stitching happens. And where ghosts appear. 👻
- 🧠 **Video models also "compress".** They squeeze what a scene looks like into numbers, then decode it back as frames.

## 🙋 who should care in 2026

- 👓 **Glasses teams.** Spatial video is coming to your face. Budget every bit.
- 🎬 **Brand video folks.** Spatial ads are next after vertical. Get the pipes ready.
- 🧑‍🎓 **Students.** Projections are a great maths playground. Spheres, cubes, trig. Fun.
- 🤖 **AI video builders.** Generation still ends in a codec. Learn the codec.
- 🦣 **Anyone who loves FFmpeg.** Go contribute. Docs count too. 🙏

## 💌 12. p.s.

To 2016 me:
You forked a giant to add a tiny filter.
That was the right move then.
The idea won. It's in FFmpeg now.
Not your code. But your kind of idea.
That's still a win. 🏆

To whoever maintains FFmpeg:
Thank you. Seriously. 🙏
Half the internet's video runs on your weekends.
You fix bugs nobody sees.
You review patches nobody thanks you for.
You keep old formats alive so old memories still play.
That's quiet, heroic work. 🦸

To you, reading:
If you're building spatial video, glasses, or AI video pipes,
come say hi at [@5aitec](https://x.com/5aitec) or [5aitec.com](https://5aitec.com).

---

<div align="center">

```
     🌐  sphere ──► 🧊 cube ──► 👀 eyes ──► 🤖 models
    ─────────────────────────────────────────────
         pushed 2016 · revived 2026 · still rolling
```

*part of the [GitHub Run](https://github.com/saitec/github-run): one fix, one insight, every day.*

</div>
