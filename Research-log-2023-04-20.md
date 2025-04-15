# 2023-04-20

1. ok, so, how do we do the thing
    1. first problem: for some reason it started throwing `zmq message arrived on closed channel` errors
        1. https://github.com/jupyter/notebook/issues/6721 recommends downgrading, `pip install --upgrade "pyzmq<25" "jupyter_client<8"`; that worked
    1. second problem: it fails at `torch._C._cuda_init()` with `RuntimeError: CUDA unknown error`.
        1. aha, we did apt upgrade of the kernel; the answer about reinstating the relevant kernel module sounds about right, let's try
        1. yep that worked, thank you `https://stackoverflow.com/questions/54998936/unable-to-connect-to-the-local-runtime-in-google-colab` & `> sudo rmmod nvidia_uvm` & `sudo modprobe nvidia_uvm` & relaunching the jupyter server.

1. ---sudden discovery that 128/4 on digit=2 takes about 2500 epochs to start descending to 0!
    1. ......._sometimes_. most of the runs it doesn't. huh.

1. ok, anyway, let's do the 