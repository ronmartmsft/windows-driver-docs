---
Description: 'This topic describes best practices about implementing the remote wakeup capability in a client driver.'
MS-HAID:
- 'usbpower\_18d34c21-29a2-449c-a3ac-fce27187c246.xml'
- 'buses.remote\_wakeup\_of\_usb\_devices'
MSHAttr:
- 'PreferredSiteName:MSDN'
- 'PreferredLib:/library/windows/hardware'
title: Remote Wakeup of USB Devices
---

# Remote Wakeup of USB Devices


This topic describes best practices about implementing the remote wakeup capability in a client driver.

USB devices that can respond to external wake signals while suspended are said to have a *remote wakeup* capability. Examples of devices that have a remote wakeup capability are mice, keyboards, USB hubs, modems (wake on ring), NICs, wake on cable insertion. All of these devices are capable of producing remote wake signaling. Devices that are not capable of generating remote wake signaling include video cameras, mass storage devices, audio devices, and printers.

Drivers for devices that support remote wakeup signaling must issue an [**IRP\_MN\_WAIT\_WAKE**](https://msdn.microsoft.com/library/windows/hardware/ff551766) IRP, also known as a wait wake IRP, to arm the device for remote wakeup. The wait wake mechanism is described in the section [Supporting Devices That Have Wake-Up Capabilities](https://msdn.microsoft.com/library/windows/hardware/ff563907).

## When Does the System Enable Remote Wakeup on a USB Leaf Device?


In USB terminology, a USB device is enabled for remote wakeup when its DEVICE\_REMOTE\_WAKEUP feature is set. The USB specification specifies that host software must set the remote wakeup feature on a device "only just prior" to putting the device to sleep.

For this reason, the USB stack does not set the DEVICE\_REMOTE\_WAKEUP feature on a device after receiving a wait wake IRP for the device. Instead, it waits until it receives a [**IRP\_MN\_SET\_POWER**](https://msdn.microsoft.com/library/windows/hardware/ff551744) request to change the WDM device state of the device to D1/D2. Under most circumstances, when the USB stack receives this request, it both sets the remote wakeup feature on the device and puts the device to sleep by suspending the device's upstream port. When you design and debug your driver, you should keep in mind that there is a loose relationship between arming a USB device for wakeup in software, by means of a wait wake IRP, and arming the device for wakeup in hardware by setting the remote wakeup feature.

The USB stack does not enable the device for remote wakeup when it receives a request to change the device to a sleep state of D3, because according to the WDM power model, devices in D3 cannot wake the system.

## Why Does Attaching or Detaching My Device Produce a Different Wakeup Behavior in Windows XP and Windows Vista and later versions of Windows?


Another unique aspect of the USB implementation of the WDM power mode regards the arming of USB hubs for remote wakeup. In Microsoft Windows XP all hub devices between host controller and the USB device are armed for wakeup whenever the USB device is armed for wakeup. This produces the surprising consequence that when sleeping devices are detached they will wake up the system.

In Windows Vista and later versions of Windows, if a USB leaf device on the bus is armed for wake, the USB stack will also arm the USB host controller for wake, but it will not necessarily arm any of the USB hubs upstream of the device. The USB hub driver arms a hub for remote wakeup only if the USB stack is configured to wake up the system on attach and detach (plug/unplug) events.

**Note**  UHCI (Universal Host Controller Interface) USB host controllers do not distinguish between remote wake signaling and connect change events on root hub ports. This means the system will always wake from a low system power state if a USB device is connected to or disconnected from a root hub port, if there is at least one device behind the UHCI controller that is armed for wake.

 

## Related topics


[USB Power Management](usb-power-management.md)

 

 

[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20%5Busbcon\buses%5D:%20Remote%20Wakeup%20of%20USB%20Devices%20%20RELEASE:%20%281/26/2017%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")




