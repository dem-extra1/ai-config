# Debugging notes

## ARDI/iterate: must poll for new review after pushing
- After pushing fixes during an iterate loop, DON'T declare "clean" based on
  the previous review. A new push triggers a new auto-review.
- Poll until a review note appears that references your latest commit SHA.
- Wait ~30-60s, then check. If nothing after ~2min, check pipeline status.
- The iterate skill now has explicit polling instructions for both GitHub and GitLab.

## bash "syntax error: unexpected end of file" at last line
- Almost always an unclosed construct (`if`/`fi`, quote, `$(`).
- BUT a sneaky cause is **CRLF line endings**: `\r` makes bash read `then\r`/`fi\r`
  as non-keywords, so the `if` never closes -> EOF error.
- Diagnose: `sed -n 'A,Bp' file | od -c | head` and look for `\r \n`.
- Fix: `perl -i -pe 's/\r\n/\n/g' file`, then re-run `bash -n file`.
- Prevent: add `.gitattributes` with `*.sh text eol=lf`.
