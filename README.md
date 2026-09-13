# Docker Tasks in **GitHub Codespaces** - 15 Points

This assignment is a hands-on introduction to Docker. You don't need any prior Docker experience. Every task below explains **what** you are doing and **why**, so that by the end you'll understand the core concepts behind containers, images, and how Docker actually works.

- In this Codespace, a Docker daemon runs inside the workspace container. The docker CLI is preinstalled and connected. You can open the Terminal and run commands right away, with no setup needed.
- First study [Docker Basics](https://dipaish.github.io/OS22/index.html) properly. I recommend completing all the tasks from the learning materials in the same environment to become familiar with the Docker environment before attempting the practical exercises below. ***You are also free to install Docker in your own personal device if you wish to.***
- You can check your score yourself. You can get a total of **15 points** from the tasks below.
- Remember to frequently commit and push the changes [Task 17. Commit and push](#17-commit-and-push)

---

##  Docker Concepts You Need First

**What is Docker?** Docker is a tool that lets you package an application together with everything it needs to run, code, runtime, libraries, and settings, into a single, portable unit called a **container**. That container runs the same way on your laptop, a classmate's laptop, or a cloud server, because it carries its own environment with it instead of depending on whatever happens to be installed on the machine. This solves the classic "it works on my machine" problem and is why Docker has become the standard way to build, ship, and run applications consistently.

Before you start, it helps to know these four terms. You will meet all of them in the tasks below:

| Term | What it means |
|---|---|
| **Image** | A read-only "template" or blueprint (e.g. `alpine`, `hello-world`). It contains everything needed to run an application: code, runtime, libraries. You **build or download** an image, but you don't run "an image" directly. Instead, you run a *container* from it. |
| **Container** | A running (or stopped) **instance** of an image. Think of it like a class (image) versus an object (container). You can create many containers from the same image, and each one is isolated from the others and from your host machine. |
| **Dockerfile** | A text file with step-by-step instructions (`FROM`, `RUN`, `CMD`, ...) that tells Docker how to **build your own custom image**. |
| **Volume / Bind mount** | Containers are normally *ephemeral*, meaning that when you delete a container, its filesystem changes are gone. A volume/bind mount is a folder shared between your host and the container so data can **persist** even after the container is removed. |

---

## Class Walkthrough: Concepts and Live Demo

This section is meant to be worked through in class first, live, before you attempt the graded tasks below. It explains each core Docker concept and demonstrates it with examples that are close to the graded tasks but not identical to them. By the end, you should understand *why* each command works the way it does, so you can complete the tasks below on your own.

### Session Roadmap

1. What is a container, really? (client and daemon, image vs container)
2. Running your first containers (`docker run`, `hello-world`, pulling images)
3. Interactive containers and the container filesystem
4. Container lifecycle and inspection (`ps`, `start`, `stop`, `rm`, `inspect`, `logs`)
5. Detached services, port publishing, environment variables
6. Stopping, removing, and cleaning up containers
7. Building your own image with a `Dockerfile`
8. Persisting data with bind mounts and volumes

Keep a terminal open the whole time and type the commands live instead of pasting them.

### 1. Concept: Client and Daemon Architecture

The `docker` command you type is like a TV remote control. The Docker daemon (`dockerd`) is the actual TV, running quietly in the background, that does the real work of pulling images, starting containers, and managing storage and networking.

When you press a button on the remote (type a `docker` command), it just sends a signal to the TV (the daemon). The remote itself does not create or run anything. If the TV is off (the daemon is not running), pressing buttons on the remote does nothing, and in the same way, `docker` commands will fail if `dockerd` is not running, even though the `docker` CLI is installed and working fine.

In this Codespace, Docker is already installed and the daemon is already running for you, so you don't need to install or start anything. But if you want to use Docker on your own personal device, the first thing you need to do is install Docker Desktop (or the Docker Engine) for your OS, then make sure the Docker daemon is started (Docker Desktop starts it for you, or on Linux you may need to run `sudo systemctl start docker`). Only once the daemon is running can you actually use any `docker` command.

**Demo:**
```bash
docker version   # shows both Client and Server (daemon) sections
docker info      # daemon level details: number of containers, images, storage driver
```
**Q:** If the daemon is the background service actually running your containers, what do you think happens to those containers if the daemon crashes or restarts?

**A:** since the daemon is what keeps track of and runs everything, your containers can stop responding, and you won't be able to run any new `docker` commands until the daemon is back up. That is the daemon is the real engine doing all the work, while the `docker` command you type is just how you talk to it.

### 2. Concept: Images vs Containers, and Pulling

**Docker Image:** An image is like a recipe or blueprint. It is a read only package that has everything an application needs to run: the code, the tools, the settings, all bundled together. By itself, an image does not do anything, it just sits there on disk (or on Docker Hub) waiting to be used.

**Docker Container:** A container is what you get when you actually run that image. If the image is the recipe, the container is the meal you cooked from it. You can take the exact same recipe (image) and cook it as many times as you want, each time you get a separate meal (container). Each container runs independently: you can start one, stop another, delete one, and it has no effect on the others or on the original recipe.

The first time you use an image you don't already have on your computer, Docker needs to download it from somewhere, usually Docker Hub. This download step is called **pulling**. Once pulled, the image is cached locally, so the next time you use it, Docker skips the download and reuses the copy already on disk.

**What is Docker Hub?** [Docker Hub](https://hub.docker.com/) is the default public registry where Docker images are stored, shared, and downloaded from, similar to how GitHub hosts code repositories. When you run `docker run alpine`, Docker looks locally first, and if the image is not found, it reaches out to Docker Hub and pulls it from there automatically. Anyone can browse Docker Hub in a web browser to see what an image contains, how big it is, and how to use it.

Docker Hub hosts hundreds of thousands of images. A few worth knowing as a beginner:
- [`alpine`](https://hub.docker.com/_/alpine) and [`busybox`](https://hub.docker.com/_/busybox), tiny Linux base images, often just a few MB.
- [`hello-world`](https://hub.docker.com/_/hello-world), a minimal image made purely to test that Docker works.
- [`ubuntu`](https://hub.docker.com/_/ubuntu), a full featured Linux distribution, much bigger than alpine.
- [`nginx`](https://hub.docker.com/_/nginx) and [`httpd`](https://hub.docker.com/_/httpd), ready to use web servers.
- [`node`](https://hub.docker.com/_/node), [`python`](https://hub.docker.com/_/python), and other language runtimes, useful as a base for your own applications.

**Demo, pull and run something that just prints and exits:**
```bash
docker run hello-world
```
Read the printed explanation together. It literally walks through the pull, create, run, stream output, then exit lifecycle.

**Demo, a real tiny Linux environment running a single command:**
```bash
docker run busybox echo "hello from busybox"
```
Notice that the first time this runs, Docker pulls the image (watch the `Pulling from library/busybox` lines). Run it again and no pull happens, because it is now cached locally.

```bash
docker images
```
Show `busybox` and `hello-world` sitting in the local cache along with their sizes.

### 3. Concept: Interactive Mode and Filesystem Isolation

So far every container we ran did one thing and then exited on its own. Now let's actually go inside a container and look around, like stepping into a tiny computer of its own.

A few flags make this possible:

| Flag | What it does |
|---|---|
| `-i` | Keeps the input channel open so you can type things into the container. |
| `-t` | Gives you a proper terminal screen to see what you're typing and its output. |
| `-it` | The two combined, giving you a live, interactive shell inside the container. The experience feels a bit like SSHing into a remote machine, you type commands and see their output as if you were sitting inside it, though technically no network or login is involved, Docker just connects your terminal directly to the container's process through the daemon. |
| `--name` | Gives the container an easy to read name (like `busybox-demo`) instead of a random one Docker would generate for you, so you can refer to it easily afterward. |

**Do this:** start an interactive container and open a shell inside it.
```bash
docker run -it --name busybox-demo busybox /bin/sh
```
You are now inside the container's shell, not your Codespace's shell. Try running a few commands to prove this is a separate little filesystem:
```sh
ls /
mkdir -p /demo/one /demo/two
ls /demo
exit
```
Notice that `ls /` shows a totally different set of folders than your own Codespace file system. That's because the container has its own private filesystem, completely isolated from your host and from any other container.

**Do this next:** type `exit` to leave the shell, then check what happened to the container.
```bash
docker ps            # nothing running
docker ps -a         # busybox-demo shown as "Exited"
```
`exit` only stops the container, it does not delete it. The container and everything inside it (including your `/demo` folders) is still sitting there, just not running.

**Do this too:** bring the container back and prove your files are still there.
```bash
docker start busybox-demo
docker exec busybox-demo ls /demo
```
`docker start` resumes the same container (not a new one), and `docker exec` runs a single command inside it without needing a full interactive shell. Your `/demo/one` and `/demo/two` folders are still there, exactly where you left them.

**Q:** If you now ran `docker rm -f busybox-demo` and then created a brand new container from the `busybox` image, would your `/demo` folders still be there?

**A:** No. `docker rm -f` permanently deletes the container, and its filesystem (including `/demo/one` and `/demo/two`) is deleted along with it. A brand new container starts fresh from the image, with none of the changes made inside the old container. The image itself never changes, only the container built from it did, and now that container is gone for good.

### 4. Concept: Container Lifecycle, Inspecting, and Logs

A container is never just "on" or "off". Over its life it moves through different states: `created` (exists but never started), `running` (currently active), `paused`, and `exited` (stopped, but not deleted). Knowing these states helps you understand what a container is actually doing at any moment, instead of just guessing.

Docker also keeps a full record of everything about a container, its settings, its network details, what it printed to the screen, and more. Two commands let you look at that record without ever needing to go inside the container.

**Do this:** list every container you have, no matter its state.
```bash
docker ps -a
```
Compare this with plain `docker ps` (no `-a`), which only shows containers that are currently running. Stopped containers do not disappear, `docker ps -a` is how you find them again.

**Do this next:** look at the full configuration of a container.
```bash
docker inspect busybox-demo
```
This prints a large block of JSON. Look through it and notice the `"Config"` section (environment variables, the command it runs) and the `"NetworkSettings"` section (its IP address, ports). Whenever something about a container is not behaving as expected, `docker inspect` is the first place to look.

**Do this too:** read whatever the container has printed out.
```bash
docker logs busybox-demo
```
`docker logs` replays everything a container has written to its standard output or error, even if the container is running quietly in the background where you cannot normally see it.

**Q:** If a container has been stopped for a while, will `docker logs` still show you the output it produced before it stopped?

**A:** Yes. Docker keeps the log output attached to the container itself, not just to the live process. As long as the container has not been removed with `docker rm`, its logs are still there for you to read at any time, running or not.

### 5. Concept: Detached Mode, Port Publishing, Environment Variables

Everything so far has run in the foreground, taking over your terminal until it finished. A real application, like a web server, needs to keep running for a long time, so you need a way to start it and get your terminal back. You also need a way to actually reach it from your browser, and a way to configure it without changing its code. Three flags solve these three problems:

| Flag | What it does |
|---|---|
| `-d` | Starts the container in the background, known as detached mode, so your terminal is free to keep working. |
| `-p HOST:CONTAINER` | Publishes a port, connecting a port on your machine to a port inside the container. Without it, the container's network is completely sealed off and invisible from the outside, even though it is running. |
| `-e KEY=VALUE` | Sets an environment variable inside the container at start time, a common way apps read in configuration like a username, a password, or a mode flag, without you having to rebuild anything. |

**Do this:** start a small web server in the background, with a custom environment variable and a published port.
```bash
docker run --name demo-web -e DEMO_MSG="Hello from class" -d -p 9999:80 httpd
```
Notice your terminal returned immediately, the container is running quietly behind the scenes.

**Do this next:** reach the web server from outside the container.
```bash
curl http://127.0.0.1:9999
```
Or click the forwarded port notification if you are in Codespaces. Either way, you are talking to something running inside the container, through the port you published.

**Do this too:** confirm the environment variable actually made it inside the container.
```bash
docker logs demo-web
docker inspect demo-web | grep -A3 '"Env"'
```
Look for `DEMO_MSG=Hello from class` in the output. That is proof the `-e` flag really did pass your value into the container's environment.

Clean up this demo container before moving on:
```bash
docker stop demo-web && docker rm demo-web
```

**Q:** If you had run the container without `-p 9999:80`, would `curl http://127.0.0.1:9999` still have worked?

**A:** No. The web server would still be running fine inside the container, but without publishing the port, there is no path connecting your machine's network to the container's network. From the outside, it would look like nothing is listening on port 9999 at all.

### 6. Concept: Stopping, Removing, and Cleaning Up

By now you have created quite a few containers, and part of working with Docker responsibly is knowing how to clean up. Two commands handle this, and it matters that you understand they do different things. `stop` asks a running container to shut down gracefully, and afterward the container still exists, just sitting in the `exited` state, like the containers you saw earlier with `docker ps -a`. `rm` goes a step further and permanently deletes the container object itself. Removing a container never touches the image it was created from, so you can always create a fresh container from that same image again later.

**Do this:** create a throwaway container, then walk it through both commands.
```bash
docker run -d --name demo-cleanup busybox sleep 300
docker stop demo-cleanup
docker rm demo-cleanup
```
Run `docker ps -a` after each step and watch the container move from `running`, to `exited`, to gone completely.

**Do this next:** clear out everything created during this class demo, a good habit to build early.
```bash
docker rm -f busybox-demo 2>/dev/null
docker rmi demo-image 2>/dev/null
docker images   # confirm what's left locally
```

**Q:** After running `docker rm demo-cleanup`, could you get that exact container back with `docker start demo-cleanup`?

**A:** No. `docker start` only works on a container that still exists, in the `created` or `exited` state. Once you run `docker rm`, the container object itself is gone for good. You could still run a brand new container from the same `busybox` image, but it would be a fresh container with a clean filesystem, not the one you just deleted.

### 7. Concept: Dockerfile and Building Your Own Image (Optional)

Every image used so far was built by someone else and downloaded from Docker Hub. But you can also design your own image from scratch, which is how you would package your own application to run inside Docker. The instructions for building an image live in a plain text file called a `Dockerfile`.

A `Dockerfile` reads a bit like a recipe, a short list of steps Docker follows in order. `FROM` picks the base image to start from, so you are not starting completely from nothing. `LABEL` attaches extra metadata, like who maintains the image. `CMD` sets the default command that runs automatically whenever a container is started from this image. Once the `Dockerfile` is written, `docker build` reads it and produces an actual image you can run, just like any other.

**Do this:** create a folder called `myimage` in this assignment repo to hold the practice Dockerfile, so it stays separate from the real `Dockerfile` you'll write for the assignment later.
```bash
mkdir -p myimage && cd myimage
```

**Do this next:** create a file named `Dockerfile` in that folder with the following content.
```dockerfile
FROM busybox:latest
LABEL maintainer="Instructor Demo"
CMD ["echo", "Hello from a custom image built in class!"]
```

**Do this too:** build the image, then run a container from it.
```bash
docker build -t demo-image .
docker run --name demo-container demo-image
```
The `.` at the end of the build command tells Docker to look for the `Dockerfile` in the current folder. Notice that once it is built, your custom image behaves exactly like `busybox` or `alpine`, Docker does not treat images you built any differently from images you pulled.

Clean up before moving on:
```bash
cd -   # back to previous directory
docker rm demo-container
```

**Q:** If you changed the `CMD` line in the Dockerfile and ran `docker run --name demo-container demo-image` again without rebuilding, would you see the new message?

**A:** No. An image is a snapshot, it only reflects the Dockerfile as it was at the moment you last ran `docker build`. Editing the Dockerfile afterward changes nothing about the image already built. You would need to run `docker build` again to bake the new `CMD` into a new version of the image.

### 8. Concept: Bind Mounts, Making Data Outlive a Container

We already saw that removing a container deletes its filesystem along with it. That is fine for throwaway containers, but it is a problem for anything you actually care about, like a website's files or a database's data. A bind mount solves this by linking a folder on your host machine to a folder inside the container. They are not two copies kept in sync, they are literally the same folder, viewed from two places at once. Whatever you do to one side instantly shows up on the other, and because the real files live on your host, they are completely unaffected by a container being removed.

**Do this:** create a small folder and a starting web page on the host.
```bash
mkdir -p /tmp/demo-site
echo "<h1>Draft page from class</h1>" > /tmp/demo-site/index.html
```

**Do this next:** start an nginx web server, bind mounting that folder into the location nginx serves files from.
```bash
docker run -d --name demo-bind -p 8081:80 -v "/tmp/demo-site:/usr/share/nginx/html" nginx
```
Open `http://localhost:8081` in your browser and you should see the draft page.

**Do this too:** edit `/tmp/demo-site/index.html` directly on the host, change the text, save it, then refresh the browser. Notice there is no restart and no rebuild, the website updates immediately because nginx is reading straight from the bind mounted folder.

**Now prove the data survives even without the container:**
```bash
docker rm -f demo-bind
docker run -d --name demo-bind-2 -p 8081:80 -v "/tmp/demo-site:/usr/share/nginx/html" nginx
```
Refresh the browser again. Your edited content is still there, even though `demo-bind` no longer exists and `demo-bind-2` is a completely new container. The content never lived inside the container, it lived in `/tmp/demo-site` on your host the whole time.

```bash
docker rm -f demo-bind-2
```

**Q:** If instead of a bind mount, you had just copied the file into the container's filesystem with `docker cp`, would it still be there after `docker rm -f`?

**A:** No. A plain copy becomes part of that specific container's own filesystem, so removing the container removes the copy along with it. A bind mount is different because the file was never really inside the container to begin with, it only ever lived on the host and was simply made visible to the container.

### Bridging to the Tasks Below

Here is how each concept demoed above connects to a specific graded task, without giving away the exact commands:

| Concept demoed | Maps to task |
|---|---|
| Client and daemon (`docker version`, `docker info`) | Task 1 |
| Pull and run a single use container | Task 2, 3 |
| `-it --name` interactive shell | Task 4, 5 |
| `docker ps -a` plus saving output with `tee` | Task 6 |
| `-d -p -e` background web service | Task 7, 8 |
| `docker inspect` | Task 9 |
| `docker logs` | Task 10 |
| `docker images` | Task 11 |
| Running a new image in detached mode | Task 12 |
| `docker exec -it` into a running container | Task 13 |
| `docker cp` | Task 14 |
| `stop` and `rm` | Task 15 |
| `Dockerfile` plus `docker build`, running your own image, bind mount persistence | Optional bonus tasks (not graded) |

A few reminders before you start:
- Use the exact names and images the tasks specify (`alpine-it-demo`, `static-site-2`, `redis-demo`, port `8888`, and so on), since the checker script matches on these exact strings.
- Save required outputs into `outputs/` using `tee` or `docker cp`, as described in each task below, or the checker won't find them.
- Run `.github/classroom/check.sh` (Task 16) after each attempt to check progress, and commit and push (Task 17) often.

---

## Task Summary (1-15, 15 points)

Each row links to the full task below, with its concept, detailed instructions, and exact requirements.

| # | Task | Points |
|---|---|---|
| 1 | [Verify Docker in the Codespace](#task-1) | 1 |
| 2 | [Run your first container: `hello-world`](#task-2) | 1 |
| 3 | [Run a container based on the `alpine` image](#task-3) | 1 |
| 4 | [Run a named, interactive alpine container: `alpine-it-demo`](#task-4) | 1 |
| 5 | [Create directories inside the container](#task-5) | 1 |
| 6 | [List all containers and save the output](#task-6) | 1 |
| 7 | [Run a static website container: `static-site-2`](#task-7) | 1 |
| 8 | [View the website in your browser](#task-8) | 1 |
| 9 | [Inspect the container's configuration](#task-9) | 1 |
| 10 | [View and save the container's logs](#task-10) | 1 |
| 11 | [List all downloaded images and save the output](#task-11) | 1 |
| 12 | [Run a detached container from a new image: `redis`](#task-12) | 1 |
| 13 | [Get into a running container's terminal with `docker exec -it`](#task-13) | 1 |
| 14 | [Copy a file out of a container with `docker cp`](#task-14) | 1 |
| 15 | [Stop and remove a container](#task-15) | 1 |

---

<a id="task-1"></a>
## 1. Verify Docker in the Codespace

**Concept: the Docker client/daemon architecture.** The `docker` command you type is the *client*; it talks to a background service called the *daemon* (or "engine") which actually creates and runs containers. This task just confirms both are alive and can talk to each other.

Run the command that prints both the client and server (daemon) version information, then run the separate command that prints daemon-level details such as the number of containers and images and the storage driver in use. Neither command should produce an error.

**✅ Task 1 (1 point):** Both the version and info commands should run without any errors.

---

<a id="task-2"></a>
## 2. Run your first container: `hello-world`

**Concept: images vs. containers, and `docker pull`.** When you run a container from an image you don't have locally yet, Docker automatically **pulls** (downloads) it from Docker Hub first, then creates a container from it.

Run a container based on the `hello-world` image. No extra flags are needed, if you don't already have the image, Docker will pull it automatically before running it. Read the message it prints, it explains the exact steps Docker just performed: pull the image, create a container, run it, stream the output back to you, and then exit.

**✅ Task 2 (1 point):** A container based on the `hello-world` image has been run.

---

<a id="task-3"></a>
## 3. Run a container based on the `alpine` image

**Concept: minimal base images.** `alpine` is a tiny (~5MB) Linux distribution, commonly used as a lightweight base for other images. Unlike `hello-world` (which just prints a message and exits), `alpine` gives you an actual mini Linux environment you can interact with.

Run a one-off container from the `alpine` image whose only job is to print the text `hello from alpine` and then exit. This container doesn't need a name or an interactive shell, a single throwaway run is enough to prove `alpine` works.

**✅ Task 3 (1 point):** A container based on `alpine` has been run.

---

<a id="task-4"></a>
## 4. Run a **named, interactive** alpine container: `alpine-it-demo`

**Concept: interactive mode (`-it`) and naming containers.** By default, `docker run` gives your container a random name and exits as soon as its command finishes. The `-it` flags (interactive + pseudo-TTY) keep an open shell session inside the container so you can type commands into it, just like SSH-ing into a machine. Naming a container with `--name` makes it easier to refer to later (`docker start alpine-it-demo`, `docker exec alpine-it-demo ...`) instead of using random IDs.

Start a new container from the `alpine` image with an interactive shell attached, so you land inside a live `/bin/sh` prompt rather than the container running once and exiting. Give this container the exact name `alpine-it-demo`, since later tasks and the checker both refer to it by that name.

You are now *inside* the container's shell. Notice that this is a completely separate filesystem from your Codespace. Running `ls /` will look nothing like your repo.

**✅ Task 4 (1 point):** A container named `alpine-it-demo` exists.

---

<a id="task-5"></a>
## 5. Create directories inside the container

**Concept: container filesystem isolation.** Every container gets its own private filesystem, layered on top of its image. Changes you make, like creating folders, only exist inside that container. They don't affect your host, and they don't affect other containers, even ones created from the same image.

By this point you've likely already left the `alpine-it-demo` shell from Task 4, which stopped the container. That's fine, the container still exists, just not running, and its filesystem is preserved exactly as it was. Start it back up (hint: `docker start`) and get an interactive terminal inside it again (hint: `docker exec -it`), using that same container, don't create a new one.

Once you're back inside its shell, create two folders, `/lab/dir1` and `/lab/dir2` (one command that creates both, including any missing parent folders, is enough). List the contents of `/lab` to confirm both exist, then leave the shell.

Leaving the shell **stops** the container again. It isn't removed, so its filesystem, including your two folders, is preserved until you run `docker rm` on it.

**✅ Task 5 (1 point):** `/lab/dir1` and `/lab/dir2` both exist inside `alpine-it-demo`.

---

<a id="task-6"></a>
## 6. List all containers (running + stopped) and save the output

**Concept: container lifecycle states.** A container can be `created`, `running`, `paused`, `exited`, or `removed`. Plain `docker ps` only shows **running** containers. You need the `-a` flag, short for "all", to also see stopped ones, like `alpine-it-demo` right now.

Make sure a folder named `outputs` exists in the repo root (create it if it doesn't). Then list every container on this system, running and stopped alike, and save that full listing into `outputs/ps-a.txt`, in a way that also shows the same output in your terminal so you can see it worked (hint: pipe the command's output through `tee`).

**✅ Task 6 (1 point):** `outputs/ps-a.txt` exists and lists `alpine-it-demo`.

---

<a id="task-7"></a>
## 7. Run a static website container: `static-site-2`

**Concept: detached mode (`-d`), port publishing (`-p`), and environment variables (`-e`).** Long-running services, like a web server, shouldn't tie up your terminal. The `-d` flag runs the container in the background, known as "detached" mode. Containers have their own private network by default, so `-p HOST:CONTAINER` **publishes** a container port to a port on your machine so you can actually reach it. The `-e` flag passes an environment variable into the container, a common way to configure apps without changing their code.

Start a container from the `dockersamples/static-site` image in detached (background) mode, named exactly `static-site-2`. Pass in an environment variable named `AUTHOR` set to your own name, and publish the container so that host port `8888` maps to the container's port `80`.

***Check and read about the `static-site` image on Docker Hub to see what it does.***

**✅ Task 7 (1 point):** `static-site-2` is running with host port `8888` mapped to container port `80`.

---

<a id="task-8"></a>
## 8. View the website in your browser

**Concept: why port publishing matters.** Without `-p 8888:80`, the container's port 80 would only be reachable from *inside* Docker's internal network, so it would stay invisible to you. Publishing it is what lets your browser, outside the container, reach the app running inside the container.

Confirm the website actually responds on port 8888. You can either click **Open in Browser** on the port 8888 notification, or make an HTTP request to `127.0.0.1` on port 8888 from the terminal and check you get a response back.

**✅ Task 8 (1 point):** The website responds on port 8888.

---

<a id="task-9"></a>
## 9. Inspect the container's configuration

**Concept: using `docker inspect` to view metadata about a container.** Every container has a large JSON document describing its config: environment variables, network settings, mounted volumes, IP address, and more. This is the go-to command when you need to debug "why isn't this container behaving the way I expect?".

Inspect the full configuration of `static-site-2` and save that output into `outputs/inspect-static-site-2.txt` (hint: pipe the output through `tee`). Search the saved file for the `"AUTHOR="` entry, you should find the environment variable you passed in Task 7, which proves it made it into the container's config.

**✅ Task 9 (1 point):** `outputs/inspect-static-site-2.txt` exists and contains your `AUTHOR` environment variable.

---

<a id="task-10"></a>
## 10. View and save the container's logs

**Concept: using `docker logs` to read stdout/stderr of a container.** Containers don't have a screen; whatever a process inside prints to standard output/error is captured by Docker and can be replayed at any time with `docker logs`, even for containers running in the background. This is the first place to look when something is misbehaving.

Read the logs produced by `static-site-2`, and save that output into `outputs/static-site-2-logs.txt` (hint: pipe the output through `tee`).

**✅ Task 10 (1 point):** `outputs/static-site-2-logs.txt` exists and is not empty.

---

<a id="task-11"></a>
## 11. List all downloaded images and save the output

**Concept: the local image cache.** Every image you've pulled or built is cached locally so it doesn't need to be re-downloaded every time. `docker images` shows this cache, including image size, which helps you understand why some images, like `alpine`, are preferred for being small.

List every image currently cached on your machine, and save that listing into `outputs/images.txt` (hint: pipe the output through `tee`). The saved file should include `alpine`, `hello-world`, and `dockersamples/static-site` among the entries.

**✅ Task 11 (1 point):** `outputs/images.txt` exists and lists `alpine`, `hello-world`, and `dockersamples/static-site`.

---

<a id="task-12"></a>
## 12. Run a detached container from a new image: `redis`

**Concept: exploring a new image beyond the ones you've already used.** So far you've used `hello-world`, `alpine`, and `dockersamples/static-site`. Docker Hub hosts hundreds of thousands more images, each with its own purpose. `redis` is a popular in-memory data store, often used for caching. Running it shows that the same pattern you've already used for detached containers works for *any* image, not just the ones you've already tried.

Start a new container from the `redis` image in detached (background) mode, named exactly `redis-demo`. No port publishing or environment variables are required for this task, just get it running in the background.

***Check and read about the `redis` image on Docker Hub to see what it does.***

**✅ Task 12 (1 point):** A container named `redis-demo`, based on the `redis` image, is running.

---

<a id="task-13"></a>
## 13. Get into a running container's terminal with `docker exec -it`

**Concept: `docker exec` vs. `docker run -it`.** In Task 4, `docker run -it` started a *brand-new* container with an interactive shell. But `redis-demo` is *already running* in the background from Task 12. `docker exec -it` lets you jump into an *existing* running container's terminal without stopping or restarting it, here using redis's own command-line client.

**Step 1:** Attach an interactive terminal to your already-running `redis-demo` container using its own command-line client (hint: `docker exec -it redis-demo redis-cli`), then type `PING` at the prompt. You should see `PONG` printed back, proof the interactive session reached the real `redis-demo` process. Type `exit` (or press `Ctrl+D`) to leave the client when you're done.

**Step 2:** The interactive session above is only for you to look at, it can't be piped to a file. To actually save the result for the checker, run the ping as a single, non-interactive command (drop `-it`) and pipe its output through `tee`:
```bash
docker exec redis-demo redis-cli ping | tee outputs/redis-ping.txt
```
This runs `redis-cli ping` once, prints `PONG` straight to your terminal, and saves that same output into `outputs/redis-ping.txt`.

**✅ Task 13 (1 point):** `outputs/redis-ping.txt` exists and contains `PONG`.

---

<a id="task-14"></a>
## 14. Copy a file out of a container with `docker cp`

**Concept: moving files between a container and your host without a bind mount.** A bind mount (covered as an optional bonus task) is great for ongoing, live syncing, but sometimes you just need to grab *one* file out of a container, for example to inspect a config file or log after the fact. `docker cp` copies files or folders between your host and any container, running or stopped, no special setup required.

Start `alpine-it-demo` back up in the background first (hint: `docker start`), then copy the `/etc/hostname` file out of it onto your host, saving it as `outputs/hostname-copy.txt`. Every container has this file, Docker creates it automatically, so it's a safe, reliable target regardless of the base image.

**✅ Task 14 (1 point):** `outputs/hostname-copy.txt` exists on your host and is not empty.

---

<a id="task-15"></a>
## 15. Stop and remove a container

**Concept: cleaning up with `docker stop` and `docker rm`.** `docker stop` gracefully shuts down a running container. It still exists afterwards, in the `exited` state. `docker rm` permanently deletes a stopped container. Removing a container never removes the underlying image, so you can always create a fresh container from it again.

Pick any container you no longer need (or start a quick throwaway one just for this purpose), stop it gracefully, then remove it completely. This proves you can perform both halves of container cleanup, stopping and removing.

**✅ Task 15 (1 point):** You have successfully stopped and removed a container.

---

## Optional Bonus Tasks (Not Graded): Build Your Own Image & Persist Data

These tasks are purely for extra learning. They carry **no points** and are **not checked** by `check.sh`. They cover two more core Docker skills, building your own image with a `Dockerfile`, and persisting data with a bind mount, which are great to know even though they aren't required for your score.

### Bonus A. Write a `Dockerfile` and build your own image

**Concept: using a `Dockerfile` and `docker build` to create custom images.** So far you've only used images built by other people. A `Dockerfile` lets you define your **own** image: what base image to start from (`FROM`), and what it should do when run (`CMD`). This is the foundation of using Docker for your own applications.

Create a file named `Dockerfile` in the root of this repository:

```dockerfile
FROM alpine:latest
LABEL maintainer="Your Name"
CMD ["echo", "Hello, Docker! This is my first custom image."]
```

Then build it (the `.` means "use the Dockerfile in the current folder", `-t` gives your image a name/tag):

```bash
docker build -t my-first-image .
```

### Bonus B. Run a container from your own image

**Concept: your custom image behaves just like any other image.** Once built, `my-first-image` can be run exactly like `alpine` or `hello-world`. Docker doesn't distinguish between images you built and images you downloaded.

```bash
docker run --name my-first-container my-first-image | tee outputs/my-first-container-output.txt
```

### Bonus C. Persist data with a bind mount

**Concept: using a bind mount to make data outlive a container.** Normally, when you remove a container, everything it wrote is gone. A **bind mount** connects a folder on your host to a folder inside the container, so they are actually the *same* folder. Any file you create or edit on either side instantly shows up on the other, and it stays on your host even after the container is deleted.

**Step 1:** Create a small website folder on your host with a starting page:

```bash
mkdir -p site
echo "<h1>Version 1: created before the bind mount</h1>" > site/index.html
```

**Step 2:** Run an `nginx` web server container, bind-mounting your `site` folder into the folder nginx serves files from:

```bash
docker run -d --name bind-mount-site -p 8080:80 -v "$(pwd)/site:/usr/share/nginx/html" nginx
```

Open **http://localhost:8080** (or click the port 8080 notification). You should see "Version 1".

**Step 3:** Now edit `site/index.html` directly on your host, for example right in the VS Code editor, and change the text:

```html
<h1>Version 2: edited directly on the host, no rebuild needed!</h1>
```

Save the file, then refresh your browser. The website updates immediately, with no restart and no rebuild, because nginx is reading straight from your bind-mounted folder.

**Step 4:** Prove the data survives even when the container is gone. Remove the container completely:

```bash
docker rm -f bind-mount-site
```

Then create a **brand-new** container, bind-mounting the exact same folder:

```bash
docker run -d --name bind-mount-site-2 -p 8080:80 -v "$(pwd)/site:/usr/share/nginx/html" nginx
```

Refresh **http://localhost:8080** again. Your "Version 2" content is still there, even though `bind-mount-site` no longer exists. The content never lived inside the container. It lived in `site/index.html` on your host the whole time.

---

## Optional Extra (Not Graded): Explore a New Docker Image

This task is purely for your own additional learning. It carries **no points** and is **not checked** by `check.sh` or graded in any way.

**Concept: exploring the Docker Hub ecosystem.** So far you've only used a handful of images picked for you. Docker Hub hosts hundreds of thousands of images for almost every purpose. Being able to find, understand, and safely try out a new image is a core Docker skill.

**What to do (optional):**

1. Go to [Docker Hub](https://hub.docker.com/) and find a lightweight, official/well-maintained image you haven't used yet in this assignment. Pick something from a category that interests you, for example:
   - AI / Machine Learning (e.g. a small model-serving or notebook image)
   - Security (e.g. a vulnerability scanner or network diagnostic tool)
   - Databases (e.g. `redis`, `postgres`, `mongo`)
   - Or any other category you find interesting.
2. Write a short paragraph describing what the image/application does, why it's useful, and why you consider it lightweight (for example, its image size from `docker images`).
3. Pull and run the image with `docker run`, and confirm it actually starts and works as expected.
4. Take a screenshot showing it running, just to have a record for yourself.

---

## 16. Run the check.sh script and know your score
```bash
chmod +x .github/classroom/check.sh
.github/classroom/check.sh > result.txt

# After you have run the above command, you can check your score by typing 

cat result.txt 

```

>> Note 

- If you’re not satisfied with your score, you may correct any mistakes you’ve made. You can try as many times as you like.
- After making the corrections, **re-run Task 16** again and check your score. 
- **Always remember to commit and push the changes you have made by doing Task 17 before closing the GitHub Codespace. You need to do this frequently. Every time you work on the tasks, remember to commit and push.**


## 17. Commit and push
```bash
git add .
git commit -m "Completed 15 docker tasks"
git push
```
