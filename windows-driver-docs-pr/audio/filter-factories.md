---
title: Filter Factories
description: Filter Factories
ms.assetid: e836f941-274f-4e27-8069-753ef9ef2a06
keywords: ["audio filters WDK audio , filter factories", "filter factories WDK audio", "filters WDK audio , factories", "KS filters WDK audio , filter factories", "audio jacks WDK", "speakers WDK audio , filter factories", "instantiating filters WDK audio"]
---

# Filter Factories


## <span id="filter_factories"></span><span id="FILTER_FACTORIES"></span>


An audio adapter driver provides filter factories to manage the instantiation of filters. Each filter factory can instantiate one or more KS filters of a particular type. If a filter type encapsulates a particular hardware function, the number of filters of that type that the factory can instantiate is limited by the underlying hardware resources.

Because a filter factory manages a largely autonomous block of hardware functionality, each filter factory can be considered to be a device driver in its own right. In fact, the term adapter driver as it is used in the preceding paragraph, refers to a collection of related drivers--filter factories--that are packaged together to manage the various hardware functions on an adapter card.

As with any other Microsoft Windows Driver Model (WDM) driver, a filter factory handles power management and setup functionality. During installation, the INF file for the driver registers one or more filter device names (see [Device Identification Strings](https://msdn.microsoft.com/library/windows/hardware/ff541224)). This process loads the names into the system registry and associates each filter factory with one or more KS filter categories, as described in [Installing Device Interfaces for an Audio Adapter](installing-device-interfaces-for-an-audio-adapter.md). All audio devices are classified under KSCATEGORY\_AUDIO, but an audio device might also be classified under additional categories such as KSCATEGORY\_RENDER (for an audio rendering device) or KSCATEGORY\_CAPTURE (for an audio capture device). The driver advertises the general capabilities of a device by means of the various categories under which it registers the filter for that device. When the [SysAudio system driver](kernel-mode-wdm-audio-components.md#sysaudio-system-driver), for example, requires an audio device of a particular type, it looks in the registry for devices that fall into the appropriate categories.

The operating system uses the Setup API, as described in [Device Installation Components](https://msdn.microsoft.com/library/windows/hardware/ff728855), to discover and enumerate all the KSCATEGORY\_AUDIO filter factories in the registry. The registry entry for each factory specifies both the filter factory's friendly name and its device name, which is a long string that a client passes to the create-file call that instantiates the filter. This call might be made to [**ZwCreateFile**](https://msdn.microsoft.com/library/windows/hardware/ff566424) from kernel mode or to **CreateFile** from user mode. A filter is a kernel-mode object and is identified by a kernel handle. The create-file call returns an instance handle that clients can use to refer to the filter. User-mode clients or upstream filters in the audio graph can use this handle to send or forward IOCTL requests to the filter. For more information about **CreateFile**, see the Microsoft Windows SDK documentation.

A typical WDM audio adapter card might reside on a PCI bus, for example, and contain several I/O connectors for rendering or capturing wave data. A single audio device on this card might contain analog audio-out jacks for driving a set of speakers and a lineout cable, and analog audio-in jacks for receiving signals from a microphone and a linein cable. The WDM audio system represents the device as a filter and represents the audio jacks as pins on that filter.

The filter for an audio device is implemented as separate port and miniport drivers that are bound together to act in unison:

-   The miniport driver contains the hardware-specific code.

-   The port driver contains the generic code that is common to all filters of a particular type.

The vendor writes the miniport driver, which contains all the proprietary code that the filter needs to manage the audio hardware. The operating system provides the port driver, which is accessible through the PortCls system driver (Portcls.sys; see [Port Class Adapter Driver and PortCls System Driver](kernel-mode-wdm-audio-components.md#port-class-adapter-driver-and-portcls-system-driver)). Dividing the filter implementation into port and miniport drivers simplifies the task of writing a driver for a proprietary device.

When a filter factory instantiates a filter, it first creates the miniport driver object for the filter. The filter factory then creates an instance of the appropriate port object and binds the miniport driver object to that instance in order to form a fully functioning filter. The code example in [Subdevice Creation](subdevice-creation.md) illustrates this process. The port and miniport drivers communicate with each other through well-defined software interfaces. For more information about these interfaces, see [Miniport Interfaces](miniport-interfaces.md) and [Supporting a Device](supporting-a-device.md).

An audio filter exposes the structure of the underlying audio device as a collection of pin factories, nodes, and internal connections. The miniport driver consolidates this information into a filter descriptor, which is a structure of type [**PCFILTER\_DESCRIPTOR**](https://msdn.microsoft.com/library/windows/hardware/ff537694). This structure, in turn, contains individual descriptors for the filter's pin factories, nodes, and internal connections. These descriptors are structures of the following types:

[**PCPIN\_DESCRIPTOR**](https://msdn.microsoft.com/library/windows/hardware/ff537721)

[**PCNODE\_DESCRIPTOR**](https://msdn.microsoft.com/library/windows/hardware/ff537720)

[**PCCONNECTION\_DESCRIPTOR**](https://msdn.microsoft.com/library/windows/hardware/ff537688)

To obtain the filter descriptor from the miniport driver, the port driver calls the [**IMiniport::GetDescription**](https://msdn.microsoft.com/library/windows/hardware/ff536765) method.

For an example of how a driver sets up its PCFILTER\_DESCRIPTOR structure, see the header file Table.h in the sb16 sample audio driver in the Windows Driver Kit (WDK).

 

 


--------------------
[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20[audio\audio]:%20Filter%20Factories%20%20RELEASE:%20%287/18/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")


