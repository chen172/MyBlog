---
layout: post
title: "How linux wireless driver send and receive packet"
---

``Note: In this blog, we just use cfg80211 kernel module``
### Driver receive packet from kernel network stack and send it out
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

#### About the ```struct net_device_ops``` info
The ```net_device_ops``` structure for a network device, defined in <linux/netdevice.h>, contains a set of method pointers that specify how the system interacts with the device.

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
