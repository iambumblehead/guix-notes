From a high-level, these are the steps from building a little package locally, then contributing it to the guix upstream. These steps are described in other places already, but here they are described in my own way,

 * first make a single file package, install it locally to verify that it works
 * clone the guix source, if you needed
 * create a new branch for your package in the cloned guix repo
 * copy the package definition into the guix repo package file
 * verify guix sources build; start guix shell pure environment, configure and make
 * verify the package builds from inside guix sources
 * apply guix linting and formatting styles to the new package definition
 * commt the changes, using magit and tempel
 * smoke-test "git send-mail --dry-run" for sending the patch email
 * if all is good, send the patch "git send-email"

For creating a single package, use any package definition from the guix sources or maybe use this yambar definition


_guix.pkg.yambar.scm_
```scm
(define-module (yambar)
  #:use-module ((guix licenses) #:prefix license:)
  #:use-module (guix gexp)
  #:use-module (guix download)
  #:use-module (guix packages)
  #:use-module (guix git-download)
  #:use-module (guix build-system meson)
  #:use-module (gnu packages pkg-config)
  #:use-module (gnu packages fontutils)
  #:use-module (gnu packages flex)
  #:use-module (gnu packages bison)
  #:use-module (gnu packages freedesktop)
  #:use-module (gnu packages linux)
  #:use-module (gnu packages datastructures)
  #:use-module (gnu packages serialization)
  #:use-module (gnu packages xdisorg)
  #:use-module (gnu packages web)
  #:use-module (gnu packages mpd)
  #:use-module (gnu packages man))

(define-public yambar
 (package
   (name "yambar")
   (version "1.10.0")
   (home-page "https://codeberg.org/dnkl/yambar")
   (source
     (origin
       (method git-fetch)
       (uri (git-reference
             (url home-page)
             (commit version)))
       (file-name (git-file-name name version))
       (sha256
        (base32
         "14lxhgyyia7sxyqjwa9skps0j9qlpqi8y7hvbsaidrwmy4857czr"))))
   (build-system meson-build-system)
   (arguments
    (list
      #:build-type "release"
      #:configure-flags #~'("-Db_lto=true"
                            "-Dbackend-x11=disabled"
                            "-Dbackend-wayland=enabled")))
    (native-inputs (list pkg-config tllist scdoc wayland-protocols flex bison))
    (inputs (list
             fcft
             wayland
             pipewire
             libyaml
             pixman
             alsa-lib
             json-c
             libmpdclient
             eudev))
    (synopsis "X11 and Wayland status panel")
    (description
     "@command{yambar} is a lightweight and configurable status
 panel (bar, for short) for X11 and Wayland, that goes to great
 lengths to be both CPU and battery efficient - polling is only
 done when absolutely necessary.")
    (license license:expat)))

yambar
```

install it this way
``` bash
$ guix package --install-from-file=guix.pkg.yambar.scm
```

if there's a problem with the package, install will show an error. update the scm script then try installing again etc.  when it succeeds, try using the command "yambar" and see if the bar shows up in your desktop.  when it runs, that means the package works.





https://guix.gnu.org/manual/en/html_node/Contributing.html
https://guix.gnu.org/en/blog/2018/a-packaging-tutorial-for-guix/

Basically these commands begin the pure shell environment and build the package
``` bash
guix shell -D guix --pure --check
./bootstrap
./configure --localstatedir=/var
make
./pre-inst-env guix build --keep-failed yambar
./pre-inst-env guix lint yambar
./pre-inst-env guix style yambar
```

When the package ready for generating a patch, take a look at previous commit messages wih `git log` then from inside emacs start magit, for example `M-x magit-commit` then choose 'c' for create commit. When magit opens the commit message buffer, use `M-x tempal-insert` and when it prompts for template name use `add` and write "yambar" inside the tempel message. Finally, commit the message with magit using `C-c C-c`.

The patch will be generated from the commit. Setup `git send-email` if needed https://git-send-email.io and from inside the cloned guix repo use this command
``` bash
git send-email --dry-run --base=master --to="guix-patches@gnu.org" -1
```

When everything looks good, send the patch
``` bash
git send-email --base=master --to="guix-patches@gnu.org" -1
```

Subsequent changes or commit messages can be [amended later][10] and resent,
```bash
git add ./path/to/any-files-changed # not needed if only amending commit message
git commit --amend
```


[10]: https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History#_git_amend
