*This library is currently being beta-tested. See something that's broken? Did we get something
wrong? [Create an issue and let us know!][issues]*



<p align="center">
    <a id="ibind" href="#ibind">
        <img src="https://raw.githubusercontent.com/Voyz/ibind/master/media/ibind_logo.png" alt="IBind logo" title="IBind logo" width="600"/>
    </a>
</p>
<p align="center">
    <a href="https://opensource.org/licenses/Apache-2.0">
        <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg"/>
    </a>
    <a href="https://github.com/Voyz/ibind/releases">
        <img src="https://img.shields.io/pypi/v/ibind?label=version"/> 
    </a>
</p>

IBind is a REST and WebSocket client library for [Interactive Brokers Client Portal Web API.][ibkr-docs]

## Installation

```rich
pip install ibind
```

Use [IBeam][ibeam] along with this library for easier authentication with IBKR.

## Documentation

See full [IBind documentation][wiki].

* [Installation][wiki-installation]
* [Basic Concepts][wiki-basic-concepts]
* [IBind Configuration][wiki-ibind-configuration]
* [IbkrClient][wiki-ibkr-client] - REST Python client for [IBKR REST API][ibkr-endpoints].
* [IbkrWsClient][wiki-ibkr-ws-client] - WebSocket Python client for [IBKR WebSocket API][ibkr-websocket].
* [API Reference][api-ibkr-client]

Features:
* REST:
  * [Automated question/answer handling][wiki-question-answer]
  * [Parallel requests][wiki-parallel-requests]
  * [Rate limiting][wiki-rate-limiting]
  * [Conid unpacking][wiki-conid-unpacking]
  * [and more][wiki-advanced-api]
* WebSocket:
  * [WebSocket thread lifecycle handling][wiki-ws-lifecycle]
  * [Thread-safe Queue data stream][wiki-ws-queues]
  * [Internal subscription tracking][wiki-ws-subscriptions]
  * [Health monitoring][wiki-ws-health-monitoring]
  * [and more][wiki-advanced-websocket]

## Overview

IBind's core functionality consists of two client classes:

* [`IbkrClient`][ibkr-client-docs] - for [IBKR REST API][ibkr-endpoints]

  Using the `IbkrClient` requires constructing it with appropriate arguments, then calling the API methods.

* [`IbkrWsClient`][ibkr-ws-client-docs] - for [IBKR WebSocket API][ibkr-websocket]

  Using the `IbkrWsClient` involves handling three areas:

  * Managing its lifecycle. It is asynchronous and it will run on a separate thread, hence we need to construct it, start it and then manage its lifecycle on the originating thread.
  * Subscribing and unsubscribing. It is subscription-based, hence we need to specify which channels we want to subscribe to and remember to unsubscribe later.
  * Consuming data. It uses a queue system, hence we need to access these queues and consume their data.

Their usage differs substantially. Users are encouraged to familiarise themselves with the `IbkrClient` class first.

## Examples

See [all examples][examples].

### Basic REST Example

```python
import warnings

from ibind import IbkrClient

# In this example we provide no CAcert, hence we need to silence this warning.
warnings.filterwarnings("ignore", message="Unverified HTTPS request is being made to host 'localhost'")

# Construct the client
client = IbkrClient()

# Call some endpoints
print('\n#### check_health ####')
print(client.check_health())

print('\n\n#### tickle ####')
print(client.tickle().data)

print('\n\n#### get_accounts ####')
print(client.portfolio_accounts().data)
```

### Basic WebSocket Example

```python
from ibind import IbkrWsKey, IbkrClient, IbkrWsClient, ibind_logs_initialize

# Initialise the logger
ibind_logs_initialize(log_to_file=False)

# Construct the clients
client = IbkrClient()
ws_client = IbkrWsClient(ibkr_client=client)

# Start the WebSocket worker thread
ws_client.start()

# Acquire a QueueAccessor for the Orders channel
qa = ws_client.new_queue_accessor(IbkrWsKey.ORDERS)

# Subscribe to the Orders channel
ws_client.subscribe(channel='or', data=None, needs_confirmation=False)

# Wait for new items in the Orders queue.
while True:
  try:
    while not qa.empty():
      print(str(qa), qa.get())

  except KeyboardInterrupt:
    print('KeyboardInterrupt')
    break

# Unsubscribe from the Orders channel and shutdown the client
ws_client.unsubscribe(channel='or', data=None, needs_confirmation=False)

ws_client.shutdown()

```


## Licence

See [LICENSE](https://github.com/Voyz/ibind/blob/master/LICENSE)

## Disclaimer

IBind is not built, maintained, or endorsed by the Interactive Brokers.

Use at own discretion. IBind and its authors give no guarantee of uninterrupted run of and access to the Interactive
Brokers Client Portal Web API. You should prepare for breaks in connectivity to IBKR servers and should not
depend on continuous uninterrupted connection and functionality. To partially reduce the potential risk use Paper Account credentials.

IBind is provided on an AS IS and AS AVAILABLE basis without any representation or endorsement made and without warranty
of any kind whether express or implied, including but not limited to the implied warranties of satisfactory quality,
fitness for a particular purpose, non-infringement, compatibility, security and accuracy. To the extent permitted by
law, IBind's authors will not be liable for any indirect or consequential loss or damage whatever (including without
limitation loss of business, opportunity, data, profits) arising out of or in connection with the use of IBind. IBind's
authors make no warranty that the functionality of IBind will be uninterrupted or error free, that defects will be
corrected or that IBind or the server that makes it available are free of viruses or anything else which may be harmful
or destructive.

## Acknowledgement

IBind has been enriched by incorporating work developed in collaboration with  [Kinetic](https://www.kinetic.xyz/) and [Grant Stenger](https://github.com/GrantStenger), which now forms part of the initial open-source release. I appreciate their significant contribution to this community-driven initiative. Cheers Kinetic! 🍻

## Built by Voy

Hi! Thanks for checking out and using this library. If you are interested in discussing your project, require
mentorship, consider hiring me, or just wanna chat - I'm happy to talk.

You can email me to get in touch: hello@voyzan.com

Or if you'd just want to give something back, I've got a Buy Me A Coffee account:

<a href="https://www.buymeacoffee.com/voyzan" rel="nofollow">
    <img src="https://raw.githubusercontent.com/Voyz/voyz_public/master/vz_BMC.png" alt="Buy Me A Coffee" style="max-width:100%;" width="192">
</a>

Thanks and have an awesome day 👋


[ibeam]: https://github.com/Voyz/ibeam
[examples]: https://github.com/Voyz/ibind/blob/master/examples
[issues]: https://github.com/Voyz/ibind/issues
[api-ibkr-client]: https://github.com/Voyz/ibind/wiki/API-Reference-%E2%80%90-IbkrClient
[ibkr-client-docs]: https://github.com/Voyz/ibind/wiki/Ibkr-Client
[ibkr-ws-client-docs]: https://github.com/Voyz/ibind/wiki/Ibkr-Ws-Client

[ibkr-docs]: https://ibkrcampus.com/ibkr-api-page/webapi-doc/
[ibkr-endpoints]: https://ibkrcampus.com/ibkr-api-page/cpapi-v1/#endpoints
[ibkr-websocket]: https://ibkrcampus.com/ibkr-api-page/cpapi-v1/#websockets


[wiki]: https://github.com/Voyz/ibind/wiki
[wiki-installation]: https://github.com/Voyz/ibind/wiki/Installation
[wiki-basic-concepts]: https://github.com/Voyz/ibind/wiki/Basic-Concepts
[wiki-ibind-configuration]: https://github.com/Voyz/ibind/wiki/IBind-Configuration
[wiki-ibkr-client]: https://github.com/Voyz/ibind/wiki/Ibkr-Client
[wiki-ibkr-ws-client]: https://github.com/Voyz/ibind/wiki/Ibkr-Ws-Client

[wiki-question-answer]: https://github.com/Voyz/ibind/wiki/Ibkr-Client#-place_order
[wiki-parallel-requests]: https://github.com/Voyz/ibind/wiki/Ibkr-Client#-marketdata_history_by_symbols
[wiki-rate-limiting]: https://github.com/Voyz/ibind/wiki/Ibkr-Client#-marketdata_history_by_symbols
[wiki-conid-unpacking]: https://github.com/Voyz/ibind/wiki/Ibkr-Client#-security_stocks_by_symbol
[wiki-advanced-api]: https://github.com/Voyz/ibind/wiki/Ibkr-Client#advanced-api

[wiki-ws-lifecycle]: https://github.com/Voyz/ibind/wiki/Ibkr-Ws-Client#-managing-the-lifecycle
[wiki-ws-queues]: https://github.com/Voyz/ibind/wiki/Ibkr-Ws-Client#-consuming-data
[wiki-ws-subscriptions]: https://github.com/Voyz/ibind/wiki/Ibkr-Ws-Client#-subscribing-and-unsubscribing
[wiki-ws-health-monitoring]: https://github.com/Voyz/ibind/wiki/Ibkr-Ws-Client#health-monitoring
[wiki-advanced-websocket]: https://github.com/Voyz/ibind/wiki/Advanced-WebSocket
