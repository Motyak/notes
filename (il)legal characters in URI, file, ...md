

## Legal characters in URI:

```
A-Z
a-z
0-9
-
.
_
~
:
/
?
#
[
]
@
!
$
&
'
(
)
*
+
,
;
=
%
```

## Illegal characters in file (Windows, Linux and macOS)

```
< (less than)
> (greater than)
: (colon - sometimes works, but is actually NTFS Alternate Data Streams)
" (double quote)
/ (forward slash)
\ (backslash)
| (vertical bar or pipe)
? (question mark)
* (asterisk)
```

## When converting from valid URI/URL to filename, should encode the following characters :

```
: => %3a
/ => %2f
? => %3f
* => %2a
% => %25
```

bash script to do so using `sed` :
```bash
function uri_to_filename {
	sed -e 's/%/%25/g' \
		-e 's/\//%2f/g' \
		-e 's/:/%3a/g' \
		-e 's/?/%3f/g' \
		-e 's/\*/%2a/g' <<< "$@"
}
```
