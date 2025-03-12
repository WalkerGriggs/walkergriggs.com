+++
title = "PSSH; Primordial Soup of Secure-ish Headers"
author = ["Walker Griggs"]
date = 2024-10-16
categories = ["talks"]
draft = false
creator = "Emacs 29.4 (Org mode 9.6.15 + ox-hugo)"
weight = 2004
+++

Given at [April '24 SF Video Technology](https://www.meetup.com/sf-video-technology/events/298593592/) meetup and [Demuxed '24](https://2024.demuxed.com)


## Recording {#recording}

<iframe src="https://www.youtube-nocookie.com/embed/ZMTkdIb7RIc" allowfullscreen title="YouTube"></iframe>


## Slides {#slides}

<iframe id="pdf" src="/pdf/demuxed_2024.pdf" frameborder="0"></iframe>


## Abstract {#abstract}

Consider our friendly, neighborhood PSSH box.

The semantics are simple -- to identify encryption keys -- but, as with any permissive specification, there’s a lot more going on than meets the eye. In some cases, they contain deeply nested little-endian UTF16 XML. In others, we’ll find protocol buffers containing base64-encoded JSON. In all cases, they have surprising amount of personality.

In this talk, we will dive deep into several PSSH boxes, dissecting them bit by bit across various popular DRM schemes. Along the way, we will

1.  Explore the history of the PSSH box and how it mirrors the evolution of DRM standards.
2.  Discover how each provider has imparted their own company idioms onto the loosely-defined PSSH payload.
3.  Identify where the decisions of one provider impacted the rest.


## Transcript {#transcript}

I want everyone to close their eyes for a moment. _(Maybe not you, reading this.)_

We’re going to a happy place. It’s a crisp, April morning, the sun is shining through your window, and you have a warm cup of coffee. You sit down at your desk — as everyone does on a crisp spring morning — to lovingly scroll through your favorite manifests. Some have interstitials, some are commented, some are live multi-track, and a few have templated segments. One, though, has a 1200 character base64 PSSH box crammed into HLS key tag. This does spark joy, but, being the inquisitive video engineer you are, you have to know: “what could possibly require 900 bytes to convey 1x 16 byte key ID?”

My name is Walker Griggs, I’m a video engineer at Mux, and with our time together, we’ll attempt to answer that question. We’ll tear apart a few PSSH boxes bit by bit, we’ll discover a few interesting design choices, and we’ll discuss what those choices mean for license request semantics. My hope is that we’ll walk away today appreciating just how quirky, idiomatic, and character rich these boxes are.

Before we get hands on with some hex dumps, we should first at least cover some of the box basics. The PSSH, or Protection System Specific Header, is a flexible and general box that contains the data needed by a content protection system to play back the content -- most often just the encryption key ID. The format of that data is specified by a DRM system identifier. As the name suggests, it’s specific for a given DRM scheme and is most often used by a scheme’s CDM (content decryption module) to generate license requests.

Unlike the TrackEncryption box, the PSSH is optional regardless if the content is protected or not. Also different from the TrackEncryption box, there can be multiple PSSH boxes if the content is playable under multiple schemes. Maybe most important for this talk, it’s outside any security boundary. Please don't sue me.

```nil
aligned(8) class PSSH extends FullBox('pssh', version, flags=0)
{
  unsigned int(8) [16] SystemID;
  if (version > 0)
  {
    unsigned int(32) KID_COUNT;
    {
      Unsigned int(8)[16] KID;
    } [KID_count];
  }
  unsigned int(32) DataSize;
  unsigned int(8) [DataSize] Data;
}
```

There isn’t much to see in the spec either. The PSSH MUST contain a system ID, possibly a list of key ids depending on the box version, and then the arbitrary binary blob.

In preparing for this talk, I’ve reviewed many many PSSH boxes from major streaming services, live venues, and creator networks. The vast majority (maybe even all) of the boxes I’ve surveyed the list of key IDs have been omitted. So in practice, the PSSH is a scheme ID followed by arbitrary data.

Let’s pull one apart.

This particular example is pulled from an HLS key tag and contains a Widevine PSSH. You can see a few HLS specific bits like the key format, but otherwise it’s a base64 blob packed into a URI.

```nil
#EXT-X-KEY:METHOD=SAMPLE-AES,URI="data:text/plain;base64,AAAAknBzc2gAAAAA7e+LqXnWSs6jyCfc1R0h7QAAAHISEMtCZvRtSxeihjNfmx+ZfGQiWGV5SmhjM05sZEVsa0lqb2lPVFUyTlRjMU56RXpNRGt6T0RjM056WTNJaXdpZG1GeWFXRnVkRWxrSWpvaU9UVTJOVGM0TlRreU1qRXdNakl6TVRFeEluMD1I88aJmwY=",KEYID=0xcb4266f46d4b17a286335f9b1f997c64,KEYFORMAT="urn:uuid:edef8ba9-79d6-4ace-a3c8-27dcd51d21ed",KEYFORMATVERSION="1"
```

If we decode that URI, we’ll find the full PSSH box. We can read through it following the spec. If we read the box in 4 byte increments, we know it’s 146 bytes long, it is indeed a PSSH box, it’s version 0, and it’s specific to Widevine. We know it’s a version 0 PSSH box so we know that the following 4 bytes describe the payload length — 114 bytes. After that, though, we’re in the wild west.

```nil
xxd -g1 widevine.bin

00000000: 00 00 00 92 70 73 73 68 00 00 00 00 ed ef 8b a9  ....pssh........
00000010: 79 d6 4a ce a3 c8 27 dc d5 1d 21 ed 00 00 00 72  y.J...'...!....r
00000020: 12 10 cb 42 66 f4 6d 4b 17 a2 86 33 5f 9b 1f 99  ...Bf.mK...3_...
00000030: 7c 64 22 58 65 79 4a 68 63 33 4e 6c 64 45 6c 6b  |d"XeyJhc3NldElk
00000040: 49 6a 6f 69 4f 54 55 32 4e 54 63 31 4e 7a 45 7a  IjoiOTU2NTc1NzEz
00000050: 4d 44 6b 7a 4f 44 63 33 4e 7a 59 33 49 69 77 69  MDkzODc3NzY3Iiwi
00000060: 64 6d 46 79 61 57 46 75 64 45 6c 6b 49 6a 6f 69  dmFyaWFudElkIjoi
00000070: 4f 54 55 32 4e 54 63 34 4e 54 6b 79 4d 6a 45 77  OTU2NTc4NTkyMjEw
00000080: 4d 6a 49 7a 4d 54 45 78 49 6e 30 3d 48 f3 c6 89  MjIzMTExIn0=H...
00000090: 9b 06
```

If we’re taking this investigation from first principles and pretending like we don’t already know how this data is encoded, the context clue here is that Widevine is a Google product and Google **loves** to encode structured data with Protocol Buffers.

If you haven’t worked with protocol buffers before, they’re a way to serialize structured data into a tightly packed, non-canonical, wire format. It uses a few tricks like optional fields, variable width integers, and a tag-length-value scheme to optimize for space efficiency. The trouble is, to decode a buffer effectively, the receiver needs the message definition because encoded protobuf discards field names entirely. We can deduce field indices, types, and values, but not semantics.

For example, the message Foobar contains one, int64 field called A at index 1. If we set A to 150 and encode, we get the following three bytes: `08 96 01`.

If we look at our Widevine payload (everything after byte 32), the first two bytes if our payload (`12 10`) are clever varints that tell us the field is the second index, is a variable length type like a string, list, byte array, or nested proto, and it’s 16 bytes long. See, [Protobuf's encoding documentation](https://protobuf.dev/programming-guides/encoding/) for more detail.

Any time I’m working with MPEG CENC, 16 byes should immediately signal initialization vector, key id, or key material. Given that the PSSH is squarely outside of our security boundary, we can safely rule out the latter. This is likely our key ID.

I wont subject you to parsing the full proto, so we can use a tool like Protoscope here to fast forward this process and to decode the entire blob for us. You’ll see that we did get the first field right — index two is a single, 16 byte key id. Field four looks like even MORE base64, and field 9 looks like a timestamp or integer or something.

```nil
dd status=none skip=32 bs=1 if=widevine.bin | protoscope

2: { `cb4266f46d4b17a286335f9b1f997c64` }
4: { "eyJhc3NldElkIjoiOTU2NTc1NzEzMDkzODc3NzY3IiwidmFyaWFudElkIjoiOTU2NTc4NTkyMjEwMjIzMTExIn0=" }
9: 1667392371
```

Conveniently, the Shaka packager hosts their version of the PSSH spec. It’s not the official Widevine, top secret, internal spec, but it’ll more than help us out here. Index 2 is a key ID. Check. Index 9 is actually our protection scheme, represented by it’s FourCC uint32 value. In our case, this media is encrypted with the CBCS scheme

```nil
// (no comment, but key_id tells us everything we need)
repeated bytes key_id = 2;

// A content identifier, specified by content provider.
optional bytes content_id = 4;

// Protection scheme identifying the encryption algorithm.
// Represented as one of the following 4CC values: 'cenc' (AES-CTR),
// 'cbc1' (AES-CBC), 'cens' (AES-CTR subsample), 'cbcs'
// (AES-CBC subsample).
optional uint32 protection_scheme = 9;
```

Index 4 though, brings us to our first, meaningful design decision. Widevine PSSH actually have fields for key id ****and**** content ID. Earlier I mentioned that an important detail to pay attention to is the distinction between holistic content and specific data. Widevine have chosen to identify content not by a series of keys or tracks, but by the broader content. This lets Widevine support multi-key license requests, but you could argue that, semantically, that license licenses the content holistically. Your client is allowed to view the content given your playback environment and various licensing factors. Keep this in mind as we look at the next two schemes.

What, then, is in this deeply nested base64? Why it’s JSON of course! The content ID is set by the content provider. In this case, the provider identifies content with an asset and variant ID pair. And this brings us to the bottom one our Widevine PSSH. Base64 encoded JSON, encoded into a protocol buffer, boxed up in a PSSH, base64’d again, and string formatted into a manifest. It’s boxes all the way down and you can’t convince me otherwise.

```nil
dd status=none skip=52 count=88 bs=1 if=widevine.bin | base64 -d | jq

{
  "assetId": "956575713093877767",
  "variantId": "956578592210223111"
}
```

Time for PlayReady.

It’s almost Halloween but I’ll give you a jump scare warning anyway. I wasn’t kidding about an 850 byte key tag. I’ll be honest though, at this point I can’t even see the code anymore.

Where Google has used protobuf to encode a their full payload, Microsoft has invented a full serial encoding. To their credit, they acknowledge that this is not part of their security boundary and document it thoroughly. We don’t have to make many informed guesses. The top level PlayReady Object is just a container with a total length and number of elements. The sub elements are called PlayReady Object Records and each record has a type, length, and arbitrary byte array. The type can either be an embedded license store or the PlayReady Header, but for the happy-path use case you’ll always see the PRH. At this point in the talk, you should get suspicious any time we give developers space for arbitrary data. At the very least, dejavu.

```nil
PlayReady Object
-> Length
-> PlayReady Object Record Count
-> PlayReady Object Records
                -> Record Type (PlayReady Header)
                -> Record Length
                -> Record Value
```

I spent some time coming up with a way to effectively describe the contents of the PRH record for you, but I’ll tell you this joke instead. An Intel 8008, the universal coded character set TWO, and a standard generalize markup language walk into a bar…

The punchline reads something like "UTF16 little endian, XML". It’s always fun how a few decisions made by Intel architect the 8000 series, Microsoft developing Windows NT, and W3C creating a markup language for the web trickle into out streaming tech.

But I digress. Let’s take a quick pass through this PSSH box. Playready PSSHs stored in manifest are actually stripped of the fields and ONLY include their data payload. That makes for a great shortcut for this talk. We start with a little endian integer size of 834 bytes.

This PSSH contains a single record of type 1 — that’s the PRH. Logically there are 824 bytes left.

```nil
xxd -g1 -s32 -l10 playready.bin

00000020: 42 03 00 00 01 00 01 00 38 03 00 00              B.......8.
```

PlayReady give us a myriad of things to look at in these objects. First and foremost, this is a version 4.3.0.0 PlayReadyHeader. Each header contains a protection info node which contains a list of keys. Multi-key support was added in 2015 in version 4.2.0.0, which is pretty recent as far as DRM is concerned.

```nil
dd status=none skip=42 bs=1 if=playready.bin | iconv -f UTF-16LE -t UTF-8 | xq

<WRMHEADER xmlns="http://schemas.microsoft.com/DRM/2007/03/PlayReadyHeader" version="4.3.0.0">
  <DATA>
    <PROTECTINFO>
      <KIDS>
        <KID ALGID="AESCBC" VALUE="igXBva1eUgNB45tKQrDx5w=="/>
          </KIDS>
        </PROTECTINFO>
  </DATA>
</WRMHEADER>
```

Each key is assigned an algorithm -- in this case, AES CBCS. PlayReady also packs with it the license acquisition url, domain service IDs, even room for custom attributes identify publishers, creation dates, etc. The important note here is, whereas Widevine gravitates towards identifying the content holistically, PlayReady is focused on a set of keys which may or may not encompass the content. Either way, there’s something to glean from “this is my content” and “these are the keys that protect my content”.

Now for our third and final: Fairplay.

How can I describe Apple’s approach to DRM except for “they just don’t." In classic Apple fashion, they march by the beat of their own drum. They do not use a PSSH anywhere. The spec said “optional” and they ran with it. To be fair, Apple supports a streaming key delivery which is effectively an arbitrary URI to identify single keys. You’ll often see these in manifests but they’re not strongly enforced.

```nil
skd://demuxed?keyId=bdc1058a5ead035241e39b4a42b0f1e7
```

In fact, when you look at HLS.js `onMediaEncrypted` implementation, you’ll notice that the function is one giant condition that starts with “if Fairplay." Everyone else, parse the PSSH.

```nil
if (
        initDataType === 'sinf' &&
        this.config.drmSystems[KeySystems.FAIRPLAY]
) {
        const sinf = base64Decode(JSON.parse(json).sinf);
        const tenc = parseSinf(new Uint8Array(sinf));
} else {
        const psshInfo = parsePssh(initData);
}
```

This makes more sense when you consider the track encryption box. The TENC, like PSSH, defines protection parameters. Unlike the PSSH though, the TENC is required for protected tracks. It specifies all sorts of defaults: default key id, default initialization vector, etc. If you consider Fairplay, this is all they need!

They don’t support CTR, so they don’t need a protection scheme indicator. They don’t support multi-key license requests nor content identification. As far as the track encryption box is concerned, they only ever need version 0. The crypt and skip byte blocks control the pattern which encryption is applies to subsamples, but HLS though has dictated “9 off, 1 on”. Apple has simplified their DRM story be using and vehemently stick to defaults across the board. If they want to rotate keys over a single track, you can override the encryption parameters for with the sample group description. In the end of the day, that’s everything the decoder needs to make license requests and playback protected content.

```nil
aligned(8) class TENC extends FullBox('tenc', version, flags=0)
{
  if (version!=0) {
    unsigned int(4) default_crypt_byte_block;
    unsigned int(4) default_skip_byte_block;
  }

  unsigned int(8)     default_isProtected;
  unsigned int(8)     default_Per_Sample_IV_Size;
  unsigned int(8)[16] default_KID;

  if (default_isProtected == 1 && default_Per_Sample_IV_Size == 0) {
    unsigned int(8) default_constant_IV_size;
    unsigned int(8)[default_constant_IV_size] default_constant_IV;
  }
}
```
