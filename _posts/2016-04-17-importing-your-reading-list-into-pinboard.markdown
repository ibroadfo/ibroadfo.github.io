---
layout: post
title: "importing your icloud reading list into pinboard"
date: "2016-04-17 16:13:55 +0100"
---
Recently my icloud/safari reading list overflowed; i started hunting for an exit strategy.

Because apple likes to use their [plist format](https://en.wikipedia.org/wiki/Property_list) for most things, actually getting at the reading list items is fiddly; they're stored in the same binary Bookmarks.plist file as your regular in-browser bookmarks, but `File -> Export Bookmarks` only gives you the non-reading list ones.

Alex Chan [posted about](http://alexwlchan.net/2015/11/export-urls-from-safari-reading-list/) his method of extracting reading list urls using python. Importing into pinboard is [well-supported](https://pinboard.in/howto/#import), but you have to structure your urls in a way it can parse. My first few attempts at shoving the urls exported using the script were only half-successful, as pinboard expects bookmarks to have titles as well as urls, so i ended up with pages of UNTITLED links.

I held my nose, figured out how to get page titles out of the Bookmarks.plist file, randomly bashed my keyboard until python came out and [voilà](https://gist.github.com/ibroadfo/97200c641b4dada6185a8b20d5647a8f)!

You'll have to paste the resulting chunk of awful html into an exported Safari Bookmarks file; if you're importing everything into pinboard then just add it at the end, before the final closing `</HTML>` tag. If you want just your reading list and not the other bookmarks, go ahead and replace everything between `<H1>Bookmarks</H1>` and `</HTML>`.

##### Epilogue

After all of that drama, it turns out that Chrome imports reading list articles natively, so you could just go safari => chrome => pinboard; but then i wouldn't hit my blogging quota.
