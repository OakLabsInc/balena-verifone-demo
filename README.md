# Verifone (Balena) Demo Application

This project allows for a demo site that builds a cart to send a payload to a Verifone P400 terminal and then sends the same cart to a TM-T88V POS printer. This is to be used as a POC or demo.

## Containers

This project uses 5 containers together within the BalenaOS to provide the necessary elements to accomplish a complete demo.

## Balena Blocks

Balena provides certain `Block Containers` for common use cases. In our case we use two of them.

### Xserver Container

Provided as a block from Balena Hub, it creates a window server for our application to display an Electron window to house the remote url provided in the application's environmental variable. For more information on this block, see: <https://github.com/balenablocks/xserver>

### Wifi Connect Container

This is a very useful container to allow your unit to connect to a local WIFI SSID. If there is no internet connection and the BalenaOS unit is powered on, this block container provides an access point that can be connected to by your mobile device or LAN computer. It will present an interface to enter your WIFI credentials and pass those to the unit for it to connect. For more information about this block see: <https://github.com/balena-os/wifi-connect>

## Payment Container

Provides the middleware interface to the Verifone P400 using the Verifone PSDK to process a cart payload from the application container to the POS terminal at the given `terminalIp` and return card transaction validation. This container expects this payload:

``` json
{
  "items": [
    {
      "name": "Item 1",
      "subtotal": 20.2,
      "taxRate": 0.085,
      "total": 21.916999999999998
    },
    {
      "name": "Item 2",
      "subtotal": 20.2,
      "taxRate": 0.085,
      "total": 21.916999999999998
    }
  ],
  "cart": {
    "total": 40.4,
    "tax": 3.434,
    "taxRate": 0.085,
    "grandTotal": 43.833999999999996
  },
  "terminalIp": "192.168.31.44"
}
```

- note that the `terminalIp` should be the IP of the POS Terminal being used.
- the items array can have as many items necessary, but for this demo to print correctly, all the same field are required.
- The JSON object payload contract above is a fixed structure and all the field names shown are required.

## Printer Container

Provides the printer description and drivers for the TM-T88V POS printer. This container expects a valid print command. See Cups printing for more information. At the time of this printing, only ppds for the Epson TM-T88V POS printer are provided.

## Application Container

As well as serving as an API interface using NodeJS, it will connect the UI/UX contained in the `REMOTE_URL` environmental variable through the Electron window provided. The NodeJS API will also connect the application UI to the `Verifone Payment Container` which passes the payload on to the POS Terminal. The response recieved will be a transaction validation. If the response is a `SUCCESS` then the UI should be able to pass the same payload to the `Printer Container` to print a receipt. There are some `API` endpoints for sending the cart payload to both the  `Oak` npm module listeners.

There are environment variables that will need to be set for this container to work properly.

1. `DISPLAY`

