# enet.c3l
(wip~) c3 bindings for enet via https://github.com/zpl-c/enet

```c3
import std;
import std::io;
import enet;

const int MAX_CLIENTS = 32;

fn int main(String[] args) {
    
    if (enet::initialize () != 0) {
        io::printfn("An error occurred while initializing ENet.");
        return 1;
    }
    else {
        io::printfn("Successfully initialized ENet.");
    }

    ENetAddress address = {};

    address.host = enet::HOST_ANY; /* Bind the server to the default localhost. */
    address.port = 27000;          /* Bind the server to port 7777              */

    /* create a server */
    ENetHost * server = enet::host_create(&address, MAX_CLIENTS, 2, 0, 0);

    defer enet::deinitialize();
    defer enet::host_destroy(server);

    if (server == null) {
        io::printfn("An error occurred while trying to create an ENet server host.");
        return 1;
    }

    io::printfn("Started server %h:%s", address.host, address.port);

    ENetEvent event;


    /* Wait up to 1000 milliseconds for an event. (WARNING: blocking) */
    while (enet::host_service(server, &event, 5000)) {

        switch (event.type) {
            case EVENT_TYPE_CONNECT:
                io::printfn("A new client connected from %s:%d.",  event.peer.address.host, 
                                                                   event.peer.address.port);

                /* Store any relevant client information here. */
                event.peer.data = "Client information";
                break;

            case EVENT_TYPE_RECEIVE:
                io::printf("A packet of length %d containing %s was received from %s on channel %d.\n",
                        event.packet.dataLength,
                        event.packet.data,
                        event.peer.data,
                        event.channelID);

                /* Clean up the packet now that we're done using it. */
                enet::packet_destroy (event.packet);
                break;

            case EVENT_TYPE_DISCONNECT:

                io::printfn("%s disconnected.", event.peer.data);
                /* Reset the peer's client information. */
                event.peer.data = null;
                break;

            case EVENT_TYPE_DISCONNECT_TIMEOUT:

                io::printfn("%s disconnected due to timeout.", event.peer.data);
                /* Reset the peer's client information. */
                event.peer.data = null;
                break;

            case EVENT_TYPE_NONE:
                io::printfn("EVENT_TYPE_NONE");
                break;
        }
    }
    
    return 0;
}
```
