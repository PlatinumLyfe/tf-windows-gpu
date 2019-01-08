# tf-windows-gpu
Tensorflow Nightly GPU is not being updated for Windows, so here's a repo of wheels I made myself.

__All are compiled with CUDA 10.0__
Not CUDA 9.0 or 9.2 since somewhere in the commits of Tensorflow, they updated it to CUDA 10.0 and later version of cuDNN

Also I have them compiled for Python 3.6.6 and Python 3.7.2, but may be dropping 3.6.6 soon since 3.7.2 seems a bit quicker.

They are also compiled for the later headers and libraries for MSVC 2017 (or should be at least) which means you may require the VS2017 VC++ redist - I can't know and I can't test.

I will chuck up a guide on how I got it to compile myself, since it required a bit of work FOR SOMETHING WHICH SHOULD FREAKING WORK OUT OF THE BOX... right?

I am still trying to see if I can manage to get XLA to compile for Windows but it looks very unlikely.
Earlier commits were closer to getting XLA to compile on windows than later commits were...

Two steps backwards, half a step forwards...
