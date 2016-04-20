% XPANDLATEX(1)

# NAME

xpandlatex - expand LaTeX elements

# SYNOPSIS

`xpandlatex` [flags] [-f macro\_file] ... file

# OPTIONS

An [on|off] switch with no argument turns the function on.

-h, --help
:   show a help message and exit
-x [on|off], --expand-macros [on|off]
:   expand macros (default: on)
-r [on|off], --expand-refs [on|off]
:   expand \\refs (default: off)
-b [on|off], --expand-cites [on|off]
:   expand bibliography \\cite, \\citep, \\citet ... (default: off)
-m [on|off], --read-macros [on|off]
:   read macro definitions that appear in main file (default: off)
-X [on|off], --expand-all [on|off]
:   control all expansion (overrides individual flags)
-I [on|off], --merge-inputs [on|off]
:   merge \\input files (default: off)
-T [on|off], --merge-tocs [on|off]
:   merge <name>.{toc,lof,lot} (default: off)
-B [on|off], --merge-bibliography [on|off]
:   merge compiled bibliography from <name>.bbl (default: off)
-M [on|off], --merge-all [on|off]
:   control all merges (overrides individual flags)
-f \<file\>, --macro-file \<file\>
:   read macros from file 
-D \#
:   set debug level


# DESCRIPTION

`xpandlatex` processes a LaTeX file to do some or all of the following

- expand locally or externally defined macros 
- replace `\ref`s with corresponding targets
- replace `\cite`\* with citation targets
- include files as directed by: `\include{file}`, `\tableofcontents`,
  `\listoffigures`, `\listoftables`, `\bibliography{}`
- interpret special commands beginning with ’%XP’


##Macro handling

Macros may be defined with `\def`, `\newcommand` or `\renewcommand`.
Definitions in the main file are only read with the `-m` switch.
Definitions may also be provided from other files using `-f`.

See also environments below.

##Label/ref handling

With the `-r` switch, label definitions are read from the aux file,
and `\ref`s expanded.  References to labels in other files are also
expanded correctly by following any `\externaldocument[prefix]{file}`
commands (see `xr.sty`).

##Citation handling
With the `-b` switch `xpandlatex` replaces
```
\citetype[opt1][opt2]{key1,key2}
```
with
```
[(][opt1 ]\XPcitetype{expansion1}, \XPcitetype{expansion2}[ opt2][)]
```
where `\citetype` may be plain `\cite` or natbib `\citep`, `\citet`,
`\citealt` or `\citeauthor`. The expansion (at least using natbib with
apalike.bst) has 4 parts, and the `\XPcitetype` macros should be
defined to handle them correctly. The following commands work for the
natbib/apalike combination:
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
rather than prefix, as in natbib.

##Merging files

With `-I`, `xpandlatex` will merge any external files referenced by
`\input{file}`.

With `-T`, `xpandlatex` will include `.toc`, `.lof` and `.lot` files
in place of the respective commands `\tableofcontents`,
`\listoffigures` and `\listoftables` which cause LaTeX to generate
them.  An appropriate `\section*{\namemacro}` line is inserted.  The
special macros `\XPtocbegin` and `\XPtocend`, if defined, will be
expanded before and after the `\tableofcontents` inclusion, but after
the `\section{}`.  Similar macros work for 'lof' and 'lot'.

With `-B`, `xpandlatex` will include the contents of a
bibtex-generated `.bbl` file in place of
`\bibliography{database.bib}`. It does not read the `.bib` file
itself.  No special `\begin`/`\end` macros are expanded: the
`{thebibliography}` environment within the `.bbl` file can be
redefined instead.

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
the `\foo` macro definition will be seen by xpandlatex but not by LaTeX;
the `\bar` macro definition will be seen by LaTeX but not by
xpandlatex; and the text "`%XPCUT won't be cut %XPTUC`" will be
copied unchanged.

Note the use of % on the %XP line: if omitted the words
would be copied uncommented to the xpandlatex output.

Note also that %XP specials that appear in comments will not be
interepreted, and those in macro definitions will not be interpreted
until the macro is expanded.

See also environment handling specials below.

##Environment handling
xpandlatex reads `\newenvironment` and `\renewenvironment`
commands, and expands corresponding `\begin{env}...\end{env}` code.
It can also interpret a special `\XPenvironment` command to
execute special actions on the **body** of a LaTeX environment. The
definition takes the form:
``` 
\XPenvironment{name}{begin code}{end code}{body actions} 
```
The `{name}`, `{begin code}` and `{end code}` are as
for `\newenvironment`; except that a special ’\#\#’ parameter is
replaced by xpandlatex’s count of the number of times this environment
has been called. The final argument may contain the following special
symbols:

%XPcopy
:	Copy out (and interpret) the body as usual

%XPdiscard
:	Discard the body completely 

%XPwritefile
: 	Write the body to a file called ’name\_\#\#.tex’, where \#\# is
	xpandlatex’s count for the number of times this environment
	has been encountered. Note this is **not** a LaTeX counter,
	and so will not be affected by LaTeX commands such as
	`\setcounter`. The body is not copied to the main output.

Multiple body actions may appear: so `{%XPwritefile %XPcopy}` will copy
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

#EXAMPLES

##Extracting floats 
The file `paper.tex` contains figures within floats and no
`\listoffigures`.  We wish to extract each figure to its own file, and
add a list of figure legends to the end of the file, without affecting
the LaTeX output from `paper.tex` itself.

As `paper.tex` has no `\listoffigures` command this operation must be
performed in two stages.  We add this code to paper.tex:
```
%% near the beginning of the file:
%XPVERB
%XP \XPenvironment{figure}{\begin{center}[Figure ## about here]\end{center}}{}{%XPwritefile}
%XPBREV

%% near the end of the file:
%% Double-wrap listoffigures for xpandlatex.  
%XPVERB\def\listfigurename{Figure Legends}%XPBREV
%XPVERB\listoffigures%XPBREV
```

Then run: (a Makefile may be useful)
```
xpandlatex -X off -M off paper.tex > paper-int.tex
[pdf]latex paper-int.tex
xpandlatex -m -T on paper-lof.tex > paper-fin.tex
```
The first `xpandlatex` call does nothing except strip the `%%XPVERB`
environments, exposing the `%XP` line and the `\listoffigures`
commands.  The LaTeX compile creates the `.lof` file.  The final
`xpandlatex` call includes this into the output.  It also processes the
\XPenvironment define protected by `%XP` (note the `-m` flag) and
splits the figures into individual files.  This environment definition
could also be placed in a helper 'macro' file and included with the
`-f` flag instead. 

To compile the figures create a wrapper file `fig_wrapper.tex`:
```
\documentclass{article} 

%% include packages needed for graphics
\usepackage{graphicx,tikz} 

%% remove figure captions
\usepackage{caption}
\DeclareCaptionFormat{blank}{}
\captionsetup[figure]{format=blank}

%% no page numbers
\pagestyle{empty}

\begin{document}
\begin{figure}
  \input{\jobname}
\end{figure}
\end{document}
```
and use the following Makefile rule to compile:
```
figures: figure_*.pdf 

figure_*.pdf: %.pdf: %.tex figure_wrapper.tex
	pdflatex -jobname $* figure_wrapper.tex
```

If the final xpanded LaTeX file is to be processed by `pandoc(1)` then
it may be useful to add the following code to `paper.tex` (or,
unprotected, to the macro file) as `pandoc` seems not to
understand LaTeX TOC commands:
```
%XPVERB
%XP \newcommand{\XPlofbegin}{\begin{description}}
%XP \newcommand{\contentsline}[3]{#2\par}
%XP \newcommand{\numberline}[1]{\item[Figure #1]}
%XP \newcommand{\XPlofend}{\end{description}}
%XPBREV
```


#BUGS

It does what I need today, but has not been tested widely

#AUTHOR

Maneesh Sahani (xpandlatex @github)

