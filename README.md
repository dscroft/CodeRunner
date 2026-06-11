<!--
author:   André Dietrich

email:    LiaScript@web.de

version:  0.0.4

language: en

narrator: US English Female

@onload
window.CodeRunner = {
    ws: undefined,
    handler: {},
    connected: false,
    error: "",
    url: "",
    firstConnection: true,
    pingTimer: null,

    init(url, step = 0) {
        this.url = url
        if (step  >= 10) {
           console.warn("could not establish connection")
           this.error = "could not establish connection to => " + url
           return
        }

        this.ws = new WebSocket(url);

        const self = this
        
        const connectionTimeout = setTimeout(() => {
          self.ws.close();
          console.log("WebSocket connection timed out");
        }, 5000);
        
        
        this.ws.onopen = function () {
            clearTimeout(connectionTimeout);
            self.log("connections established");
            self.connected = true
            if (self.pingTimer) clearInterval(self.pingTimer)
            self.pingTimer = setInterval(function() {
              try { self.ws.send("ping") } catch (_) {}
            }, 15000);
        }
        this.ws.onmessage = function (e) {
            // e.data contains received string.

            let data
            try {
                data = JSON.parse(e.data)
            } catch (e) {
                self.warn("received message could not be handled =>", e.data)
            }
            if (data) {
                self.handler[data.uid](data)
            }
        }
        this.ws.onclose = function () {
            clearTimeout(connectionTimeout);
            if (self.pingTimer) { clearInterval(self.pingTimer); self.pingTimer = null }
            self.connected = false
            self.warn("connection closed ... reconnecting")
            setTimeout(function(){
                console.warn("....", step+1)
                self.init(url, step+1)
            }, 1000)
        }
        this.ws.onerror = function (e) {
            clearTimeout(connectionTimeout);
            if (self.pingTimer) { clearInterval(self.pingTimer); self.pingTimer = null }
            self.warn("an error has occurred")
        }
    },
    log(...args) {
        window.console.log("CodeRunner:", ...args)
    },
    warn(...args) {
        window.console.warn("CodeRunner:", ...args)
    },
    handle(uid, callback) {
        this.handler[uid] = callback
    },
    send(uid, message, sender=null, restart=false) {
        const self = this
        if (this.connected) {
          message.uid = uid
          this.ws.send(JSON.stringify(message))
        } else if (this.error) {

          if(restart) {
            sender.lia("LIA: terminal")
            this.error = ""
            this.init(this.url)
            setTimeout(function() {
              self.send(uid, message, sender, false)
            }, 2000)

          } else {
            //sender.lia("LIA: wait")
            setTimeout(() => {
              sender.lia(" " + this.error)
              sender.lia(" Maybe reloading fixes the problem ...")
              sender.lia("LIA: stop")
            }, 800)
          }
        } else {
          setTimeout(function() {
            self.send(uid, message, sender, false)
          }, 2000)
          
          if (sender) {
            
            sender.lia("LIA: terminal")
            if (this.firstConnection) {
              this.firstConnection = false
              setTimeout(() => { 
                sender.log("stream", "", [" Waking up execution server ...\n", "This may take up to 30 seconds ...\n", "Please be patient ...\n"])
              }, 100)
            } else {
              sender.log("stream", "", ".")
            }
            sender.lia("LIA: terminal")
          }
        }
    }
}

//window.CodeRunner.init("wss://coderunner.informatik.tu-freiberg.de/")
//window.CodeRunner.init("ws://localhost:4000/")
window.CodeRunner.init("wss://ancient-hollows-41316.herokuapp.com/")
@end


@LIA.ada:               @LIA.eval(`["main.ada"]`, `gnatmake main.ada`, `./main`)
@LIA.algol:             @LIA.eval(`["main.alg"]`, `none`, `a68g main.alg`)
@LIA.apl:               @LIA.eval(`["main.apl"]`, `none`, `dyalog -script main.apl`)
@LIA.awk:               @LIA.eval(`["main.awk"]`, `none`, `awk -f main.awk`)
@LIA.basic:             @LIA.eval(`["main.bas"]`, `none`, `bwbasic main.bas`)
@LIA.c:                 @LIA.eval(`["main.c"]`, `gcc -Wall main.c -o a.out`, `./a.out`)
@LIA.clojure:           @LIA.eval(`["main.clj"]`, `none`, `clojure -M main.clj`)
@LIA.clojure_withShell: @LIA.eval(`["main.clj"]`, `none`, `clojure -M -i main.clj -r`)
@LIA.cpp:               @LIA.eval(`["main.cpp"]`, `g++ main.cpp -o a.out`, `./a.out`)
@LIA.cobol:             @LIA.eval(`["main.cob"]`, `cobc -x --free main.cob`, `./main`)
@LIA.coq:               @LIA.eval(`["file.v"]`, `coqc file.v`, `coqtop -lv file.v`)
@LIA.d:                 @LIA.eval(`["main.d"]`, `gdc main.d`, `./a.out`)
@LIA.elixir:            @LIA.eval(`["main.exs"]`, `none`, `elixir main.exs`)
@LIA.elixir_withShell:  @LIA.eval(`["main.exs"]`, `none`, `iex main.exs`)
@LIA.erlang:            @LIA.eval(`["hello.erl"]`, `erlc hello.erl`, `erl -noshell -s hello hello -s init stop`)
@LIA.erlang_withShell:  @LIA.eval(`["hello.erl"]`, `erlc hello.erl`, `erl -noshell -s hello hello`)
@LIA.forth:             @LIA.eval(`["main.fs"]`, `none`, `gforth main.fs -e BYE`)
@LIA.forth_withShell:   @LIA.eval(`["main.fs"]`, `none`, `gforth main.fs`)
@LIA.fortran:           @LIA.eval(`["main.f90"]`, `gfortran main.f90 -o a.out`, `./a.out`)
@LIA.go:                @LIA.eval(`["main.go"]`, `go build main.go`, `./main`)
@LIA.groovy:            @LIA.eval(`["main.groovy"]`, `none`, `groovy main.groovy`)
@LIA.haskell:           @LIA.eval(`["main.hs"]`, `ghc main.hs -o main`, `./main`)
@LIA.haskell_withShell: @LIA.eval(`["main.hs"]`, `none`, `ghci main.hs`)
@LIA.haxe:              @LIA.eval(`["Main.hx"]`, `none`, `haxe -main Main --interp`)
@LIA.inform:            @LIA.eval(`["main.inf"]`, `inform -o main.inf > compile.log && [ -f "main.z5" ] || { cat compile.log >&2; exit 1; }`, `/usr/games/dfrotz main.z5`)
@LIA.io:                @LIA.eval(`["main.io"]`, `none`, `io main.io`)
@LIA.io_withShell:      @LIA.eval(`["main.io"]`, `none`, `io -i main.io`)
@LIA.java:              @LIA.eval(`["@0.java"]`, `javac @0.java`, `java @0`)
@LIA.julia:             @LIA.eval(`["main.jl"]`, `none`, `julia main.jl`)
@LIA.julia_withShell:   @LIA.eval(`["main.jl"]`, `none`, `julia -i main.jl`)
@LIA.kotlin:            @LIA.eval(`["main.kt"]`, `kotlinc main.kt -include-runtime -d main.jar`, `java -jar main.jar`)
@LIA.lua:               @LIA.eval(`["main.lua"]`, `none`, `lua main.lua`)
@LIA.mono:              @LIA.eval(`["main.cs"]`, `mcs main.cs`, `mono main.exe`)
@LIA.nasm:              @LIA.eval(`["main.asm"]`, `nasm -felf64 main.asm && ld main.o`, `./a.out`)
@LIA.nim:               @LIA.eval(`["main.nim"]`, `nim c main.nim`, `./main`)
@LIA.nodejs:            @LIA.eval(`["main.js"]`, `none`, `node main.js`)
@LIA.ocaml:             @LIA.eval(`["main.ml"]`, `none`, `ocaml main.ml`)
@LIA.perl:              @LIA.eval(`["main.pl"]`, `perl -c main.pl`, `perl main.pl`)
@LIA.perl_withShell:    @LIA.eval(`["main.pl"]`, `perl -c main.pl`, `perl -d main.pl`)
@LIA.php:               @LIA.eval(`["main.php"]`, `none`, `php main.php`)
@LIA.postscript:        @LIA.eval(`["input.ps"]`, `none`, `gs -sDEVICE=png16m -r300 -o output.png input.ps`)
@LIA.prolog:            @LIA.eval(`["main.pl"]`, `none`, `swipl -s main.pl -g @0 -t halt`)
@LIA.prolog_withShell:  @LIA.eval(`["main.pl"]`, `none`, `swipl -s main.pl`)
@LIA.python:            @LIA.python3
@LIA.python_withShell:  @LIA.python3_withShell
@LIA.python2:           @LIA.eval(`["main.py"]`, `python2.7 -m compileall .`, `python2.7 main.pyc`)
@LIA.python2_withShell: @LIA.eval(`["main.py"]`, `python2.7 -m compileall .`, `python2.7 -i main.pyc`)
@LIA.python3:           @LIA.eval(`["main.py"]`, `none`, `python3 main.py`)
@LIA.python3_withShell: @LIA.eval(`["main.py"]`, `none`, `python3 -i main.py`)
@LIA.r:                 @LIA.eval(`["main.R"]`, `none`, `Rscript main.R`)
@LIA.r_withShell:       @LIA.eval(`["main.R"]`, `none`, `sh -c "cat main.R - | R --interactive"`)
@LIA.racket:            @LIA.eval(`["main.rkt"]`, `none`, `racket main.rkt`)
@LIA.rexx:              @LIA.eval(`["main.rexx"]`, `none`, `rexx ./main.rexx`)
@LIA.ruby:              @LIA.eval(`["main.rb"]`, `none`, `ruby main.rb`)
@LIA.ruby_withShell:    @LIA.eval(`["main.rb"]`, `none`, `irb --nomultiline -r ./main.rb`)
@LIA.rust:              @LIA.eval(`["main.rs"]`, `rustc main.rs`, `./main`)
@LIA.scala:             @LIA.eval(`["@0.scala"]`, `scalac @0.scala`, `scala @0`)
@LIA.scheme:            @LIA.eval(`["main.scm"]`, `none`, `guile --no-auto-compile main.scm`)
@LIA.selectscript:      @LIA.eval(`["main.s2"]`, `none`, `S2c -x main.s2`)
@LIA.smalltalk:         @LIA.eval(`["main.st"]`, `none`, `gst main.st`)
@LIA.solidity:          @LIA.eval(`["@0.sol"]`, `solcjs --abi @0.sol`, `python3 -mjson.tool @0_sol_@0.abi`)
@LIA.tcl:               @LIA.eval(`["main.tcl"]`, `none`, `tclsh main.tcl`)
@LIA.v:                 @LIA.eval(`["main.v"]`, `v main.v`, `./main`)
@LIA.v_withShell:       @LIA.eval(`["main.v"]`, `none`, `sh -c "cat main.v - | v repl"`)
@LIA.verilog:           @LIA.eval(`["main.v"]`, `iverilog -o main.vvp main.v`, `vvp main.vvp`)
@LIA.vhdl:              @LIA.eval(`["@0.vhdl"]`, `ghdl -a @0.vhdl && ghdl -e @0`, `ghdl -r @0`)
@LIA.zig:               @LIA.eval(`["main.zig"]`, `zig build-exe ./main.zig -O ReleaseSmall`, `./main`)

@LIA.dotnet
```xml    -project.csproj
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```
@LIA.eval(`["Program.cs","project.csproj"]`, `dotnet build -nologo`, `dotnet run`)
@end

@LIA.fsharp
```xml    -project.csproj
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <Compile Include="Program.fs" />
  </ItemGroup>
</Project>
```
@LIA.eval(`["Program.fs", "project.fsproj"]`, `dotnet build -nologo`, `dotnet run`)
@end

@LIA.qsharp
```xml    -project.csproj
<Project Sdk="Microsoft.Quantum.Sdk/0.28.302812">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
</Project>
```
@LIA.eval(`["Program.qs", "project.csproj"]`, `dotnet build -nologo`, `dotnet run`)
@end

@LIA.eval:  @LIA.eval_(false,`@0`,@1,@2,@3)

@LIA.evalWithDebug: @LIA.eval_(true,`@0`,@1,@2,@3)

@LIA.eval_
<script>
function random(len=16) {
    let chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    let str = '';
    for (let i = 0; i < len; i++) {
        str += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return str;
}

function get_file(path) {

    var baseUrl = window.location.href.split('#')[0].split('?')[1];
    var baseDir = baseUrl.substring(0, baseUrl.lastIndexOf('/') + 1);
    var url = new URL(path, baseDir).href;

    var xhr = new XMLHttpRequest();
    xhr.open('GET', url, false);
    xhr.send(null);
    if (xhr.status >= 200 && xhr.status < 300 || xhr.status === 0) {
        return xhr.responseText;
    }
    throw new Error('Could not access ' + path + ', ' + xhr.status + ' ' + xhr.statusText);
}

function add_file(name, content) {
    if (Array.isArray(name)) {
        content = get_file(name[1])
        name = name[0]
    }

    files.push([name, content])
}

const uid = random()
var order = @1
var files = []

var pattern = "@4".trim()

if (pattern.startsWith("\`")){
  pattern = pattern.slice(1,-1)
} else if (pattern.length === 2 && pattern[0] === "@") {
  pattern = null
}

function f() {
    try
    {
        if (order[0])
            add_file(order[0], `@'input(0)`)
        if (order[1])
            add_file(order[1], `@'input(1)`)
        if (order[2])
            add_file(order[2], `@'input(2)`)
        if (order[3])
            add_file(order[3], `@'input(3)`)
        if (order[4])
            add_file(order[4], `@'input(4)`)
        if (order[5])
            add_file(order[5], `@'input(5)`)
        if (order[6])
            add_file(order[6], `@'input(6)`)
        if (order[7])
            add_file(order[7], `@'input(7)`)
        if (order[8])
            add_file(order[8], `@'input(8)`)
        if (order[9])
            add_file(order[9], `@'input(9)`)
    }
    catch (error)
    {
        console.warn(error.message);
        return "LIA: stop"
    }

    send.handle("input", (e) => {
        CodeRunner.send(uid, {stdin: e}, send)
    })
    send.handle("stop",  (e) => {
        CodeRunner.send(uid, {stop: true}, send)
    });

    CodeRunner.handle(uid, function (msg) {
        switch (msg.service) {
            case 'data': {
                if (msg.ok) {
                    CodeRunner.send(uid, {compile: @2}, send)
                }
                else {
                    send.lia("LIA: stop")
                }
                break;
            }
            case 'compile': {
                if (msg.ok) {
                    if (msg.message) {
                        if (msg.problems.length)
                            console.warn(msg.message);
                        else
                            console.log(msg.message);
                    }

                    send.lia("LIA: terminal")
                    CodeRunner.send(uid, {exec: @3, filter: pattern})

                    if(!@0) {
                    console.clear()
                    }
                } else {
                    send.lia(msg.message, msg.problems, false)
                    send.lia("LIA: stop")
                }
                break;
            }
            case 'stdout': {
                if (msg.ok)
                    console.stream(msg.data)
                else
                    console.error(msg.data);
                break;
            }

            case 'stop': {
                if (msg.error) {
                    console.error(msg.error);
                }

                if (msg.images) {
                    for(let i = 0; i < msg.images.length; i++) {
                        console.html("<hr/>", msg.images[i].file)
                        console.html("<img title='" + msg.images[i].file + "' src='" + msg.images[i].data + "' onclick='window.LIA.img.click(\"" + msg.images[i].data + "\")'>")
                    }
                }

                if (msg.videos) {
                    for(let i = 0; i < msg.videos.length; i++) {
                        console.html("<hr/>", msg.videos[i].file)
                        console.html("<video controls style='width:100%' title='" + msg.videos[i].file + "' src='" + msg.videos[i].data + "'></video>")
                    }
                }

                if (msg.files) {
                    let str = "<hr/>"
                    for(let i = 0; i < msg.files.length; i++) {
                        str += `<a href='data:application/octet-stream${msg.files[i].data}' download="${msg.files[i].file}">${msg.files[i].file}</a> `
                    }

                    console.html(str)
                }

                window.console.warn(msg)

                send.lia("LIA: stop")
                break;
            }

            default:
                console.log(msg)
                break;
        }
    })


    CodeRunner.send(
        uid, { "data": files }, send, true
    );

    return "LIA: wait"
}

f()
</script>
@end
-->


# CodeRunner

Same as André's original version https://github.com/liascript/CodeRunner excepting
that you can now specify additional code files based on relative path, do not have 
to include all code in codeblocks within the markdown.

If you want to load a file then instead of providing just a filename for in the list of 
files for parameter 1 of `@LIA.eval` and `@LIA.evalWithDebug`you can provide a list of two elements.
Where the first element is the filename to be used and the second element is the relative path to load the file from. 

Will probably do a pull request to the original once I figure how to handle the 
asynchronous file loading properly, which is currently done synchronously for simplicity.

Only tested with C++ so far.

Should be fully backwards compatible with the original.

````
```cpp main.cpp
#include <iostream>

#include "func.h"

int main() {
    std::cout << "The Ultimate Answer is: " << ultimate_answer() << std::endl;
    return 0;
}
```
@LIA.eval(`["main.cpp", ["func.h", "assets/func.h"]]`, `g++ main.cpp`, `./a.out`)
````

```cpp
#include <iostream>

#include "func.h"

int main() {
    std::cout << "The Ultimate Answer is: " << ultimate_answer() << std::endl;
    return 0;
}
```
@LIA.eval(`["main.cpp", ["func.h", "assets/func.h"]]`, `g++ main.cpp`, `./a.out`)
