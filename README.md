# Lisp-Format --- A tool to format lisp code.

This lisp-format script aims to provide the same functionality as
clang-format only for lisp languages instead of for C languages.
Emacs is used to perform formatting.  The behavior of clang-format is
followed as closely as possible with the goal of a direct drop in
replacement in most contexts.

This script is suitable for (re)formatting lisp code using an external
process.  This script may be marked as executable and executed
directly on the command line as follows.

```shell
chmod +x lisp-format
./lisp-format -h
```

This script is appropriate for use in git commit hooks.


This script is appropriate for use in git commit hooks.  In fact
the git-clang-format script [1] may be fairly trivially converted
into a git-lisp-format script by replacing "clang" with "lisp" and
changing the in-scope file extensions as follows.

```shell
curl https://llvm.org/svn/llvm-project/cfe/trunk/tools/clang-format/git-clang-format \
    |sed \
    "s/clang-format/lisp-format/g;s/clangFormat/lispFormat/;
     s/default_extensions =.*\$/default_extensions = ','.join(['lisp','cl','asd','scm','el'])/;
     /# From clang\/lib\/Frontend\/FrontendOptions.cpp, all lower case/,/])/d" \
    > /usr/bin/git-lisp-format

chmod +x /usr/bin/git-lisp-format
```

After the resulting git-lisp-format is added to your path then git can
execute this file by running "git lisp-format."

See [2] for an example description of a work flow leveraging git hooks
and git-clang-format to ensure that code is well formatted before
every commit (i.e., by adding [3] to an executable file named
pre-commit in a repository's .git/hooks/ directory).  This work flow
may be trivially adopted to use git-lisp-format for lispy code.

Clang-format allows for customized "styles" to be specified by
writing .clang-format files in the base of code repository and
passing the -style=file flag to clang-format.  Lisp-format
supports the same using .lisp-format files.  These files may hold
arbitrary emacs-lisp code, and they will be loaded before every
run of lisp-format.  The following example file will (1) load
slime assuming you have slime installed via quicklisp, (2) adjust
the syntax-table for special reader macro characters, and (3)
specify indentation for non-standard functions (e.g., from the
popular common lisp Alexandria package).

```lisp
;;;; -*- emacs-lisp -*-
(set-default 'indent-tabs-mode nil)
(mapc (lambda (dir) (add-to-list 'load-path dir))
      (directory-files
       (concat (getenv "QUICK_LISP")
               "/dists/quicklisp/software/") t "slime-v*"))
(require 'slime)

;;; Syntax table extension for curry-compose-reader-macros
(modify-syntax-entry ?\[ "(]" lisp-mode-syntax-table)
(modify-syntax-entry ?\] ")[" lisp-mode-syntax-table)
(modify-syntax-entry ?\{ "(}" lisp-mode-syntax-table)
(modify-syntax-entry ?\} "){" lisp-mode-syntax-table)
(modify-syntax-entry ?\« "(»" lisp-mode-syntax-table)
(modify-syntax-entry ?\» ")«" lisp-mode-syntax-table)

;;; Specify indentation levels for specific functions.
(mapc (lambda (pair) (put (first pair) 'lisp-indent-function (second pair)))
      '((if-let 1)
        (if-let* 1)))
```

As described in the "git lisp-format -h" output, you can use "git
config" to change the default style to "file" with the following
command (run in a git repository).

```shell
git config lispFormat.style "file"
```

Running the above and adding a .lisp-format file to the based of a
git repository enables customization of the lisp-format behavior.

[1] https://llvm.org/svn/llvm-project/cfe/trunk/tools/clang-format/git-clang-format

[2] https://dx13.co.uk/articles/2015/4/3/Setting-up-git-clang-format.html

[3] `.git/hooks/pre-commit`

```shell
#!/bin/bash
OUTPUT=$(git lisp-format --diff)
if [ "${OUTPUT}" == "no modified files to format" ] ||
   [ "${OUTPUT}" == "lisp-format did not modify any files" ];then
    exit 0
else
    echo "Run git lisp-format, then commit."
    exit 1
fi
```
