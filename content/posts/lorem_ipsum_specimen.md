+++
title = "Lorem Ipsum: A Specimen"
author = ["Walker Griggs"]
description = "A demonstration essay exercising the marginalia theme — headers, sidenotes, margin notes, epigraphs, figures, tables, and code."
date = 2026-07-13
categories = ["essays"]
draft = true
+++

{{< epigraph author="Cicero, *De Oratore*" >}}
Neither can embellishment of language be found without arrangement and expression of thoughts, nor can thoughts be made to shine without the light of language.
{{< /epigraph >}}

{{< newthought >}}Lorem ipsum dolor{{< /newthought >}} sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.{{< sidenote >}}This is a sidenote — the Tufte answer to the footnote. It sits in the margin, level with its reference, so the eye never has to leave the passage. On a narrow screen it collapses and reveals on tap.{{< /sidenote >}} Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.

Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.{{< sidenote >}}A second note, numbered automatically by a CSS counter — you never type the number yourself.{{< /sidenote >}} Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

{{< newthought >}}A thematic break{{< /newthought >}} precedes this paragraph — pure whitespace, no rule — and because it no longer directly follows another paragraph, it resets to flush with no indent. Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium.

## On Arrangement

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam.{{< marginnote >}}A margin note is a sidenote without a number — an aside, not a citation. The ⊕ is only shown on small screens as its toggle.{{< /marginnote >}} Eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo.

Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt. Neque porro quisquam est, qui dolorem ipsum quia dolor sit amet, consectetur, adipisci velit.

<figure>
<svg viewBox="0 0 320 44" role="img" aria-label="A sparkline of an example time series" xmlns="http://www.w3.org/2000/svg">
  <polyline fill="none" stroke="#111111" stroke-width="1"
    points="0,30 20,26 40,31 60,20 80,24 100,14 120,22 140,11 160,18 180,9 200,16 220,7 240,13 260,5 280,12 300,3 320,10"/>
  <circle cx="300" cy="3" r="2" fill="#a00000"/>
</svg>
<figcaption>A sparkline: a small, intense, word-sized graphic. Its data-ink ratio approaches one — all data, no chartjunk.</figcaption>
</figure>

### A Digression in the Third Degree

Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur,{{< sidenote image="/img/zettelkasten_rhizomes_and_you/zettel_1.webp" alt="A note card" >}}A sidenote can carry a small image, sized to the margin — a caption, a diagram, or a scrap of evidence set beside the text.{{< /sidenote >}} vel illum qui dolorem eum fugiat quo voluptas nulla pariatur.

At vero eos et accusamus et iusto odio dignissimos ducimus qui blanditiis praesentium voluptatum deleniti atque corrupti:

- Temporibus autem quibusdam et aut officiis debitis
- Aut rerum necessitatibus saepe eveniet
- Ut et voluptates repudiandae sint et molestiae non recusandae

Itaque earum rerum hic tenetur a sapiente delectus, ut aut reiciendis voluptatibus maiores alias consequatur aut perferendis doloribus asperiores repellat.

## On Expression

{{< newthought >}}Et harum quidem{{< /newthought >}} rerum facilis est et expedita distinctio. Nam libero tempore, cum soluta nobis est eligendi optio cumque nihil impedit quo minus id quod maxime placeat facere possimus.{{< sidenote >}}The small-caps opener marks a new movement in the argument without the heaviness of another heading.{{< /sidenote >}}

<figure>
<img src="/img/data_preservation_alfs_room_and_spicy_p/blinkenlights.webp" alt="Rack of blinking status lights">
<figcaption>A raster figure in the main column, held to the measure. Colour is welcome here — it is a figure, not prose.</figcaption>
</figure>

> Omnis voluptas assumenda est, omnis dolor repellendus. Temporibus autem quibusdam et aut officiis debitis aut rerum necessitatibus.

A table, ruled only horizontally — no grid, no boxes:

| Author        | Work            | Year  |
| ------------- | --------------- | ----- |
| Cicero        | De Oratore      | 55 BC |
| Manutius      | De Aetna        | 1495  |
| Tufte         | Beautiful Evidence | 2006 |

And a specimen of code, held to the measure and scrolling if it must:

```go
func Measure(text string) int {
    // every glyph earns its place
    return len([]rune(text))
}
```

Nam libero tempore, cum soluta nobis est eligendi optio cumque nihil impedit quo minus id quod maxime placeat.{{< sidenote >}}A final note, to close the set.{{< /sidenote >}} Temporibus autem quibusdam et aut officiis debitis aut rerum necessitatibus saepe eveniet ut et voluptates repudiandae sint et molestiae non recusandae.
