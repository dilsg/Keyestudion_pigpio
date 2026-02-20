I made this on my Raspberry 4B like
https://github.com/joan2397/pigpio/

Installiere Zeugs
sudo apt install -y python3-setuptools python3-full

sudo apt-get --yes --allow-change-held-packages install --no-install-recommends python3-setuptools python3-full
wget https://github.com/joan2937/pigpio/archive/refs/tags/v79.tar.gz
tar zxf v79.tar.gz
cd pigpio-79
make
sudo make install
sudo ldconfig
sudo systemctl daemon-reload
sudo systemctl enable --now pigpiod

Hat bei mir nicht geklappt - kein Service gestartet

Also 

Code
<code>
DOLLAR="\$"
	cat << EOF > config_pigpiod_service.sh
#! /bin/bash
PIGPIOD_SERVICE="/lib/systemd/system/pigpiod.service"
	if [[ ! -f "${DOLLAR}{PIGPIOD_SERVICE}" ]] ; then
	  # The configuration of pigpiod appears to be different on 32 bit and 64 bit systems
  # pigpiod must be run with -t0 on a 64 bit system, and -t1 on a 32 bit system.
  # The 32 bit behaviour contradicts what is documented in the pigpiod web documentation ;-(
  ARCH=${DOLLAR}(uname -m)
	  if [[ "${DOLLAR}{ARCH}" == "aarch64" ]] ; then
    T_COMMAND="-t0"
  elif [[ "${DOLLAR}{ARCH}" == "armv7l" ]] ; then
    T_COMMAND="-t1"
  elif [[ "${DOLLAR}{ARCH}" == "armv6l" ]] ; then
    T_COMMAND="-t1"
  else
    echo "uname -m returns '${DOLLAR}{ARCH}' which is an unexpected value. Time to debug..."
    echo
    echo "I have not seen armv8 on my Pi3, but I believe this might happen if running"
    echo "64 bits. I am not sure whether pigpiod requires -t0 or -t1 in this case."
    echo "You'll need to experiment."
    echo
    echo "I believe armv6 is a possibility on a Pi2 or earlier?"
    echo "I guess use same settings as for armv6l?"
    exit 1
  fi
	  # We are going to overwrite the systemd configuration file for pigpiod
  # because we are going to change the launch options
  cat << END > "${DOLLAR}{PIGPIOD_SERVICE}"
[Unit]
Description=Daemon required to control GPIO pins via pigpio
	[Service]
Type=forking
ExecStart=/usr/local/bin/pigpiod -l -s10 ${DOLLAR}{T_COMMAND} -x0xFFFFFFF
Restart=always
ExecStop=/bin/systemctl kill pigpiod
	[Install]
WantedBy=multi-user.target
END
else
  echo "${DOLLAR}{PIGPIOD_SERVICE} already exists. I refuse to change it."
fi
EOF
	chmod 755 config_pigpiod_service.sh
</code>	
	Aus <https://github.com/joan2937/pigpio/issues/632> 
	
<img width="730" height="999" alt="image" src="https://github.com/user-attachments/assets/f36ab0ea-f36c-405c-a5c4-36cdf398f112" />

