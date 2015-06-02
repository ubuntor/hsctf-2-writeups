# Connect the Colors

## Writeup
We notice that even though `toggle` checks for bounds, it doesn't return for
invalid bounds. This happens to be a `.bss` overflow vuln. If we try "negative"
indices, we can overwrite a `.got.plt` entry with the address of `print_flag()`,
giving us the flag next time the overwritten function is called.
