Forked Mosquitto 2.0.14 with web socket support for CentOS 8
=================
There is zero support on the internet for Mosquitto > 1.6 in a CentOS based environment. The intention of this repo and README is to change that.

The built-in CentOS package managers, `dnf` and `yum`, distribute Mosquitto 1.6, eventhough the Eclipse foundation is on v2.0.14 as of this writing.

Mosquitto v2 is required in order to get updated Mosquitto functionality such as `mosquitto_ctrl`, which makes dynamic security to an mqtt server more modern and flexible.

Specifically, this repo is forked from Eclipse Mosquitto to help support Mosquitto v2 w/ web socket support on a CentOS 8 environment.

## Steps to reproduce

Fork the repo from Eclipse https://github.com/eclipse/mosquitto

In your CentOS 8 environment, clone the forked repo into `/usr/bin/`

    cd /usr/bin
    sudo git clone https://github.com/{youruserid}/mosquitto

Now install dependencies for the build process

    sudo yum install cmake wget gcc openssl-devel -y
    
`cJSON` is required in order to build Mosquitto from source. I found it to be problematic to install via `yum` in a CentOS environment. Using `wget` to retrieve `cJSON.c` and `cJSON.h` is all we need. I placed them initially in the `/usr/bin/mosquitto/apps/mosquitto_ctrl/` directory.

    cd /usr/bin/mosquitto/apps/mosquitto_ctrl/
    sudo mkdir cjson/
    cd json/
    sudo wget https://raw.githubusercontent.com/DaveGamble/cJSON/master/cJSON.c
    sudo wget https://raw.githubusercontent.com/DaveGamble/cJSON/master/cJSON.h

We will also need to place this `cjson` dir where `cJSON` is referenced throughout the Makefiles.

    sudo cd /usr/bin/mosquitto/
    sudo cp -v -r apps/mosquitto_ctrl/cjson/ apps/mosquitto_passwd/
    sudo cp -v -r apps/mosquitto_ctrl/cjson/ include/
    sudo cp -v -r apps/mosquitto_ctrl/cjson/ lib/
    sudo cp -v -r apps/mosquitto_ctrl/cjson/ client/
    sudo cp -v -r apps/mosquitto_ctrl/cjson/ plugins/dynamic-security

Because we chose to onboard `cjson` with only a .h and .c file, we need to replace the `-lcjson` string in Makefiles to `cjson/cJSON.c`. The `-lcjson` references are in:

    #   config.mk
    #   apps/mosquitto_ctrl/Makefile
    #   plugins/dynamic-security/Makefile

In order to build Mosquitto with web socket support, we need to build `libwebsockets` from source; specifically version `2.4.2` for `LWS_WITH_EXTERNAL_POLL` support.

    cd /usr/bin
    sudo wget https://github.com/warmcat/libwebsockets/archive/refs/tags/v2.4.2.tar.gz
    sudo tar -xvzf v2.4.2.tar.gz
    cd libwebsockets-2.4.2/
    sudo mkdir build
    cd build
    sudo cmake .. -DCMAKE_C_COMPILER=/usr/bin/gcc
    sudo make
    sudo make install

Now we need to modify `config.mk` in order for the build to have web socket support

    #change no
    WITH_WEBSOCKETS:=no
    #to yes
    WITH_WEBSOCKETS:=yes

Now we can finally move forward with the Mosquitto build

    cd /usr/bin/mosquitto/
    sudo make
    sudo make install

After that, we need to link libmosquitto and libwebsockets to `/usr/lib/` and reload the `ldconfig`. This represents our final set of operations enable the `mosquitto` and `mosquitto_ctrl` commands.

    sudo ln -s /usr/local/lib/libmosquitto.so.1 /usr/lib/libmosquitto.so.1
    sudo ln -s /usr/local/lib/libwebsockets.so.19 /usr/lib/libwebsockets.so.19
    sudo ln -s /usr/local/lib/libwebsockets.so.12 /usr/lib/libwebsockets.so.12
    sudo /sbin/ldconfig
    
Verify `mosquitto` and `mosquitto_ctrl` are functional by entering them in your console.

## Credits

For collaboration on this project https://github.com/taclog

For help building libwebsockets from source https://github.com/eclipse/mosquitto/issues/538#issuecomment-326290742

For developing this awesome tool https://github.com/eclipse/mosquitto
