# Hack FaaSter on Play With Docker

These instructions will run through getting [OpenFaaS](https://www.openfaas.com/) ([GitHub](https://github.com/openfaas/faas)) and the [OpenFaaS CLI](https://github.com/openfaas/faas-cli) running in a playground environment on [Play with Docker](https://labs.play-with-docker.com/).

Most of the steps you follow here would be exactly the same as running this on your local machine (once you have Docker installed)

If you want to learn more about Docker itself, check out the [Play with Docker Classroom](http://training.play-with-docker.com/).

## Setup Play With Docker

* https://labs.play-with-docker.com/
* Click 'Login'
    * Login with a docker ID, or create a new one
* Click 'Start'
* Click 'Add new instance'
    * A new instance should be created and return a linux shell for you
* Sometimes it seems to randomly not work.. close the session and try again

## Setup Docker Swarm

* https://docs.docker.com/engine/swarm/
* Docker swarm allows multiple servers to act as a single managed cluster.
* We won't go into specifics here.. but OpenFaaS runs on top of swarm mode, so we need to initialise that.
* In the provided linux shell..
    * `docker swarm init --advertise-addr eth0`

eg.

```
$ docker swarm init --advertise-addr eth0
Swarm initialized: current node (ymqf03rsjx3z862yme7w9iex9) is now a manager.

To add a worker to this swarm, run the following command:

..etc..
```

## Setup OpenFaaS

* https://github.com/openfaas/faas
* In the provided linux shell..
    * `git clone https://github.com/openfaas/faas.git`
    * `cd faas`
    * `./deploy_stack.sh`

eg.

```
$ git clone https://github.com/openfaas/faas.git
Cloning into 'faas'...
remote: Counting objects: 12561, done.
remote: Compressing objects: 100% (319/319), done.
remote: Total 12561 (delta 105), reused 381 (delta 87), pack-reused 12145
Receiving objects: 100% (12561/12561), 18.44 MiB | 20.66 MiB/s, done.
Resolving deltas: 100% (3837/3837), done.

$ ./deploy_stack.sh
Deploying stack
Creating network func_functions
Creating service func_nodeinfo
Creating service func_hubstats
Creating service func_alertmanager

..etc..
```

* Check that it deployed properly
    * `docker stack ls`
    * `docker service ls`

eg.

```
$ docker stack ls
NAME                SERVICES
func                11

$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                             PORTS
mmtk7oiuilf3        func_alertmanager   replicated          1/1                 functions/alertmanager:latest      *:9093->9093/tcp
6frmcirixo16        func_base64         replicated          1/1                 functions/alpine:latest
qf6hhr2e289p        func_decodebase64   replicated          1/1                 functions/alpine:latest

..etc..
```

## Setup OpenFaaS CLI

* https://github.com/openfaas/faas-cli
* This provides the command line tool for interacting with the OpenFaaS installation and our functions.
* In the provided linux shell..
    * `curl -sSL https://cli.openfaas.com | sudo sh`
    * `faas-cli --help`
* We can check that it is all working by listing the available functions
    * `faas-cli list`

eg.

```
$ curl -sSL https://cli.openfaas.com | sudo sh
x86_64
Getting package https://github.com/openfaas/faas-cli/releases/download/0.4.30/faas-cli
Attemping to move faas-cli to /usr/local/bin
New version of faas-cli installed to /usr/local/bin

$ faas-cli --help

Manage your OpenFaaS functions from the command line

Usage:
  faas-cli [flags]
  faas-cli [command]

Available Commands:

..etc..

$ faas-cli list
Function                        Invocations     Replicas
func_base64                     0               1
func_echoit                     0               1
func_markdown                   0               1

..etc..
```

## Invoking functions with the CLI

* Functions can be invoked from the OpenFaaS CLI tool using the `invoke` command
* In the provided linux shell..
    * `faas-cli invoke --help`
    * `echo "Hack the planet!" | faas-cli invoke func_echoit`
        * This will send the text "Hack the planet!" to the `func_echoit` function on the OpenFaaS server
        * When this function executes, it will just return any text sent to it
* You can play around with any of the other functions shown in `faas-cli list`

eg.

```
# Echo
$ echo "Hack the planet!" | faas-cli invoke func_echoit
Hack the planet!

# Convert Markdown into HTML
echo -e '# Hack FaaSter\n\nThis is cool!\n\n* Hack\n* Hack FaaSter\n* ???\n* Profit' | faas-cli invoke func_markdown

# Base64 Encode
$ echo '1337 encr(ode)tion!' | faas-cli invoke func_base64
MTMzNyBlbmNyeXB0aW9uISg/KQo=

# Base64 Decode
$ echo 'SSBhbSBkZWZpbml0ZWx5IHRoZSBtb3N0IGg0eDByIQo=' | faas-cli invoke func_decodebase64
I am definitely the most h4x0r!
```

* Because `faas-cli` generally follows the linux principals of 1 task to 1 function, and composing these together, you can actually chain the output from 1 function invocation to the next.
* A completely contrived example..

```
# Echo -> Base64 Encode -> Base64 Decode
echo "Hack the planet!" | faas-cli invoke func_echoit | faas-cli invoke func_base64 | faas-cli invoke func_decodebase64
Hack the planet!
```

* There is also a web interface available.
* From your 'Play with Docker' session, click on the `8080` link near the top of the page.
* This will open the OpenFaas Portal, which allows you to create and invoke functions from a web UI.

## Community and Sample Functions

* There are lots of sample functions
    * https://github.com/openfaas/faas/tree/master/sample-functions
* And a pile of community-contributed OpenFaaS functions, with more popping up all the time!
    * https://github.com/faas-and-furious
        * https://github.com/faas-and-furious/openfaas-mememachine
        * https://github.com/faas-and-furious/youtube-dl
        * etc
* We can even deploy them directly from GitHub with OpenFaaS CLI
    * `faas-cli deploy --help`

eg.

```
# MemeMachine
faas-cli deploy -f https://raw.githubusercontent.com/faas-and-furious/openfaas-mememachine/master/mememachine.yml

echo '{"top": "HACK", "bottom": "FAASTER!", "image": "https://i.ytimg.com/vi/u3CKgkyc7Qo/hqdefault.jpg"}' | faas-cli invoke mememachine > meme.jpg

# Youtube Download + Gif Maker (might be slow..)
faas-cli deploy -f https://raw.githubusercontent.com/0xdevalias/faas-youtube-dl/modernize-mystack/stack.yml
faas-cli deploy -f https://raw.githubusercontent.com/openfaas/faas/master/sample-functions/samples.yml --filter "*gif*"

echo "https://www.youtube.com/watch?v=BBJa32lCaaY" | faas-cli invoke youtubedl | faas-cli invoke gif-maker > ra.gif
```

* Since we can't display the images/etc in our shell.. we can cheat and run a little python webserver
* Now we could do this pretty easily with python directly, but then we would have to figure out the URL to access it:
    * `python -m SimpleHTTPServer 1337`
* It'll be something like this.. (you could click the `8080` link then change `8080` to `1337` if you wanted..)
    * http://ipTHEIPBITHERE-SOMERANDOMBITSHERE-1337.direct.labs.play-with-docker.com/meme.jpg
* But since 'Play with Docker' will create a nice link for us if we use docker for it.. let's do that!
    * `docker run --rm -it -p 1337:8000 -v `pwd`:/HackFaaSter -w /HackFaaSter python:alpine python -m http.server`
* This will..
    * Download and run the python alpine container
    * `-v`: Map our current directory (`pwd`) into the container at `/HackFaaSter`
    * `-w`: Change the working directory inside the container to `/HackFaaSter`
    * Start a python webserver on port `8000` inside the container
    * `-p`: Map port `8000` inside the container to port `1337` on our server
* Now if we look at the top of the 'Play With Docker' interface, we should see a link to `1337`
    * Clicking on this will open our webserver with a directory listing
    * And if we find and click on `meme.jpg`, we should see our pic!
    * Do the same for `ra.gif`, etc as desired
* To close the webserver, back in our shell, just press `Ctrl-C`
* Note: sometimes the functions don't run perfectly as expected, so if you got a black image, try running it again

## Gobuster

* Turning the focus to more security-relevant tools, we can also run gobuster as a function
    * https://github.com/0xdevalias/faas-gobuster/
* Gobuster is a "Directory/file & DNS busting tool written in Go", that may be used during the discovery phase of an engagement, to find previously unknown files and subdomains.

```
faas-cli deploy -f https://raw.githubusercontent.com/0xdevalias/faas-gobuster/master/stack.yml
echo "-w /words/quicktestwords.txt -u http://google.com" | faas-cli invoke gobuster
```

## Creating your own function

* OpenFaaS makes it pretty easy to create your own functions too
    * `faas-cli new --help`
* You can list the templates available
    * `faas-cli new --list`
* And then create a new function
    * `faas-cli new --lang python3 foo`

eg.

```
⇒  faas-cli new --lang python3 foo
2017/11/11 16:04:51 No templates found in current directory.
2017/11/11 16:04:51 HTTP GET https://github.com/openfaas/faas-cli/archive/master.zip
2017/11/11 16:04:55 Writing 250Kb to master.zip

..snip..

Folder: foo created.
  ___                   _____           ____
 / _ \ _ __   ___ _ __ |  ___|_ _  __ _/ ___|
| | | | '_ \ / _ \ '_ \| |_ / _` |/ _` \___ \
| |_| | |_) |  __/ | | |  _| (_| | (_| |___) |
 \___/| .__/ \___|_| |_|_|  \__,_|\__,_|____/
      |_|


Function created in folder: foo
Stack file written: foo.yml

⇒  ls
foo/  foo.yml  template/

⇒  ls foo/
__init__.py  handler.py  requirements.txt
```

* `foo.yml` contains the stack file for your function
* The `/foo` folder contains your functions code files, at this stage it's basically just standard python
    * `requirements.txt` is used to list dependencies for your project
    * `handler.py` is where you'll put your code

Let's make it do something.. add the following to `./foo/requirements.txt` (you can use `echo pyfiglet >> ./foo/requirements.txt`)

```
pyfiglet
```

And edit `./foo/handler.py` to contain the following code (`vi ./foo/handler.py`):

```
from pyfiglet import figlet_format

def handle(st):
    print(figlet_format(st, font='starwars'))

```

You can use the following if you're not up for console based text editing (`vi` <3):

```
echo "from pyfiglet import figlet_format

def handle(st):
    print(figlet_format(st, font='starwars'))
" > ./foo/handler.py
```

Now we can build, deploy and run our function with OpenFaaS!

```
faas-cli build -f foo.yml
faas-cli deploy -f foo.yml

echo 'Hack the planet!' | faas-cli invoke foo
```
