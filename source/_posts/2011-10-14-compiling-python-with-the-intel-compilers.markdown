---
layout: post
title: "Compiling Python With the Intel Compilers"
date: 2011-10-14 00:20
comments: true
categories: python
---

I'm a heavy user of [Python][] for work, but on must clusters I've worked on the
installation has had significant problems. Usually either the builds of
[numpy][] and [scipy][] aren't built against any optimized numerical libraries
(such as [Intel MKL][mkl]), or they've been compiled with some mixed combination
of compilers which makes building C extensions difficult (especially with [Cython][]).

I've been struggling for a while to build a [Python][] toolchain completely around
the Intel compilers and math libraries, and I've just succeeded. It's a little involved so I've attempted to document the process here.

## Compiling Python ##

First, go and grab a copy of the Python source, and uncompress it. The
unmodified source cannot be built by the Intel compilers, as the `_ctypes`
library uses some gcc specific features. So before doing anything else, the file
`Modules/_ctypes/libffi/src/x86/ffi64.c` needs to be patched. The relevant
changes are below (taken from two Intel forum posts [here][1] and [here][2]).


{% codeblock patch for ffi64.c lang:diff %}
--- ffi64.old.c 2011-10-12 17:27:47.952517000 -0400
+++ ffi64.c	    2011-10-12 17:29:48.435726000 -0400
@@ -36,6 +36,8 @@
 #define MAX_GPR_REGS 6
 #define MAX_SSE_REGS 8
 
+typedef struct { int64_t m[2]; } __int128_t;
+
 struct register_args
 {
   /* Registers for argument passing.  */
@@ -470,10 +472,14 @@
		  break;
 		case X86_64_SSE_CLASS:
 		case X86_64_SSEDF_CLASS:
-		  reg_args-&gt;sse[ssecount++] = *(UINT64 *) a;
+		  reg_args-&gt;sse[ssecount].m[0] = *(UINT64 *) a;
+		  reg_args-&gt;sse[ssecount++].m[1] = 0;
 		  break;
 		case X86_64_SSESF_CLASS:
-		  reg_args-&gt;sse[ssecount++] = *(UINT32 *) a;
+		  reg_args-&gt;sse[ssecount].m[0] = *(UINT32 *) a;
+		  reg_args-&gt;sse[ssecount++].m[1] = 0;
 		  break;
   		default:
 		  abort();
{% endcodeblock %}

Then with this done python can be compiled. To force it to compile against the
Intel compilers instead of *GCC*, we need to set some environment variables, and
then configure and build it.

{% codeblock Set compilation environment lang:bash %}
$ export CC=icc
$ export CXX=icpc
$ ./configure --without-gcc
$ make
$ make install
{% endcodeblock %}

## Compiling Numpy and Scipy ##

Now to building *numpy* and *scipy*. There some detailed instructions posted
[here][3], which were useful, though I think some parts are out of date. The
main thing to do is to create a `site.cfg` file at the root of the numpy build
directory setting all the MKL variables (in the following `$MKLROOT` must be
explicitly expanded to the root of the MKL library, usually defined in the
environment variable of the same name).

{% codeblock site.cfg lang:ini %}
[mkl]
library_dirs = $MKLROOT/lib/intel64
include_dirs = $MKLROOT/include
mkl_libs = mkl_def,mkl_intel_lp64,mkl_intel_thread,mkl_core,mkl_mc
lapack_libs = mkl_lapack95_lp64
{% endcodeblock %}

Next we need to edit the Intel compiler setup in numpy's distutils. Edit
`numpy/distutils/intelccompiler.py` and change all definitions of `cc_exe` to be
`cc_exe = 'icc -O3 -g -fomit-frame-pointer -openmp -fPIC'`, not forgetting the
`self.cc_exe = ...` lines in the two constructors.

We can now build numpy with:

{% codeblock compile numpy lang:bash %}
$ python setup.py config --compiler=intel build_clib --compiler=intelem build_ext --compiler=intelem install
{% endcodeblock %}

Fire up python and check to see if it all works with (you'll need to have
installed *nose* to run the tests):

{% codeblock Test python lang:python %}
import numpy as np
np.test()
{% endcodeblock %}

In my build *numpy* imported, but one of the tests seemed to hang. Other than
that (!), everything else seemed to work correctly.

I found that in the most recent versions of *scipy* it wasn't necessary to make
all the changes detailed in the post linked [above][3], just copy the *numpy* `site.cfg` into the scipy
directory. However, we also seem to need to patch one of the files
`scipy/spatial/qhull/src/qhull_a.h`

{% codeblock patch for qhull_a.h lang:diff %}
--- qhull_a.h   2011-02-27 18:57:02.000000000 -0500
+++ /home/jrs65/qhull_a.h       2011-10-12 20:42:36.898520000 -0400
@@ -103,9 +103,10 @@
 #   define QHULL_OS_WIN
 #endif
 #if defined(__INTEL_COMPILER) &amp;&amp; !defined(QHULL_OS_WIN)
-template 
-inline void qhullUnused(T &amp;x) { (void)x; }
-#  define QHULL_UNUSED(x) qhullUnused(x);
+// template 
+//void qhullUnused((int *) x) { (void)x; }
+// #  define QHULL_UNUSED(x) qhullUnused(x);
+#  define QHULL_UNUSED(x) (void)x;
 #else
 #  define QHULL_UNUSED(x) (void)x;
 #endif
{% endcodeblock %}

Then the only step left is to build *scipy*. Do that by running:

{% codeblock Building scipy lang:bash %}
$ python setup.py config --compiler=intelem --fcompiler=intelem build_clib --compiler=intelem --fcompiler=intelem build_ext --compiler=intelem --fcompiler=intelem install
{% endcodeblock %}

Voila. In theory you should have a working Python/Numpy/Scipy toolchain. On with science!


[python]: http://www.python.org/
[cython]: http://www.cython.org/
[mkl]: http://software.intel.com/en-us/articles/intel-mkl/
[numpy]: http://numpy.scipy.org/
[scipy]: http://www.scipy.org/
[1]: http://software.intel.com/en-us/articles/build-firefox-35-with-intel-c-compiler/
[2]: http://origin-integration-software.intel.com/en-us/forums/showthread.php?t=56652&amp;o=d&amp;s=lr
[3]: http://blog.sun.tc/2010/11/numpy-and-scipy-with-intel-mkl-on-linux.html
