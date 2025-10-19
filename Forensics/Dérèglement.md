# Dérèglement

## Challenge description

A Microsoft Office document (`2021-fcsc-reglement_de_participation.docx`) was corrupted during editing. The task is to recover the file contents and retrieve the flag.

## Analysis steps

1. **Extract embedded files / unpack the DOCX**

```bash
# run binwalk to extract embedded/carved files
binwalk -e /path/to/2021-fcsc-reglement_de_participation.docx
```

`binwalk` will create a directory named something like `_2021-fcsc-reglement_de_participation.docx.extracted`. Inside that directory you should find a `word` folder containing `document.xml`.

2. **Locate the `document.xml` file**

```bash
# change into the extracted dir (name depends on binwalk's output)
cd _2021-fcsc-reglement_de_participation.docx.extracted/word
ls -lah
# you should see document.xml (or similarly named xml fragments)
```

3. **Search for the flag string(s)**

Because `document.xml` contains many XML tags and formatting markup, use `grep` to extract lines containing the token `FCSC`. Use `-a` to treat binary files as text and `-n` to show line numbers.

```bash
# simple search showing context
grep -a -n "FCSC" document.xml

# narrow to only the matching strings (Perl-compatible regex)
grep -a -Po "FCSC\{.*?\}" document.xml

```

## Findings

* By extracting the DOCX with `binwalk -e` and searching `word/document.xml`, the flag string was located.
* **Recovered flag:** `FCSC{9bc5a6d51022ac}`

---
