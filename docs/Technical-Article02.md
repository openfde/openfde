# Break free from VNC and unlock a true multi-window Linux experience on Android.

<img width="2864" height="1610" alt="image" src="https://github.com/user-attachments/assets/33934278-7434-4c2a-921d-efa09650f9f6" />

Those who have read our previous technical article should already know that we used to run Linux applications on the desktop with the VNC solution，but this solution has a flaw：because it directly projects the Linux screen，there is always only one window during use，no matter how many pages you open，they are all confined within that frame，which is quite inconvenient.

<img width="2864" height="1610" alt="image" src="https://github.com/user-attachments/assets/7cb378fd-afb9-4e48-9d6f-49d5f37125cc" />

So after research and exploration，we adopted a new technical solution—xserver，which can directly deliver program display content without screen image forwarding；the biggest change in practice is that you can open multiple windows，and the overall interaction is no different from Linux（for example，you can merge and split windows directly by dragging，and you are prompted whether to save when closing a document，etc.）。

##### **Principle Explanation：**

In case anyone does not know，before explaining the principle，let me mention that we are actually a Linux desktop based on AOSP，with the underlying layer based on Waydroid，so it is somewhat like an Android desktop that can run on Linux
<img width="1676" height="800" alt="image" src="https://github.com/user-attachments/assets/2c1999fc-924f-49ab-8cb8-e0e7522bc4d2" />




This is a comparison of the technical structures of the two：you can see that VNC forwards operations and images through vncserver，while the xserver process is much simplified，performing better in aspects such as performance，compatibility，and user experience；next，we will explain the xserver solution in detail：
<img width="1528" height="873" alt="image" src="https://github.com/user-attachments/assets/a9b9cf57-8bb1-4498-a48a-1397e152e887" />




This is a detailed structure diagram of xserver. Let me first briefly introduce the background：Xserver is a core component of the X Window System，and X Window is actually one of the foundations of the graphical environments of Linux distributions；popular Linux desktop environments such as the well-known GNOME and KDE Plasma are all built on the X Window System. So what you can see is that we ported the xserver originally used in Linux systems to our FDE desktop，where it is responsible for managing display devices and processing requests from X clients，allowing Linux applications to run directly on our desktop through this approach.

Next，let us explain this structure in detail：the orange part on the left consists of the two modules responsible for display in Linux，which pass all display information，including window information，to the ported xserver；our Xserver runs in an Android service（XWindowService）in an independent process，and in this service native functions start a TCP server or a Unix socket server，so that Linux programs supporting the X11 protocol can connect to this server，send drawing instructions to it，and receive input events from it.

Many extensions and supplementary protocols on Xserver make it still the most compatible and efficient display solution on Linux，supporting both local and remote connections with very small memory usage. Compiling and debugging in the Android NDK is also quite convenient，with no redundant dependencies，and there are many mature solutions we can learn from.

Below XWindowService there is another module，which is **WindowManager（window manager）**，a program used to manage and control X Window System (X11) windows. It handles operations such as opening，closing，moving，and resizing windows，and determines the appearance and behavior of windows. On OpenFDE，the Xserver connects to a custom-developed basic window manager，which runs in FDE-X11，has no window decoration function，and additionally implements functions such as a compositor，window property synchronization，and clipboard synchronization.
<img width="1776" height="963" alt="image" src="https://github.com/user-attachments/assets/0031b3f6-ef96-4ca3-8582-f78208a64c39" />


In the window manager，there is a relatively innovative point of ours called\*\*Compositor（compositor）image redirection，\*\*the Compositor extension is enabled in the window manager. Composite allows redirecting the output content of windows during the creation and display of all windows，redirecting the rendering of all windows in a window tree to internal storage，and then finally obtaining the image buffer of the window through the window pointer. The principle by which the compositor implements window effects is also based on this.

In this way，the image buffer of each window can be separated out and handed over to Android for drawing，that is，directly taking the image data from the Linux window into the Android window.

<img width="1528" height="873" alt="image" src="https://github.com/user-attachments/assets/bdf5c9cf-0c1b-4581-8e36-4080c4f2e84d" />

And the green part on the far right is **SurfaceManager and Activity，responsible for displaying content and receiving events**

Among them，SurfaceManager is a module abstracted out by FDE-X11，used to manage all Surfaces that draw buffers. Different Surfaces are passed into the EGL environment of Xserver so that different Surfaces correctly draw different buffers. For some OverrideRedirect windows，no new Surface will be created，and they will be drawn on the window they depend on.

As for the Activity part，it is mainly used as the window corresponding one-to-one with Linux. In the Android window system，the level handled by an APP window is confined to one level range，and only windows within this level range will have the same interaction behavior as windows of other APPs；currently，the only selectable Android components are Activity and Dialog. Their role is to create a SurfaceView to display the output image，receive Android input events，and send them to Xserver. Under the Android freeform mode used by FDE，the behavior is roughly equivalent to a Linux desktop，and in theory it can even be completely identical.

<img width="1321" height="880" alt="image" src="https://github.com/user-attachments/assets/c80d254a-ab4c-45dc-922d-a0ab1bf28252" />

When a Linux window is created，an Activity is created in Android，and the image of the Linux window is displayed on the SurfaceView of this Activity；if the Activity receives keyboard or mouse input，it is sent to the corresponding coordinates of Xserver. There are many problems to handle in this process，such as lifecycle synchronization，window types，event redirection，window icon and title bar operations，and so on.

In addition to what is mentioned above，we also did related work such as **Android and Linux window property synchronization，Android and Linux clipboard synchronization**，which I will not elaborate on here.

In summary，unlike the current Xserver implementations on various platforms，because FDE-X11 itself is a desktop environment，with the power of AOSP and the ideas of some open-source solutions（such as Termux:X11），this innovative solution has been realized. The research and development of this solution took quite a long time，but there are still many immature aspects：for example，only part of the complex features of X windows have been implemented；in addition，Android windows are limited by the original design of the framework，and modifying them requires a huge amount of work，especially since the lifecycle overhead is one or two orders of magnitude higher than that of X windows；there is also the more efficient Wayland protocol. All of these leave a lot of room for optimizing user experience.

Hope this can give you all some reference and inspiration～The code of this solution is also placed in our code repository，and you can take it as you need. If you have any other questions，feel free to leave a message，and they will all be answered one by one.
