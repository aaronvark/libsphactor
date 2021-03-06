[![Build Status](https://travis-ci.org/hku-ect/libsphactor.svg?branch=master)](https://travis-ci.org/hku-ect/libsphactor)
[![Build status](https://ci.appveyor.com/api/projects/status/2hggurtvkok3uqvl?svg=true)](https://ci.appveyor.com/project/sphaero/libsphactor-bdyyp)

# libsphactor

"Extended nodal actor framework based on zactor"

# Prototype Build Guide

## OSX

### Building Dependencies

There are two main dependencies:

 * libzmq
 * czmq

Dependencies for the build process / dependencies are:

   * git, libtool, autoconf, automake, cmake, make, pkg-config, pcre,

Building them should be pretty straight forward:

 * Get dependencies via brew:
```
brew install libtool autoconf automake pkg-config cmake make zeromq czmq
```

### Building Libsphactor

Once the above dependencies are installed, you are ready to build libsphactor. The process for this is much the same:

 * Clone the repo
```
git clone http://github.com/hku-ect/libsphactor.git
```
 * Build the project
```
cd libsphactor
./autogen.sh
./configure
make
# optionally test the library
SPHACTOR_SOCKET_LIMIT=30 make check
sudo make install
```
See the minimal example below. Should work on OSX as well.

### Creating an XCode project

* Create a new macOS - Command Line Tool project
* Enter a product name and other details, select C as the language
* Select a folder
* You'll be provided with an example C file. Paste the minimal example source from below to replace the provided C example
* Select the project on the left pane. Goto Build Settings and select All + Combined
* Look for "Search paths"
  * Add /usr/local/include to "Header Search Paths"  
  * Add /usr/local/lib to "Library Search Paths"  
* Look for "Linking"
  * Add "-lsphactor" and "-lczmq" to "Other Linker Flags"

You can now run the program.

If you want to work on the library it self it is best to use Cmake to build a Xcode project. from the repository's root run:

```
mkdir xcodeproj
cd xcodeproj
cmake -G Xcode ..
```
This should generate a valid Xcode project that can run and pass tests.

### Latest dependencies

Sometimes you need the latest dependencies. Here's how:

First uninstall already installed libs (optional):
```
brew uninistall zeromq czmq
```

Clone & build libzmq & czmq
```
git clone git://github.com/zeromq/libzmq.git
cd libzmq
./autogen.sh
./configure
make check
sudo make install
cd ..

git clone git://github.com/zeromq/czmq.git
cd czmq
./autogen.sh && ./configure && make check
sudo make install
cd ..
```
---

## Debian/Ubuntu Linux

(tested on ubuntu 16.04)

Install dependencies

```
sudo apt-get update
sudo apt-get install -y \
    build-essential libtool \
    pkg-config autotools-dev autoconf automake \
    uuid-dev libpcre3-dev libsodium-dev libzmq5-dev libczmq-dev
```

Clone this repo, build and install

```
git clone https://github.com/hku-ect/libsphactor.git
cd libsphactor
./autogen.sh
./configure && make
sudo make install
```
You can now use libsphactor as a static (/usr/local/lib/libsphactor.a) or dynamic lib (/usr/local/lib/libsphactor.so). Include file are in /usr/local/include.

---
## Windows

For windows you'll need the latest Visual Studio (i.e. 2019) and cmake.

From a Visualstudio command prompt use the following commands to build everything in c:\projects with binaries in c:\tmp

libzmq
```
set INSTALL_PREFIX=C:\tmp\build
set LIBZMQ_SOURCEDIR=C:\projects\libzmq
set LIBZMQ_BUILDDIR=%LIBZMQ_SOURCEDIR%\build
git clone --depth 1 --quiet https://github.com/zeromq/libzmq.git "%LIBZMQ_SOURCED
md "%LIBZMQ_BUILDDIR%"
cd "%LIBZMQ_BUILDDIR%"
cmake .. -DBUILD_STATIC=OFF -DBUILD_SHARED=ON -DZMQ_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX="%INSTALL_PREFIX%"
cmake --build . --config %Configuration% --target install
```
cmzq
```
set CZMQ_SOURCEDIR=C:\projects\czmq
set CZMQ_BUILDDIR="%CZMQ_SOURCEDIR%\build"
git clone --depth 1 --quiet https://github.com/zeromq/czmq.git "%CZMQ_SOURCEDIR%"
md "%CZMQ_BUILDDIR%"
cd "%CZMQ_BUILDDIR%"
cmake .. -DCZMQ_BUILD_STATIC=OFF -DCZMQ_BUILD_SHARED=ON -DCMAKE_PREFIX_PATH="%INSTALL_PREFIX%" -DCMAKE_INSTALL_PREFIX="%INSTALL_PREFIX%"
cmake --build . --config %Configuration% --target install
```
libsphactor
```
set SPH_SOURCEDIR=C:\projects\libsphactor
set SPH_BUILDDIR=%SPH_BUILDDIR%\build
git clone --depth 1 --quiet https://github.com/hku-ect/libsphactor.git "%SPH_SOURCEDIR%"
md "%SPH_BUILDDIR%"
cd "%SPH_BUILDDIR%"
cmake .. -DSPHACTOR_BUILD_STATIC=OFF -DSPHACTOR_BUILD_SHARED=ON -DCMAKE_C_FLAGS="-DCZMQ_BUILD_DRAFT_API=1" -DCMAKE_PREFIX_PATH="%INSTALL_PREFIX%" -DCMAKE_INSTALL_PREFIX="%INSTALL_PREFIX%"
cmake --build . --config Debug --target install
ctest -C Debug -V
```

---
## Minimal example in C (Linux with gcc)

test.c:
```c
#include <sphactor_library.h>
#include <czmq.h>

//  An actor is a method wich receives a sphactor_event as an argument
//  and returns a zmsg_t. Zmsg_t is just a message we can send.
static zmsg_t *
hello_sphactor(sphactor_event_t *ev, void *args)
{
    if ( ev->msg == NULL ) return NULL;
    assert(ev->msg);
    //  just echo what we receive
    char *cmd = zmsg_popstr(ev->msg);
    zsys_info("Hello actor %s says: %s", ev->name, cmd);
    zstr_free(&cmd);
    // if there are strings left publish them
    if ( zmsg_size(ev->msg) > 0 )
    {
        return ev->msg;
    }
    else
    {
        zmsg_destroy(&ev->msg);
    }
    return NULL;
}

//  This is identical to the first hello_actor
static zmsg_t *
hello_sphactor2(sphactor_event_t *ev, void *args)
{
    if ( ev->msg == NULL ) return NULL;
    assert(ev->msg);
    // just echo what we receive
    char *cmd = zmsg_popstr(ev->msg);
    zsys_info("Hello2 actor %s says: %s", ev->name, cmd);
    zstr_free(&cmd);
    // if there are strings left publish them
    if ( zmsg_size(ev->msg) > 0 )
    {
        return ev->msg;
    }
    else
    {
        zmsg_destroy(&ev->msg);
    }
    return NULL;
}

int main()
{
    //  create two actors
    sphactor_t *hello1 = sphactor_new ( hello_sphactor, NULL, NULL, NULL);
    sphactor_t *hello2 = sphactor_new ( hello_sphactor2, NULL, NULL, NULL);

    //  connect the two actors to each other
    sphactor_connect(hello1, sphactor_endpoint(hello2));
    sphactor_connect(hello2, sphactor_endpoint(hello1));

    //  send multiple strings to the first actor
    zstr_sendm( sphactor_socket(hello1), "SEND");  // this is an API command
    zstr_sendm( sphactor_socket(hello1), "HELLO"); // first string
    zstr_sendm( sphactor_socket(hello1), "WORLD"); // etc...
    zstr_sendm( sphactor_socket(hello1), "AND");
    zstr_sendm( sphactor_socket(hello1), "ALIEN");
    zstr_send( sphactor_socket(hello1), "SPACELINGS"); // finish message construction and send the message

    zclock_sleep(10); //  give some time for the test to complete, since it's threaded

    sphactor_destroy (&hello1);
    sphactor_destroy (&hello2);

    return 0;
}

```
Build this file and run:

```
gcc -o test main.c -lczmq -lsphactor
./test
```

If correct it wil show:
```
I: 19-12-03 13:20:49 Hello2 actor A8D382 says: HELLO
I: 19-12-03 13:20:49 Hello actor C3FA9D says: WORLD
I: 19-12-03 13:20:49 Hello2 actor A8D382 says: AND
I: 19-12-03 13:20:49 Hello actor C3FA9D says: ALIEN
I: 19-12-03 13:20:49 Hello2 actor A8D382 says: SPACELINGS
```

You can also use the static lib instead of the dynamic one:
```
gcc -o test main.c /usr/local/lib/libsphactor.a /usr/lib/x86_64-linux-gnu/libczmq.a -l zmq -lpthread
```

## Minimal example in C++

Please note for C++ you need to include *libsphactor.hpp* instead of libsphactor.h!

```cpp
#include <iostream>
#include "libsphactor.hpp"

// g++ sphactor_selftest.cpp -o test -I ../include/ .libs/libsphactor.a -l czmq

class Test
{
public:

    Test() {
    }

    ~Test() { }

    zmsg_t *
    handleMsg( sphactor_event_t *event ) {
        // just print the event
        printf("name: %s, type: %s", ev->name, ev->type);
        if ( event->msg == nullptr ) return nullptr;
        char *cmd = zmsg_popstr(event->msg);
        zsys_info("Cpp actor %s says: %s", event->name, cmd);
        // if there are strings left publish them
        if ( zmsg_size(event->msg) > 0 )
        {
            return event->msg;
        }
        else
        {
            zmsg_destroy(&event->msg);
        }
        return nullptr;
    }
};

int main()
{
    Test *a = new Test()
    Test *b = new Test();

    // register the Test class as an actor
    sphactor_register("cpp test", sphactor_member_handler<Test>);

    // create actor controllers for the actors
    sphactor_t *actora = sphactor_new(a, "hello-a", nullptr);
    sphactor_t *actorb = sphactor_new_by_type("cpp test", b, "hello-b", nullptr);

    // get the actor name
    const char *name = sphactor_ask_name(actora);
    assert( streq(name, "hello-a"));

    // set actora to be verbose
    sphactor_ask_set_verbose(actora, true);

    // connect the actors to each other
    sphactor_ask_connect(actora, sphactor_ask_endpoint(actorb));
    sphactor_ask_connect(actorb, sphactor_ask_endpoint(actora));
    zclock_sleep(10); // give time to connect

    // initiate a SEND (SEND API command just sends what it receives)
    zstr_sendm(sphactor_socket(actora), "SEND");
    zstr_sendm(sphactor_socket(actora), "HELLO");
    zstr_sendm(sphactor_socket(actora), "WORLD");
    zstr_sendm(sphactor_socket(actora), "AND");
    zstr_sendm(sphactor_socket(actora), "ALIEN");
    zstr_send(sphactor_socket(actora), "SPACELINGS");
    zclock_sleep(10); // give time to see messages printed

    // cleanup
    sphactor_destroy(&actora);
    sphactor_destroy(&actorb);
    delete( a );
    delete( b );
    sphactor_dispose();

    return 0;
}
```

Compile with:
```
g++ test.cpp -o test -lsphactor -lczmq
```

As a convenience you can also inherit the Sphactor class:

```
class Test : public Sphactor
{
public:

    Test() {
    }

    ~Test() { }
    
    // handle initialisation
    zmsg_t * handleInit(sphactor_event_t *ev)
    
    // handle timed events
    zmsg_t * handleTimer(sphactor_event_t *ev) { if ( ev->msg ) zmsg_destroy(&ev->msg); return nullptr; }

    // handle your own API commands
    zmsg_t * handleAPI(sphactor_event_t *ev) { if ( ev->msg ) zmsg_destroy(&ev->msg); return nullptr; }

    // handle receiving messages
    zmsg_t * handleSocket(sphactor_event_t *ev) { if ( ev->msg ) zmsg_destroy(&ev->msg); return nullptr; }

    // handle stopping your actor (destroying it is handled for you in the base class)
    zmsg_t * handleStop(sphactor_event_t *ev) { if ( ev->msg ) zmsg_destroy(&ev->msg); return nullptr; }
};
```
