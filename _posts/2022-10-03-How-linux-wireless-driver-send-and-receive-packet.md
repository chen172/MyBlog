---

layout: post
title: "How Linux Wireless Driver Sends and Receives Packets"

---

<font color="red">**Note**: This blog focuses on the `cfg80211` kernel module.</font>

---

### 1. Overview of the Linux Wireless Subsystem

![linux\_wireless\_subsystem](/MyBlog/assets/picture/linux_wireless_subsystem.png){\:class="img-responsive"}

The Linux wireless subsystem is responsible for managing wireless network devices, such as Wi-Fi adapters, and facilitates communication between the network stack and the hardware. The `cfg80211` module plays a key role in this process by providing a common interface for managing wireless devices.

### 2. Driver Receives Packets from the Kernel Network Stack and Sends Them Through the Network Device Interface

When the kernel's network stack processes packets, the wireless driver is responsible for transmitting them through the network device interface. The driver uses the callback function `ndo_start_xmit` from the `struct net_device_ops` to send packets. This function must be implemented by the driver.

Here’s an example of how the driver implements the `ndo_start_xmit` callback:

```c
static struct net_device_ops nvf_ndev_ops = {
    .ndo_start_xmit = nvf_ndo_start_xmit,
};

static netdev_tx_t nvf_ndo_start_xmit(struct sk_buff *skb, struct net_device *dev) {
    printk("Received packet from kernel, need to transmit\n");
    
    // Clean up the socket buffer (skb)
    kfree_skb(skb);
    return NETDEV_TX_OK;
}
```

* **Explanation of the `net_device_ops` structure**:

  * The `net_device_ops` structure, defined in `<linux/netdevice.h>`, contains a set of function pointers that allow the kernel to interact with the network device.

  For more details, you can refer to the [definition of `net_device_ops`](https://elixir.bootlin.com/linux/latest/source/include/linux/netdevice.h#L1381).

```c
struct net_device_ops {
    int (*ndo_init)(struct net_device *dev);
    void (*ndo_uninit)(struct net_device *dev);
    int (*ndo_open)(struct net_device *dev);
    int (*ndo_stop)(struct net_device *dev);
    netdev_tx_t (*ndo_start_xmit)(struct sk_buff *skb, struct net_device *dev);
    // More methods omitted...
};
```

### 3. Driver Receives Packets from Network Device Interface and Sends Them to Kernel Network Stack

When the network device receives a packet, it follows a series of steps to pass the packet to the kernel’s network stack:

1. The network device stores the packet in a buffer located in its memory.
2. The device raises an interrupt to notify the kernel of a new packet.
3. The interrupt handler allocates and initializes a new socket buffer (`skb`) to hold the packet.
4. The interrupt handler copies the packet from the device’s memory into the socket buffer.
5. The interrupt handler invokes the `netif_rx()` function to notify the networking subsystem that a new packet has arrived and should be processed.

Here's an example of how the `netif_rx()` function is used in the driver to pass a packet to the kernel:

```c
static void virwifi_monitor_rx(struct sk_buff *skb, struct net_device *dev) {
    struct sk_buff *newskb;

    // Copy skb to a new buffer
    newskb = skb_copy_expand(skb, 0, 0, GFP_ATOMIC);
    if (newskb == NULL) {
        printk("Failed to allocate skb\n");
        return;
    }

    newskb->dev = dev;
    skb_reset_mac_header(newskb);
    newskb->ip_summed = CHECKSUM_UNNECESSARY;
    newskb->pkt_type = PACKET_OTHERHOST;
    newskb->protocol = htons(ETH_P_802_2);
    memset(newskb->cb, 0, sizeof(newskb->cb));

    // Pass the packet to the kernel's network stack
    netif_rx(newskb);
}
```

* **Explanation of `netif_rx()`**:

  * The `netif_rx()` function is defined in `/net/core/dev.c` and is used to pass packets from the network driver to the upper layers of the network stack.
  * It handles packets from both interrupt and process contexts and queues them for processing by the appropriate protocol handlers (such as IP or ARP).

For more details, you can check out the [definition of `netif_rx()`](https://elixir.bootlin.com/linux/latest/source/net/core/dev.c#L4999):

```c
/**
 * netif_rx - Post buffer to the network code
 * @skb: buffer to post
 *
 * This function receives a packet from a device driver and queues it for
 * further processing. It is used in both interrupt and process context.
 *
 * Return values:
 * NET_RX_SUCCESS (no congestion)
 * NET_RX_DROP (packet dropped)
 */
int netif_rx(struct sk_buff *skb)
```

### 4. Key Concepts and Data Structures

* **`struct sk_buff` (Socket Buffer)**:
  The `sk_buff` structure is used to represent a network packet. It contains metadata about the packet, such as its headers, payload, and various flags related to its processing.

* **`net_device_ops`**:
  This structure defines the callbacks used by the kernel to interact with network devices. It includes functions for transmitting data (`ndo_start_xmit`), managing device state (`ndo_open`, `ndo_stop`), and retrieving statistics (`ndo_get_stats`), among others.

* **Interrupt Handling**:
  When a packet is received by the network device, an interrupt is generated, and the corresponding interrupt handler is executed. The handler processes the packet and passes it to the kernel’s network stack using `netif_rx()`.

### 5. Conclusion

This blog post described the key steps in how the Linux wireless driver sends and receives network packets. We covered the role of `net_device_ops`, the `ndo_start_xmit` function for sending packets, and the process of receiving packets and passing them to the kernel using `netif_rx()`. Understanding these components is crucial for anyone working with the Linux network stack and wireless drivers.

---

### References

1. [Understanding the Linux Kernel Networking Subsystem](https://www.oreilly.com/library/view/understanding-the-linux/0596002130/ch18s04.html)
2. [Linux Wireless Overview](https://www.cnblogs.com/rain-blog/p/linux-wireless.html)
