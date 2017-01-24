---
layout: post
title : 修改vmware-tools以适应高版本Linux内核
category : tool
---
{% include JB/setup %}

VMware Workstation8中的vmware-tools在高版本Linux内核环境下，有些模块会编译出错，比如我关心的shared folder就会出错。导致的结果就是在虚拟机中可以看到/mnt/hgfs文件夹,但是无法查看shared folder里设置的宿主机共享文件夹。

编译出错的原因是高版本的Linux内核中的一些数据结构定义和函数生命与2.6.\*之前的版本有所变化。

例如老版本ethtool.h中结构体ethtool_ops声明如下：

```cpp
struct ethtool_ops {
        int     (*get_settings)(struct net_device *, struct ethtool_cmd *);
        int     (*set_settings)(struct net_device *, struct ethtool_cmd *);
        void    (*get_drvinfo)(struct net_device *, struct ethtool_drvinfo *);
        int     (*get_regs_len)(struct net_device *);
        void    (*get_regs)(struct net_device *, struct ethtool_regs *, void *);
        void    (*get_wol)(struct net_device *, struct ethtool_wolinfo *);
        int     (*set_wol)(struct net_device *, struct ethtool_wolinfo *);
        u32     (*get_msglevel)(struct net_device *);
        void    (*set_msglevel)(struct net_device *, u32);
        int     (*nway_reset)(struct net_device *);
        u32     (*get_link)(struct net_device *);
        int     (*get_eeprom_len)(struct net_device *);
        int     (*get_eeprom)(struct net_device *,
                              struct ethtool_eeprom *, u8 *);
        int     (*set_eeprom)(struct net_device *,
                              struct ethtool_eeprom *, u8 *);
        int     (*get_coalesce)(struct net_device *, struct ethtool_coalesce *);
        int     (*set_coalesce)(struct net_device *, struct ethtool_coalesce *);
        void    (*get_ringparam)(struct net_device *,
                                 struct ethtool_ringparam *);
        int     (*set_ringparam)(struct net_device *,
                                 struct ethtool_ringparam *);
        void    (*get_pauseparam)(struct net_device *,
                                  struct ethtool_pauseparam*);
        int     (*set_pauseparam)(struct net_device *,
                                  struct ethtool_pauseparam*);
        u32     (*get_rx_csum)(struct net_device *);
        int     (*set_rx_csum)(struct net_device *, u32);
        u32     (*get_tx_csum)(struct net_device *);
        int     (*set_tx_csum)(struct net_device *, u32);
        u32     (*get_sg)(struct net_device *);
        int     (*set_sg)(struct net_device *, u32);
        u32     (*get_tso)(struct net_device *);
        int     (*set_tso)(struct net_device *, u32);
        void    (*self_test)(struct net_device *, struct ethtool_test *, u64 *);
        void    (*get_strings)(struct net_device *, u32 stringset, u8 *);
        int     (*set_phys_id)(struct net_device *, enum ethtool_phys_id_state);
        void    (*get_ethtool_stats)(struct net_device *,
                                     struct ethtool_stats *, u64 *);
        int     (*begin)(struct net_device *);
        void    (*complete)(struct net_device *);
        u32     (*get_ufo)(struct net_device *);
        int     (*set_ufo)(struct net_device *, u32);
        u32     (*get_flags)(struct net_device *);
        int     (*set_flags)(struct net_device *, u32);
        u32     (*get_priv_flags)(struct net_device *);
        int     (*set_priv_flags)(struct net_device *, u32);
        int     (*get_sset_count)(struct net_device *, int);
        int     (*get_rxnfc)(struct net_device *,
                             struct ethtool_rxnfc *, u32 *rule_locs);
        int     (*set_rxnfc)(struct net_device *, struct ethtool_rxnfc *);
        int     (*flash_device)(struct net_device *, struct ethtool_flash *);
        int     (*reset)(struct net_device *, u32 *);
        int     (*set_rx_ntuple)(struct net_device *,
                                 struct ethtool_rx_ntuple *);
        int     (*get_rxfh_indir)(struct net_device *,
                                  struct ethtool_rxfh_indir *);
        int     (*set_rxfh_indir)(struct net_device *,
                                  const struct ethtool_rxfh_indir *);
        void    (*get_channels)(struct net_device *, struct ethtool_channels *);
        int     (*set_channels)(struct net_device *, struct ethtool_channels *);
        int     (*get_dump_flag)(struct net_device *, struct ethtool_dump *);
        int     (*get_dump_data)(struct net_device *,
                                 struct ethtool_dump *, void *);
        int     (*set_dump)(struct net_device *, struct ethtool_dump *);

};
```

新版本声明如下：

```cpp
struct ethtool_ops {
        int     (*get_settings)(struct net_device *, struct ethtool_cmd *);
        int     (*set_settings)(struct net_device *, struct ethtool_cmd *);
        void    (*get_drvinfo)(struct net_device *, struct ethtool_drvinfo *);
        int     (*get_regs_len)(struct net_device *);
        void    (*get_regs)(struct net_device *, struct ethtool_regs *, void *);
        void    (*get_wol)(struct net_device *, struct ethtool_wolinfo *);
        int     (*set_wol)(struct net_device *, struct ethtool_wolinfo *);
        u32     (*get_msglevel)(struct net_device *);
        void    (*set_msglevel)(struct net_device *, u32);
        int     (*nway_reset)(struct net_device *);
        u32     (*get_link)(struct net_device *);
        int     (*get_eeprom_len)(struct net_device *);
        int     (*get_eeprom)(struct net_device *,
                      struct ethtool_eeprom *, u8 *);
        int     (*set_eeprom)(struct net_device *,
                              struct ethtool_eeprom *, u8 *);
        int     (*get_coalesce)(struct net_device *, struct ethtool_coalesce *);
        int     (*set_coalesce)(struct net_device *, struct ethtool_coalesce *);
        void    (*get_ringparam)(struct net_device *,
                                 struct ethtool_ringparam *);
        int     (*set_ringparam)(struct net_device *,
                                 struct ethtool_ringparam *);
        void    (*get_pauseparam)(struct net_device *,
                                  struct ethtool_pauseparam*);
        int     (*set_pauseparam)(struct net_device *,
                                  struct ethtool_pauseparam*);
        void    (*self_test)(struct net_device *, struct ethtool_test *, u64 *);
        void    (*get_strings)(struct net_device *, u32 stringset, u8 *);
        int     (*set_phys_id)(struct net_device *, enum ethtool_phys_id_state);
        void    (*get_ethtool_stats)(struct net_device *,
                                     struct ethtool_stats *, u64 *);
        int     (*begin)(struct net_device *);
        void    (*complete)(struct net_device *);
        u32     (*get_priv_flags)(struct net_device *);
                       struct ethtool_eeprom *, u8 *);
        int     (*set_eeprom)(struct net_device *,
                              struct ethtool_eeprom *, u8 *);
        int     (*get_coalesce)(struct net_device *, struct ethtool_coalesce *);
        int     (*set_coalesce)(struct net_device *, struct ethtool_coalesce *);
        void    (*get_ringparam)(struct net_device *,
                                 struct ethtool_ringparam *);
        int     (*set_ringparam)(struct net_device *,
                                 struct ethtool_ringparam *);
        void    (*get_pauseparam)(struct net_device *,
                                  struct ethtool_pauseparam*);
        int     (*set_pauseparam)(struct net_device *,
                                  struct ethtool_pauseparam*);
        void    (*self_test)(struct net_device *, struct ethtool_test *, u64 *);
        void    (*get_strings)(struct net_device *, u32 stringset, u8 *);
        int     (*set_phys_id)(struct net_device *, enum ethtool_phys_id_state);
        void    (*get_ethtool_stats)(struct net_device *,
                                     struct ethtool_stats *, u64 *);
        int     (*begin)(struct net_device *);
        void    (*complete)(struct net_device *);
        u32     (*get_priv_flags)(struct net_device *);
       int     (*get_sset_count)(struct net_device *, int);
        int     (*get_rxnfc)(struct net_device *,
                             struct ethtool_rxnfc *, u32 *rule_locs);
        int     (*set_rxnfc)(struct net_device *, struct ethtool_rxnfc *);
        int     (*flash_device)(struct net_device *, struct ethtool_flash *);
        int     (*reset)(struct net_device *, u32 *);
        u32     (*get_rxfh_indir_size)(struct net_device *);
        int     (*get_rxfh_indir)(struct net_device *, u32 *);
        int     (*set_rxfh_indir)(struct net_device *, const u32 *);
        void    (*get_channels)(struct net_device *, struct ethtool_channels *);
        int     (*set_channels)(struct net_device *, struct ethtool_channels *);
        int     (*get_dump_flag)(struct net_device *, struct ethtool_dump *);
        int     (*get_dump_data)(struct net_device *,
                                 struct ethtool_dump *, void *);
        int     (*set_dump)(struct net_device *, struct ethtool_dump *);
};
```

主要是vmhgfs和vmxnet两个模块的错误，将编译出错的字段删除即可。

你可以[在这里](http://vdisk.weibo.com/s/u6erG)下载已经修改好的压缩包，将压缩包中的文件替换掉原文件即可。其中vmhgfs、vmxnet模块在/usr/lib/vmware-tools/modules/source目录下。

ps:感谢ceofit的分享,但是其提供的补丁包下载页面已经失效,于是我重新做了一个分享。[原文](http://blog.csdn.net/ceofit/article/details/7723348)有更加详细的说明！