# Deluge::RPC

Ruby RPC client library for [Deluge 2.0+ torrent client](https://deluge-torrent.org/).
Provides dynamic API bindings depending on RPC server.
Multi-threading friendly, thanks to ``concurrent-ruby`` gem.

*Deluge 1.x is not supported anymore*.

## Reference

* [Official RPC protocol documentation](https://deluge.readthedocs.io/en/latest/reference/rpc.html)
* [Deluge RPC API reference](https://deluge.readthedocs.io/en/latest/reference/api.html)
* [Official repository](https://github.com/deluge-torrent/deluge)
* Other client implementations:
  [Go](https://github.com/gdm85/go-libdeluge),
  [Python](https://github.com/JohnDoee/deluge-client)

## Usage

```ruby
require 'deluge'

# Initialize client
client = Deluge::Rpc::Client.new(
    host: 'localhost', port: 58846,
    login: 'username', password: 'password'
)

# Start connection and authenticate
client.connect

# Get auth level
client.auth_level
# => 5

# Get available methods
client.api_methods
# => ['daemon.add_torrent_file', 'core.shutdown', ...]

# Get deluge version
client.daemon.info
# => "1.3.10"

# Get torrents list
client.core.get_torrents_status({}, ['name', 'hash'])
# => [{name: 'Hot Chicks Action', hash: '<torrent_hash>'}, ...]

# Get namespace
core = client.core
# => <Deluge::Rpc::Namespace name="core">

# Get namespace methods
core.api_methods
# => ['core.get_session_status', 'core.get_upload_rate', ....]

# Invoke namespace method
core.get_config
# => {"info_sent"=>0.0, "lsd"=>true, "send_info"=>false, ... }

# Close connection
client.close
```

## Events

Receiving events is dead simple:

```ruby
client.register_event('TorrentAddedEvent') do |torrent_id|
    puts "Torrent #{torrent_id} was added! YAY!"
end

# You can pass symbols as event name,
# they would be converted to proper camelcase event names aka "EventNameEvent"
client.register_event(:torrent_removed) do |torrent_id|
    puts "Torrent #{torrent_id} was removed ;_;"
end
```

Unfortunately there is no way to listen ALL events, due to Deluge architecture.
You have to register each event you need.

**Keep in mind event blocks would be executed in connection thread, NOT main thread!**
Avoid time-consuming code!

### Known events

 Event Name               | Arguments               | Description
--------------------------|-------------------------|------------------------------------------------------
``TorrentAddedEvent``         | torrent_id              | New torrent is successfully added to the session.
``TorrentRemovedEvent``       | torrent_id              | Torrent has been removed from the session.
``PreTorrentRemovedEvent``    | torrent_id              | Torrent is about to be removed from the session.
``TorrentStateChangedEvent``  | torrent_id, state       | Torrent changes state.
``TorrentQueueChangedEvent``  | &nbsp;                  | The queue order has changed.
``TorrentFolderRenamedEvent`` | torrent_id, old, new    | Folder within a torrent has been renamed.
``TorrentFileRenamedEvent``   | torrent_id, index, name | File within a torrent has been renamed.
``TorrentFinishedEvent``      | torrent_id              | Torrent finishes downloading.
``TorrentResumedEvent``       | torrent_id              | Torrent resumes from a paused state.
``TorrentFileCompletedEvent`` | torrent_id, index       | File completes.
``NewVersionAvailableEvent``  | new_release             | More recent version of Deluge is available.
``SessionStartedEvent``       | &nbsp;                  | Session has started.  This typically only happens once when the daemon is initially started.
``SessionPausedEvent``        | &nbsp;                  | Session has been paused.
``SessionResumedEvent``       | &nbsp;                  | Session has been resumed.
``ConfigValueChangedEvent``   | key, value              | Config value changes in the Core.
``PluginEnabledEvent``        | name                    | Plugin is enabled in the Core.
``PluginDisabledEvent``       | name                    | Plugin is disabled in the Core.

This list was extracted from Deluge 1.3.11 sources. Events for your version could be different.
There is no official documentation.

Current events could be found here:
https://github.com/deluge-torrent/deluge/blob/master/deluge/event.py

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'deluge-rpc'
```

And then execute:

```shell
$ bundle
```

Or install it yourself as:

```shell
$ gem install deluge-rpc
```
