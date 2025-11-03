# minecraft-server-docker

A simple template to quickly set up your own Minecraft server in a Docker container! This has been done on my home server running Ubuntu Server.



# Setting-up the server

## Nginx configuration

Firstly, install and enable nginx on your host with:
```bash
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

Then, in `minecraft.conf` replace `to_fill` with your server domain (either IP or domain name). You can optionally change you dynmap port if you want to.

Now, it's time to add the minecraft nginx configuration file to `/etc/nginx/conf.d`:
```bash
sudo ln -s path/to/minecraft-server-docker/minecraft.conf /etc/nginx/conf.d/ # replace with your actual path to minecraft-server-docker
```

Test the nginx conf file with `sudo nginx -t`. If the file is correct, reload nginx with `sudo nginx -s reload`.

## Start your server!

You're almost there! Have a look at *docker-compose.yml* where you can:
- name your docker container
- set [environment variables](https://docker-minecraft-server.readthedocs.io/en/latest/variables/) to configure your server and override some server properties. It is here that you can select your **server type**, **server version**, and do some other useful configuration.
- configure your ports. The syntax is *exposed_port:container_port*. 25565 is the default Minecraft server port, and 8123 is the port I choose for dynmap.

Run your server with `docker-compose up -d`. The first start is taking a while because it is setting up everything, you can check the status by running `docker-compose logs -f`. 

To stop the server, run `docker-compose down`.

To see the **server logs** in real time, use `docker logs -f <container_name>`.

To launch **server commands**, use `docker exec -it <container_name> /bin/bash`. Then input `rcon-cli <command>`.

The server has been configured to allow only **whitelisted** players. To manage the whitelist, you can use `whitelist add <user>`, `whitelist remove <user>`, and `whitelist list`. You can access all whitelist info in the *data/whitelist.json* file. Use `/whitelist reload` to apply changes made to this file. If you don't want this, set `WHITELIST: "FALSE"` in *docker-compose.yml*.

**Add an admin**: `op <player_name>`



# Server Worlds

The worlds are in their respective folders *world*, *world_nether*, and *world_the_end*. To recreate a new world, simply delete these directories, create empty new ones and restart the server.



# Configure the server

You can **edit some server properties using environment variables provided in the _docker-compose.yml_ file**. You can find the list [here](https://docker-minecraft-server.readthedocs.io/en/latest/variables/).

But you can't edit everything with the aforementionned technique. For more control, you can edit */minecraft/data/server.properties* to configure what you want.

For example:
- Add `bonus_chest=true` to enable it.
- Set `online_mode=false` to allow cracked versions to connect (**If true, I advise you to add a plugin for authentication**)
- "motd" is the server name
- "difficulty" can be set to *peacefull, easy, normal, hard*
- `force-gamemode=true` so that users can't use the /gamemode command


*Note: the `JVM_DD_OPTS: "disable.watchdog:true"` variable is to tell the server to not start the watchdog server, which crashes the server when using autopause because it thinks the tick is taking forever.



# Server Plugins

To add a plugin, simply download the *.jar* file, and place it inside the */minecraft/plugins* folder. Then restart the server and the plugin will automatically be launched.


## Dynmap

Download the *.jar* file, then upload it to the server in the *plugins* directory.

Restart the server, it will set-up the plugin and create a **dynmap** folder.

In this folder, you can **edit *configuration.txt* to configure dynmap**. Here is what I configured:
- Change *deftemplatesuffix* to **vlowres** for better performances
- Change your *webserver-port* to **any number over 1024**, I used 8123 (**be consistent with your *docker-compose.yml* and in *minecraft.conf***)
- Change the *image format* to jpg
- In the *dynmap/worlds.txt* folder, **enable your world like showed in the template**. I just used:
```bash
worlds:
  - name: world
    title: "Your server name"
    enabled: true 
```

Now, restart the server.
- Go to the server console and type `dynmap fullrender <worldname>`, or as a player in the server run `dynmap fullrender`
- Go to the web browser at address *server_ip:dynmap_port*

You can now see your map in the web browser!

### Commands

- **Pause rendering**: `/dynmap pause all`
- **Resume rendering**: `/dynmap pause none`
- **Toggle render messages**: `/dynmap quiet`

You can also add markers to the map:
- /dmarker add "<label> (icon:)"
- /dmarker delete "<label>"
- /dmarker delete id:<id>
- /dmarker list
- /dmarker icons


## Luckperms

This plugin is used to manage users permissions. You can find it here: [Luckperms](https://luckperms.net/download).

The admin user may need to be an op.

Here are the commands:

- Get a UI for managing everything: `/lp editor all`
- Create a group: `/lp creategroup <group>`
- Add an user to a group: `/lp user <username> parent add <group>`
- Add permissions to a group: `/lp group <groupname> permission set <permission> true`
- Get group infos: `/lp group <groupname> info`
- Get user infos: `/lp user <username> info`
- Get all groups: `/lp listgroups`
- Export to see everything in a file: `/lp export`

Examples:

```
/lp creategroup admin
/lp group admin permission set luckperms.* true
/lp group admin permission set minecraft.command.* true
/lp creategroup default
/lp group default permission set minecraft.command.* false
/lp group default permission set luckperms.* false
/lp user Route_Goudronnee parent add admin
```


## EssentialsX

EssentialsX provides lots of useful commands and is lightweight, in my case I use it to protect my world from new non identified players.

To install it, simply download essentialsX and essentialsXAntiBuild, and put the *.jar* files inside the */plugins* folder.

After restarting the server, the config is located into */plugins/Essentials/config.yml*, in the "AntiBuild" section.

Thanks to *luckperms*, we can set **essentials.build.\*** command to **false** for the *default* group to disallow them all interactions with our world. You can do that with `/lp editor all`. Create another group than default for your trusted users.



# Server backup

Stop the server.

Run `sudo tar -cvzf path/to/backup.tar.gz path/to/minecraft-server-docker/minecraft` to create a full backup of the server.
You can automate it using `cron`, I let you do your own researches for that.



# Updating to new Minecraft version

- First, stop the server
- Create a backup of the server
- Delete old plugins *.jar* files
- Update the new ones in *minecraft/plugins* folder
- Update the *docker-compose.yml* to use the new Minecraft version
- Start the server



# Re-generate a part of the world

To do so, you can use the very cool app [Amulet](https://www.amuletmc.com/).

Make sure to do a **backup** of your world server before attempting anything.

Then, open amulet (amulet_app.exe), and open your world. On the left, open "3D Editor". You can then move as you want in your world. You can switch between 3D/2D view, and between normal selection and chunk selection. If you want to delete and regenerate a part of the world, just select it, and click "suppr". After you finished, don't forget to save.

You can then just replace the old "world" folder with the new one saved from amulet. All deleted parts will be reloaded like it was originally.
