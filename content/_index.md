---
title: "Welcome to Jinwoo and Lambda :cat: 's blog"
description: "Main page of pmnxis.github.io"
---

{{< lead >}} Welcome to Jinwoo and Lambda :cat: 's blog {{< /lead >}}

<img src="LambdaClub.jpg">

This blog mainly cover with Linux, Rust, Embedded, and electronic circuits, and articles in Korean and English are mixed. And sometimes my cat `Lambda`**λ** appears frequently, so I would appreciate it if you liked it. Nyaa

```go
{{ define "main" }}
  <section class="mt-8">
    {{ range .Pages }}
      <article class="pb-6">
        <a class="flex" href="{{ .Params.externalUrl }}">
          <div class="mr-3 text-3xl text-neutral-300">
            <span class="relative inline-block align-text-bottom">
              {{ partial "icon.html" .Params.icon }}
            </span>
          </div>
          <div>
            <h3 class="flex text-xl font-semibold">
              {{ .Title }}
            </h3>
            <p class="text-sm text-neutral-400">
              {{ .Description }}
            </p>
          </div>
        </a>
      </article>
    {{ end }}
  </section>
{{ end }}
```