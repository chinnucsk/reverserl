practical_erlang_tutorial
=========================

A practical tutorial about how to program with Erlang.

The goal of this example is to show how to create an Erlang project that isn't
too far from an actual real-world application.

License
=======

3-clause BSD

Overview
========

Introducing... reverserl!

Let's build a little Erlang application that does something really simple: reverse strings.
To show a bit more of what Erlang can do it will also have a simple session based web API.

We'll try to cover:

- Some of OTP's principles (gen_server, application, ...)
- Basics of functional programming with Erlang
- The rebar build tool for Erlang
- Testing using eunit/QuickCheck mini (Proper should work too -
  https://github.com/manopapad/proper)

Functional Overview
=====================

At its core reverserl has a server that handles incoming requests to perform string
reversal operations.

Each request needs to happen within a session:

- Client creates a session with server
- Server responds with a session id
- Client uses session id to call string reversal service
- Client closes the session

On top of that each session should have a timeout to ensure that it is closed even if the
client forgets to do so. Moreover, if a client tries to connect without a session id or with
an invalid session id the server should not allow it to call its string reversal service.

A simple HTTP-based protocol will be used to implement the exchanges between the server and
its clients.

Architectural Overview
======================

The core of the application will be an Erlang gen_server which will handle the logic for:

- Session creation requests
- Session destruction requests
- Session timeout detection
- Management of the actual lifetime of the sessions

The last point is important here since we'll be leveraging Erlang's lightweight processes:
each session will be represented by an actual Erlang process. We'll see that this is an
extremely natural fit and that Erlang makes programming things as they would occur in
real life easy.

The actual work of reversing the strings will thus be done by each session's process.

For the HTTP portion we'll be using the built-in Erlang web server. However, there are a few
other very good open source web servers if you want to play with them (mochiweb, yaws, cowboy,
webmachine, ...).

Step 0 - Install Erlang and rebar
=================================

This assumes that you have Erlang installed on your system. You can install it from source or
from pre-compiled binaries (from either http://www.erlang.org or http://www.erlang-solutions.com).

- Install rebar from github
  - git clone https://github.com/basho/rebar.git
  - cd rebar; ./bootstrap.sh
  - Put the generated rebar executable into your path or directly within the project directory we'll
    create next

Step 1 - Create the core source files using rebar
=================================================

Our application will reside in its own directory:

    mkdir reverserl

Using rebar, create the core application source files:

    cd reverserl
    rebar create-app appid=reverserl

You should see something like:

    ==> reverserl (create-app)
    Writing src/reverserl.app.src
    Writing src/reverserl_app.erl
    Writing src/reverserl_sup.erl

This created two Erlang source files for two standard OTP components: an _application_ and a
_supervisor_. The third file (.app.src) will contain some metadata regarding the application.
Source files go under the _src_ directory.

An OTP application is the standard way of implemention applications in Erlang. More information
is available here: http://www.erlang.org/doc/design_principles/applications.html. An OTP supervisor
(http://www.erlang.org/doc/design_principles/sup_princ.html) is a specialized Erlang process that
is responsible to monitor the lifetime of children processes and act accordingly if they crash
(i.e. restart them, ...).

Here we used rebar to have it generate pre-populated source files from its built-in template files.
Although they should compile file, we'll need to tweak them a little so that they actually do something.

To compile the current _rebarized_ project:

    rebar compile

You should see something like:

    ==> reverserl (compile)
    Compiled src/reverserl_app.erl
    Compiled src/reverserl_sup.erl

This created compiled Erlang bytecode files (along with the processed .app file) under the _ebin_ directory:

    /ebin
    ./ebin/reverserl.app
    ./ebin/reverserl_app.beam
    ./ebin/reverserl_sup.beam

