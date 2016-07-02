# DC2 Setup

0. [Unpacking your DC2](#0-unpacking-your-dc2)
1. [Power up your DC2](#1-power-up-your-dc2)
2. [Find the DC2 on your network](#2-find-the-dc2-on-your-network)
3. [Configure the DC2](#3-configure-the-dc2)
4. [Add the DC2 as a docker machine](#4-setup-dc2-as-a-docker-machine)
5. [Assemble the DC2](#5-dc2-assembly)
6. [FAQ](#6-faq)

## 0. Unpacking your DC2

- The first time you open the container doors, be really careful. The container is a model and was not designed to be an enclosure. If a hinge breaks, you can set it aside. If you have success gluing your hing back on, please share.

- We booted all of the MinnowBoard Turbots to make sure they would cleanly boot. See the FAQ below if your MinnowBoard does not boot up.

## 1. Power up your DC2

1. Let's make sure your MinnowBoard boots up and get it all setup while we can easily see the lights etc. before putting it in the container. If you have more than one DC2, let's set them up separately.

2. Set your MinnowBoard on the pink foam to protect it from shorting out.

3. Connect your USB drive to the MinnowBoard. Plug it into the USB socket that is blue.

4. Connect your MinnowBoard with an ethernet cable to the same network your PC that you will use to setup the DC2 is connected to.

5. OPTIONALLY connect your MinnowBoard to a display with a micro HDMI connector.

6. The last connection will be connecting your MinnowBoard to power.

7. You should see the USB drive light flicker. If you have an display connected, you should see the the boot sequence scroll across.

    ![test setup](./images/test_setup.jpg)


## 2. Find the DC2 on your network

1. Make sure you have Bonjour / mDNS.

    When you connect your DC2 to your network, it will be using DHCP to get an IP address. To work with the DC2 from other machines, we will need to find out where it is. The DC2 will default to having the '''dc2.local''' hostname which will be broadcast with mDNS aka Bonjour. If you have a Mac or Windows 10 machine, you are good to go. If you have an earlier version of Windows, you will need to install [Bonjour Services](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US), If you are on a linux box, you can install [Avahi](https://wiki.archlinux.org/index.php/Avahi).

2. If you have more than one DC2, then set them up one at a time so you can change the host names to differentiate between them.

3. Check you can connect to your DC2.

    Open up [http://DC2.local:8765](http://DC2.local:8765) with your browser -- this should return the configuration information from your DC2.

    It can take 40 seconds for the DC2 to boot, and 30 seconds for the dc2-node script to run and Bonjour to have completed its broadcast so that loading the web page to work. You should see something like this:

    ```json
    {
        "latestVersion": "1.0.4",
        "version": "1.0.4",
        "hostname": "dc2",
        "ip": "a.b.c.d",
        "MAC": "ff:ff:ff:ff:ff:ff",
        "callHomeResponse": 200
    }
    ```

    Where `a.b.c.d` is the IP address of your DC2 and `ff:ff:ff:ff:ff:ff` is the MAC address of your DC2 ethernet port.

## 3. Configure the DC2

1. Check that you can SSH into your DC2.

    - `username`: `jack` is the preconfigured username
    - `password`: `hardtware` is the preconfigured password

    ```shell
    ssh jack@dc2.local
    ```

2. Change your password

    ```shell
    passwd
    ```

    Enter `hardtware` for the old password, and then enter your new password.

3. If you don't have an ssh key, on *another* machine run the following:

    ```shell
    ssh-keygen -b 4096 -f ~/.ssh/id_rsa.dc2
    ```

    You will be prompted to create a passphrase, create one and enter it in twice as prompted.

4. Copy your SSH public key to your DC2.

    First `exit` your SSH session with your DC2.

    If your SSH public key is at `~/.ssh/id_rsa.pub` (if created above, use `~/.ssh/id_rsa.dc2`) then from your machine:

    ```shell
    ssh jack@dc2.local "cat >> .ssh/authorized_keys" < ~/.ssh/id_rsa.pub
    ```

    Enter your new password when prompted.

    You should now be able to `ssh jack@dc2.local` and not be prompted for a password.

5. If you are not in the `America/Los Angeles` timezone you can change it with:

    ```shell
    sudo dpkg-reconfigure tzdata
    ```

6. If you want to change the locale, follow directions at [https://help.ubuntu.com/community/Locale](https://help.ubuntu.com/community/Locale).

7. If you have more than one DC2, change your hostname by replacing `dc2` to a new hostname unique on your network (eg. dc2a, dc2b, dc2c) in the `/etc/hostname` and `/etc/hosts` files.

    For example, if you wanted to change the hostname to `dc2a`, you would run:

    ```shell
    sudo echo dc2a /etc/hostname
    sudo sed -i 's/dc2/dc2a/1' /etc/hosts
    ```

    Then restart `hostname` and `avahi` services with:

    ```shell
    sudo service hostname restart
    sudo /etc/init.d/avahi-daemon restart
    ```

    > NOTE: Everwhere you see `dc2` below, change that to the new hostname you have given your DC2.

8. OPTIONALLY change the username from `jack` to something else. This requires a few more commands and should be considered a bit more advnaced of a topic.

    In our example we will change the username to `bob` and remove `jack` entirely. For this example, start my being logged in as `jack`.

    Create the new user:

    ```shell
    sudo useradd -m bob
    ```

    Set a `sudo` password for the user:

    ```shell
    sudo passwd bob
    ```

    Create the `ssh` directory for the user:

    ```shell
    sudo mkdir /home/bob/.ssh
    sudo chown bob:bob /home/bob/.ssh
    sudo chmod 700 /home/bob/.ssh
    ```

    If you created and copied over a ssh key for jack, let's copy it over to bob.

    ```shell
    sudo cp /home/jack/.ssh/authorized_keys /home/bob/.ssh/authorized_keys
    sudo chown bob:bob /home/bob/.ssh/authorized_keys
    ```

    The last item is to add `bob` to all of the appropriate groups that `jack` is a member of:

    ```shell
    sudo usermod -a -G adm bob
    sudo usermod -a -G cdrom bob
    sudo usermod -a -G sudo bob
    sudo usermod -a -G dip bob
    sudo usermod -a -G plugdev bob
    sudo usermod -a -G lpadmin bob
    sudo usermod -a -G dc2 bob
    sudo usermod -a -G sambashare bob
    ```

    In a new terminal window, try to SSH now as bob using pubkey auth:

    ```shell
    ssh -o IdentityFile=~/.ssh/id_rsa bob@dc2.local
    ```

    > NOTE: You will need to change `~/.ssh/id_rsa` to your respective key path from above and `dc2.local` to whatever you changed your hostname to above as well.

    If you are able to SSH in at this point you have correctly created another user and setup their SSH key correctly.

    As the final step, you may remove the `jack` user from the device.

    ```shell
    sudo userdel -r jack
    ```

9. OPTIONALLY disable password authentication and only allow pubkey authentication. This is recommended for most users but is considered optional because it is more advanced.

    Bring up `/etc/ssh/sshd_config` in your favorite editor and uncomment the line that has `PasswordAuthentication`. Change the value to `no` if it is not set as such.

    Once completed, run:

    ```shell
    sudo service ssh restart
    ```

10. OPTIONALLY update your packages.

    Run:

    ```shell
    sudo apt-get update
    sudo apt-get upgrade
    ```

    You may need to enter `y` to approve the updates.

## 4. Setup DC2 as a Docker Machine

1. Make sure you have the [Docker Toolbox](https://www.docker.com/products/docker-toolbox) intalled on your computer.

2. Find the IP address of your DC2:

    ```shell
    ping dc2.local
    ```

    Substitute `a.b.c.d` in the following commands with your DC2's IP address

3. Check that you can use SSH against your IP address:

    ```shell
    ssh dc2@a.b.c.d
    ```

   If you had previously used SSH against that IP address, your will likely get an error and you will need to update your `known_hosts` file.

4. Add your SSH key to your agent.

    ```shell
    ssh-add ~/.ssh/id_rsa
    ```

    You will be prompted for your SSH key passphrase. You may need to change `~/.ssh/id_rsa` to the path of your key that you created.

5. OPTIONALLY create a separate user to run `docker`. This is a security measure.

    On the DC2, create a user and password for the user:

    ```shell
    sudo useradd -m dockeradmin
    sudo passwd dockeradmin
    sudo mkdir /home/dockeradmin/.ssh
    sudo chown dockeradmin:dockeradmin /home/dockeradmin/.ssh
    sudo chmod 700 /home/dockeradmin/.ssh
    ```

    Back on your machine, create a SSH key and copy it to the DC2. Make sure *not* to enter a passphrase for this user, unlike your SSH key pair which most certainly should have a passphrase.

    ```shell
    ssh-keygen -b 4096 -f ~/.ssh/id_rsa.dc2.docker
    ```

    With your favorite editor, create `/home/dockeradmin/.ssh/authrorized_keys` to contain the public half of the keypair (`~/.ssh/id_rsa.dc2.docker.pub`). And make sure to set the proper permissions:

    ```shell
    sudo chown dockeradmin:dockeradmin /home/dockeradmin/.ssh/authorized_keys
    sudo chmod 600 /home/dockeradmin/.ssh/authorized_keys
    ```

    Give the ability to this user full `sudo` access:

    ```shell
    sudo visudo
    ```

    Copy/paste the following towards the bottom after the `%sudo` definition:

    ```shell
    # Docker
    dockeradmin     ALL=(ALL) NOPASSWD: ALL
    ```

    If everything was done perfectly, you should now be able to SSH as this user to the DC2.

    ```shell
    ssh -o IdentityFile=~/.ssh/id_rsa.dc2.docker dockeradmin@a.b.c.d
    ```

6. Setup the DC2 as a generic machine:

    ```shell
    docker-machine create --driver generic --generic-ssh-user=jack --generic-ip-address=a.b.c.d dc2
    ```

    You may need to change the username if you created a separate one above. This command takes a while as it will be updating the DC2. It will create add your DC2 as a machine to run containers in. Sometimes `docker-machine` emits an error about certs.

    An example successful command and output might look like the following (with a couple more options):

    ```shell
    docker-machine create --driver generic --generic-ssh-user dockeradmin --generic-ssh-key ~/.ssh/id_rsa.dc2.docker --generic-ip-address 10.1.12.2 --generic-ssh-port 22 dc2
    ```

    ```
    Running pre-create checks...
    Creating machine...
    (dc2) Importing SSH key...
    Waiting for machine to be running, this may take a few minutes...
    Detecting operating system of created instance...
    Waiting for SSH to be available...
    Detecting the provisioner...
    Provisioning with ubuntu(upstart)...
    Installing Docker...
    Copying certs to the local machine directory...
    Copying certs to the remote machine...
    Setting Docker configuration on the remote daemon...
    Checking connection to Docker...
    Docker is up and running!
    To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env dc2
    ```

7. Check the status of your DC2 docker machine:

    ```shell
    docker-machine status dc2
    ```

    It should state `Running` at this point.

8. Setup the DC2 as your docker machine:

    To view the environment for your docker machine:

    ```shell
    docker-machine env dc2
    ```

    It should output several lines.

    For assistance with this or any `docker` command you can always run `docker-machine help <command>`.

    Where command can be `env` or any other `docker-machine` command.

9. Run the hello-world container:

    First, you need to setup your local environment by running the above `env` command. The output of that command at the end should have something like:

    ```shell
    # Run this command to configure your shell:
    # eval $(docker-machine env dc2)
    ```

    Run the `eval ...` command.

    After that, run:

    ```shell
    docker run hello-world
    ```

    You should see "Hello from Docker" along with various other output. You now have a functining Desktop Computer Container!

10. Shutting down your DC2. Make sure you properly shutdown your DC2 so that the file system does not get corrupted. If it does, you will need to connect a monitor and keyboard to fix the errors on boot.

    ```shell
    ssh jack@dc2.local
    sudo shutdown -h now
    ```

    Success! Disconnect power, ethernet and HDMI (if connected) and finish assembly.

## 5. DC2 Assembly

1. Connect the MinnowBoard Turbot to the faceplate.

    1. Get a #1 (or #2 if need be) philips head screwdriver and screw the 2 M3 philips screws each in a few turns. You want the screws to go in from the left side if you are looking at the front of the faceplate. We are doing this to loosen the threads as the faceplate threads are 3D printed.

        ![mounting screws](./images/mounting_screws.jpg)

    2. Take out the screws, set the MinnowBoard Turbot on the faceplate, and screw the screws back in. Tighten enough that the MinnowBoard Turbot does not wobble around on the faceplate. If you have ordered a SilverJaw Lure, you can first mount it to the MinnowBoard Turbot placing the spacers between the two boards, then screw the screws into the faceplate, and then screw the screws into the standoff at the other end of the boards.

        ![mounting screws](./images/base_assembly.jpg)

        With SilverJaw

        ![mounting screws](./images/Silverjaw_Assembly.jpg)

2. Connnect the USB drive to the MinnowBoard if it is not connected.

3. If you have an OLED you need to connect it before installing the MinnowBoard Turbot into the container.

    1. Just as we did with the screws for mounting the MinnowBoard, we need to clean out the threads for the OLED mount. Screw the M1.6 screws into the OLED mounting locations with the supplied allen wrench, and then take them back out.

        ![mounting screws](./images/OLED_screws.jpg)

    2. Insert the memory card into the OLED. This image also shows how the cable will be connected.

        ![mounting screws](./images/OLED_assembly.jpg)

    3. Mount the OLED with the 4 M1.6 hex head screws.

        ![mounting screws](./images/OLED_mounting.jpg)

    4. Connect the cable from the USB 2.0 connector to the OLED. Make sure you connect per the image and then coil the cable as indicated. You can use the twist tie from the power supply to hold the OLED cable.

        ![mounting screws](./images/OLED_connection.jpg)

        Full Assembly

        ![mounting screws](./images/full_assembly.jpg)

    5. NOTE: we are still working on software to drive the OLED.

4. Slide the assembled MinnoBoard and faceplate into the container. It should click a little when fully installed.

6. Reconnect ethernet and power and enjoy your DC2!

## 6. FAQ

1. **Question**: Something is not working?

    **Answer**: Check the FAQ below. If not answered, file an issue at https://github.com/hardtware/DC2/issues

2. **Question**: What is the CR1225 battery for?

    **Answer**: It is a spare battery. Your MinnowBoard Turbot should have a CR1225 battery in it already.

3. **Question**: I see `WRITE SAME failed. Manually zeroing.` on boot. What is happening?

    **Answer**: You don't need to worry about this and can ignore it. If you would like to make this message go away since it is unrelated to this hardware (it has to do with SCSI driver which is irrelevant for us), you can! Copy the following script to `/usr/local/sbin/disable-write-same`:

    ```shell
    #! /bin/sh
    # Disable SCSI WRITE_SAME, which is not supported by underlying disk
    # emulation.  Run on boot from, eg, /etc/rc.local
    #
    # See http://www.it3.be/2013/10/16/write-same-failed/
    #
    # Written by Ewen McNeill <ewen@naos.co.nz>, 2014-07-17
    #---------------------------------------------------------------------------

    find /sys/devices -name max_write_same_blocks |
        while read DISK; do
            echo 0 >"${DISK}"
        done
    ```

    And then add a call on boot to this script before the `exit` in `/etc/rc.local`.

    On reboot you should no longer see this message.

    Source [link](http://ewen.mcneill.gen.nz/blog/entry/2014-07-17-mininet-on-ubuntu-14.04-in-kvm/).

4. **Question**: The machine did not boot. What happened?

    **Answer**: Sometimes something gets out of sync somewhere with the onboard memory and the drive and Ubuntu thinks something is corrupted and you have to tell it to continue. If you have a display attached you can see it. You can just hit `i` to ignore and everything will be fine. You might need to hit `i` a few times. If you don't have a monitor, try connecting a USB keyboard and hit `i` a few times. If you see the drive light flash, you should be booting.

5. **Question**: How do I remove the faceplate once I have assmebled it?

    **Answer**: To take out the faceplate and the board, put your fingers in the large hex holes, push the faceplate down towards the bottom of the container as the top of the faceplate hooks to the top inside of the container, then wiggle and pull to get it out.

6. **Question**: How can I stack up my containers?

    **Answer**: [remonlam](https://github.com/remonlam) has created an STL you can 3D print to stack up your containers [here](./stacker.md) 
