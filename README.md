# pyBlufi

**NOTE**: This is a WIP. Feel free to clone and try it out, but not installable
with pip yet.

Python utility to interface with esp32 Blufi component. Mimics
[EspBlufiForAndroid](https://github.com/EspressifApp/EspBlufiForAndroid)
and [EspBlufiForiOS](https://github.com/EspressifApp/EspBlufiForiOS).

## Why

Useful for testing/debugging Blufi commands without having to use a
phone. Thus making it easier to implement new features, build on top of
Blufi and not having to design apps and protocols from scratch.

## MTU

On some platforms, negotiated MTU is not accessible via API. On Linux, bluez
does not provide a way (yet?) to get negotiated MTU via dbus. For this reason,
payloads must be artificially capped using the `setPostPackageLengthLimit`
method. Using `setPostPackageLengthLimit(256)` should work in most cases. If
the client PC has a requirement for lower MTU, then this must be set lower. Or
if/when `bleak` can correctly obtain MTU for a given platform, it can be removed.

As far as MTUs > 256:

1. ESP-IDF defaults to 256, but can be configured to use up to 512.

1. The ESP-IDF Blufi protocol is limited to 256 bytes. You could likely modify
it if you wanted to, and maybe it could still work with MTU negotiation but I
have not put much thought into it.


## Install

```
pip install git+https://github.com/someburner/pyBlufi.git
```

## Usage (sending custom data)
```
import blufi
import bleak
import asyncio

client = blufi.BlufiClient()

async def scanDevices():
    print("Scanning for 5 seconds, please wait...")

    devices = await bleak.BleakScanner.discover(
        return_adv=True
    )

    for d, a in devices.values():
        print(f"{d} | {a}")

# Set customData received callback
def onCustomDataCallback(self, data):
    print(f"Received data: {data}")

blufi.BlufiClient.onCustomData = onCustomDataCallback

# Scan for devices
asyncio.run(scanDevices())

# Exemple to post customData to a discovered device
client.connectByName(name="the-discovered-device-name")
client.negotiateSecurity()
client.setPostPackageLengthLimit(256)

client.postCustomData(data="your-custom-data")

```