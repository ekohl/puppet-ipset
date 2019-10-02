# Reference
<!-- DO NOT EDIT: This document was generated by Puppet Strings -->

## Table of Contents

**Classes**

* [`ipset`](#ipset): module to install the ipset tooling and to manage individual ipsets

**Defined types**

* [`ipset::set`](#ipsetset): Declare an IP Set.
* [`ipset::unmanaged`](#ipsetunmanaged): Declare an IP set, without managing its content.  Useful when you have a dynamic process that generates an IP set content, but still want to 

**Data types**

* [`IPSet::Options`](#ipsetoptions): list of options you can configure on an ipset
* [`IPSet::Set::Array`](#ipsetsetarray): type to allow an array of ip addresses
* [`IPSet::Set::File_URL`](#ipsetsetfile_url): type to allow a static file on the target system as source for ipsets
* [`IPSet::Set::Puppet_URL`](#ipsetsetpuppet_url): type to allow a file on the puppetserver as source for ip addresses for ipsets
* [`IPSet::Settype`](#ipsetsettype): different datatypes that provides prefixes for the actual ipset
* [`IPSet::Type`](#ipsettype): type to allow all different hash setups for ipsets

## Classes

### ipset

module to install the ipset tooling and to manage individual ipsets

#### Parameters

The following parameters are available in the `ipset` class.

##### `packages`

Data type: `Array[String[1]]`

The name of the package we want to install

##### `service`

Data type: `String[1]`

The name of the service that we're going to manage

##### `service_ensure`

Data type: `Boolean`

Desired state of the service. If true, the service will be running. If false, the service will be stopped

##### `enable`

Data type: `Boolean`

Boolean to decide if we want to have the service in autostart or not

##### `firewall_service`

Data type: `Optional[Pattern[/\.service$/]]`

An optional service name. if provided, the ipsets will be configured before this. So your firewall will depend on the chains. The name should end with `.service`. This is only supported on systemd-based Operating Systems

Default value: `undef`

##### `package_ensure`

Data type: `Enum['present', 'absent', 'latest']`



##### `config_path`

Data type: `Stdlib::Absolutepath`



## Defined types

### ipset::set

Declare an IP Set.

#### Examples

##### An IP set containing individual IP addresses, specified in the code.

```puppet
ipset::set { 'a-few-ip-addresses':
  set => ['10.0.0.1', '10.0.0.2', '10.0.0.42'],
}
```

##### An IP set containing IP networks, specified with Hiera.

```puppet
ipset::set { 'hiera-networks':
  set  => lookup('foo', IP::Address::V4::CIDR),
  type => 'hash:net',
}
```

##### An IP set of IP addresses, based on a file stored in a module.

```puppet
ipset::set { 'from-puppet-module':
  set => "puppet:///modules/${module_name}/ip-addresses",
}
```

##### An IP set of IP networks, based on a file stored on the filesystem.

```puppet
ipset::set { 'from-filesystem':
  set => 'file:///path/to/ip-addresses',
}
```

##### setup multiple ipsets based on a hiera hash with multiple arrays and multiple IPv4/IPv6 prefixes. Use the voxpupuli/ferm module to create suitable iptables rules.

```puppet
$ip_ranges = lookup('ip_net_vlans').flatten.unique
$ip_ranges_ipv4 = $ip_ranges.filter |$ip_range| { $ip_range =~ Stdlib::IP::Address::V4 }
$ip_ranges_ipv6 = $ip_ranges.filter |$ip_range| { $ip_range =~ Stdlib::IP::Address::V6 }

ipset::set{'v4':
  ensure => 'present',
  set    => $ip_ranges_ipv4,
  type   => 'hash:net',
}

ipset::set{'v6':
  ensure  => 'present',
  set     => $ip_ranges_ipv6,
  type    => 'hash:net',
  options => {
    'family' => 'inet6',
  },
}

ferm::ipset{'INPUT':
  ip_version => 'ip6',
  sets       => {
    'v6' => 'ACCEPT',
  },
}

ferm::ipset{'INPUT':
  ip_version => 'ip',
  sets       => {
    'v4' => 'ACCEPT',
  },
}
```

#### Parameters

The following parameters are available in the `ipset::set` defined type.

##### `set`

Data type: `IPSet::Settype`

IP set content or source.

##### `ensure`

Data type: `Enum['present', 'absent']`

Should the IP set be created or removed ?

Default value: 'present'

##### `type`

Data type: `IPSet::Type`

Type of IP set.

Default value: 'hash:ip'

##### `options`

Data type: `IPSet::Options`

IP set options.

Default value: {}

##### `ignore_contents`

Data type: `Boolean`

If ``true``, only the IP set declaration will be
managed, but not its content.

Default value: `false`

##### `keep_in_sync`

Data type: `Boolean`

If ``true``, Puppet will update the IP set in the kernel
memory. If ``false``, it will only update the IP sets on the filesystem.

Default value: `true`

### ipset::unmanaged

Declare an IP set, without managing its content.

Useful when you have a dynamic process that generates an IP set content,
but still want to define and use it from Puppet.

<aside class="warning">
When changing IP set attributes (type, options) contents won't be kept,
set will be recreated as empty.
</aside>

#### Examples

##### 

```puppet
ipset::unmanaged { 'unmanaged-ipset-name': }
```

#### Parameters

The following parameters are available in the `ipset::unmanaged` defined type.

##### `ensure`

Data type: `Enum['present', 'absent']`

Should the IP set be created or removed ?

Default value: 'present'

##### `type`

Data type: `IPSet::Type`

Type of IP set.

Default value: 'hash:ip'

##### `options`

Data type: `IPSet::Options`

IP set options.

Default value: {}

##### `keep_in_sync`

Data type: `Boolean`

If ``true``, Puppet will update the IP set in the kernel
memory. If ``false``, it will only update the IP sets on the filesystem.

Default value: `true`

## Data types

### IPSet::Options

list of options you can configure on an ipset

* **See also**
http://ipset.netfilter.org/ipset.man.html#lbAI

Alias of `Struct[{
    Optional[family]   => Enum['inet', 'inet6'],
    Optional[hashsize] => Integer[128],
    Optional[maxelem]  => Integer[128],
    Optional[netmask]  => IP::Address,
    Optional[timeout]  => Integer[1],
}]`

### IPSet::Set::Array

type to allow an array of ip addresses

Alias of `Array[String]`

### IPSet::Set::File_URL

type to allow a static file on the target system as source for ipsets

Alias of `Pattern[/^file:\/\/\//]`

### IPSet::Set::Puppet_URL

type to allow a file on the puppetserver as source for ip addresses for ipsets

Alias of `Pattern[/^puppet:\/\//]`

### IPSet::Settype

different datatypes that provides prefixes for the actual ipset

Alias of `Variant[IPSet::Set::Array, IPSet::Set::Puppet_URL, IPSet::Set::File_URL, String]`

### IPSet::Type

type to allow all different hash setups for ipsets

* **See also**
http://ipset.netfilter.org/ipset.man.html#lbAW
documentation for all different hash options

Alias of `Enum['hash:ip', 'hash:ip,port', 'hash:ip,port,ip', 'hash:ip,port,net', 'hash:ip,mark', 'hash:net', 'hash:net,net', 'hash:net,iface', 'hash:net,port', 'hash:net,port,net', 'hash:mac']`
