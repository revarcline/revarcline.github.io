[systemd](https://systemd.io/) is a modern init system for linux created by Red Hat. It's not without its detractors, who tend to decry it as bloated and overly-centralized, but given its inclusion in most popular linux distributions, like Debian, Ubuntu, Red Hat, and Arch, like it or not it's here to stay. 

For an end-user, systemd offers a fairly easy way to automate jobs or run persistent processes. For the most part, your distribution will handle setting up these services for you and automatically enable them to run as needed. In some cases, you may want to run a script or program persistently from launch, and making your own systemd service to do this is a pretty painless endeavor.
<!--more-->

In this post, we're going to use the example of my [dr. bronner soap label twitter bot](https://github.com/revarcline/bronnerbot), which currently posts once an hour to [@alloneallbot](https://twitter.com/alloneallbot) and is deployed on a headless ubuntu server in my living room. I wanted to make sure the program stayed running, relaunched if something went wrong, and launched on boot. My original implementation was to just directly run the python script in a tmux pane (lol) and wanted to make something a little more robust for it. This is where systemd came in handy.

Like a lot of python projects, I created a [virtual environment](https://docs.python.org/3/library/venv.html) for it, to ensure that the necessary dependencies could be easily installed. Basically, a python virtual environment installs the relevant python binary and pip packages in a subdirectory of the project, allowing you to create the exact environment necessary to run it. As such, in order to run the bronner bot, the script needs to be run by the version of python in the bronnerbot venv.

The (example) service unit file I created deploy the bot looks something like this:

```
#bronnerbot.service

[Unit]
Description=markov chain dr bronner twitter bot
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=1
User=markovbot
# set to project root directory
WorkingDirectory=/path/to/bronnerbot
# be sure to exec with venv python, abs path
ExecStart=/path/to/bronnerbot/venv/bin/python3 /path/to/bronnerbot/bot.py

[Install]
WantedBy=multi-user.target
```

Let's go through this line by line. First, note how there are three sections to the file, each enclosed with square brackets: `[Unit]`, `[Service]`, and `[Install]`.

In the `[Unit]` section, we give a user-readable description of the service in the `Description` key, then instruct the service when to start with the `After` key. In this case, since the bot requires network connectivity to work, we want to run it after `network.target`.

The meat of our instructions live in the `[Service]` section. The `Type` key is set to `simple`, which is the default start-up type, which will run the command denoted by `ExecStart`. To see other types and when you might want to use them, check the [systemd documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html#Type=). Next, we set `Restart` to `always` and `RestartSec` to `1`. This tells systemd to always restart the program if it fails after waiting for one second. The `User` key denotes which user will run the process. I created a user called `markovbot` to run it the bot, but it can be run by any user on the system. If the key is set to a user that does not exist, the process will not run.

Now, in order to properly run the python script, since it relies on files in the project directory, we set the `WorkingDirectory` key to the **absolute path** of the project. If it is sitting in `~/code/bronnerbot`, for instance, you will want to set it to `/home/my_user/code/bronnerbot`. Similarly, you will need to use the absolute path in the `ExecStart` to the virtual environment's python executable (in `venv/bin/python3`) and the script itself. There are a number of useful options available for this key, and are available [in the documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecStart=).

Finally, we have the `[Install]` section. Here, we designate what particular systemd target unit the program should run under. These correlate to the old-school SysVinit runlevels. In the case of `multi-user.target`, we are basically saying we want the bot to run when the system can run a login shell. This is one of the most common levels at which systemd service units run, and is the runlevel of most headless servers. By including this setting, we denote to systemd that we want this service to run automatically.

Now that the bot has a service file, installing it and getting it working is just a matter of root access and a few commands. First, you want to copy your service file to the `systemd/system` folder:
```
sudo cp bronnerbot.service /etc/systemd/system/
```
Next, have systemd reload all of its service files with
```
sudo systemctl daemon-reload
```
To get the service running, run the following two commands:
```
sudo systemctl start bronnerbot.service
sudo systemctl enable bronnerbot.service
```
The first one, `start`, activates the service unit. By running `enable`, we ensure that it is activated on boot going forward.

To check if your new service is working and see log output, run `systemctl status bronnerbot`. This will show you relevant information like whether it's active, the PID, memory usage, and program output.

There are a wide variety of other ways you can use systemd service units, including timers and sockets. I highly recommend diving into it as a linux user, you'll find you can solve a lot of sticky problems with the tools available through systemd.
