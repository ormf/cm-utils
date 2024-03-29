cm-utils adds coroutine-like process constructs for realtime common
music 2. It allows for a more general concept of a process. Unlike
cm's process macro, which is akin common lisp's loop macro, rt-proc
will work on arbitrary lisp bodys enabling multiple waits and
recursive synchronous subprocesses.

As the package depends on cl-cont (via cl-coroutine), the limitations
of cl-cont apply to this as well.

Dependencies:

- [incudine](http://incudine.sourceforge.net/)
- [fudi-incudine](https://github.com/ormf/fudi-incudine)
- [cm](https://github.com/ormf/cm)
- [cm-incudine](https://github.com/ormf/cm-incudine) for realtime output
- [cl-coroutine](https://github.com/takagi/cl-coroutine)

Implemented macros are:

rt-proc: similar to cm's "process" macro, allowing arbitrary lisp forms within its body

rt-wait: similar to cm's wait.

rt-sprout: a "non-blocking" subprocess, similar to cm's sprout

rt-sub: a "blocking" subprocess. In contrast to sprout subproc will finish
        the subprocess before continuing.

NOTE: cm's "events" function can also be used with rt-proc! 

examples comparing the traditional cm process syntax to cm-util's
syntax:

cm:
```cl
(sprout
 (process
   for keynum in '(60 62 64 65)
   output (new midi :time (now) :keynum keynum)
   wait 0.5))
```
cm-utils:
```cl
(sprout
 (rt-proc
   (loop
      for keynum in '(60 62 64 65)
      do (progn
           (output (new midi :time (now) :keynum keynum))
           (rt-wait 0.5)))))

;;; the same using dolist:

(sprout
 (rt-proc
   (dolist (keynum '(60 62 64 65))
      (output (new midi :time (now) :keynum keynum))
      (rt-wait 0.5))))

;;; using multiple waits:

(sprout
 (rt-proc
   (loop
      for keynum in '(60 62 64 65)
      do (progn
           (output (new midi :time (now) :keynum keynum))
           (rt-wait 0.1)
           (output (new midi :time (now) :keynum (+ keynum 11)))
           (rt-wait 0.2)))))

;;; defining as function:

(defun proc-01 ()
 (rt-proc
   (loop
      for keynum in '(60 62)
      do (progn
           (output (new midi :time (now) :keynum keynum))
           (rt-wait 0.1)
           (output (new midi :time (now) :keynum (+ keynum 11)))
           (rt-wait 0.2)))))

(sprout (proc-01))

;;; using a synchronous subprocess:

(defun proc-02 ()
  (rt-proc
   (loop
      for keynum in '(50 52)
      do (progn
           (output (new midi :time (now) :keynum keynum))
           (rt-wait 0.5)
           (rt-sub (proc-01))
           (output (new midi :time (now) :keynum (+ keynum 11)))
           (rt-wait 0.2)))))

(sprout (proc-02))

;;; events also works:

(events (proc-02) "/tmp/test.midi")
```

The code is (c) Orm Finnendahl, released under the GPL (version 2 or
later), without any warranties. Use at your own risk.
