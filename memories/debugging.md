# Debugging notes

## bash "syntax error: unexpected end of file" at last line
- Almost always an unclosed construct (`if`/`fi`, quote, `$(`).
- BUT a sneaky cause is **CRLF line endings**: `\r` makes bash read `then\r`/`fi\r`
  as non-keywords, so the `if` never closes -> EOF error.
- Diagnose: `sed -n 'A,Bp' file | od -c | head` and look for `\r \n`.
- Fix: `perl -i -pe 's/\r\n/\n/g' file`, then re-run `bash -n file`.
- Prevent: add `.gitattributes` with `*.sh text eol=lf`.
