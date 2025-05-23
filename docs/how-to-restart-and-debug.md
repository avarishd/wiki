---
tableofcontents: 1
---

# How to restart and debug

AzerothCore is composed of two services: authserver and worldserver.
Authserver only acts as an authenticator and a router for your realms redirecting your authorized client connections to the selected realm address.
The worldserver instead handles all connections related to the game mechanics and it is the single source of truth for everything related to a single realm.

Authserver and worldservers can be placed on different environments. However, in the following guide we will explain how to run them together on the same environment.

## How to start the services

Both authserver and worldserver can be started by simply running the compiled binaries after [completing the installation](Installation).

## How to configure a restarter

Restarting and debugging an application works in many different ways depending on your operating system. That is why we always suggest to use our Docker solution that is fully supported on all platforms.

However, if you need to keep your server up and running after a crash and checking what is going on with your code, you can do it using a restarter and a debugger.

Below is an explanation on how to use our integrated restarter scripts and the [GDB](https://www.gnu.org/software/gdb/) (GNU Project Debugger) utility as well to generate crash-dumps.

### Restarter using acore dashboard (only for bash)

You can use `./acore.sh run-worldserver` and `./acore.sh run-authserver`.

They both work out of the box when you compile with the dashboard.

{% include note.html content="To enable GDB, you can use <b>AC_RESTARTER_WITHGDB=true</b> as an environment variable or by adding it to your <b>/conf/config.sh</b> file.</br>
If the server crashes after enabling GDB, you will find the crashdump file (gdb.txt) within the /env/ folder. <b>Keep in mind that you should compile your code with one of the following compilation types: Debug or RelWithDebInfo, otherwise GDB will not work properly</b>" %}

### Using Docker (cross-platform)

Our Docker system integrates the scripts above within the docker-compose. It means that enabling GDB works exactly in the same way in Docker too.
Moreover our docker-compose uses the [restart-policy feature](https://docs.docker.com/config/containers/start-containers-automatically/) to keep the containers up and running.

For more information, please refer to the [Install-with-Docker](install-with-docker) documentation. 
You will also find a guide on how to debug your code by using VSCode combined with its Remote Docker extension.

### Advanced restarter (only for bash)

For more advanced restarters that include several other useful configurations, you can try our "run-engine" system written in bash.

Here you can find the restarters for linux/bash environments: https://github.com/azerothcore/azerothcore-wotlk/tree/master/apps/startup-scripts

Those scripts are automatically copied after the compilation to the `/dist` directory if you're using our `./acore.sh` dashboard

You can copy the `conf.sh.dist` and create a `conf-world.sh` file to customize those documented configurations (do the same for the `conf-auth.sh`). This way you will have both the restarter and GDB pre-configured to create a `gdb.txt` (crashdump) file when the core crashes. Make sure to use `Debug` or `RelWithDebInfo` compilation (in your CMake command) in order to get meaningful crash reports.

Then copy the `restarter-world.sh` and the `restarter-auth.sh` from the "examples" beside your conf file and in the same folder of the "run-engine" file.

Eventually you will have something like this:

[![example][1]][1]

Run those two restarter scripts to have both authserver and worldserver restarters with GDB support.


### Manual way (crossplatform)

Always make sure to use **Debug** or **RelWithDebInfo** compilation (in your CMake command) in order to get meaningful crash reports.

Create a file called `gdb.conf` with this inside:

    set logging on
    set debug timestamp
    run -c ../etc/worldserver.conf
    bt
    bt full
    info thread
    thread apply all backtrace full

To debug or create a crashdump, you can then use the GDB command as described in its documentation:

```
gdb -x gdb.conf --batch ./worldserver
```

This command should be enough to both attach your IDE to debug your code and also generate a crashdump when the server crashes.

For a more advanced and "universal" restarter, personally I'm using [PM2][2].

```
pm2 start "gdb -x gdb.conf --batch ./worldserver"
```

It should be enough to automatically restart, monitor, and utilize GDB with your server.


  [1]: assets/images/how-to-restart-and-debug/vscode-files-explorer.png
  [2]: https://pm2.keymetrics.io/
