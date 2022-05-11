## What is this

Netbox API wrapper for Crystal, in a very early stage of development

## Installation

TODO: Write installation instructions here

## Usage

```
# Get hosts in tenants =>

  require "netbox"

  nbx = Netbox::API.new(
    NETBOX_URL,
    API_TOKEN,
    debug: true
  )

  tens = nbx.get_entities(:tenants).as(Array(Netbox::APITenants))

  host_chans = [] of Channel(Tuple(String, Array(Netbox::API::HostsCommon)))
  res = Hash(String, Array(Netbox::API::HostsCommon)).new
  tens.each do |ten|
    {:virtual_machines, :hosts}.each do |hosts_type|
      host_chans.push(ch_res = Channel(Tuple(String, Array(Netbox::API::HostsCommon))).new(1))
      spawn do
        hosts = nbx.get_entities(hosts_type, tenant: ten.slug).unsafe_as(Array(Netbox::API::HostsCommon))
                   .reject! {|h| ! (h.host_name && h.primary_ip4 && h.status.value == "active")  }
        ch_res.send({ten.slug, hosts})
      end
    end # <- each hosts_type
  end

  host_chans.each do |ch|
    slug, hosts = ch.receive
    if hosts.size > 0
      if res[slug]?
        res[slug].concat(hosts)
      else
        res[slug] = hosts
      end
    end
  end
  
  pp res
```
## Development

TODO: Write development instructions here

## Contributing

1. Fork it (<https://github.com/DRVTiny/crystal-netbox/fork>)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Contributors

- [Andrey A. Konovalov](https://github.com/DRVTiny) - creator and maintainer
