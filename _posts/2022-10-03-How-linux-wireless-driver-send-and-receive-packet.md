---
layout: post
title: "How linux wireless driver send and receive packet"
---

<font color="red">Note: In this blog, we just use cfg80211 kernel module</font>
### 1. Driver receive packet from kernel network stack and send it through network device interface
Driver use callback function ```netdev_tx_t		(*ndo_start_xmit)(struct sk_buff *skb, struct net_device *dev);``` from ```struct net_device_ops``` to receive packet from 
the network stack. The callback function need implement by the driver. for example:

```c
static struct net_device_ops nvf_ndev_ops = {
        .ndo_start_xmit = nvf_ndo_start_xmit,
};

static netdev_tx_t nvf_ndo_start_xmit(struct sk_buff *skb,
                               struct net_device *dev) {
    printk("receive packet from kernel, need to be transmit\n");
    
    // cleanup skb
    kfree_skb(skb);
    return NETDEV_TX_OK;
}
```

#### About the ```struct net_device_ops``` info (You can get defintion from [here](https://elixir.bootlin.com/linux/latest/source/include/linux/netdevice.h#L1381))
The ```net_device_ops``` structure for a network device, defined in ```<linux/netdevice.h>```, contains a set of method pointers that specify how the system interacts with the device.

```c
struct net_device_ops {
        int                     (*ndo_init)(struct net_device *dev);
        void                    (*ndo_uninit)(struct net_device *dev);
        int                     (*ndo_open)(struct net_device *dev);
        int                     (*ndo_stop)(struct net_device *dev);
        netdev_tx_t             (*ndo_start_xmit) (struct sk_buff *skb,
                                                   struct net_device *dev);
        u16                     (*ndo_select_queue)(struct net_device *dev,
                                                    struct sk_buff *skb);
        void                    (*ndo_change_rx_flags)(struct net_device *dev,
                                                       int flags);
        void                    (*ndo_set_rx_mode)(struct net_device *dev);
        void                    (*ndo_set_multicast_list)(struct net_device *dev);
        int                     (*ndo_set_mac_address)(struct net_device *dev,
                                                       void *addr);
        int                     (*ndo_validate_addr)(struct net_device *dev);
        int                     (*ndo_do_ioctl)(struct net_device *dev,
                                                struct ifreq *ifr, int cmd);
        int                     (*ndo_set_config)(struct net_device *dev,
                                                  struct ifmap *map);
        int                     (*ndo_change_mtu)(struct net_device *dev,
                                                  int new_mtu);
        int                     (*ndo_neigh_setup)(struct net_device *dev,
                                                   struct neigh_parms *);
        void                    (*ndo_tx_timeout) (struct net_device *dev);

        struct rtnl_link_stats64* (*ndo_get_stats64)(struct net_device *dev,
                                                     struct rtnl_link_stats64 *storage);
        struct net_device_stats* (*ndo_get_stats)(struct net_device *dev);

        void                    (*ndo_vlan_rx_register)(struct net_device *dev,
                                                        struct vlan_group *grp);
        void                    (*ndo_vlan_rx_add_vid)(struct net_device *dev,
                                                       unsigned short vid);
        void                    (*ndo_vlan_rx_kill_vid)(struct net_device *dev,
                                                        unsigned short vid);
        /* Several lines omitted */
};
```

### 2. Driver receive packet from network device interface and send it to kernel network stack
1. The network device saves the packet in a buffer in the device’s memory.
2. The network device raises an interrupt.
3. The interrupt handler allocates and initializes a new socket buffer for the packet. 
4. The interrupt handler copies the packet from the device’s memory to the socket buffer.
5. The interrupt handler invokes the ```netif_rx()``` function to notify the Linux networking code that a new packet is arrived and should be processed.

Here is a example of ```netif_rx()``` usage:
```c
// copy skb to newskb, then send it to network stack
static void virwifi_monitor_rx(struct sk_buff *skb,
				      struct net_device *dev) {
	struct sk_buff *newskb;
  
	newskb = skb_copy_expand(skb, 0, 0, GFP_ATOMIC);
	if (newskb == NULL) {
		printk("allocate skb failed\n");
		return;
	}
  
	newskb->dev = dev;
	skb_reset_mac_header(newskb);
	newskb->ip_summed = CHECKSUM_UNNECESSARY;
	newskb->pkt_type = PACKET_OTHERHOST;
	newskb->protocol = htons(ETH_P_802_2);
	memset(newskb->cb, 0, sizeof(newskb->cb));

	netif_rx(newskb);
}
```

#### About the ```netif_rx()``` function info (You can get defintion from [here](https://elixir.bootlin.com/linux/latest/source/net/core/dev.c#L4999))
It defined in ```/net/core/dev.c```
```c
/**
 *	netif_rx	-	post buffer to the network code
 *	@skb: buffer to post
 *
 *	This function receives a packet from a device driver and queues it for
 *	the upper (protocol) levels to process via the backlog NAPI device. It
 *	always succeeds. The buffer may be dropped during processing for
 *	congestion control or by the protocol layers.
 *	The network buffer is passed via the backlog NAPI device. Modern NIC
 *	driver should use NAPI and GRO.
 *	This function can used from interrupt and from process context. The
 *	caller from process context must not disable interrupts before invoking
 *	this function.
 *
 *	return values:
 *	NET_RX_SUCCESS	(no congestion)
 *	NET_RX_DROP     (packet was dropped)
 *
 */
int netif_rx(struct sk_buff *skb)
 ```
 
##### reference
1. https://www.oreilly.com/library/view/understanding-the-linux/0596002130/ch18s04.html
2. https://www.cnblogs.com/rain-blog/p/linux-wireless.html
