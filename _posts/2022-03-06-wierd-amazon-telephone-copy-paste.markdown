---
layout: post
title: "Phone number copied from Amazon is invalid?"
date:   2022-03-06
---

---

Sometimes when ordering online a website's form requires a phone number, for those cases I use the phone number registered on my Amazon's address book: I go to my Amazon's address book, select the number, copy the text and paste into the other form (yes I need something more efficient). 

Here is the thing: frequently, the website's validation code rejects the field with the phone number, failing the form validation and sometimes even clearing all the other fields forcing me to type everyting again. A few years ago I found the cause, a very simple one but new to me and documented here. (I found it just by pasting the clipboard value into a terminal).

The cause is Unicode.

Most phone number fields in web forms expect just plain numbers but Amazon wraps the phone number with a couple of invisible Unicode characters that are included when copying the selected number. What are those characters?

A website like amazon.com is available in a lot of languages but not all languages are read from left to right, so what happens when you have a paragraph in Arabic that includes a number? The text must be read from right to left but the number from left to right, how do you do that?

Of course Unicode has an answer to that question.

# LRE and PDF

In the fascinating [Unicode Standard Annex #9](https://unicode.org/reports/tr9/) we find the "Unicode Bidirectional Algorithm" and its relevant section: "Explicit Directional Embeddings". In that section we find the solution to the problem of how to specify numbers inside right-to-left languages: we must wrap the number inside two Unicode characters:
- Start: Code point U+202A (LEFT-TO-RIGHT EMBEDDING or LRE)
- End: Code point U+202C (POP DIRECTIONAL FORMATTING or PDF)

Those code points indicate to the text processor that the text between them should be displayed left-to-right, that's it. Being amazon.com a multilingual web application, it is essential that numbers are shown correctly independently of the language used so they wrap the numbers in the appropiate Unicode code points and when local or server code in other websites validate phone numbers as plain numbers without any Unicode characters as part of the string it triggers their invalid phone number code path. Guesses: validating the size of the value, validating a regex numbers class like "[0-9]".

It looks like this in a Python shell:

{% highlight python %}
>>> s = "‪123 1231234‬"
>>> s
'\u202a123 1231234\u202c'
{% endhighlight %}

Evidently it is not only useful for numbers. For example, if you have an Arabic text that quotes a Spanish one you will also need to wrap the Spanish text between code points LRE (U+202B) and PDF (U+202C). 









