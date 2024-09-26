This document is written by [Joon Ho Ryu](https://github.com/junhoryu).

This page is partially copied from https://gstreamer.freedesktop.org/documentation/tutorials/index.html?gi-language<br/>
you might need required libraries for other tutorials which is not in this page.<br/>
For more detail reference the page.<br/>

Make sure you have superuser(root) access rights to install GStreamer

**1. Download prerequisites**
```
$ apt-get install libgstreamer1.0-0 gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 gstreamer1.0-qt5 gstreamer1.0-pulseaudio
```
**2. Download GStreamer**
```
$ git clone https://gitlab.freedesktop.org/gstreamer/gst-docs
```
**3. Example source code**

`basic-tutorial-1.c` is from the link below.
- https://gstreamer.freedesktop.org/documentation/tutorials/basic/hello-world.html?gi-language=c
```c
$ cat basic-tutorial-1.c
#include <gst/gst.h>

int
main (int argc, char *argv[])
{
  GstElement *pipeline;
  GstBus *bus;
  GstMessage *msg;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Build the pipeline */
  pipeline =
      gst_parse_launch
      ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm",
      NULL);

  /* Start playing */
  gst_element_set_state (pipeline, GST_STATE_PLAYING);

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

  /* Free resources */
  if (msg != NULL)
    gst_message_unref (msg);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  return 0;
}
```
**4. Build the tutorials**
```
$ gcc -pg basic-tutorial-1.c -o basic-tutorial-1 `pkg-config --cflags --libs gstreamer-1.0`
```
**5. Record basic-tutorial-1.c**
```
$ uftrace record -P . basic-tutorial-1
```
**6. Replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 15661] | main() {
   5.788 ms [ 15661] |   gst_init();
   1.110 ms [ 15661] |   gst_parse_launch();
 741.006 us [ 15661] |   gst_element_set_state();
   0.593 us [ 15661] |   gst_element_get_bus();
   3.114  s [ 15661] |   gst_bus_timed_pop_filtered();
            [ 15661] |   gst_message_unref() {
   2.987 us [ 15661] |     gst_mini_object_unref();
   3.872 us [ 15661] |   } /* gst_message_unref */
   0.808 us [ 15661] |   gst_object_unref();
  60.858 ms [ 15661] |   gst_element_set_state();
 311.514 us [ 15661] |   gst_object_unref();
   3.183  s [ 15661] | } /* main */
```
**7. uftrace tui**
```
$ uftrace tui
TOTAL TIME : FUNCTION
    3.183  s : (1) basic-tutorial-1                                                                                           
    3.183  s : (1) main
    5.788 ms :  ├─(1) gst_init
             :  │
    1.110 ms :  ├─(1) gst_parse_launch
             :  │
   61.599 ms :  ├─(2) gst_element_set_state
             :  │
    0.593 us :  ├─(1) gst_element_get_bus
             :  │
    3.114  s :  ├─(1) gst_bus_timed_pop_filtered
             :  │
    3.872 us :  ├─(1) gst_message_unref
    2.987 us :  │ (1) gst_mini_object_unref
             :  │
  312.322 us :  └─(2) gst_object_unref
```