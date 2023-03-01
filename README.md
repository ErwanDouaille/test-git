# The task is to create a compositor based on Mir

## Technical choices

By reading multiple blogs/articles, I found an "user-friendly" api for creating a compositor based on Mir. This library is named Miral. Really easy to use for creating a simple compositor. I was able to get my first window with miral after few minutes.

I decided to get all applications with fullscreen by modifying the `place_new_window` and `handle_modify_window` (personnal choice).

The window manager policy also include a set of shortcuts such as :

* alt + tab : focus next application
* alt + F4 : close current applicaiton
* alt + shift + F8 : print count application
* alt + shift + F9 : print applications informations

The MirRunner comes with shortcuts as well :
* ctrl + alt + shift + backspace : stop MirRunner
* ctrl + alt + shift + G : launch gedit
* ctrl + alt + shift + X : launch xfce4-terminal

I lost a lot of time trying to get my own wallpaper class. Without success I decided to re-use egmde classes and make it works in few minutes. I had issues trying to get wayland surface trough miral api. I tried to found a solution but api is quite opaque ...

## Souce code explanations

### CMakeLists

A simple CMakelist, I added the miral module plus freetype, wayland_client and boost for the printer and wallpaper.

### main.cpp

The main file is composed of a a MirRunner. An internal launcher allowing us to launch application from our mir compositor. An external launcher allowing us to handle external application inside our compositor, tested with : 
``
WAYLAND_DISPLAY=wayland-99 xfce4-terminal.
``

A wallpaper class :
``
erwanTest::Wallpaper wallpaper;
runner.add_stop_callback([&] { wallpaper.stop(); });\
``
  
At startup with a Configuration option gedit is launch.
``
auto run_startup_apps = [&](std::string const& apps)
``
  
The event filter allow us to launch applications and quit the current mir session.
``
AppendEventFilter event_filter{[&](MirEvent const* event)
``
  
MirRunner class is run with multiple options composed of what is describe above.
  
### erwan_window_management h/cpp

ErwanWindowManagement handle the behavior of windows and inputs. Inherit from CanonicalWindowManagerPolicy virtual class.

I removed the touch event because I don't need it.

I added events filter inside this function : 
``
handle_keyboard_event
``
I tried to print and play with workspace and trees (removed) but AFAIK there is one session with one default workspace and I didn't succeed switching between multiple workspace even if they were created by myself. 

Inside functions `place_new_window` and `handle_modify_window` I forced the fullscreen state for all windows.

### erwan_wallpaper h/cpp

Copied from [3]. Needed for the wallpaper.

### printer h/cpp

Copied from [3]. Needed for the text on top of the wallpaper.

### egfullscreenclient.cpp

Copied from [3]. Needed for the wallpaper/printer. I simply removed a depency with xkb because I don't need it.

## Ressources :

1. [Developing a Wayland Compositor using Mir](https://mir-server.io/docs/developing-a-wayland-compositor-using-mir?fbclid=IwAR0EWhSwoP4NCO629JqjhEpC26WyL80CysuYJhlBl1Fv9kmHa6POSIlfJ9o)
2. [Egmde: Wayland reboot](https://discourse.ubuntu.com/t/egmde-wayland-reboot/4911)
3. [Egmde Alan Griffiths github repo](https://github.com/AlanGriffiths/egmde)
4. [Miral github repository](https://github.com/ubports/miral/tree/master)
5. [Blog mir](https://ubuntu.com/blog/creating-graphical-shells-try-mir-in-a-virtual-machine)
6. [Mir documentation](https://mir-server.io/doc/index.html)
7. [MirServer github repository](https://github.com/MirServer/mir/tree/main/examples/miral-kiosk?fbclid=IwAR2IGGfFTMPE5-3OHSh9WdwYQhCuNG_gyu59kIcNJO3E4UfaAAM8ekc7mR0)
