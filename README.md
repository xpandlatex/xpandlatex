% XPANDLATEX(1)
% April 2016

# NAME

xpandlatex - expand LaTeX elements

# SYNOPSIS

`xpandlatex` [flags] [-m macro\_file] ... file

# OPTIONS

-h, \--help
:   show a help message and exit
-D \#
:   set debug level
-x, --expand-macros
:   expand macros [default]
-X, --no-expand-macros
:   don’t expand macros
-r, --expand-refs
:   expand \\refs (to sections, figures, equations etc.)
-R, --no-expand-refs
:   don’t expand \\refs [default]
-b, --expand-cites
:   expand bibliography \\cite, \\citep, \\citet etc.
-B, --no-expand-cites
:   don’t expand cites [default]
--macros
:   read macro definitions that appear in main file
--no-macros
:   don’t read macros in main file [default]
--includes
:   include input files [default]
--no-includes
:   don’t include inputs
-m \<file\>, --macro-file \<file\>
:   read macros from file

# DESCRIPTION

`xpandlatex` processes a LaTeX file to do the following

- expand locally defined macros 
- replace \\refs with corresponding targets
- replace \\cite\* with citation targets
- include files as directed by: \\include{file}, \\tableofcontents,
  \\listoffigures, \\listoftables, \\bibliography{}
- interpret special commands beginning with ’%XP’


##Macro handling

Macros may be defined with `\def`, `\newcommand` or `\renewcommand`.
See also environments below.

##Label/ref handling

Label definitions are read from the aux file, and `\refs` expanded.
References to labels in other files are also expanded correctly by
following `\externaldocument[prefix]{file}` commands (see `xr.sty`).

##Citation handling
`xpandlatex` replaces
```
\citetype[opt1][opt2]{key1,key2}
```
by
```
[(][opt1 ]\XPcitetype{expansion1}, \XPcitetype{expansion2}[ opt2][)]
```
where \\citetype may be plain \\cite or natbib \\citep, \\citet,
\\citealt or \\citeauthor. The expansion (at least using natbib with
apalike.bst) has 4 parts, and the \\XPcitetype macros should be
defined to handle them correctly. The following should work:
```
\newcommand{\XPcite}[4]{#1}
\newcommand{\XPcitep}[4]{#3 #2}
\newcommand{\XPcitet}[4]{#3 (#2)}
\newcommand{\XPcitealt}[4]{#3 #2}
\newcommand{\XPciteauthor}[4]{#3}
```

The opening delimeters are chosen appropriately for the citation type,
and cannot currently be altered.

If only one optional argument appears it is assumed to be a postfix
rather than prefix.

##%XP directives 
 The LaTeX file may contain special tokens that begin with ’%XP’ that
are interpreted by xpandlatex but ignored (along with the rest of the
line) by the LaTeX compiler. Currently available specials are:

%XP
:    Ignore the special, but process the rest of the line.

%XPCUT ... %XPTUC
:    Ignore everything (including multiple lines, which may or may not be
     commented) between the tokens. Cannot be nested.

%XPVERB ... %XPBREV
:    Copy everything (comments, %XP specials, warts and all) to the
     output without performing any expansions.  Cannot be nested.

Thus, in the code
```
 %XP \def\foo{bar} % latex doesn’t see this
 %XPCUT
 \def\bar{foo} % xpandlatex doesn’t see this
 %XPTUC
 %XPVERB%XPCUT won't be cut %XPTUC%XPBREV
 
```
the \\foo macro definition will be seen by xpandlatex but not by LaTeX;
while the \\bar macro definition will be seen by LaTeX but not by
xpandlatex, while the text "`%XPCUT won't be cut %XPTUC`" will be
copied unchanged.

Note the use of % on the %XP line: if omitted the words
would be copied uncommented to the xpandlatex output.

Note also that %XP specials that appear in comments will not be
interepreted, and those in macro definitions will not be interpreted
until the macro is expanded.

See also environment handling specials below.

##Environment handling
 xpandlatex interprets and expands \\newenvironment and
\\renewenvironment commands. It can also interpret a special
\\XPenvironment command to execute special actions on the **body** of a
LaTeX environment. The definition takes the form:
```
\XPenvironment{name}{begin code}{end code}{body actions}
```
The {name}, {begin code} and {end code} are as for \\newenvironment;
except that a special ’\#\#’ parameter is replaced by xpandlatex’s count
of the number of times this environment has been called. The final
argument may contain the following special symbols: 

%XPcopy
:	Copy out (and interpret) the body as usual

%XPdiscard
:	Discard the body completely 

XPwritefile
: 	Write the body to a file called ’name\_\#\#.tex’, where \#\# is
	xpandlatex’s count for the number of times this environment
	has been encountered. Note this is **not** a LaTeX counter,
	and so will not be affected by LaTeX commands such as
	\\setcounter. The body is not copied to the main output.

Multiple body actions may appear: so {%XPwritefile %XPcopy} will copy
the body both to the main output and a separate file.

For example:
```
\XPenvironment{figure} 
{\begin{center}[Figure ## about here]\end{center}} 
{} 
{%XPwritefile}
```
writes figure contents to ’figure\_1.tex’ etc, placing marker text in
the main output stream.

#BUGS

It does what I need, but has not been tested widely

#AUTHOR

Maneesh Sahani (xpandlatex@users.github.com)

