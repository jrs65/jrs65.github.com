+++
title = "Software"
template = "simple-section.html"
+++

Modern Physics is as much an exercise in software development as anything else. I
strongly believe we need to make the software required to generate and analyse datasets
open source, both for transparency and reproducibility, but also to share
the benefits with others. Software packages that I have made major contributions to:

### Digital instrumentation
- [Kotekan](https://github.com/kotekan/kotekan/) is a C++ framework for the real-time
  processing of streams of data. The CHIME X-engine and the cosmology real-time system is
  built on this package.

### 21cm data analysis

- [draco](https://github.com/radiocosmology/draco/) is a Python package that contains
the pipeline building blocks for analysis of the data from general transit radio
interferometers. It is built upon the pipeline framework in
[caput](https://github.com/radiocosmology/caput/) which contains tools for pipeline
management and handling large (1 TB+) datasets in a distributed memory environment.
- [ch_pipeline](https://github.com/chime-experiment/ch_pipeline) contains pipeline
building blocks specific to CHIME.
- [driftscan](https://github.com/radiocosmology/driftscan/) is a Python package for
modelling the transfer function of transit telescopes, and generating the derived
products needed to filter out foregrounds and estimate the power spectrum of 21 cm data.
- [cora](https://github.com/radiocosmology/cora/) is a Python package for generating
simulations of the low-frequency radio sky with a focus on polarised synchrotron
foregrounds from both our own galaxy and point sources, and cosmological 21 cm signal.

### Miscellaneous

- [scalapy](https://github.com/jrs65/scalapy) is a Python wrapper around the ScaLAPACK
library for distributed memory linear algebra.
- MagCAMB is an extension of [CAMB](https://camb.info) for calculating the effects of
primordial magnetic fields on the CMB and matter power spectrum. It's a bit old, and
really needs updating (hence it's not on Github), but apparently people still find it
useful. If you're interested in getting a copy, send me an email.
- [prym](https://github.com/chime-experiment/prym/) is a Python interface for performing
prometheus queries and returning them as sensible structured arrays or data frames.