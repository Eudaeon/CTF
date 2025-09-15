# Scrambled Payload

## Recon

VBS script.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace]
└─$ file payload.vbs               
payload.vbs: ASCII text, with very long lines (60674), with no line terminators
```

## Analysis

Code is obfuscated. It's like:

```vb
Set A = "{System.__ComObject}"
A.dataType = "bin.base64"
A.text = ""
```
