LPF_GS(1) General Commands Manual LPF_GS(1)

## NAME

lpf_gs - printer filter to convert Postscript to PCL


## SYNOPSIS

lpf_gs [-cdhijlnvw] [-h host] [-i indent] [-j job] [-l length] -n user [-w 
       width] acct-file


## DESCRIPTION

lpf_gs is an lpd(8) accounting filter that takes a PostScript, PDF or
text file via stdin and converts it to an output format based on the
calling name of the program using Ghostscript. Accouting information
is logged to acct-name.

lpf_gs is based on the default lpd(8) accounting filter lpf, though
written in sh(1) rather than C. The options are as follows:

```
 -c
    Provided for backwards compatibility. Ignored.
 -d
    Enable debug.
 -h
    Print to the host (ip or hostname) specified.
 -i
    Provided for backwards compatibility. Ignored.
 -j
    job name to use when logging.
 -l
    Provided for backwards compatibility. Ignored.
 -n
    user who printed the job.
 -v
    Print the version.
 -w
    Provided for backwards compatibility. Ignored.
```

## CREATING ACCOUNTING FILTERS

Accounting filters are created by copying or making a symlink to
lpf_gs and giving it the name of a Ghostscript text filter.

The calling name of the filter must be in the form of lpf_device where
device is an output device known by Ghostscript. *E.g.*, lpf_pxlmono.

Using the calling name is necessary because printcap(5) definitions
only allow specifying the accounting filter path and name. The few
parameters sent to the filter are specified and sent by lpd(8). Using
the calling name to specify the output device works around this
limitation.

In most cases just a couple of links are required. Most printers
accept one or more of just a few printer-control languages (PCLs) or
page-description languages (PDLs).

The most common are:
```
  ASCII + control flow (dot-matrix and line printers)
  PCL (typically PCL3 - PCL6)
  PostScript
```

Check your printer manual for details about which PCLs or PDLs your
printer accepts.


## ENVIRONMENT
     LPF_GS_DEBUG		  variable enables debugging.


## FILES
```
/path_to_lpf_device/gs_init
    Postscript configuration file read by lpf_gs if found in the same directory
    as the symlinked filter.
    cf Ghostscript documentation for more details about using gs_init.
```


## EXIT STATUS

lpf_gs utility exits with one of the following values:

```
 0
    lpf_gs printed successfully.
 1
    Errors were detected. Try again.
```


## EXAMPLES

To use an lpd(8) definition with lpf_gs you'll need an entry in the
system printcap(5) file specifying the accounting filter.

```
  lp|local line printer:\
        :af=/var/log/lpd-acct:\          # accounting file (optional)
        :if=/usr/local/bin/lpf_pxlmono:\ # a link to lpf_<device>
        :lf=/var/log/lpd-errs:\          # error log
        :lp=9100@192.168.68.10:\         # printer raw port
        :mx#0:\                          # turn-off max file size
        :sd=/var/spool/output/lpd:\      # spool directory
        :sh:                             # suppress burst-page header
```

The definition above will convert the input print job to PCL 6
(monochrome), which many printers will accept.

The "lp:9100@192.168.68.10" line directs lpd(8) to send the output of
the accounting filter to port 9100 on the remote system.

Port 9100 is a common port for printers to listen for raw streams.
Port 515 is the default port for lpd(8) services which may have their
own accounting filters which may convert the file in unexpected ways.

Use a remote lpd(8) only when you're sure the remote server can
process the file and use lpr(8) with an appropriate printcap(5) entry.

A list of available output devices can be found by executing gs -h.
The following command creates a link to lpf_gs that converts from
postscript to PCL 6.

```
  ln -s /usr/local/bin/lpf_gs /usr/local/bin/lpf_pxlmono
```

(**N.B.** *pxlcolor* work for PCL 6 color printers.)

In addition to converting input formats, because it's an accounting
filter, lpf_gs can log accounting information if given an accounting
file to log to in the printcap(5) definition.

Lastly, to print to text files see the documentation for a2ps(1) or
enscript(1).


## DIAGNOSTICS

lpf_gs depends on Ghostscript (gs(1)) to work. lpf_gs will fail if
Ghostscript cannot be found.

lpf_gs will check whether gs(1) supports the device specified by the
name of the print filter and will fail if the device is not supported.


## SEE ALSO

a2ps(1), enscript(1), gs(1), lpq(1), lprm(1), pr(1), symlink(2),
printcap(5), lpc(8), lpd(8)


## AUTHORS

The lpf_gs utility was written by Aaron Poffenberger <akp@hypernote.com>.


## BUGS

lpd(8) accepts the following return codes:
```
 -1 non-recoverable error
 0 success
 1 try again
 2 success but with some errors
```

The exit command in sh(1) only allows values ranging from 0 - 255.
lpf_gs exits with 1 for all error conditions and prints an error
message to stderr. However, this doesn't seem to be a problem. lpd(8)
tries 3 times and aborts if it doesn't receive 0 or 2.

June 19, 2019
