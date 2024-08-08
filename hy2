#!/bin/bash
export UUID=${UUID:-'39e8b439-06be-4783-ad52-6357fc5e8743'}         
export NEZHA_SERVER=${NEZHA_SERVER:-'nz.f4i.cn'}             
export NEZHA_PORT=${NEZHA_PORT:-'5555'}            
export NEZHA_KEY=${NEZHA_KEY:-''}
export PASSWORD=${PASSWORD:-'admin888'} 
export PORT=${PORT:-'6789'}  
USERNAME=$(whoami)
HOSTNAME=$(hostname)

[[ "$HOSTNAME" == "s1.ct8.pl" ]] && WORKDIR="domains/${USERNAME}.ct8.pl/logs" || WORKDIR="domains/${USERNAME}.serv00.net/logs"
[ -d "$WORKDIR" ] || (mkdir -p "$WORKDIR" && chmod 777 "$WORKDIR" && cd "$WORKDIR")

# Download Dependency Files
ARCH=$(uname -m) && DOWNLOAD_DIR="." && mkdir -p "$DOWNLOAD_DIR" && FILE_INFO=()
if [ "$ARCH" == "arm" ] || [ "$ARCH" == "arm64" ] || [ "$ARCH" == "aarch64" ]; then
    FILE_INFO=("https://github.com/etjec4/tuic/releases/download/tuic-server-1.0.0/tuic-server-1.0.0-x86_64-unknown-freebsd.sha256sum web" "https://github.com/eooce/test/releases/download/ARM/swith npm")
elif [ "$ARCH" == "amd64" ] || [ "$ARCH" == "x86_64" ] || [ "$ARCH" == "x86" ]; then
    FILE_INFO=("https://github.com/etjec4/tuic/releases/download/tuic-server-1.0.0/tuic-server-1.0.0-x86_64-unknown-freebsd web" "https://eooce.2go.us.kg/npm npm")
else
    echo "Unsupported architecture: $ARCH"
    exit 1
fi
for entry in "${FILE_INFO[@]}"; do
    URL=$(echo "$entry" | cut -d ' ' -f 1)
    NEW_FILENAME=$(echo "$entry" | cut -d ' ' -f 2)
    FILENAME="$DOWNLOAD_DIR/$NEW_FILENAME"
    if [ -e "$FILENAME" ]; then
        echo -e "\e[1;32m$FILENAME already exists,Skipping download\e[0m"
    else
        curl -L -sS -o "$FILENAME" "$URL"
        echo -e "\e[1;32mDownloading $FILENAME\e[0m"
    fi
    chmod +x $FILENAME
done
wait

# Generate cert
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout $WORKDIR/server.key -out $WORKDIR/server.crt -subj "/CN=bing.com" -days 36500

# Generate configuration file
cat > config.json <<EOL
{
  "server": "[::]:$PORT",
  "users": {
    "$UUID": "$PASSWORD"
  },
  "certificate": "$WORKDIR/server.crt",
  "private_key": "$WORKDIR/server.key",
  "congestion_control": "bbr",
  "alpn": ["h3", "spdy/3.1"],
  "udp_relay_ipv6": true,
  "zero_rtt_handshake": false,
  "dual_stack": true,
  "auth_timeout": "3s",
  "task_negotiation_timeout": "3s",
  "max_idle_time": "10s",
  "max_external_packet_size": 1500,
  "gc_interval": "3s",
  "gc_lifetime": "15s",
  "log_level": "warn"
}
EOL

# running files
run() {
  if [ -e npm ]; then
    tlsPorts=("443" "8443" "2096" "2087" "2083" "2053")
    if [[ "${tlsPorts[*]}" =~ "${NEZHA_PORT}" ]]; then
      NEZHA_TLS="--tls"
    else
      NEZHA_TLS=""
    fi
    if [ -n "$NEZHA_SERVER" ] && [ -n "$NEZHA_PORT" ] && [ -n "$NEZHA_KEY" ]; then
      export TMPDIR=$(pwd)
      nohup ./npm -s ${NEZHA_SERVER}:${NEZHA_PORT} -p ${NEZHA_KEY} ${NEZHA_TLS} >/dev/null 2>&1 &
	    sleep 1
      echo -e "\e[1;32mnpm is running\e[0m"
    else
        echo -e "\e[1;35mNEZHA variable is empty,skiping runing\e[0m"
    fi
  fi

  if [ -e web ]; then
    nohup ./web -c config.json >/dev/null 2>&1 &
    sleep 1
    echo -e "\e[1;32mweb is running\e[0m"
  fi

}
run

# get ip
ipv4=$(curl -s ipv4.ip.sb)
if [ -n "$ipv4" ]; then
    HOST_IP="$ipv4"
else
    ipv6=$(curl -s --max-time 1 ipv6.ip.sb)
    if [ -n "$ipv6" ]; then
        HOST_IP="$ipv6"
    else
        echo -e "\e[1;35m鏃犳硶鑾峰彇IPv4鎴朓Pv6鍦板潃\033[0m"
        exit 1
    fi
fi
echo -e "\e[1;32m鏈満IP: $HOST_IP\033[0m"

# get ipinfo
ISP=$(curl -s https://speed.cloudflare.com/meta | awk -F\" '{print $26"-"$18}' | sed -e 's/ /_/g')

# get hy2 node
echo -e "\e[1;32mTuic瀹夎鎴愬姛\033[0m"
echo ""
echo -e "\e[1;33mV2rayN 鎴� Nekobox锛岃烦杩囪瘉涔﹂獙璇侀渶璁剧疆涓簍rue\033[0m\n"
echo -e "\e[1;32mtuic://$UUID:$PASSWORD@$HOST_IP:$PORT?congestion_control=bbr&alpn=h3&sni=www.bing.com&udp_relay_mode=native&allow_insecure=1#$ISP\e[0m"
echo ""

# delete files
rm -rf npm web config.json

exit 0

帮我改成Hysteria2 
