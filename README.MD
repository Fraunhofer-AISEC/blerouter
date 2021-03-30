# IPv6 BLE router for Raspberry Pi

This script configures a Raspberry PI to act as IPv6 over BLE border router 



## Network Set up

```bash
            	  BLE                                        ETH                  		wifi
nrf5x   	--------------- RPI Border router        --------------- WIFI router --------------- PC
2001:db8::1        		2001:db8::2  2001:db9::1     					 2001:db9::2
```

## BLE device set up
 * Set up similar to the [IPSP sample provided by Zephyr OS](https://docs.zephyrproject.org/latest/samples/bluetooth/ipsp/README.html)

## Rpi Set up
 * Install RADVD if not installed (to be done only once) sudo apt-get install radvd
 * Configure RADVD -> open /etc/radvd.conf and paste#

```bash
interface bt0
{
    AdvSendAdvert on;
    prefix 2001:db8::/64
    {
        AdvOnLink off;
        AdvAutonomous on;
        AdvRouterAddr on;
    };
};
```

## PC Set up

Assign static IPv6 address to the PC (use the wifi interface - not necessarily **wlp4s0** on every device) 

```bash
sudo ifconfig wlp4s0 add 2001:db9::2/64
ifconfig
```


ping RPI from PC AFTER connecting the BLE device and the router

```bash
ping6 -I wlp4s0 2001:db9::1
```


Add route to the BLE device 

```bash
sudo /sbin/ip -6 route add 2001:db8::/64 via 2001:db9::1
```

Disable the firewall of the PC. On Ubuntu this can be done with Gufw Firewall GUI.

## Connect the BLE device and the router

Execute the blerouter script as sudo. 

```bash
sudo ./blerouter.sh -d dev-name
```
Here dev-name is the name of the device to which we want to connect. 

## Testing 

Ping the BLE device

```bash
ping6 -I wlp4s0 2001:db8::1
```

Send a udp packet to port 4242

```bash
echo "some data" > /dev/udp/2001:db8::1/4242
```

