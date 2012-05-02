# A short excursion into Pry



# Why Pry?

The Ruby ecosystem lacks one major feature which is present in the same
class of languages: the REPL.

REPL is `read-eval-print-loop`.

When you fire up Scheme, Lisp or Smalltalk consoles, what you get is the
REPL.

Pry is a first step in implementing REPL for Ruby.

## Two primary uses for Pry

* A *better* Irb
* An alternative debugger



# But Irb is a REPL...?

Actually, yes, technically Irb is a REPL.

And it's not bad, especially when attached to large applications like
Rails.

Pry makes Irb a lot better.


# REPL read-eval-print-loop

~~~~
@@@ scheme
$ sbcl 
This is SBCL 1.0.55, an implementation of ANSI Common Lisp.
* (defun foo ( * 3 4 ))
; in: DEFUN FOO
;     (SB-INT:NAMED-LAMBDA FOO
;         (* 3 4)
;       (BLOCK FOO))
; ==>
;   #'(SB-INT:NAMED-LAMBDA FOO
;         (* 3 4)
;       (BLOCK FOO))
; 
; caught ERROR:
;   Required argument is not a symbol: 3
; 
; compilation unit finished
;   caught 1 ERROR condition

debugger invoked on a SB-INT:COMPILED-PROGRAM-ERROR:
  Execution of a form compiled with errors.
Form:
  #'(NAMED-LAMBDA FOO
      (* 3 4)
    (BLOCK FOO))
Compile-time error:
  Required argument is not a symbol: 3

Type HELP for debugger help, or (SB-EXT:QUIT) to exit from SBCL.

restarts (invokable by number or by possibly-abbreviated name):
  0: [ABORT] Exit debugger, returning to top level.

(#:EVAL-THUNK)
0] (quit)
~~~~



# Try again

~~~~
@@@ lisp
$ sbcl 
This is SBCL 1.0.55, an implementation of ANSI Common Lisp.
* (defun foo () ( * 3 4 ))

FOO
* (foo)

12
* (quit)
~~~~

# Lingering in the Kernel...

[Sae
Romy Hong](http://syntacticsugar.github.com/blog/2012/02/23/lingering-in-the-kernel/)
captures the Pry experience well. Here's what she found...


“Navigate into the kernel.”

`[0] pry(main)> cd Kernel`

“Look around you, do you see anything?”

`[1] pry(Kernel)> ls` “Ah, glorious. We have our Kernel methods…”

“Yes. Navigate into String and look around.”

`[2] pry(String)> cd String`

`[3] pry(String)> ls` “I don’t see much, other than some… locals.”

“Yes. Now, try creating an instance of a String.”

`[4] pry(String)> x = "wombat"`

“Navigate into that instance, look around.”

`[5] pry("wombat")> cd x`

`[6] pry("wombat")> ls`

“Tell me, what do you see?”

----

(And [do visit
Romy](http://syntacticsugar.github.com/blog/2012/02/23/lingering-in-the-kernel/).)

# [6] pry("wombat")> ls

This is what I see:

~~~~
@@@ ruby
Comparable#methods: <  <=  >  >=  between?
String#methods: %  *  +  <<  <=>  ==  ===  =~  []  []=  ascii_only?  bytes  bytesize  byteslice  capitalize  capitalize!  casecmp  center chars  chomp  chomp!  chop  chop!  chr  clear  codepoints  concat  count crypt  delete  delete!  downcase  downcase!  dump  each_byte  each_char each_codepoint  each_line  empty?  encode  encode!  encoding  end_with?  eql?  force_encoding  getbyte  gsub  gsub!  hash  hex  include?  index insert  inspect  intern  length  lines  ljust  lstrip  lstrip!  match next  next!  oct  ord  partition  prepend  replace  reverse  reverse!  rindex  rjust  rpartition  rstrip  rstrip!  scan  setbyte  shellescape shellsplit  size  slice  slice!  split  squeeze  squeeze!  start_with?  strip  strip!  sub  sub!  succ  succ!  sum  swapcase  swapcase!  to_c to_f  to_i  to_r  to_s  to_str  to_sym  tr  tr!  tr_s  tr_s!  unpack upcase  upcase!  upto  valid_encoding?  
self.methods: __binding_impl__
locals: _  _dir_  _ex_  _file_  _in_  _out_  _pry_
[7] pry("wombat"):1> 
~~~~

That's cool.

# Investigation: Ruby macros

What are these things, macros? 

Can Pry help us understand?

# Comparison with ruby-debug

`ruby-debug` is pretty useful, provided it's properly employed.


Note: taking the time to really learn how a debugger works often changes
one's opinions of debuggers.


# The Pry ecosystem


As noted at the beginning, Pry is being used as an alernative to
`ruby-debug`. Since Pry doesn't have explicit support for standard
debugging operations such as `step`, `continue`, etc., we look to
Rubygems and find:

* `pry-nav`: 
* `pry-stack_explorer`:
* `pry-exception_explorer`:
  
There are more, this will for now, see the links at the end of this
presentation.

# pry-nav

`pry-nav` is a muct have when using Pry as a debugger. It adds following
commands:



# pry-stack_explorer

Navigate the call stack with `pry-stack_explorer`.

# pry-exception_explorer

Intercept exceptions with `pry-exception_explorer`.

Let's try an example...

~~~~
@@@ ruby
a = 1/0
~~~~

# Python coolness

Python has this cool feature in the interpreter which returns a little
documentation about the object and method, when invoked without
parentheses:

~~~~
@@@ python
>>> file.read
<built-in method read of file object at 0x1004c38b0>
>>> file.read()
'This is the contents of the file'
~~~~

# Links 
You might find these links useful:

* [Pry Github account](https://github.com/pry/pry)
* [Pry home page](http://pry.github.com/). Make sure to watch and
  rewatch the video on this page. 
* [Pry Railscast](http://railscasts.com/episodes/280-pry-with-rails)
* [Pry Wiki](https://github.com/pry/pry/wiki)
* [Turning Irb on its head with
  Pry](http://banisterfiend.wordpress.com/2011/01/27/turning-irb-on-its-head-with-pry/)
* [The Pry
  ecosystem](http://banisterfiend.wordpress.com/2012/02/14/the-pry-ecosystem/)

Thanks for reading or listening along. Feel free to ask questions:
[@doolin](http://twitter.com/doolin).


