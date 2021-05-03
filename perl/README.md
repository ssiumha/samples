# Perl

perl tip and tricks

## -0
```
find . -name '*-' -print0 | perl -0ne unlink
```

## ..
```
perl -ne 'print if /----BEGIN PGP/../----END PGP/'
```

## \K

```
perl -pale 's/^Form:\K.*/ Changed: <abc\@example>/'
# > From: def@example
# > From: Changed <abc@example>
```

## $ENV

```
re="'" perl -lane 'print if $F[4] =~ /$ENV{re}/' /etc/password
```

## -MRegexp::Common

```
ip address list \
  | perl -MRegexp::Common -lne 'print $1 if /($RE{net}{IPv4})/'
```
