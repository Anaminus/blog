+++
title = "Dumping all my tweets"
description = "In which I add 15 years of tweets to my blog."
date = 2025-09-16 17:00:00
tags = ["meta"]
+++

> What if I were to dump all my tweets into my blog, which will involve coming
> up with titles and tags for every single one of them?
>
> --- Me (idiot)

The thing about social media is that your content isn't really yours. If your
platform of choice randomly, algorithmically, or hallucinatorily decides to nuke
your account, all of that content you made is gone.

I like to assume that I've said some pretty good stuff over the years, so it
would be a shame if any of that were to disappear. If I'm not in control of my
[Twitter][twit], I am in control of my blog. What would it take to convert every
tweet into a blog post?

## Step one: export
First, I need to get an archive of all my tweets. Fortunately I already had one
lying around that's fairly recent, so I'll just use that.

What about other social media? I also have a [BlueSky][bsky] and
[Mastodon][mast] account, but those have mostly been mirroring Twitter. There
may be some platform-exclusive replies, but I can look through those manually.
Most reply-type posts wont make the cut anyway.

## Step two: parse
Next, I need to write a script to parse this exported data. My default language
of choice is [Go](https://go.dev/), which compiles so fast that I regularly use
it for one-off scripting.

The first thing to do is to figure out where the meat is. The archive
conveniently provides a local web-based interface for browsing the data, so
there's a lot of HTML and JavaScript files lying around. Fortunately, there's
also a README that explains the content of the archive in an amount of detail
that is refreshingly surprising. Hats off to whoever took the time to do that.

Anyway, the important file is `tweets.js`. While this is a javascript file, it
contains mostly JSON. Specifically, the root value is an array, but it also has
JS code to assign this array to some field:

```javascript
window.YTD.tweets.part0 = [
  {
    "tweet" : {
...
```

By trimming the file up to the first `[`, the assignment is removed, and now I
have a pure-JSON file to parse.

```go
type Data struct {
	//TODO
}

func main() {
	f, _ := os.ReadFile(os.Args[1])
	f = f[bytes.Index(f, []byte{'['}):]

	var data []Data
	d := json.NewDecoder(bytes.NewReader(f))
	fmt.Println(d.Decode(&data))
...
```

Next is to fill out this Data struct by inspecting the structure of the JSON and
deciding which parts are important. Since this is a one-off, I'm keeping the
structure extremely simple. If I'm using UnmarshalJSON, then I'm doing it wrong.

Here's the structure, which only contains relevant fields that I will actually
be using:

```go
type Data struct {
	// Structure is two levels deep. Probably for
	// forward compatibility purposes which I,
	// for one, appreciate.
	Tweet Tweet `json:"tweet"`
}

type Tweet struct {
	// Unique identifier that appears in URLs.
	// Will be used as key for mapping.
	ID string `json:"id"`
	// Timestamp. Unparsed now because it's easier
	// to just do it later.
	CreatedAt string `json:"created_at"`
	// The raw content of the tweet.
	FullText string `json:"full_text"`
	// Various interpreted portions of the text,
	// such as media, URLs, mentions, and
	// hashtags.
	NEntities Entities `json:"entities"`
	Entities  Entities `json:"extended_entities"`
	// ID of tweet this tweet is replying to, if
	// applicable. When the ID is another tweet in
	// this archive, this can be used to
	// reconstruct threads.
	ReplyTo string `json:"in_reply_to_status_id"`
	// User name of the tweet being replied to.
	// Needed for generating links.
	ReplyName string `json:"in_reply_to_screen_name"`
}
```

Next, I'll take these Tweet structs and convert them into a structure that
better fits my blog. I'm calling such posts "briefs" (🩲), so that will be the
name of the struct:

```go
type Brief struct {
	// Unique ID, corresponding to Tweet.ID.
	ID int
	// Parsed date.
	Date time.Time
	// Final content, after applying various
	// transformations.
	Content string
	// Reply ID, corresponding to Tweet.ReplyTo.
	ReplyTo int
	// Corresponds to Tweet.ReplyName.
	ReplyName string
	// List of IDs replying to this post.
	Replies []int
	// Associated media.
	Media *BriefMedia
}
```

I originally assumed that a tweet can only have one piece of media attached, so
the Media field only allows zero or one values. While it turns out that more
than one can be attached, I only ever used this feature once, so I just handled
that one tweet manually.

### Media
Media is produced by inspecting the `Entities` fields. There are two fields:
`entities` and `extended_entities`. The first one contains text-related things
like mentions, hashtags, and URLs, and images as well. The extended entities
appears to contain only media-type things, including images. That is, the same
image object will appear in both fields, so it's important to only handle one of
them to avoid duplicates. Since I'm dealing only with media at this point, I
only need to consider `extended_entities`.

Getting the right media is somewhat involved. The `type` field indicates one of
`photo`, `animated_gif` or `video`. The first two are pretty straightforward,
with the `media_url` field corresponding to the actual file.

```json
{
  "extended_entities": {
    "media": [
      {
        "type": "photo",
        "media_url": "http://pbs.twimg.com/.../photo.jpg",
...
```

Videos, on the other hand, have multiple formats, several of them being images.
I want to select the best file that is an actual video, because that is what is
stored in the archive. In this case, I want to ignore `media_url`. The
`video_info.variants` field lists variations. The best file will be the variant
with the greatest `bitrate` field (if it has one at all). The URL is just the
`url` field.

```json
{
  "extended_entities" : {
    "media" : [
      {
        "type": "video",
        "video_info" : {
          "variants" : [
            {
              "bitrate" : "2176000",
              "content_type" : "video/mp4",
              "url" : "https://video.twimg.com/.../video.mp4",
...
```

Regardles of file type, to get a path to the file in the archive, I have to
parse the media URL, get the base file name of the path, append it to the ID of
the tweet, then join it with the media path in the archive. For example:

```
Base from URL:                         smtZI8u_MFMb0-ie.mp4
ID:                1783949913195159552
Path: tweets_media/1783949913195159552-smtZI8u_MFMb0-ie.mp4
```

### Content
Rendering the content of a tweet is interesting, because entities are used to
replace the raw text. Each entity has an `indices` field that is an array of two
integers, which indicates the range of characters (not bytes) in the raw text
that represents the entity.

```go
// Only includes entities I'm interested in.
type Entities struct {
	// I want to replace the shortened URL with
	// the actual URL.
	URLs []URL `json:"urls"`
	// I want to remove any media URLs entirely,
	// since they will be inlined anyway.
	Media []Media `json:"media"`
}

type URL struct {
	Indices [2]string `json:"indices"`
	...
}

type Media struct {
	Indices [2]string `json:"indices"`
	...
}
```

To handle these different types doing similar things, I define a `Replacer`
interface to be implemented by each entity type:

```go
type Replacer interface {
	ReplaceIndices() (int, int)
	ReplaceWith() string
}
```

To render the content, I round up all entities into an array as Replacers. This
array is sorted by the first index so that replacers are applied in the correct
order. Then, the content is built piece by piece:

```go
var content strings.Builder
for i, r := range replacers {
	// Last index of the previous replacer, or the
	// start of the text.
	z := 0
	if i > 0 {
		_, z = replacers[i-1].ReplaceIndices()
	}
	// First index of the current replacer.
	a, _ := r.ReplaceIndices()
	// Write the unreplaced text between the
	// previous and current replacer.
	if z <= a {
		content.WriteString(uslice(t.FullText, z, a))
	}
	// Write the replaced text.
	content.WriteString(r.ReplaceWith())
}
// Write the unreplaced text after the last
// replacer.
_, b := replacers[len(replacers)-1].ReplaceIndices()
content.WriteString(uslicex(t.FullText, b))
```

The `uslice` functions simply slice a string across characters instead of bytes:

```go
// Does the compiler recognize these patterns?
func uslice(s string, a, b int) string {
	return string([]rune(s)[a:b])
}
func uslicex(s string, a int) string {
	return string([]rune(s)[a:])
}
```

Finally, any leading and trailing whitespace is trimmed. The result is also run
through a text wrapper, since that's what I prefer, and don't want to have to do
it myself for every single post.

### Generate files

To build a tree of replies, I add all posts to a `briefs` map, inspect each
ReplyTo field, and add to the replied post's list:

```go
briefs := map[int]*Brief{}
for _, d := range data {
	// Convert Data to Brief.
	tw := d.Tweet
	id := atoi(tw.ID)
	briefs[id] = &Brief{
		ID:        id,
		ReplyTo:   atoi(tw.ReplyTo),
		ReplyName: tw.ReplyName,
		// "Mon Jan 02 15:04:05 -0700 2006"
		Date: tw.Date(),
		// Renders content.
		Content: tw.Content(),
		// Produces media.
		Media: tw.Media(),
	}
}

// Build reply tree.
for id, b := range briefs {
	if r, ok := briefs[b.ReplyTo]; ok {
		r.Replies = append(r.Replies, id)
	}
}
```

Next is to sort the posts by ID, which naturally sorts by date.

```go
ids := slices.Collect(maps.Keys(briefs))
slices.Sort(ids)
for _, id := range ids {
	brief := briefs[id]
	if briefs[brief.ReplyTo] != nil {
		// Is reply to another brief that was
		// handled previously.
		continue
	}
...
```

Self-reply posts are skipped, because I'll be combining threads of tweets into a
single post. Originally, I had threads as a distinct type of post with their own
formatting, but decided that this would be too difficult to handle during manual
processing. Not all related tweets are directly associated, so it's easier to
inspect if they're all next to each other on the same timeline.

To generate posts, I use a `postBuilder` struct. This contains a
`strings.Builder` along with needed context, like other posts and related media.

```go
type postBuilder struct {
	// Content of post file.
	b strings.Builder
	// Maps unique ID to post.
	briefs map[int]*Brief
	// Maps media object to file name.
	media map[*BriefMedia]string
}
```

When a post includes media, it needs to be bundled with the post. First, I have
a function that finds all media in the post tree:

```go
// Recursively populates a list of media files.
func (p *postBuilder) findMedia(brief *Brief, media *[]*BriefMedia) {
	if brief == nil {
		return
	}
	if brief.Media != nil {
		*media = append(*media, brief.Media)
	}
	for _, r := range brief.Replies {
		p.findMedia(p.briefs[r], media)
	}
}
```

Then, I decide whether the post should be a single file or a bundle based on the
presence of media:

```go
out := "../content/briefs"

var media []*BriefMedia
builder.findMedia(brief, &media)
if len(media) == 0 {
	out = filepath.Join(out, brief.Filename()+".md")
} else {
	out = filepath.Join(out, brief.Filename(), "index.md")
}
os.MkdirAll(filepath.Dir(out), 0755)
```

Then I have to locate and copy the media files:

```go
for i, m := range media {
	// ContentFile is already pointing to the
	// correct location of the media file in the
	// archive, starting from the directory of
	// tweets.js.
	contentPath := filepath.Join(filepath.Dir(os.Args[1]), m.ContentFile)

	// A simple number is used as the filename.
	name := fmt.Sprintf("%02d%s", i, m.Ext)
	outputPath := filepath.Join(filepath.Dir(out), name)

	// Open the input and output files, then copy.
	cin, _ := os.Open(contentPath)
	cout, _ := os.Create(outputPath)
	io.Copy(cout, cin)
	cin.Close()
	cout.Close()

	// Map the media to the filename so that the
	// builder knows which name to use when
	// generating references to the media.
	builder.media[m] = name
}
```

Finally, the content of the post is generated.

```go
func (p *postBuilder) build(brief *Brief) {
	// Generate front matter.
	p.b.WriteString("+++\n")
	fmt.Fprintf(&p.b, "title = %q\n", brief.Filename())
	fmt.Fprintf(&p.b, "date = %s\n", brief.Date.Format("2006-01-02 15:04:05"))
	p.b.WriteString("tags = []\n")
	if brief.ReplyTo > 0 {
		p.b.WriteString("[params]\n")
		fmt.Fprintf(&p.b, "replyto = \"https://twitter.com/%s/status/%d\"\n", brief.ReplyName, brief.ReplyTo)
	}
	p.b.WriteString("+++")

	// Write content.
	p.section(brief)

	p.b.WriteString("\n")
}
```

If the post is a reply to an external tweet, then a link to the tweet is
generated in the front matter, which will be rendered by the post template. This
is an addition to make manual processing a little easier.

Instead of producing posts for each tweet, the entire reply tree is flattened
and written together. The `section` method basically writes the Content field of
the post, generates markup for displaying media if there is any, then calls
`section` on each reply to the post.

For the name of each post file, I decided to just use the timestamp up to the
minute:

```go
func (b Brief) Filename() string {
	return b.Date.Format("200601021504")
}
```

This makes each post somewhat hard to identify at a glance within a file
explorer, but it's the best way to ensure that every file is unique.

After running this script, the final result is a lot of new files under
`content/briefs` of my blog's repository.

## Step three: the hard part
The final step is to manually walk through each generated post, and decide what
to with it.

The thing is, when I said "a lot", apparently I meant approximately **4,000**
(that's as many as four thousands, and that's terrible). Having already
processed over a hundred posts, "a lot" is starting to feel like an
understatement.

> no, lol
>
> --- Me, in response to "Just have an AI do it"

Anyway, here are some techniques I'm applying to make the process easier:

### Generate as much as possible
The front matter is already mostly filled in. I just need to replace the title,
fill in the tags, and remove a reply parameter, if present.

The content is mostly well-formatted. Threads of tweets are grouped into the
same file. Each tweet is a paragraph, and any media is placed in its own block.
Long lines are already wrapped to 80 characters. URLs may have to be adjusted,
but they're usually placed at the end of the tweet, so it's easy enough to move
them onto their own line.

Mentions are usually removed. Sometimes, if it feels relevant enough, I might
include a tweet from someone else as a quote. This might be the most manual
part, but it hasn't been too bad so far.

### Include context
Many posts are replies to tweets from other people, which wont appear in my
archive. It's good to have the context, so I render a link to the reply when
available. This is the purpose of the `replyto` parameter in the front matter:

```html
{{- if .Params.replyto }}
	<a href="{{.Params.replyto}}" target="_blank">⤷ Reply</a>
{{- end }}
```

This makes it very easy to view the tweet, and include it as a quote, if
desired.

### Make posts easy to view alongside each other
It's a lot easier to determine whether or not a number of tweets are related
when they're displayed right next to each other. I'm already rendering posts in
a timeline on the [briefs]({{% relref "briefs" %}}) page, so I just reuse that
by serving my blog locally.

### Make posts easy to edit
Since I'm viewing posts through my browser, I would like a streamlined way to
edit them. I'm not going to write a whole-ass browser extension for it, though.
After some research, I found that Firefox has a config that lets you use View
Source to open files in an external editor.

```js
// Enable external view-source editor.
user_pref("view_source.editor.external", true);
// Set path to preferred editor.
user_pref("view_source.editor.path", "/usr/bin/subl");
```

That's great, but I need to be able to get to the file in the first place.
Firefox will interpret `file://` links, but it requires some more configuration:

```js
// Enable policy.
user_pref("capability.policy.policynames", "localfilelinks");
// Needed for reasons I'm not interested in looking up.
user_pref("capability.policy.localfilelinks.checkloaduri.enabled", "allAccess");
// Add local Hugo-served blog to whitelist.
user_pref("capability.policy.localfilelinks.sites", "http://localhost:1313");
```

Then, when rendering a post locally, I've added a link to the actual file.
`target=_blank` forces the file to be opened in a new tab, which means I can
left-click instead of middle-click, which is slightly easier.

```html
{{- if hugo.IsDevelopment -}}
	<a href="file://{{.File.Filename}}" target="_blank">
		{{- .File.ContentBaseName -}}
	</a>
{{- end -}}
```

After opening the file, I can right-click and select View Page Source, and the
file now appears directly in my editor.

## Step four: conclude
After churning though a hundred of these posts at once, I got sick of the whole
thing for an entire day. So I've determind that I'll have to go through them
little by little. Surely I'll get through them all eventually. Oh, by the way,

Am I deleting my social media posts once I'm done?

No!

Will I stop posting there and only make posts here from now on?

No!

As long as a platform doesn't require me to upload my face and government ID
(for real, wtf), you'll still find me posting there. Consider this website to be
yet another mirror.

[twit]: https://twitter.com/Anaminus
[bsky]: https://bsky.app/profile/anaminus.bsky.social
[mast]: https://mastodon.gamedev.place/@anaminus
