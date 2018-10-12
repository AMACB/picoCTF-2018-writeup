# Forensics Warmup 2

> Hmm for some reason I can't open this PNG? Any ideas?

> Hints:
> How do operating systems know what kind of file it is? (It's not just the ending!  
> Make sure to submit the flag as picoCTF{XXXXX}

When we try to open this file as a PNG image, it fails. The hint suggests that the file might not actually be a PNG file.
We can either try guessing file extensions, or we can use the unix command `file`:

```
$ file flag.png
flag.png: JPEG image data, JFIF standard 1.01, resolution (DPI), density 75x75, segment length 16, baseline, precision 8, 909x190, frames 3
```

Aha! The file is actually a JPEG file. Renaming the file to a `.jpeg` (or `.jpg`) allows us to open it and get the flag:

> `picoCTF{extensions_are_a_lie}`