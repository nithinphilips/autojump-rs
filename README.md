# autojump-rs

A port of the wildly popular helper application [`autojump`][aj] to Rust.

[aj]: https://github.com/wting/autojump


## License

As this project is technically a fork, the license is the same as `autojump`,
which is GPL, either version 3 or any later version. See [LICENSE](LICENSE)
for details.


## Install

This isn't published on crates.io yet, but you can always clone the repository
and `cargo build --release` yourself. Assuming you already have original
`autojump` installed and properly set up, it's only a simple matter of putting
the result executable on your `$PATH`, overriding the original script. (You
may need to do a `hash -r` before using in your currently open shells, of
course.)


## Features

Why do a port when the original version works? Primarily for two reasons:

* The author is *really* tired of `autojump` breakage inside Python virtualenvs, and
* Rust is simply *awesome* for CLI applications, with its performance and (code) slickness!

Indeed, being written in a compiled language, **`autojump-rs` is very light on
modern hardware**. As the program itself is very short-running, the overhead of
setting up and tearing down a whole Python VM could be overwhelming,
especially on less capable hardware. With `autojump-rs` this latency is
greatly reduced. Typical running time is like this on the author's Core
i7-2670QM laptop, with a directory database of 256 entries:

```
time ./autojump/bin/autojump au
/home/xenon/src/autojump-rs
./autojump/bin/autojump au  0.05s user 0.02s system 96% cpu 0.080 total

time ./autojump-rs/target/release/autojump au
/home/xenon/src/autojump-rs
./autojump-rs/target/release/autojump au  0.02s user 0.00s system 94% cpu 0.020 total
```

The time savings are more pronounced on less powerful hardware, where every
cycle shaved off counts. On a Loongson 3A2000 box running at 1.0 GHz the
timings are like this, with a database of the same size:

```
time ./autojump/bin/autojump au
/opt/store/src/autojump-rs
./autojump/bin/autojump au  0.15s user 0.02s system 97% cpu 0.178 total

time ./autojump-rs/target/release/autojump au
/opt/store/src/autojump-rs
./autojump-rs/target/release/autojump au  0.04s user 0.01s system 96% cpu 0.051 total
```

And, of course, the port no longer interacts with Python in any way, so the
virtualenv-related crashes are no more. Say goodbye to the dreaded
`ImportError`'s *every `$PS1` in a virtualenv with the system-default
Python*!

```
# bye and you won't be missed!
Traceback (most recent call last):
  File "/usr/lib/python-exec/python2.7/autojump", line 43, in <module>
    from autojump_data import dictify
ImportError: No module named autojump_data
```


## Compatibility

The on-disk format of the text database should be identical. That said, edge
cases certainly exist. The author is developing and using this on Linux, so
other platforms may need a little more love.

That said, there're some IMO very minor deviations from the original Python
implementation. These are:

*   Argument handling and help messages.

    Original `autojump` uses Python's `argparse` to parse its arguments. There
    is [a Rust port of it][rust-argparse], but it's nowhere as popular as the
    [`docopt.rs`][docopt.rs] library, as is shown in `crates.io` statistics
    and GitHub activities. So it's necessary to re-arrange the help messages
    at least, as the `docopt` family of argument parsers mandate a specific
    style for them. However this shouldn't be any problem, just that it's
    different. Again, who looks at the usage screen all the day? XD

*   Different algorithm chosen for fuzzy matching.

    The Python version uses the [`difflib.SequenceMatcher`][difflib] algorithm
    for its fuzzy matches. Since it's quite a bit of complexity, I chose to
    leverage the [`strsim`][strsim-rs] library instead. The [Jaro-Winkler
    distance][jaro] is computed between every filename and the last part of
    query needles respectively, and results are filtered based on that.

*   `jc` doesn't work correctly at the moment.

    Exact reason may be different filtering logic involved, but I'm not very
    sure about this one. I only use plain `j` mostly, so if you're heavily
    reliant on `jc` and its friends please open an issue!


[rust-argparse]: https://github.com/tailhook/rust-argparse
[docopt.rs]: https://github.com/docopt/docopt.rs
[difflib]: https://docs.python.org/3.5/library/difflib.html
[strsim-rs]: https://github.com/dguo/strsim-rs
[jaro]: https://en.wikipedia.org/wiki/Jaro%E2%80%93Winkler_distance


## Future plans

After initial porting, it's now time to re-organize the code to be more
library-like (rather than application-like) and Rustic. After that the project
would likely be published on crates.io. Hell I even want to write a `fasd`
backend too, but I don't presently have *that* much free time. Anyway,
contributions and bug reports are welcome!


<!-- vim:set ai et ts=4 sw=4 sts=4 fenc=utf-8: -->
