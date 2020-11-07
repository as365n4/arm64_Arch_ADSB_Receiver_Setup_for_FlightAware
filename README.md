# arm64 Arch ADSB Receiver Setup for FlightAware

#### 1.)  Follow the instructions in my repo `Arch_on_Rock64` or the instructions	given on the Arch Wiki https://archlinuxarm.org/platforms/armv8/rockchip/rock64	A 16GB eMMC module is sufficient as the whole setup requires only 2.5GB in space when finished.

#### 2.)  Install system monitoring

`sudo pacman -S screenfetch htop python-pip python-psutil python-future python-bottle hddtemp lm_sensors glances`

`sudo mkdir /etc/glances`

`sudo cp /usr/share/doc/glances/glances.conf /etc/glances/glances.conf`

`sudo nano /etc/glances/glances.conf`     comment / uncomment various sections as required

`sudo cp /usr/lib/systemd/system/glances.service /etc/systemd/system/glances.service`

`sudo nano /etc/systemd/system/glances.service`

    [Unit]
    Description=Glances
    After=network.target

    [Service]
    ExecStart=/usr/bin/glances -w
    Restart=on-abort

    [Install]
    WantedBy=multi-user.target

`sudo systemctl enable glances.service`

`sudo systemctl start glances.service`

`sudo systemctl status glances.service`

`sudo reboot`     to restart the machine and all systemd services, once machine is up again check `192.168.X.X:61208` if glances is running correctly

`glances`     shows a variety of system/machine data which can be	configured by changing `/etc/glances/glances.conf` and pressing ‘q’ returns to console

#### 3.)  Install GPS Software

`sudo pacman -S gpsd`

`sudo nano /lib/systemd/system/gpsd.socket`     change `ListenStream=127.0.0.1:2947` to `ListenStream=0.0.0.0:2947`

`sudo nano /etc/default/gpsd`

    START_DAEMON=”true”
    USBAUTO=”true”
    DEVICES=”/dev/ttyXYZ”
    GPSD_OPTIONS=”-n”
    GPSD_SOCKET=”/var/run/gpsd.sock”

`sudo systemctl enable gpsd`

`sudo systemctl start gpsd`

`sudo systemctl status gpsd`

`sudo reboot`

Use `cgps` or `gpsmon` to check GPS data and position.

#### 4.)  Install FlightAware Software

`sudo pacman -S rtl-sdr lighttpd bladerf git tcl tk autoconf net-tools fakeroot pkgconf which wget`

    git clone https://aur.archlinux.org/tclx.git
    cd tclx
    nano PKGBUILD						set arch to ‘any’
    makepkg -si
    cd ..

    git clone https://aur.archlinux.org/tcllib.git
    cd tcllib
    makepkg -si
    cd ..

    git clone https://aur.archlinux.org/tcllauncher.git
    cd tcllauncher
    makepkg -si
    cd ..

    git clone https://aur.archlinux.org/tcltls.git
    cd tcltls
    makepkg -si
    cd ..

    git clone https://aur.archlinux.org/mlat-client-git.git
    cd mlat-client-git
    makepkg -si
    cd ..

    git clone https://aur.archlinux.org/dump1090-fa-git.git
    cd dump1090-fa-git
    nano PKGBUILD						set arch to ‘any’
    makepkg -si
    cd ..

`sudo systemctl enable dump1090`

`sudo systemctl start dump1090`

`sudo systemctl status dump1090`

    git clone https://aur.archlinux.org/piaware-git.git
	  cd piaware-git
  	nano PKGBUILD			                        set arch to ‘any’
	  makepkg -si
	  cd ..

`sudo nano /etc/piaware.conf`

    FlightAwareUser				            remove line
	  FlightAwarePassword			            remove line
	  piaware-config allow-auto-updates no
	  piaware-config allow-manual-updates no	    change to yes
	  piaware-config feeder-id XXXXX              	    replace X with Unique Identifier found on FlightAware web site.

`sudo systemctl enable piaware`

`sudo systemctl start piaware`

`sudo systemctl status piaware`

`sudo cp /usr/share/dump1090/lighttpd.conf /etc/lighttpd/lighttpd.conf`

`sudo nano /etc/lighttpd/lighttpd.conf`         change `server.port` to `8080` and change `index-file.names` to `( “gmap.html”, “index.html” )`

`lighttpd -t -f /etc/lighttpd/lighttpd.conf`    check if syntax is OK

`sudo systemctl enable lighttpd`

`sudo systemctl start lighttpd`

`sudo systemctl status lighttpd`

`sudo reboot`

#### 5.)  Enable console auto log-in and start glances

`sudo nano /etc/systemd/logind.conf`            uncomment `#NAutoVTs=6` and set to `NAutoVTs=2`

`sudo mkdir /etc/systemd/system/getty@tty1.service.d`

`sudo nano /etc/systemd/system/getty@tty1.service.d/override.conf`

    [Service]
	  ExecStart=
	  ExecStart=-/usr/bin/agetty --autologin user --noclear %I $TERM
	  Type=simple

`sudo systemctl enable getty@tty1.service`

`nano .bashrc`                                  add `glances` to the END of the file

`sudo reboot`

Done, enjoy your new ADSB receiver !
