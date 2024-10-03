# Ansible Role: Multipath

This role will install and configure multipath(d).

The 'multipath' tool is used to detect and coalesce multiple paths to devices, for fail-over or performance reasons.

The 'multipathd' daemon is in charge of checking for failed paths and reconfiguring the affected multipath map 
when a failure occurs. For this it executes the multipath tool, which in turn will signal multipathd when it is done.   

## Requirements

This role has to be executed as user 'root'.

## Variables

| Variable                      | Type   | Required | Default | Comment                                                                       |
|-------------------------------|--------|----------|---------|-------------------------------------------------------------------------------|
| multipath_user_friendly_names | bool   | No       | false   | Use the multipath bindings file to assign alias to multipath instead of WWID  |
| multipath_blacklist_wwids     | list() | No       | N/A     | Devices to exclude from multipath topology discovery                          |
| multipath_devices             | list() | No       | N/A     | Device-specifc settings                                                       |

### Multipath devices

```yaml
# INFO: repeat_count syslog issue: https://access.redhat.com/solutions/3484411
# INFO: Test tur with: sg_turs -vvvt -n 4 /dev/sdX
base_multipath_devices:
  - |
    vendor "SYNOLOGY"
    product "Storage"
    path_grouping_policy "multibus"
    path_selector "round-robin 0"
    path_checker "tur"
    prio "alua"
    failback immediate
    no_path_retry "fail"
    flush_on_last_del "yes"
```

### Multipath blacklist

Following command can be used to find the WWID of all devices:

```shell
$ multipathd show paths raw format "%d %w"
sda ST1000DM010-2EP102_ZN1YZ1Y8
sdb 360014052bd202b2da441d4299da1ead7
sdc 360014052bd202b2da441d4299da1ead7
```

All non multipath devices can be added to the blacklist:

```yaml
multipath_blacklist_wwid:
  - ST1000DM010-2EP102_ZN1YZ1Y8
```

**Note:** Once a device is blacklisted it will not show up in the multipath command anymore.

## Management

### Multipathd

The multipathd command line tool can be used to check the multipath path, devices and blacklist status.

```shell
$ multipathd show paths
hcil    dev dev_t pri dm_st  chk_st dev_st  next_check      
8:0:0:1 sdb 8:16  50  active ready  running XXXXXXXXX. 19/20
9:0:0:1 sdc 8:32  50  active ready  running XXXXXXXXX. 19/20
```

```shell
$ multipathd show devices
available block devices:
    sda devnode whitelisted, unmonitored
    sda1 devnode whitelisted, unmonitored
    sda2 devnode whitelisted, unmonitored
    sda3 devnode whitelisted, unmonitored
    sda4 devnode whitelisted, unmonitored
    sdb  devnode whitelisted, monitored
    sdc  devnode whitelisted, monitored
    dm-0 devnode blacklisted, unmonitored
    dm-1 devnode blacklisted, unmonitored
    dm-2 devnode blacklisted, unmonitored
    dm-3 devnode blacklisted, unmonitored
    dm-4 devnode blacklisted, unmonitored
    dm-5 devnode blacklisted, unmonitored
    dm-6 devnode blacklisted, unmonitored
```

```shell
$ multipathd show blacklist
device node rules:
- blacklist:
        (default rule)     !^(sd[a-z]|dasd[a-z]|nvme[0-9])
- exceptions:
        <empty>
udev property rules:
- blacklist:
        <empty>
- exceptions:
        <empty>
protocol rules:
- blacklist:
        <empty>
- exceptions:
        <empty>
wwid rules:
- blacklist:
        (config file rule) ST1000DM010-2EP102_ZN1YZ1Y8
- exceptions:
        <empty>
device rules:
- blacklist:
        (default rule)     SGI:Universal Xport
        (default rule)     ^DGC:LUNZ
        (default rule)     EMC:LUNZ
        (default rule)     DELL:Universal Xport
        (default rule)     FUJITSU:Universal Xport
        (default rule)     IBM:Universal Xport
        (default rule)     IBM:S/390
        (default rule)     LENOVO:Universal Xport
        (default rule)     (NETAPP|LSI|ENGENIO):Universal Xport
        (default rule)     STK:Universal Xport
        (default rule)     SUN:Universal Xport
        (default rule)     (Intel|INTEL):VTrak V-LUN
        (default rule)     Promise:VTrak V-LUN
        (default rule)     Promise:Vess V-LUN
- exceptions:
        <empty>
```

### Multipath

The multipath command line tool provides an overview of the current multipath device maps and configuration.

```shell
$ multipath -l
360014052bd202b2da441d4299da1ead7 dm-6 SYNOLOGY,Storage
size=1.0T features='0' hwhandler='1 alua' wp=rw
`-+- policy='round-robin 0' prio=0 status=active
  |- 8:0:0:1 sdb 8:16 active undef running
  `- 9:0:0:1 sdc 8:32 active undef running
```

```shell
$ multipath -t
defaults {
...
}
blacklist {
...
}
blacklist_exceptions {
...
}
devices {
...
}
overrides {
...
}
```
