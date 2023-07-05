# Strings, bytes, runes and characters in Go

以前的博客解释了slices在Go中怎么工作 [previous blog post](https://blog.golang.org/slices)， 用了大量的示例说明了其实现背后的机制。在此背景下，这篇文章讨论了 Go 中的字符串。首先，字符串对于一篇博客文章似乎很简单，但是要很好的使用，不仅需要知道它们是怎么工作，而且需要知道不同的介于byte, character, rune, 不同的介于Unicode和UTF-8，不同的介于string和string literal，之间细微的区别。

处理该主题的方式是将其视为对常见问题的思考，“When I index a Go string at position n, why don't I get the nth character?” 正如你所见，这个问题引导我们了解文本是如何工作的大量细节。

著名的Joel Spolsky's 博客对其中一些问题进行的出色的介绍，与Go无关, [The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](http://www.joelonsoftware.com/articles/Unicode.html). 他提出的许多观点将在这里得到回应。

## What is a string?

让我们从一些基础知识开始。

在Go中，string是bytes只读的slice。如果你不确定什么是bytes的slice或它是怎么工作，请阅读[previous blog post](https://blog.golang.org/slices); 我们假设你知道。

在前面申明任意字节的字符串是重要的。没有要求Unicode text, UTF-8 text, 或其它预定义格式。就字符串的内容而言，它等同于bytes的slice。

这是string literal(稍后会介绍)，它使用 `\xNN` 符号定义包含特殊字节值的(peculiar value) string constant. (当然，bytes的范围包括00到FF)

```
const sample = "\xbd\xb2\x3d\xbc\x20\xe2\x8c\x98"
```

## Printing strings

Because some of the bytes in our sample string are not valid ASCII, not even valid UTF-8, printing the string directly will prorduce ugly output. The simple print statement

```
fmt.Println(sample)
```

produces this mess (whose exact appearance varies with the environment):

```
��=� ⌘
```

To find out what that string really holds, we need to take it apart and examine the pieces. There are several ways to do this. The most obvious is to loop over its contents and pull out the bytes individually, as in this `for` loop:

```
for i := 0; i < len(sample); i++ {
	fmt.Printf("%x ", sample)
}
```

As implied up front, indexing a string accesses individual bytes, not characters. We'll return to that topic in detail below. For now, let's stick with just the bytes. This is the output from the byte-by-byte loop:

```
bd b2 3d bc 20 e2 8c 98
```

Notice how the individual bytes match the hexadecimal escapes that defined the string.

A shorter way to generate presentable output for a messy string is to use the `%x` (hexadecimal) format verb of `fmt.Printf`. It just dumps out the sequential bytes of the string as hexadecimal digits, two per byte.

```
fmt.Printf("%x\n", sample)
```

Compare its output to that above:

```
bdb23dbc20e28c98
```

A nice trick is to use the "space" flag in that format, putting a space between the % and the `x`. Compare the format string used here to the one above,

```
fmt.Printf("% x\n", sample)
```

and notice how the bytes come out with spaces between, making the result a little less imposing:

```
bd b2 3d bc 20 e2 8c 98
```

There's more. The `%q` (quoted) verb will escape any non-printable byte sequences in a string so the output is unambiguous.

```
fmt.Printf("%q\n", sample)
```

This technique is handy when much of the string is intelligible as text but there are peculiarities to root out; it produces:

```
"\xbd\xb2=\xbc ⌘"
```

if we squint at that, we can see that buried in the noise is one ASCII equals sign, along with a regular space, and at the end appears the well-known Swedish "Place of Interest" symbol. That symbol has Unicode value U+2318, encoded as UTF-8 by the bytes after the space (hex value 20): `e2 8c 98`.

If we are unfamiliar or confused by strange values in the string, we can use the "plus" flag to the %q verb. This flag causes the output to escape not only non-printablee sequences, but also any non-ASCII bytes, all while interpreting UTF-8. The result is that it exposes the Unicode values of properly formatted UTF-8 that represents non-ASCII data in the string:

```
fmt.Printf("%+q\n", sample)
```

With that format, the Unicode value of the Swedish symbol shows up as a \u escape:

```
"\xbd\xb2=\xbc \u2318"
```

These printing techniques are good to know when debugging the contents of strings, and will be handy in the discussion that follows. It's worth pointing out as well that all these methods behave exactly the same for byte slices as they do for strings.

Here's the full set of printing options we've listed, presented as a complete program you can run (and edit) right in the browser:

```
package main

import "fmt"

func main() {
    const sample = "\xbd\xb2\x3d\xbc\x20\xe2\x8c\x98"

    fmt.Println("Println:")
    fmt.Println(sample)

    fmt.Println("Byte loop:")
    for i := 0; i < len(sample); i++ {
        fmt.Printf("%x ", sample[i])
    }
    fmt.Printf("\n")

    fmt.Println("Printf with %x:")
    fmt.Printf("%x\n", sample)

    fmt.Println("Printf with % x:")
    fmt.Printf("% x\n", sample)

    fmt.Println("Printf with %q:")
    fmt.Printf("%q\n", sample)

    fmt.Println("Printf with %+q:")
    fmt.Printf("%+q\n", sample)
}
```

## UTF-8 and string literals

As we saw, indexing a string yields its bytes, not its characters: a string is just a bunch of bytes. That means that when we store a character value in a string, we store its byte-at-a-time representation. Let's look at a more controlled example to see how that happens.

Here's a simple program that prints a string constant with a single character three different ways, once as a plain string, once as an ASCII-only quoted string, and once as individual bytes in hexadecimal. To avoid any confusion, we create a "raw string", enclosed by back quotes, so it can contain only literal text. (Regular strings, enclosed by double quotes, can contain escape sequences as we showed above.)

```
func main() {
    const placeOfInterest = `⌘`

    fmt.Printf("plain string: ")
    fmt.Printf("%s", placeOfInterest)
    fmt.Printf("\n")

    fmt.Printf("quoted string: ")
    fmt.Printf("%+q", placeOfInterest)
    fmt.Printf("\n")

    fmt.Printf("hex bytes: ")
    for i := 0; i < len(placeOfInterest); i++ {
        fmt.Printf("%x ", placeOfInterest[i])
    }
    fmt.Printf("\n")
}
```

The output is:

```
plain string: ⌘
quoted string: "\u2318"
hex bytes: e2 8c 98
```

which reminds us that the Unicode character value U+2318, the "Place of Interest" symbol ⌘, is represented by the bytes `e2` `8c` `98`, and that those bytes are the UTF-8 encoding of the hexadecimal value 2318.

It may be obvious or it may be subtle, depending on your familiarity with UTF-8, but it's worth taking a moment to explain how the UTF-8 representation of the string was created. the simple fact is: it was created when the source code was written.

Source code in Go is defined to be UTF-8 text; no other representation is allowed. That implies that when, in the source code, we write the text

```
`⌘`
```

the text editor used to create the program places the UTF-8 encoding of the symbol  ⌘ into the source text. When we print out the hexadecimal bytes, we're just dumping the data the editor placed in the file.

In short, Go source code is UTF-8, so the source code for the string literal is UTF-8 text. If that string literal contains no escape sequences, which a raw string cannot, the constructed string will hold exactly the source text between the quotes. Thus by definition and by construction the raw string will always contain a valid UTF-8 representation of its contents. Similarly, unless it contains UTF-8-breaking escapes like those from the previous section, a regular string literal will also always contain valid UTF-8.

Some people think Go strings are always UTF-8, but they are not: only string literals are UTF-8. As we showed in the previous section, string values can contain arbitrary bytes; as we showed in this one, string literals always contain UTF-8 text as long as they have no byte-level escapes.

## Code points, characters, and runes

We've been very careful so far in how we use the words "byte" and "character". That's partly because strings hold bytes, and partly because the idea of "character" is a little hard to define. The Unicode standard uses the term "code point" to refer to the item represented by a single value. The code point U+2318, with hexadecimal value 2318, represents the symbol  ⌘. 

To pick a more prosaic example, the Unicode code point U+0061 is the lower case letter 'A': a.

But what about the lower case grave-accented letter 'A', à? That's a character, and it's also a code point (U+00E0), but it has other representations. For example we can use the "combining" grave accent code point, U+0300, and attach it to the lower case letter a , U+0061, to create the same character  à. In general, a character may be represented by a number of different sequences of code points, and therefore different sequences of UTF-8 bytes.

The concept of character in computing is therfore ambiguous, or at least confusing, so we use it with care. To make things dependable, there are *normalization* techniques that guarantee that a given character is always represented by the same code points, but that subject takes us too far off the topic for now. A later blog post will explain how the Go libraries address normalization.

"Code point" is a bit of a mouthful, so Go introduces a shorter term for the concept: rune. The term appears in the libraries and source code, and means exactly the same as "code point", with one interesting addition.

The Go language defines the word `rune` as an alias for the type int32, so programs can be clear when an integer value represents a code point. Moreover, what you might think of as a character constant is called a rune constant in Go. The type and value of the expression

```
'⌘'
```

is `rune` with integer value 0x2318.

To summarize, here are the salient points:

- Go source code is always UTF-8.
- A string holds arbitrary bytes.
- A string literal, absent byte-level escapes, always holds valid UTF-8 sequences.
- Those sequences represent Unicode points, called runes.
- No guarantee is made in Go that characters in strings are normalized.

## Range loops

Besides the axiomatic detail that Go source code is UTF-8, there's really only one way that Go treats UTF-8 specially, and that is when using a `for range` loop on a string.

We've seen what happens with a regular `for` loop. A `for range` loop, by contrast, decodes one UTF-8-encoded rune on each iteration. Each time around the loop, the index of the loop is the starting position of the current rune, measured in bytes, and the code point is its value. Here's an example using yet another handy `Printf` format, %#U, which shows the code point's Unicode value and its printed representation:

```
    const nihongo = "日本語"
    for index, runeValue := range nihongo {
        fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
    }
```

The output shows how each code point occupies multiple bytes:

```
U+65E5 '日' starts at byte position 0
U+672C '本' starts at byte position 3
U+8A9E '語' starts at byte position 6
```

## Libraries

Go's standard library provides strong support for interesting UTF-8 text. If a `for range` loop isn't sufficient for your purposes, chances are the facility you need is provided by a package in the library.

The most important such package is `unicode/utf8`, which contains helper routines to validate, disassemble, and reassemble UTF-8 strings. Here is a program equivalent to the `for range` example above, but using the `DecodeRuneInString` function from that package to do the work. The return values from the function are the rune and its width in UTF-8-encoded bytes.

```
    const nihongo = "日本語"
    for i, w := 0, 0; i < len(nihongo); i += w {
        runeValue, width := utf8.DecodeRuneInString(nihongo[i:])
        fmt.Printf("%#U starts at byte position %d\n", runeValue, i)
        w = width
    }
```

Run it to see that it performs the same. The `for range` loop and `DecodeRuneInString` are defined to produce exactly the same iteration sequence.

Look at the  [documentation](https://go.dev/pkg/unicode/utf8/) for the unicode/utf8 package to see what other facilities it provides.



